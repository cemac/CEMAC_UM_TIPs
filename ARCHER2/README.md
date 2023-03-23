# ARCHER with 2FA FROM PUMA

These are some tips for transfering from ARCHER to ARCHER 2 (access via PUMA)

## information

NCAS CMS have some usefull information on how to modify your suite.



* LINUX must be set to background (from at)
* site/archer.rc and ncas_cray  might be required to run

### Met office password cahce
`mosrs-cache-password `

### Permission errors
```
rm -f  ~/.ssh/environment.puma
eval `ssh-agent -s`
ssh-add -D
ssh-add ~/.ssh/id_rsa_archerum
```
weird FCM make permission error... not always ? Sometimes runs fine ?


# ARCHER 2

[CMS UM on archer 2 pages](https://cms.ncas.ac.uk/archer2/unified-model/)

Note nothing will work out of the box.

1. Follow on boarding instructions
2. run example suite
  1. ~/.bash_profile  edit on ARCHER 2
  2. edit or add `site/archer2.rc` type file or `site/ncas_cray_ex/suite-adds.rc`
      * `--chdir=/work/n02/n02/<your ARCHER2 user name>`  line must be edited

# SLURM COMMANDS

* `sinfo`  information on nodes available
* `squeue -l | grep $USER` check if your jobs are running
* `squeue -l | grep PENDING | wc -l` how long is the queue
* `squeue -R shortqos` check the short queue

# Gotchas

* WALLCLOCK LIMIT is in minutes not seconds there 60 is 1 hour not 1 min
this can be fixed editing `bin/setup_metadata`
```
fp.write("WALL_CLOCK_LIMIT=20\n")
fp.write("range=1:172800\n")
```
and `meta/rose-meta.conf` swapping `range=60:172800` to `range=1:172800`
* Node is 8x8 CPUS (actually 128 cores as hyper threaded)
* Python mismatches - if loading in cray-python must load after other cyclc tasks and unload in after in pre scripts
* Nesting Suite archer suite had reference to short partition which does not exist short q is accessed by short reservation on standard partition
* short q limited to 1 running 1 queuing



# Archer2 new 23 cabinet system


## Host Key verification failed / submit failed

`[FAIL] Host key verification failed.`

Occurs because Archer2 now has a different key the 4 cabinet system

`ssh-keygen -R login.archer2.ac.uk`

should resolve it

some users might need to check there's no other toublesome keys

`ssh -o BatchMode=yes -o StrictHostKeyChecking=no  login.archer2.ac.uk`
might pop up a warning:

`Warning: the ECDSA host key for 'login.archer2.ac.uk' differs from the key for the IP address '193.62.216.45'``

telling you the other offending key to remove e.g.:

`ssh-keygen -R 193.62.216.45`

Now you should have no more mismatching keys. (possibly you'll need to repeat this step a few times to cover all the ip addresses!)

*NB only do this when you are expecting the keys to mismatch - otherwise a key mismatch could highlight something malicious*

## Module fails

Job fails with error logs not finding modules

`module restore module` yields : `Lmod has detected the following error:  User module collection: "2020.12.14" does not exist.`

`module restore` is no longer a command. `module load um` will load um modules

## Transitioning to 23-cab from 4 cab

Main files that need altering:

`site/ncas-cray-ex/suite-adds.rc`

all `module restore` commands must be modified to something like:

```bash
export MODULEPATH=$MODULEPATH:/work/n02/n02/simon/modulefiles
module load um
```

the CAP9.1 path must be altered

```bash
# replace
# export PATH=/work/n02/n02/grenvill/CAP9.1/build/bin:$PATH
# with
export PATH=/work/y07/shared/umshared/CAP9.1/build/bin:$PATH
```

CAP9 path must also be altered in:

`app/install_cold/opt/rose-app-ncas-cray-ex.conf`

```bash
# replace
# source=/work/n02/n02/grenvill/CAP9.1/build/bin
# with
source=/work/y07/shared/umshared/CAP9.1/build/bin
```

# UM source code mods #

Depending on which version of the UM more or less modifications will be required:

## CCE fixes ##

[UKCA CC12 fixes](https://code.metoffice.gov.uk/trac/um/ticket/6464)


An easy way to find the required mods is to browse [Jeff Coles UM code](https://code.metoffice.gov.uk/trac/um/browser/main/branches/dev/jeffcole) select the version you're interested in and search the revision log for the "cce12 fixes" in Jeff Coles code. View the revision log and view the changes related the CCE12

e.g. for vn 11.2 the following files need to be modified 

```
atmosphere/AC_assimilation/getobs.F90 (4 diffs)
atmosphere/UKCA/ukca_activ_mod.F90 (1 diff)
atmosphere/UKCA/ukca_calc_drydiam.F90 (2 diffs)
atmosphere/UKCA/ukca_volume_mode.F90 (6 diffs)
atmosphere/boundary_layer/bdy_expl2.F90 (2 diffs)
utility/qxreconf/rcf_h_int_nearest_mod.F90 (2 diffs)
```

This is highlighted here: [UKCA and other Code fixes examples](https://code.metoffice.gov.uk/trac/um/changeset?reponame=&new=104769%40main%2Fbranches%2Fdev%2Fjeffcole%2Fvn11.2_archer2_fixes&old=61261%40main%2Ftrunk)

### Troubleshooting:

1. Access issue
 `ssh login.archer2.ac.uk`

 returns

 `PTY allocation request failed on channel 0`
 `bash: /home1/z00/puma-access-route/bin/pumawrapper: No such file or
 directory`

check the path in

 `/home1/system/puma-access-route/keys/$USER`

 `/home1/z00/` should be `/home1/system/`
