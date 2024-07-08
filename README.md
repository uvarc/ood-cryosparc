# Batch Connect - CryoSPARC

A Batch Connect app designed for Open OnDemand that launches [CryoSPARC] within a SLURM batch job.
[CryoSPARC]: https://cryosparc.com/

Based off of https://github.com/OSC/bc_osc_comsol.

## Requirements
Requires [Lmod], and for [Apptainer] to be installed as an lmod module.  These need to be available on the
**compute nodes** that the batch job is intended to run on (**NOT** the
OnDemand node)
[Apptainer]: https://apptainer.org
[Lmod]: https://www.tacc.utexas.edu/research-development/tacc-projects/lmod

additionally, to use the webapp, users must install [this browser addon]
[this browser addon]: https://github.com/wrightquist/ood_clientside_linkfix

## Future Directions
* Figure out how to shut down CyroSPARC cleanly.  Currently cryosparc just gets killed by slurm when it times out/is deleted.  This messes up the db state and required repairs.  To fix the timeout issue, you could add a timer to the actual script based on the time set by the user in the form but with a minute of so to spare to do the shutdown.  This would not fix the issue of the script finishing early due to the job being manually cancelled but you could disable manual canceling for this app.
* Fix webapp urls serverside instead of in a browser addon.  Cryosparc uses site-root-relative urls (ie. paths prefixed with "/") for loading all of its clientside resources.  This is incompatable with the way OOD's reverse proxy works, and is what nesccesitates the browser addon.  To fix this serverside.  There's a couple possible options for fixing this
    - Go to /opt/cryosparc/cryosparc\_master/cryosparc\_app/custom-server/dist/client and edit the files to replace the site-root-relative urls ("/") with current-directory-relative urls ("./") + on server launch, dynamically set the <base> attribute of index.html to <ood hostname>/rnode/<node hostname>/<port> (ood hostname being ood.hpc.virginia.edu or ood1.hpc.virginia.edu).  The base attribute needs to be set so that resources load correctly even when the user navigates to a path other than home.  There's no super clean way to do the link replacement because a lot of resources are loaded dynamically with minified+obfusated js but with enough time and testing it could be done
    - Or, in the same place, just do the text replacements that the browser addon does on the clientside files.  these would need to be done dynamically when the server is launched because they rely on knowing the node hostname + port.  Then, in the proxy server, add some logic that recreates the browser addon's request redirecting (ie. if a request is made to ood.hpc.virginia.edu without the correct path prefix, but the request's referrer header includes the path prefix, prepend the prefix to the path).  This might involve setting some extra headers/cookies to fully identify which requests need to be redirected.  This would be less tedious than looking through all the clientside files to replace links but would require some extra configuration