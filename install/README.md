#### GNU parallel

GNU parallel is a shell tool to execute jobs in parallel. Some chunks in the R Notebooks are written in bash and use GNU parallel to run commands in parallel and reduce computation time.
Make sure you have GNU parallel installed before starting.

On debian linux, you can install GNU parallel using the `apt-get` package manager command:
```
$ apt-get install parallel
```

On macOS, using the terminal (command + space -> terminal.app), you will first need to install Homebrew, at which point you can then easily install GNU parallel, python 3, and git to facilitate installation of the required packages:

```
# Install Homebrew (useful package installer) with the following command:
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null

# Next, you can install GNU parallel:
$ brew install parallel

# You will also need to install python 3:
$ brew install python # should install python 3.7+

# Finally install git:
$ brew install git # should install git 2.26+
# git is used later on to copy GitHub repositories
```

#### illumina-utils

Illumina-utils is a small FASTQ files-processing library.
Some chunks in the `dada2-pipeline.Rmd` R Notebook use it to demultiplex FASTQ files.

Best is to create a virtual environment for illumina-utils using [virtualenv](https://virtualenv.pypa.io/en/latest/). On debian linux, you can install it using Python package manager `pip` command:

```
$ pip install virtualenv
$ virtualenv -p python3 ~/illumina-utils # important to specify Python3
$ source ~/illumina-utils/bin/activate
$ pip install illumina-utils
```

For macOS users, the process is slightly different, but can be achieved using the following commands in the terminal (also uses virtualenv):

```
# The following code will install the files in your home directory
# If you want them elsewhere, adjust the paths accordingly
$ mkdir -p ~/github
$ mkdir -p ~/virtual-envs
$ cd ~/github
$ git clone git://github.com/meren/illumina-utils.git
$ virtualenv ~/virtual-envs/illumina-utils-master/bin/activate
$ cd illumina-utils
$ pip3 install -r requirements.txt

# Now to setup the environment variables:
$ echo 'export PYTHONPATH=$PYTHONPATH:~/github/illumina-utils' >> ~/virtual-envs/illumina-utils-master/bin/activate
$ echo 'export PATH=$PATH:~/github/illumina-utils/scripts' >> ~/virtual-envs/illumina-utils-master/bin/activate
$ echo 'alias illumina-utils-activate-master="source ~/virtual-envs/illumina-utils-master/bin/activate"' >> ~/.bash_profile
$ echo 'alias illumina-utils-activate-v2.7="source ~/virtual-envs/illumina-utils-v2.7/bin/activate"' >> ~/.bash_profile

# Make sure that every time you activate it, it updates itself from the master:
$ echo 'cd ~/github/illumina-utils && git pull && cd -' >> ~/virtual-envs/illumina-utils-master/bin/activate

# Initial confirmation of illumina-utils package:
$ pip3 install illumina-utils # should return text saying requirements are already satisfied

# Final confirmation of illumina-utils package:
$ iu- # type this in and press the TAB key twice
# A list of all the installed functions should appear if installation worked
```

To activate/deactivate your illumina-utils virtual environment run:
```
$ source ~/illumina-utils/bin/activate
$ deactivate
```

#### cutadapt (for ITS only)

Cutadapt is a tool to find and remove adapter sequences, primers and other types of unwanted sequence from high-throughput sequencing reads.

Same as for illumina-utils, best is to create a virtual environment for cutadapt.

```
$ virtualenv -p python3 ~/cutadapt # important to specify Python3
$ source ~/cutadapt/bin/activate
$ pip install cutadapt
```

#### git

This repository is managed using the git distributed version control system. You can get a local copy of it using the `git clone` command.
If not done yet, install git.

On debian linux, you can install git using the `apt-get` package manager command:
```
$ apt-get install git
```
