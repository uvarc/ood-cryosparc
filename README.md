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