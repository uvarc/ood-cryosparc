# Batch Connect - CryoSPARC

A Batch Connect app designed for Open OnDemand that launches [CryoSPARC](https://cryosparc.com/) within a SLURM batch job.

Based off of https://github.com/OSC/bc_osc_comsol.

## Requirements
Requires [Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod), and for [Apptainer](https://apptainer.org) to be installed as an lmod module.  These need to be available on the **compute nodes** that the batch job is intended to run on (**NOT** the OnDemand node)

additionally, to use the webapp, users must install [this browser addon](https://github.com/uvarc/ood-cryosparc-linkfix)

## Future Directions
* Figure out how to shut down CyroSPARC cleanly.  Currently CryoSPARC's processes just get killed by SLURM when the SLURM job ends (when the job times out or is ended by the user).  This prevents CryoSPARC, specifically its mongoDB database, from shutting down cleanly.  The db is not able to release its resouces (mongod.lock file not emptied) so can't restart the next time the user launches the app without the database first being [repaired)(https://www.mongodb.com/docs/manual/tutorial/recover-data-following-unexpected-shutdown/).  This is currently being handled at the top of the `/template/in_apptainer.erb.sh` file.  The --repair command is risky and may result in some data loss, as well as time consuming.  One option would be to use a timer in the script along with the bc_num_hours parameter from the job submission to start shutting CryoSPARC down just before (maybe a minute or so, depends on how long the shutdown process usually takes) before the job times out.  This would not fix the case where the user uses the "delete" button on the job card or ends the job early through the terminal.  Someone who knows more about SLURM might know of a better way.
* Do some more rigourous testing to check that the job submission form works and all job parameters are set correctly
* Fix webapp urls serverside instead of in a browser addon.
      OOD uses the url path to specify what node/port on the cluster requests should go to. So all requests to an interactive app's server running on the cluster need the correct path prefix /rnode/(host)/(port)(path).  See https://osc.github.io/ood-documentation/latest/how-tos/app-development/interactive/view.html#reverse-proxy for an explanation. CryoSPARC's webpage files specify all the web resource paths relative to the hostname (ie. src="/index.js"), but for OOD to load them correctly, they need to include the path prefix. The addon rewrites requests to the incorrect urls (ex. "/index.js") to the correct ones (ex. "/rnode/(host)/(port)/index.js") at request time so that OOD can locate the resources correctly

  There's a couple ways we could fix this without the addon
    - Change all the urls in the dist/client files to include the OOD path prefix.  There's no super clean way to do this because a lot of the urls are in the .js files, but it's theoretically posiible, it would just take time and careful testing to make sure nothing breaks.
    - Or, In the proxy server, add some logic that recreates the browser addon's request redirecting (ie. if a request is made to ood.hpc.virginia.edu without the correct path prefix, but the request's referrer header includes the path prefix, prepend the prefix to the path).  This might involve setting some extra headers/cookies to fully identify which requests need to be redirected.  This would be less tedious than looking through all the clientside files to replace links but would require some extra configuration
