## Installation
### Requirements

- Python 3.6.1+
- gcc 4.9+ for PyTorch1.0.0+
- cmake3 for some extensions
    ```bash
    # e.g. Ubuntu
    $ sudo apt-get install cmake
    # e.g. Using anaconda (If you don't have sudo privilege, the installation from conda might be useful)
    $ conda install cmake
    ```

We often use audio converter tools in several recipes:

- sox
    ```bash
    # e.g. Ubuntu
    $ sudo apt-get install sox
    # e.g. CentOS
    $ sudo yum install sox
    # e.g. Using anaconda
    $ conda install -c conda-forge sox
    ```
- sndfile
    ```bash
    # e.g. Ubuntu
    $ sudo apt-get install libsndfile1-dev
    # e.g. CentOS
    $ sudo yum install libsndfile
    ```
- ffmpeg (This is not required when insllataion, but used in some recipes)
    ```bash
    # e.g. Ubuntu
    $ sudo apt-get install ffmpeg
    # e.g. CentOS
    $ sudo yum install ffmpeg
    # e.g. Using anaconda
    $ conda install -c conda-forge ffmpeg
    ```
- flac (This is not required when insllataion, but used in some recipes)
    ```bash
    # e.g. Ubuntu
    $ sudo apt-get install flac
    # e.g. CentOS
    $ sudo yum install flac
    ```

Optionally, GPU environment requires the following libraries:

- Cuda 8.0, 9.0, 9.1, 10.0 depending on each DNN library
- Cudnn 6+, 7+
- NCCL 2.0+ (for the use of multi-GPUs)

### Supported Linux distributions and other requirements

We support the following Linux distributions with CI. If you want to build your own Linux by yourself,
please also check our [CI configurations](https://github.com/espnet/espnet/blob/master/.circleci/config.yml).
to prepare the appropriate environments

- ubuntu18
- ubuntu16
- centos7
- debian9


### Step 1) setting of the environment for GPU support

To use cuda (and cudnn), make sure to set paths in your `.bashrc` or `.bash_profile` appropriately.
```
CUDAROOT=/path/to/cuda

export PATH=$CUDAROOT/bin:$PATH
export LD_LIBRARY_PATH=$CUDAROOT/lib64:$LD_LIBRARY_PATH
export CFLAGS="-I$CUDAROOT/include $CFLAGS"
export CPATH=$CUDAROOT/include:$CPATH
export CUDA_HOME=$CUDAROOT
export CUDA_PATH=$CUDAROOT
```

If you want to use multiple GPUs, you should install [nccl](https://developer.nvidia.com/nccl)
and set paths in your `.bashrc` or `.bash_profile` appropriately, for example:
```
CUDAROOT=/path/to/cuda
NCCL_ROOT=/path/to/nccl

export CPATH=$NCCL_ROOT/include:$CPATH
export LD_LIBRARY_PATH=$NCCL_ROOT/lib/:$CUDAROOT/lib64:$LD_LIBRARY_PATH
export LIBRARY_PATH=$NCCL_ROOT/lib/:$LIBRARY_PATH
export CFLAGS="-I$CUDAROOT/include $CFLAGS"
export CPATH=$CUDAROOT/include:$CPATH
export CUDA_HOME=$CUDAROOT
export CUDA_PATH=$CUDAROOT
```


### Step 2) Install Kaldi
Related links:
- [Kaldi Github](https://github.com/kaldi-asr/kaldi)
- [Kaldi Documentation](https://kaldi-asr.org/)
  - [Downloading and installing Kaldi](https://kaldi-asr.org/doc/install.html)
  - [The build process (how Kaldi is compiled)](https://kaldi-asr.org/doc/build_setup.html)
- [Kaldi INSTALL](https://github.com/kaldi-asr/kaldi/blob/master/INSTALL)

Kaldi's requirements:
- OS: Ubuntu, CentOS, MacOSX, Windows, Cygwin, etc.
- GCC >= 4.7

We also have [prebuilt Kaldi binaries](https://github.com/espnet/espnet/blob/master/ci/install_kaldi.sh).


1. Git clone kaldi

    ```bash
    $ cd <any-place>
    $ git clone https://github.com/kaldi-asr/kaldi
    ```
1. Install tools

    ```bash
    $ cd <kaldi-root>/tools
    $ make -j <NUM-CPU>
    ```
    1. Select BLAS library from ATLAS, OpenBLAS, or MKL

    - OpenBLAS

    ```bash
    $ cd <kaldi-root>/tools
    $ ./extras/install_openblas.sh
    ```
    - MKL (You need sudo privilege)

    ```bash
    $ cd <kaldi-root>/tools
    $ sudo ./extras/install_mkl.sh
    ```
    - ATLAS (You need sudo privilege)

    ```bash
    # Ubuntu
    $ sudo apt-get install libatlas-base-dev
    ```

1. Compile Kaldi & install

    ```bash
    $ cd <kaldi-root>/src
    # [By default MKL is used] ESPnet uses only feature extractor, so you can disable CUDA
    $ ./configure --use-cuda=no
    # e.g. With OpenBLAS]
    # $ ./configure --openblas-root=../tools/OpenBLAS/install --use-cuda=no
    # If you'll use CUDA
    # ./configure --cudatk-dir=/usr/local/cuda-10.0
    $ make -j clean depend; make -j <NUM-CPU>
    ```

### Step 3-A) installation of espnet

```bash
$ cd <any-place>
$ git clone https://github.com/espnet/espnet
```

#### create miniconda environment (default)

Install Python libraries and other required tools with [miniconda](https://conda.io/docs/glossary.html#miniconda-glossary)
```sh
$ cd <espnet-root>/tools
$ make KALDI=<kaldi-root>
```

You can also specify the Python (`PYTHON_VERSION` default 3.7), PyTorch (`TH_VERSION` default 1.0.0) and CUDA versions (`CUDA_VERSION` default 10.0), for example:
```sh
$ cd <espnet-root>/tools
$ make KALDI=<kaldi-root> PYTHON_VERSION=3.6 TH_VERSION=0.4.1 CUDA_VERSION=9.0
```

#### create virtualenv from an existing python

If you do not want to use miniconda, you need to specify your python interpreter to setup `virtualenv`

```sh
$ cd <espnet-root>/tools
$ make KALDI=<kaldi-root> PYTHON=/usr/bin/python3.6
```

#### using the system Python without creating new python environment
We prepare a new Python interpreter independently in the Makefile, but if you'd like to use the System Python, for example, Google Colab originally provides an independent environment and you may use the Python as it is, then just create an empty file at `tools/venv/bin/activate`:

```sh
$ cd <espnet-root>/tools
$ rm -rf venv; mkdir -p venv/bin; touch venv/bin/activate  # Create an empty file
$ make KALDI=<kaldi-root> PYTHON=dummy
```

### Step 3-B) installation for CPU-only

To install in a terminal that does not have a GPU installed, just clear the version of `CUPY` as follows:

```sh
$ cd <espnet-root>/tools
$ make KALDI=<kaldi-root> CUPY_VERSION='' -j 10
```

This option is enabled for any of the install configuration.

### Step 4) installation check

You can check whether the install is succeeded via the following commands
```sh
$ cd <espnet-root>/tools
$ make check_install
```
or `make check_install CUPY_VERSION=''` if you do not have a GPU on your terminal.
If you have no warning, ready to run the recipe!

If there are some problems in python libraries, you can re-setup only python environment via following commands
```sh
$ cd <espnet-root>/tools
$ make clean_python
$ make python
```
