# Install atomate and Python=3.9 on GINAR from scratch with `pyenv`
Date: 2022-07-30

Author: Yuxing Fei, Bowen Deng, Fengyu Xie and Yunyeoung Choi


## Checking initial .bashrc and .bash_profile
Suppose your account is newly created and your home directory is
clean, make sure that your initial .bashrc script looks like the following.
These initial settings will allow you to use intel math libraries and
MPI.
```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi

#default aliases
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# No MP Parallelization (MKL)
export OMP_NUM_THREADS=1
export I_MPI_LINK='opt'
export MKL_NUM_THREADS=1
export MKL_DOMAIN_NUM_THREADS='MKL_BLAS=1'
export MKL_DYNAMIC='FALSE'
export OMP_DYNAMIC='FALSE'
export I_MPI_COMPATIBILITY=4

source /share/apps/intel/parallel_studio_xe_2015/bin/psxevars.sh intel64 &>/dev/null

# User specific aliases and functions

```
and .bash_profile
```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
```

## Installing Rust
`Rust` is a development environment management package, and is required by scipy. Do not customize installation,
just hit ENTER when prompted.
```
curl --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
source ~/.bashrc
```

## [Added 7/30/22] Installing gcc-7.4.0 and binutils-2.32
After the disk failure of ginar, many pre-compiled libraries under ginar /share/softwares have been lost, therefore
module load is not guaranteed to work neither.
Therefore, you can alternatively compile your own `gcc` and `binutils`. When we finished restoring /share, we will
let you know, such that you can skip installing packages and write only bashrc instead.
The procedure for installing gcc (can take up to an hour, please be patient):
```commandline
wget --no-check-certificate https://ftp.gnu.org/gnu/gcc/gcc-7.4.0/gcc-7.4.0.tar.gz
tar -zxvf gcc-7.4.0.tar.gz
mkdir $HOME/gcc
cd gcc-7.4.0

./contrib/download_prerequisites
./configure --prefix=$HOME/gcc
make
make install

echo 'export PATH=$HOME/gcc/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/gcc/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LIBPATH=$HOME/gcc/lib64:$LIBPATH' >> ~/.bashrc

cd $HOME/gcc/bin
ln -s gcc cc
source ~/.bashrc
```
The `ln` line create a soft link to redirect cc to our own gcc, rather than the system default. This is
necessary because some package's makefile is written with command `cc`, not `gcc`.

The procedure for installing binutils:
```commandline
wget --no-check-certificate https://ftp.gnu.org/gnu/binutils/binutils-2.32.tar.gz
tar -zxvf binutils-2.32.tar.gz
mkdir $HOME/binutils
cd binutils-2.32

./configure --prefix=$HOME/binutils
make
make install

echo 'export PATH=$HOME/binutils/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/binutils/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LIBPATH=$HOME/binutils/lib64:$LIBPATH' >> ~/.bashrc
source ~/.bashrc
```
Versions newer or older than these might not work. Older versions might not support C++ 11 standard required
for python>=3.9, while newer versions will require glibc >= 2.14, which is impossible to have because that
would require an update in our system kernel.


## Installing Openssl
`openssl` is a python building essential, which allows python to authenticate databases.
Locally install a newer version (>=1.1) of openssl to compile python.
```commandline
wget --no-check-certificate https://ftp.openssl.org/source/old/1.1.1/openssl-1.1.1g.tar.gz
tar -zxvf openssl-1.1.1g.tar.gz
mkdir $HOME/openssl
cd openssl-1.1.1g

mkdir build && cd build
../config --prefix=$HOME/openssl
make
make install

echo 'export PATH=$HOME/openssl/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/openssl/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

## Install liblzma
`liblzma` is a python dependency required to compile pymatgen. We use 5.0.4 version.
Install it by yourself, use the following:
```commandline
git clone https://github.com/kobolabs/liblzma.git
mkdir $HOME/xz

cd liblzma
./configure --prefix=$HOME/xz
make
make install
echo 'export PATH=$HOME/xz/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/xz/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

## Install SQLite3
`SQLite3` is a library that fireworks needs to communicate with mongodb. The latest version proved to work
is 3.39.2. 

You can download, install a `SQLite3` yourself:
```commandline
wget --no-check-certificate https://www.sqlite.org/2022/sqlite-autoconf-3390200.tar.gz
mkdir $HOME/sqlite3
tar -zxvf sqlite-autoconf-3390200.tar.gz

cd sqlite-autoconf-3390200
./configure --prefix=$HOME/sqlite3
make
make install
echo 'export PATH=$HOME/sqlite3/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/sqlite3/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

## [Added 7/30/22] Installing pkg-config
`pkg-config` is a compact toolkit that pip uses to wrap up package compilation. You need to add the pkgconfig
directory of a software in bashrc as a environemnt variable PKG_CONFIG_PATH, if you have installed the software
elsewhere other than /usr/lib. Since the systemwise pkg-config has been broken, you need to install it like below:
```commandline
wget --no-check-certificate https://pkgconfig.freedesktop.org/releases/pkg-config-0.29.2.tar.gz
mkdir $HOME/pkg-config
tar -zxvf pkg-config-0.29.2.tar.gz

cd pkg-config-0.29.2
./configure --prefix=$HOME/pkg-config
make
make install
echo 'export PATH=$HOME/pkg-config/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
Add a library to PKG_CONFIG_PATH when needed. In the following sections we shall include it in the example.

## [Added 7/30/22] Installing openblas
`openblas` is required by scipy. Here is the way to install it yourself. Note: the latest version know to work
is 0.3.5. The laterest released version does not support ginar's kernel. If you find any other newer
version that works, please update this SOP.
```commandline
wget --no-check-certificate https://github.com/xianyi/OpenBLAS/archive/refs/tags/v0.3.5.tar.gz
mv v0.3.5.tar.gz openblas-0.3.5.tar.gz
tar -zxvf openblas-0.3.5.tar.gz
mkdir $HOME/openblas
cd openblas-0.3.5

make
make PREFIX=$HOME/openblas install

echo 'export PATH=$HOME/openblas/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/openblas/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export LIBPATH=$HOME/openblas/lib:$LIBPATH' >> ~/.bashrc
echo 'export PKG_CONFIG_PATH=$HOME/openblas/lib/pkgconfig:$PKG_CONFIG_PATH' >> ~/.bashrc
source ~/.bashrc
```
Note: 1, openblas does not use ./configure to specify installation. Add the prefix when calling "make install"
instead.
2, You must add <path_to_openblas>/lib/pkgconfig to environement vairable PKG_CONFIG_PATH for pip to find
openblas.

## [Added 7/30/22] Installing libffi
`libffi` is required by atomate.
```commandline
wget --no-check-certificate https://github.com/libffi/libffi/releases/download/v3.4.2/libffi-3.4.2.tar.gz
tar -zxvf libffi-3.4.2.tar.gz
mkdir $HOME/libffi
cd libffi-3.4.2

./configure prefix=$HOME/libffi
make
make install

echo 'export LD_LIBRARY_PATH=$HOME/libffi/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export PKG_CONFIG_PATH=$HOME/libffi/lib/pkgconfig:$PKG_CONFIG_PATH' >> ~/.bashrc
source ~/.bashrc
```

## [Added 7/30/22, optional] Installing MIP dependecies
`SuiteSparce` and `glpk` are required for people who will use cvxopt, cvxpy to fit cluster expansions. It is pre-required by
cvxopt. `SuiteSparce` package does not require compilation. The latest version currently known to work is 4.5.3.
```commandline
wget --no-check-certificate https://github.com/DrTimothyAldenDavis/SuiteSparse/archive/refs/tags/v4.5.3.tar.gz
mv v4.5.3.tar.gz SuiteSparce-4.5.3.tar.gz
tar -zxvf SuiteSparce-4.5.3.tar.gz

echo 'export CVXOPT_SUITESPARSE_SRC_DIR=$HOME/SuiteSparse' >> ~/.bashrc
source ~/.bashrc
```
You can also install gurobi's binary version. Check gurobi's official website for details.
For glpk, install as:
```commandline
wget --no-check-certificate https://ftp.gnu.org/gnu/glpk/glpk-5.0.tar.gz
mkdir $HOME/glpk
tar -zxvf glpk-5.0.tar.gz
cd glpk-5.0.tar.gz

./configure --prefix=$HOME/glpk
make
make install

echo 'export PATH=$HOME/glpk/bin:$PATH' >> ~/.bashrc
echo 'export LIBRARY_PATH=$HOME/glpk/lib:$LIBRARY_PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$HOME/glpk/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export CPATH=$HOME/glpk/lib:$CPATH' >> ~/.bashrc
echo 'export CVXOPT_BUILD_GLPK=1' >> ~/.bashrc
source ~/.bashrc
```
The last line will allow your `cvxopt` compiled to work with `glpk`.
So in the future, when you install the `cvxopt` python package, use:
```commandline
CFLAG=-I/<yourhome>/glpk/include LDFLAG=-L/<yourhome>/glpk/lib pip install glpk
pip install cvxopt --no-binary cvxopt
```
Note: if you don't install glpk with cvxopt, cvxopt can still be compiled
successfully, but cvxpy will throw out warning that it can not access glpk
when you import cvxpy. Things will still work anyway.

## [Added 8/9/2022] Notes on shared libraries
We have copied compiled libraries above to /share/apps/software. If you prefer not to
compile on your own, please omit download, configure and make steps, and add the 
paths an variables into your bashrc under /share/apps/software. When compiling python,
you need to change flags accordingly.

## Installing pyenv
`pyenv` is a powerful tool to manage multiple python versions and virtualenvs.
Unlike anaconda, it compiles python and basic packages from scratch,
and is therefore more tolerant with old system kernels.

`pyenv` is compilation free. You only need to download its binary
distribution from github and configure as follows: 
```commandline
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

sed -Ei -e '/^([^#]|$)/ {a \
export PYENV_ROOT="$HOME/.pyenv"
a \
export PATH="$PYENV_ROOT/bin:$PATH"
a \
' -e ':a' -e '$!{n;ba};}' ~/.bash_profile
echo 'eval "$(pyenv init --path)"' >> ~/.bash_profile

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo 'eval "$(pyenv init --path)"' >> ~/.profile
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc

source ~/.profile
source ~/.bash_profile
source ~/.bashrc
```

## Install python
When installing python, pyenv compiles python, pip and runtime
libraries from scratch. Therefore, besides supplying paths to necessary
packages in .bashrc, you also need to pass them as compiler flags
into `pyenv`. The necessary attachments are openssl, xz and sqlite3.

```commandline

CPPFLAGS="-I/home/<your_home>/openssl/include/ \
-I/home/<your_home>/sqlite3/include/ \
-I/home/<your_home>/xz/include/ \
-I/home/<your_home>/libffi/include" \
LDFLAGS="-L/home/<your_home>/openssl/lib/ \
-L/home/<your_home>/sqlite3/lib/ \
-L/home/<your_home>/xz/lib. \
-L/home/<your_home>/libffi/lib64" pyenv install 3.9.9

pyenv local 3.9.9
```
`pyenv local` sets 3.9.9 as your python version when you call
python from the current working directory. If you wish to make
python 3.9.9 your system-wise default, please use `pyenv global 3.9.9`
instead.

## Install numpy, scipy and the pymatgen-atomate ecosystem.
The following packages should be installed in the exact order.
```commandline
pip install cython
pip install wheel
pip install importlib
pip install numpy
CFLAGS=-I/home/opts/openblas/include LDFLAGS=-L/home/opts/openblas/lib pip install scipy
pip install atomate
```
Note: scipy may require FLAGS to be compiled properly.

After this, try use pymongo MongoClient to connect a mongodb you
have access to, and execute a read operation. If that's successful, we
are all good. If not, usually pymongo will say that your systems
mongodb binary program version is too low. To adapt this, you need to
downgrade your pymongo to <=4.0.2 using pip.
Try use some functionalities in pymatgen and scipy to see
if they also work.

## Configure pymatgen
Write a .pmgrc.yaml under your $HOME as:
```yaml
PMG_VASP_PSP_DIR: /share/apps/repos/pyabinitio/resources/VASP_PSP
PMG_MAPI_KEY: YOUR_API_KEY
```
Where `PMG_VASP_PSP_DIR` sets path to VASP pseudo-potentials required
by pymatgen.io.vasp.sets. `PMG_MAPI_KEY` sets the Materials Project API key, which is
required if you wish to query Materials Project database from pymatgen.

## Configure atomate
See atomate tutorial at: https://atomate.org/installation.html Using NERSC's free mongdb service is highly recommended.
If you already have a NERSC account, you can apply for a mongdb
at: https://docs.nersc.gov/services/databases/
Here is an example of configuration files that work on
GINAR, if you are using NERSC mongodb:

db.json
```json
{
    "host": "mongodb://mongodb07.nersc.gov",
    "port": 27017,
    "database": "<your db name>",
    "collection": "tasks",
    "admin_user": "<your db username>",
    "admin_password": "<your password>",
    "readonly_user": "<your read-only username>",
    "readonly_password": "<your password>",
    "aliases": {}
}
```

my_launchpad.yaml (use the same database for storing launchpad)
```yaml
host: mongodb://mongodb07.nersc.gov
port: 27017
name: <your db>
username: <your admin user name>
password: <your password>
ssl_ca_file: null
logdir: null
strm_lvl: DEBUG
user_indices: []
wf_user_indices: []
```

If you use mongodb Atlas, the two files above will be different
because Atlas only supports uri mode authentication. First make sure that
your atomate version is >=1.0.3. Then you should instead
write:
db.json
```json
{
    "host_uri": "mongodb+srv://<username>:<yourpassword>@cluster0.so90p.mongodb.net",
    "database": "<your db name>",
    "collection": "tasks",
    "aliases": {}
}
```
and my_launchpad.yaml:
```yaml
host: mongodb+srv://<username>:<password>@cluster0.so90p.mongodb.net/<your db name>
uri_mode: true
ssl_ca_file: null
logdir: null
strm_lvl: DEBUG
user_indices: []
wf_user_indices: []
```

my_fworker and my_qadapter are not dependent on mongodb
provider. Write:
my_fworker.yaml:
```yaml
name: ginar
category: ''
query: '{}'
env:
      db_file: /path/to/your/db.json
      vasp_cmd: 'mpiexec.hydra -n $NSLOTS pvasp.5.4.4.intel'
      gamma_vasp_cmd: 'mpiexec.hydra -n $NSLOTS pvasp.5.4.4.intel.gamma'
      scratch_dir: /path/to/a/scratch/folder/scratch
```

my_qadapter.yaml:
```yaml
_fw_name: CommonAdapter
_fw_q_type: SGE
rocket_launch: rlaunch -c /path/to/your/config rapidfire --timeout 345600
walltime: 96:00:00
queue: null
job_name: atomate (or anything you like)
pre_rocket: |
          #$ -pe impi 16
          #$ -cwd
logdir: /path/to/your/logs
```

Create a file called FW_config.yaml in /path/to/your/config with the following contents:
```yaml
CONFIG_FILE_DIR: /path/to/your/config
```
where /path/to/your/config is where you put your db.json and
other setting files together.

Finally, add the following to your ~/.bashrc:
```bash
export FW_CONFIG_FILE=path/to/your/config/FW_config.yaml
```

After setting up, create and run a testing workflow as described
on: https://atomate.org/running_workflows.html. If that passed,
you are all set! Enjoy.

##Other Words

1, Since GLIBC compilation needs upgrading system kernel, which
is an highly risky operation, we should not attempt to upgrade the system
kernel unless we have the help from GINAR's manufacturer (penguin
computing). Currently, the packages in this SOP work without a newer GLIBC.
This SOP might expire once the compilation of
the packages above becomes dependent on GLIBC > 2.12.

2, If you choose to follow this SOP, please do not use anaconda
at the same time. Anaconda's python and basic libraries such as
openssl is pre-compiled with the newest system kernel and GLIBC,
 with an assumption that your system kernel is up-to-date, which
may result in dependency failure. For example, anaconda's
`python>3.7.5` was not compiled with `GLIBC < 2.12`, so when you
use anaconda python on this computer, it will throw out a `Segementation
fault.` If you treasure your life and want to avoid the
endless cycle of GINAR-hating and self-resentment, do not put yourself 
at this risk!


3, If you wish every user on GINAR to be
able to use your compilation, you can ask the admins (Yun and Xinye) to put it under /share/software.
If you only wish to use it myself, put it under your $HOME.
Do not put your compiled package under /opt, because /opt
is only visible from the login node and can not be seen from other computing nodes.
