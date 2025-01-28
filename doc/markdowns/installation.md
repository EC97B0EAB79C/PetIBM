# Installation

## Setup
```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh -b -p $HOME/miniforge
source miniforge/bin/activate
mamba create \
    -n petibm \
    -c barbagroup \
    -c nvidia \
    -c conda-forge \
    "*amgxwrapper=*=cuda114*" \
    "yaml-cpp>=0.7" \
    "symengine>=0.9" \
    "cmake>=3.23" \
    "make" \
    "pkg-config" \
    "git" \
    "gxx_linux-64>=11.2"
conda activate petibm
```

## Build
```
mkdir build && cd build
cmake \
    -DCMAKE_INSTALL_PREFIX=$HOME/sfw/petibm/petibm-linux-dbg \
    -DCMAKE_CXX_COMPILER=${CXX} \
    -DCMAKE_BUILD_TYPE=Debug \
    -DPETSC_DIR=$CONDA_PREFIX \
    -DPETSC_ARCH="" \
    -DCUDA_DIR=$CONDA_PREFIX \
    -DAMGX_DIR=$CONDA_PREFIX \
    -DAMGXWRAPPER_DIR=$CONDA_PREFIX \
    -DYAMLCPP_DIR=$CONDA_PREFIX \
    -DSYMENGINE_DIR=$CONDA_PREFIX \
    -DPETIBM_ENABLE_TESTS=ON \
    -DPETIBM_USE_AMGX=ON \
    ..
```

<!--
## Environment Setup
```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh -b -p $HOME/miniforge
source miniforge/bin/activate
mamba create -n petibm -c barbagroup -c nvidia -c conda-forge \
    "cmake>=3.23" make pkg-config git gxx_linux-64=11.2 \
    "yaml-cpp>=0.7" "symengine>=0.9" "h5py>=3.0" numpy \
    mpich=4.2.3 \
    cudatoolkit=12.6
conda activate petibm
mamba install -c conda-forge python=3.9
# mamba install -c conda-forge hdf5=1.14.3
# mamba install -c conda-forge bison flex
# mamba install -c conda-forge ptscotch
```

### Directories
```
export PETSC_DIR=$HOME/petsc/petsc-3.19.6
export PETSC_ARCH=linux-cuda12
```

## Build PETSc

```
mkdir -p $PETSC_DIR && cd $PETSC_DIR/..
wget http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-lite-3.19.6.tar.gz
tar -xzf petsc-lite-3.19.6.tar.gz
cd $PETSC_DIR
./configure \
    --with-cuda-dir=/opt/nvidia/hpc_sdk/Linux_x86_64/24.11/cuda \
    --with-cuda=1 \
    --with-cuda-arch=sm_89 \
    --with-cudac=nvcc \
    --with-cc=$CONDA_PREFIX/bin/mpicc \
    --with-cxx=$CONDA_PREFIX/bin/mpicxx \
    --download-f2cblaslapack \
    --download-hdf5 \
    --download-hypre \
    --download-metis \
    --download-parmetis \
    --download-ptscotch

# ./configure \
#    --with-cc=$CONDA_PREFIX/bin/mpicc \
#    --with-cxx=$CONDA_PREFIX/bin/mpicxx \
#    --with-fc=0 \
#    --with-cuda=1 \
#    --with-cuda-arch=sm_89 \
#    --with-hdf5-dir=$CONDA_PREFIX \
#    --with-hypre-dir=$CONDA_PREFIX \
#    --with-ptscotch-dir=$CONDA_PREFIX \
#    --download-metis \
#    --download-f2cblaslapack \
#    --download-parmetis \
```
-->

<!--
PetIBM only officially supports Linux systems. MacOS and Windows may or may not work. We will not
provide any assist to issues of using PetIBM on MacOS and Windows.

## A. Installation guide for end-users

End-users can install PetIBM with Anaconda or use it with container technologies.

### A.1. With `conda`/`mamba`

We uploaded our pre-built packages to our Anaconda channels `barbagroup`. These packages were built
with GPU support. Linux users can install
them through

```
$ mamba install -c barbagroup -c nvidia -c conda-forge "petibm=*=cuda114*"
```

We suggest using `mamba`. If `conda` is preferred, just replace `mamba` with `conda`. Replace
`install` with `create -n <env name>` instead to create a new and fresh environment.

If an older CUDA is desired (e.g., for old GPU models), we also have packages for
CUDA 10.2, which supports old GPUs like K20/K40. To do so, use `"petibm=*=cuda102*"` instead.

**Note**: The CUDA 10.2 variant may also work on newer GPUs and drivers.
If you encounter issues using the CUDA 11.4 variant with newer GPUs/drivers, it should be fine to
use the CUDA 10.2 variant.
AmgX uses somehow different code w.r.t. whether it's CUDA 10 or CUDA 11, and sometimes there are
issues in AmgX's CUDA 11 codebase.

### A.2. Using container technology

For users familiar with Docker or Singularity Container (now renamed Apptainer), we provide images
for the both to save you from compiling, buliding, and installing:

* Docker: https://hub.docker.com/repository/docker/barbagroup/petibm
* Singularity: https://github.com/barbagroup/petibm-recipes/tree/master/singularity

### A.3. Optional dependencies for example cases' post-processing

The above two installation methods only provide the solver.
Some example cases in the `examples` folder have Python scripts for post-processing.
To be able to execute these Python scripts, install the following Python packages (using `mamba`/`conda`, for example):

```shell
$ mamba install -c conda-forge pyyaml numpy h5py matplotlib
```

## B. Building from the source

### B.1. Using pre-built dependencies

If you just need to set up a development environment to extend/debug PetIBM, we suggest installing
dependencies with `conda`/`mamba`:

```
$ mamba create \
    -n <env name> \
    -c barbagroup \
    -c nvidia \
    -c conda-forge \
    "*amgxwrapper=*=cuda114*" \
    "yaml-cpp>=0.7" \
    "symengine>=0.9" \
    "cmake>=3.23" \
    "make" \
    "pkg-config" \
    "git" \
    "gxx_linux-64>=11.2"
```

The package `amgxwrapper` pulls in most of the required dependencies (e.g., petsc, cuda, etc.).
`yaml-cpp` and `symengine` are dependencies not covered by `amgxwrapper`. `cmake`, `make`, `git`, and
`gxx_linux-64` are for building PetIBM in case your system does not have them or they are too old.

Once the process is done, jump into the environment through either `mamba activate <env name>` or
`source <mamba installation prefix>/bin/activate <env name>`.

The above command only installs required packages for the flow solvers.
See section A.3 for dependencies for post-processing with example cases.

To proceed, let's assume you would like to download PetIBM's source files to path
`$HOME/sfw/petibm/PetIBM`, and to install PetIBM's final solvers and libraries to
`$HOME/sfw/petibm/petibm-linux-dbg`.

#### B.1.1. Obtaining the source 

Pull the latest development version of PetIBM from GitHub:

```shell
$ cd $HOME/sfw
$ mkdir petibm
$ cd petibm
$ git clone https://github.com/barbagroup/PetIBM.git
```

PetIBM's source files are now at `$HOME/sfw/petibm/PetIBM`. Our current location is at
`$HOME/sfw/petibm`.

#### B.1.2. Configuring

Create a folder to host temporary files from building and compilation:

```shell
$ mkdir build
$ cd build
```

Configure the build:

```shell
$ cmake \
    -DCMAKE_INSTALL_PREFIX=$HOME/sfw/petibm/petibm-linux-dbg \
    -DCMAKE_CXX_COMPILER=${CXX} \
    -DCMAKE_BUILD_TYPE=Debug \
    -DPETSC_DIR=$CONDA_PREFIX \
    -DPETSC_ARCH="" \
    -DCUDA_DIR=$CONDA_PREFIX \
    -DAMGX_DIR=$CONDA_PREFIX \
    -DAMGXWRAPPER_DIR=$CONDA_PREFIX \
    -DYAMLCPP_DIR=$CONDA_PREFIX \
    -DSYMENGINE_DIR=$CONDA_PREFIX \
    -DPETIBM_ENABLE_TESTS=ON \
    -DPETIBM_USE_AMGX=ON \
    $HOME/sfw/petibm/PetIBM
```

* PETSc, CUDA, AmgX, AmgXWrapper, and yaml-cpp were all installed through `mamba` (or `conda`), so
  their paths are `$CONDA_PREFIX`.
* `-DPETIBM_ENABLE_TESTS` controls whether to build unit tests.
* `-DPETIBM_USE_AMGX` notifies PetIBM to build against GPU-related code.

Later on when you want to have a production build, just change `-DCMAKE_BUILD_TYPE` from `Debug` to
`Release`. (And you may also want to change the installation path in `-DCMAKE_INSTALL_PREFIX`.)

#### B.1.3. Building and installing

Start to build:

```shell
$ make all -j <number of CPUs>
```

Specifying the number of CPUs with `<number of CPUS>` to speed up the compilation procedure.

We can do some unit tests:

```shell
$ make check
```

Then install the final solvers and libraries:

```shell
$ make install
```

The solvers are now available at `$HOME/sfw/petibm/petibm-linux-dbg/bin`.

### B.2. Building dependencies from scratch

If you would like to build everything from scratch, here's the list of dependencies:

**Required**:

* GNU C++ compiler g++ (9.4 and 11.2 have been test with the latest PetIBM release)
* [PETSc](https://www.mcs.anl.gov/petsc/) (3.16+) with parallel HDF5 enabled
* MPI: OpenMPI, MPICH, or Intel MPI
* [yaml-cpp](https://github.com/jbeder/yaml-cpp) (0.7.0+)
* [SymEngine](https://github.com/symengine/symengine) (0.9+)

**Optional for GPU linear solvers**:

* [AmgX](https://github.com/NVIDIA/AMGX) (v2.2.0)
* [AmgXWrapper](https://github.com/barbagroup/AmgXWrapper) (1.6.1+)

**Optional for pre- and post-processing Python scripts**:

* Python (3.6+)
* NumPy (1.12.1+)
* h5py (2.7.0+)
* Matplotlib (2.0.2+)
* pyyaml (6.0+)

**Note**:

* [yaml-cpp](https://github.com/jbeder/yaml-cpp) and
  [AmgXWrapper](https://github.com/barbagroup/AmgXWrapper)
  can be automatically installed during PetIBM configuration.
* MPI can be either installed during [PETSc configuration](#petsc) or installed explicitly by users.
  If you are using HPC clusters, it's better to use the shipped MPI packages on the clusters because
  they usually have specific optimizations for the clusters' hardware. NVIDIA's HPC-X is another
  alternative, which includes OpenMPI optimized for NVIDIA's HPC infrastructure.
* The latest release of PetIBM (0.5.4) has been tested with:
    * Arch Linux with g++\-11.2.0 and PETSc v3.16.6
    * Ubuntu 20.04 with g++\-9.4 and PETSc v3.16.6
* Though not suggested, if using older compilers are desired, previous releases of PetIBM (before
  v0.5.2) have at some point been tested with the following combinations. However, we will not
  provide any support for using older versions of PetIBM.
    * Ubuntu 16.04 with g++ v5.4, and PETSc v3.11.2
    * MacOS Sierra with g++ v6.0, and PETSc v3.8.2
    * Arch Linux with g++ v7.2, and PETSc v3.8.2

#### B.2.1. Compilers and configuration tools

Different linux distributions use different package management systems. Here we use Ubuntu 20.04 as
an example. Assuming you have permission to use `sudo`, to install compilers and tools:

```shell
$ sudo apt-get install build-essential pkg-config make git curl
```

You can check the version of G++ installed with `g++ --version`.

Now we also need `cmake`. Unfortunately, `cmake` from Ubuntu 20.04 is too old. We need to install it
from upstream directly:

```shell
$ curl -LO https://github.com/Kitware/CMake/releases/download/v3.23.1/cmake-3.23.1-linux-x86_64.sh
$ sudo sh ./cmake-3.23.1-linux-x86_64.sh --prefix=/usr --exclude-subdir --skip-license
```

This will install the `cmake` v3.23 to `/usr` as if it's a package from the package manager.


#### B.2.2. Building SymEngine

SymEngine requires some other third-party dependencies.
The building process is not trivial.
Hence we are not covering how to build and install SymEngine.
Some Linux distributions officially provide pre-built SymEngine.
For example, in Arch Linux, the pre-built SymEngine can be installed with `sudo pacman -S symengine`.
Check with your Linux distribution's package manager first.

If building SymEngine from scratch is needed, please refer to SymEngine's documentation [here](https://github.com/symengine/symengine#building-from-source).


#### B.2.3. Building PETSc

Here, we provide the command-line instructions to install PETSc v3.16.6. PETSc is powerful and has a
lot of options to configure and for different levels of optimizations. We only show the bare minimum
configuration that at least work for PetIBM.

Get and unpack PETSc:

```shell
$ cd $HOME/sfw
$ mkdir -p petsc/3.16.6
$ curl -LO http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-lite-3.16.6.tar.gz
$ tar -xvf petsc-lite-3.16.6.tar.gz -C petsc/3.16.6 --strip-components=1
$ cd petsc/3.16.6
```

Configure and build PETSc in debugging mode:

```shell
$ export PETSC_DIR=$HOME/sfw/petsc/3.16.6
$ export PETSC_ARCH=linux-dbg
$ ./configure --PETSC_DIR=$PETSC_DIR --PETSC_ARCH=$PETSC_ARCH \
    --with-cc=gcc \
    --with-cxx=g++ \
    --with-fc=0 \
    --with-pic=1 \
    --with-shared-libraries=1 \
    --with-debugging=1 \
    --with-fortran-bindings=0 \
    --download-f2cblaslapack \
    --download-openmpi \
    --download-hdf5 \
    --download-hypre \
    --download-ptscotch \
    --download-metis \
    --download-parmetis \
    --download-superlu_dist
$ make all
$ make test
```

To configure and build PETSc for production runs, change the value of `--with-debugging` form `1` to
`0`, and add `--COPTFLAGS="-O3" --CXXOPTFLAGS="-O3"` to the list of flags. You may also want to
change the name in `PETSC_ARCH`.

The above configuration also downloads and installs OpenMPI. However, it is not optimized. When
running the code on an external cluster, we suggest configure and build PETSc with a proper
MPI package for performance.

To understand and use a more advanced configuration of PETSc, please refer to
[PETSc's instructions](http://www.mcs.anl.gov/petsc/documentation/installation.html). Also, you
can run `./configure --help` to list all available configure flags.

#### B.2.3. Building PetIBM

Here we will build a CPU-only PetIBM with `yaml-cpp` being installed automatically.

Pull the latest development version of PetIBM from GitHub:

```shell
$ cd $HOME/sfw
$ mkdir petibm
$ cd petibm
$ git clone https://github.com/barbagroup/PetIBM.git
```

Configure and build PetIBM:

```shell
$ export PETSC_DIR=$HOME/sfw/petsc/3.16.6
$ export PETSC_ARCH=linux-dbg
$ cmake \
    -DCMAKE_INSTALL_PREFIX=$HOME/sfw/petibm/petibm-linux-dbg \
    -DCMAKE_CXX_COMPILER=$PETSC_DIR/$PETSC_ARCH/bin/mpicxx \
    -DCMAKE_BUILD_TYPE=Debug \
    -DPETSC_DIR=$PETSC_DIR \
    -DPETSC_ARCH=$PETSC_ARCH \
    -DSYMENGINE_DIR=<path to SymEngine> \
    -DPETIBM_ENABLE_TESTS=ON \
    -DPETIBM_USE_AMGX=OFF \
    -DPETIBM_BUILD_YAMLCPP=ON \
    $HOME/sfw/petibm/PetIBM
$ make all
$ make check -j <num CPUs>
$ make install
```

For production runs, change the value of `-DCMAKE_BUILD_TYPE` to `Release` and refer to the
corresponding PETSc builds in `-DPETSC_DIR` and `-DPETSC_ARCH`.

Et voila! Now PetIBM is installed to `$HOME/sfw/petibm/petibm-linux-dbg`.

#### B.2.4. Optional: using NVIDIA AmgX to solve linear systems on multiple GPUs

PetIBM's GPU support comes from NVIDIA's [AmgX](https://github.com/NVIDIA/AMGX). And the interface
between PETSc and AmgX is provided by [AmgXWrapper](https://github.com/barbagroup/AmgXWrapper).

Building AmgX is an expensive job. Without a powerful machine, building AmgX may take hours. Hence
we would like to refer interested readers to AmgX's manual for how to build and install it.

Once AmgX is available on your machine, AmgXWrapper can be automatically downloaded and installed
during installing PetBM:

```shell
$ export PETSC_DIR=$HOME/sfw/petsc/3.16.6
$ export PETSC_ARCH=linux-dbg
$ cmake \
    -DCMAKE_INSTALL_PREFIX=$HOME/sfw/petibm/petibm-linux-dbg \
    -DCMAKE_CXX_COMPILER=$PETSC_DIR/$PETSC_ARCH/bin/mpicxx \
    -DCMAKE_BUILD_TYPE=Debug \
    -DPETSC_DIR=$PETSC_DIR \
    -DPETSC_ARCH=$PETSC_ARCH \
    -DCUDA_DIR=<path to cuda> \
    -DAMGX_DIR=<path to amgx> \
    -DPETIBM_ENABLE_TESTS=ON \
    -DPETIBM_USE_AMGX=ON \
    -DPETIBM_BUILD_YAMLCPP=ON \
    -DPETIBM_BUILD_AMGXWRAPPER=ON \
    $HOME/sfw/petibm/PetIBM
$ make all
$ make check -j <num CPUs>
$ make install
```

## C. PetIBM examples

Examples files are installed at `<PetIBM installation>/share/petibm/examples`. For example, for
installation with `conda`/`mamba`, the `<PetIBM installation>` is `$CONDA_PREFIX`.
-->
