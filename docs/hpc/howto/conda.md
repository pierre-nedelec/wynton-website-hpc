<div class="alert alert-warning" role="alert" style="margin-top: 3ex" markdown="1">
⚠️ 2022-10-28: **Beta version!** The instructions on this page are fresh and might be updated soon. That said, they have been validated and approved by a group of experienced Conda users. If you run into issues, please let us know.
</div>

# Work with Conda

[Conda] is a package manager and an environment management system.  It's popular, because it simplifies installation of many scientific software tools.  There are two main distributions of Conda:

1. Anaconda - comes with more than 1,500 scientific packages (~3 GiB of disk space) [_not_ preinstalled on {{ site.cluster.nickname }}]
2. [Miniconda] - a small subset of the much larger Anaconda distribution (~0.5 GiB of disk space) [**recommended**; preinstalled on {{ site.cluster.nickname }}]

Both come with Python and `conda` commands.  We _recommend_ working with the smaller Miniconda distribution, especially since it is preinstalled on {{ site.cluster.nickname }}.  Using Miniconda, you can install additional scientific packages as needed using the `conda install ...` command.

**Note**: The aim of this document is to give the essential best-practices for working with Conda on {{ site.cluster.nickname }}. It is not meant to be a complete introduction on how to work with Conda in general. A good complement to this document, is the official Conda documentation on [Managing environments].


## Loading Miniconda

On {{ site.cluster.name }}, up-to-date versions of the Miniconda distribution are available via the CBI software stack.  There is no need for you to install this yourself.  To load Miniconda v3 with Python 3.9, call:

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI miniconda3-py39
```

This gives access to:

```sh
[alice@{{ site.devel.name }} ~]$ conda --version

conda 4.12.0

[alice@{{ site.devel.name }} ~]$ python --version
Python 3.9.12
```

To see what software packages come with this Miniconda distribution, call:

```sh
[alice@{{ site.devel.name }} ~]$ conda list
# packages in environment at {{ site.path.cbi_software }}/CBI/miniconda3-py39-4.12.0:
#
# Name                    Version                   Build  Channel
_libgcc_mutex             0.1                        main  
_openmp_mutex             4.5                       1_gnu  
brotlipy                  0.7.0           py39h27cfd23_1003  
ca-certificates           2022.3.29            h06a4308_1  
certifi                   2021.10.8        py39h06a4308_2  
cffi                      1.15.0           py39hd667e15_1  
charset-normalizer        2.0.4              pyhd3eb1b0_0  
colorama                  0.4.4              pyhd3eb1b0_0  
conda                     4.12.0           py39h06a4308_0  
conda-content-trust       0.1.1              pyhd3eb1b0_0  
conda-package-handling    1.8.1            py39h7f8727e_0  
cryptography              36.0.0           py39h9ce1e76_0  
idna                      3.3                pyhd3eb1b0_0  
ld_impl_linux-64          2.35.1               h7274673_9  
libffi                    3.3                  he6710b0_2  
libgcc-ng                 9.3.0               h5101ec6_17  
libgomp                   9.3.0               h5101ec6_17  
libstdcxx-ng              9.3.0               hd4cf53a_17  
ncurses                   6.3                  h7f8727e_2  
openssl                   1.1.1n               h7f8727e_0  
pip                       21.2.4           py39h06a4308_0  
pycosat                   0.6.3            py39h27cfd23_0  
pycparser                 2.21               pyhd3eb1b0_0  
pyopenssl                 22.0.0             pyhd3eb1b0_0  
pysocks                   1.7.1            py39h06a4308_0  
python                    3.9.12               h12debd9_0  
readline                  8.1.2                h7f8727e_1  
requests                  2.27.1             pyhd3eb1b0_0  
ruamel_yaml               0.15.100         py39h27cfd23_0  
setuptools                61.2.0           py39h06a4308_0  
six                       1.16.0             pyhd3eb1b0_1  
sqlite                    3.38.2               hc218d9a_0  
tk                        8.6.11               h1ccaba5_0  
tqdm                      4.63.0             pyhd3eb1b0_0  
tzdata                    2022a                hda174b7_0  
urllib3                   1.26.8             pyhd3eb1b0_0  
wheel                     0.37.1             pyhd3eb1b0_0  
xz                        5.2.5                h7b6447c_0  
yaml                      0.2.5                h7b6447c_0  
zlib                      1.2.12               h7f8727e_1
```


## Creating a Conda environment (required)

A Conda _environment_ is a mechanism for installing extra software tools and versions beyond the base Miniconda distribution in a controlled manner.  When using the **miniconda3-py39** module, a Conda environment must be used to install extra software. The following command creates a new `myjupyter` environment:

```sh
[alice@{{ site.devel.name }} ~]$ conda create -n myjupyter notebook
Collecting package metadata (current_repodata.json): done
Solving environment: done

## Package Plan ##

  environment location: {{ site.user.home }}/.conda/envs/myjupyter

  added / updated specs:
    - notebook


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    _openmp_mutex-5.1          |            1_gnu          21 KB
    argon2-cffi-21.3.0         |     pyhd3eb1b0_0          15 KB
    ...
    zeromq-4.3.4               |       h2531618_0         331 KB
    zlib-1.2.12                |       h5eee18b_3         103 KB
    ------------------------------------------------------------
                                           Total:        62.4 MB

The following NEW packages will be INSTALLED:

  _libgcc_mutex      pkgs/main/linux-64::_libgcc_mutex-0.1-main
  _openmp_mutex      pkgs/main/linux-64::_openmp_mutex-5.1-1_gnu
  ...
  zeromq             pkgs/main/linux-64::zeromq-4.3.4-h2531618_0
  zlib               pkgs/main/linux-64::zlib-1.2.12-h5eee18b_3


Proceed ([y]/n)? y


Downloading and Extracting Packages
...
jupyterlab_pygments- | 8 KB      | #################################### | 100% 
jupyter_client-7.3.5 | 194 KB    | #################################### | 100% 
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate myjupyter
#
# To deactivate an active environment, use
#
#     $ conda deactivate

[alice@{{ site.devel.name }} ~]$ 
```

By default, the environment is created in your home directory under `~/.conda/`.  To create the environment at a specific location, see [Managing environments] part of the official Conda documentation.  That section covers many useful topics such as removing a Conda environment, and creating an environment with a specific Python version.


## Activating a Conda environment (required)

After an environment is created, the next time you log in to a development node, you can set `myjupyter` (or any other Conda environment you've created) as your active environment by calling:

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI miniconda3-py39
[alice@{{ site.devel.name }} ~]$ conda activate myjupyter
(myjupyter) [alice@{{ site.devel.name }} ~]$ jupyter notebook --version
6.4.12

(myjupyter) [alice@{{ site.devel.name }} ~]$ 
```

Note how the command-line prompt is prefixed with `(myjupyter)`; it highlights that the Conda environment `myjupyter` is activated.  To deactivate an environment and return to the base environment, call:

```sh
(myjupyter) [alice@{{ site.devel.name }} ~]$ conda deactivate
[alice@{{ site.devel.name }} ~]$ jupyter notebook --version
jupyter: command not found

[alice@{{ site.devel.name }} ~]$ 
```


## Speed up software by auto-staging Conda environment (recommended)

We highly recommend configuring Conda environment to be automatically staged only on the local disk whenever activated.  This results in your software running _significantly faster_.  Auto-staging is straightforward to configure using the `conda-stage` tool, e.g.

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI miniconda3-py39
[alice@{{ site.devel.name }} ~]$ module load CBI conda-stage
[alice@{{ site.devel.name }} ~]$ conda activate myjupyter
(myjupyter) [alice@{{ site.devel.name }} ~]$ conda-stage --auto-stage=enable
INFO: Configuring automatic staging and unstaging of original Conda environment  ...
INFO: Enabled auto-staging
INFO: Enabled auto-unstaging
```

For the complete, easy-to-follow instructions, see the [conda-stage] documentation.


## Back up, migrate, and restore Conda environments (recommended)

Once you have your Conda environment built, we recommend that you back up its core configuration.  The process is quick and straightforward, and fully documented in the official Conda [Managing environments] documentation.  For example, to back up the above `myjupyter` environment, call:

```sh
[alice@{{ site.devel.name }} ~]$ conda env export --name myjupyter | grep -v "^prefix: " > myjupyter.yml
[alice@{{ site.devel.name }} ~]$ ls -l myjupyter.yml
-rw-rw-r-- 1 alice boblab 2966 Oct 19 18:21 myjupyter.yml
```

This configuration file is useful:

* when migrating the environment from {{ site.cluster.nickname }} to another Conda version, another computer, or another HPC environment

* for sharing the environment with collaborators

* for making a snapshot of the software stack used in a project

* for disaster recovery, e.g. if you remove the Conda environment by mistake

* for updating the dependencies in a Conda environment


To restore a backed up Conda environment from a yaml file, _on the target machine_:

1. download the yaml file, e.g. `myjupyter.yml`

2. make sure there is no existing environment with the same name, i.e. check `conda env list`

3. create a new Conda environment from the yaml file


For example, assume we have downloaded `myjupyter.yml` to our local machine.  Then we start by getting the name of the backed-up Conda environment and making sure it does not already exist;

```sh
{local}$ grep "name:" myjupyter.yml
name: myjupyter

{local}$ conda env list
# conda environments:
#
sandbox  /home/aliceb/.conda/envs/sandbox
base     /path/to/anaconda3
```

When have confirmed that there is no name clash, we can restore the backed up environment on our local machine using:

```sh
{local}$ conda env create -f myjupyter.yml 
```

This will install the exact same software versions as when we made the backup.

_Warning_: This is _not_ a fool-proof backup method, because it depends on packages to be available from the package repositories also when you try to restore the Conda environment.  To lower the risk for failure, keep your environments up to date with the latest packages and test frequently that your `myjupyter.yml` file can be restored.


We can also use `myjupyter.yml` to update an existing environment.  The gist is to edit the file to reflect what we want to be updated, and then run `conda env update ...`.  See [Managing environments] part of the official Conda documentation for exact instructions.



## Conda revisions

If you updated or installed new packages in your Conda environment and need to roll back to a previous version, it is possible to do this using Conda's revision utility.  To list available revisions in the current Conda environment, use:

```sh
[alice@{{ site.devel.name }} ~]$ CONDA_STAGE=false conda activate myjupyter
(myjupyter) [alice@{{ site.devel.name }} ~]$ conda list --revisions
2022-07-28 11:33:35  (rev 0)
    +_libgcc_mutex-0.1 (defaults/linux-64)
    +_openmp_mutex-5.1 (defaults/linux-64)
    +argon2-cffi-21.3.0 (defaults/noarch)
    +argon2-cffi-bindings-21.2.0 (defaults/linux-64)
    ...

2022-07-28 11:42:32  (rev 1)
     ca-certificates  {2022.07.19 (defaults/linux-64) -> 2022.6.15 (conda-forge/linux-64)}
     certifi  {2022.6.15 (defaults/linux-64) -> 2022.6.15 (conda-forge/linux-64)}
    +conda-pack-0.7.0 (conda-forge/noarch)
    +python_abi-3.10 (conda-forge/linux-64)
```

To roll back to a specific revision, say revision zero, use:

```sh
(myjupyter) [alice@{{ site.devel.name }} ~]$ conda install --revision 0
```

_Warning_: This only works with packages installed using `conda install`.  Packages installed via `python3 -m pip install` will _not_ be recorded by the revision system.  In such cases, we have to do manual backup snapshots (as explained above).


## Appendix

### Disable automatic activation of the 'base' environment (highly recommended)

If you previously have _installed_ Conda yourself, there is a risk that it installed itself into your `~/.bashrc` file such that it automatically activates the 'base' environment whenever you start a new shell, e.g.

```sh
{local}$ ssh alice@{{ site.login.hostname }}
alice1@{{ site.login.hostname }}:s password: XXXXXXXXXXXXXXXXXXX
(base) [alice@{{ site.login.name }} ~]$ 
```

If you see a `(base)` prefix in your prompt, then you have this set up and the Conda 'base' environment is active.  You can verify this by querying `conda info` as:

```sh
[alice@{{ site.devel.name }} ~]$ conda info | grep active
conda info | grep active
     active environment : base
    active env location : {{ site.user.home }}/miniconda3
```

This auto-activation might sound convenient, but we _strongly recommend_ against using it, because Conda software stacks have a great chance to cause conflicts (read: wreak havoc) with other software tools installed outside of Conda.  For example, people that have Conda activated and then run R via `module load CBI r`, often report on endless, hard-to-understand problems when trying to install common R packages.  Instead, we recommend to activate your Conda environments only when you need them, and leave them non-activated otherwise.  This will give you a much smoother day-to-day experience.  To clarify, if you never installed Conda yourself, and only used `module load CBI miniconda3-py39`, then you should not have this problem.

To reconfigure Conda to no longer activate the 'base' Conda environment by default, call:

```sh
(base) [alice@{{ site.devel.name }} ~]$ conda config --set auto_activate_base false
(base) [alice@{{ site.devel.name }} ~]$ 
```

Next time you log in, the 'base' environment should no longer be activated by default.


### Retire personal Conda installation in favor of miniconda3-py39

If you want to completely retire you personal Conda installation, and move on to only using `module load CBI miniconda3-py39`, you can uninstall the Conda setup code that were injected to your `~/.bashrc` file by calling:

```sh
[alice@{{ site.devel.name }} ~]$ conda init --reverse
no change     {{ site.user.home }}/miniconda3/condabin/conda
no change     {{ site.user.home }}/miniconda3/bin/conda
no change     {{ site.user.home }}/miniconda3/bin/conda-env
no change     {{ site.user.home }}/miniconda3/bin/activate
no change     {{ site.user.home }}/miniconda3/bin/deactivate
no change     {{ site.user.home }}/miniconda3/etc/profile.d/conda.sh
no change     {{ site.user.home }}/miniconda3/etc/fish/conf.d/conda.fish
no change     {{ site.user.home }}/miniconda3/shell/condabin/Conda.psm1
no change     {{ site.user.home }}/miniconda3/shell/condabin/conda-hook.ps1
no change     {{ site.user.home }}/miniconda3/lib/python3.9/site-packages/xontrib/conda.xsh
no change     {{ site.user.home }}/miniconda3/etc/profile.d/conda.csh
modified      {{ site.user.home }}/.bashrc

==> For changes to take effect, close and re-open your current shell. <==

[alice@{{ site.devel.name }} ~]$ 
```


[Conda]: https://conda.io
[Miniconda]: https://docs.conda.io/en/latest/miniconda.html
[conda-stage]: conda-stage.html
[Managing environments]: https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html
