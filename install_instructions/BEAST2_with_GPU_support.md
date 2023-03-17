# Installing BEAST2 with GPU support

[BEAST2](https://www.beast2.org) can run much faster with a GPU, which requires a [BEAGLE](https://github.com/beagle-dev/beagle-lib) library compiled with GPU support. This is not the version that you can currently install with [Anaconda](https://anaconda.org), so here we will show how to install the requirements with Anaconda and then compile and install BEAGLE to you conda environment.

Currently (2023) the only GPU server at the Field Museum is **Chandler**, so these instructions will only work ins that server.



## 1. Checking NVIDIA driver version

You can check the version of NVIDIA drivers installed with the following command in the terminal window:

```sh
nvidia-smi
```

This will output something like this (screenshot in March 2023):
![nvidia-smi](images/nvidia-smi.png)

In this case, it shows that the CUDA Version is 11.8 and the CUDA driver is 520.56.06. Use this [link](https://docs.nvidia.com/deploy/cuda-compatibility/#minor-version-compatibility) in NVIDIA website to check which CUDA toolkit is compatible with the driver version we have.

As of March 2023, this is the table found in the website:
![nvidia cuda compatibility table](images/nvidia-cuda-table.png)

This means that we need a CUDA driver of version 525.60.13 or above for a CUDA 12.x toolkit, which is above the version we currently have (520.56.06). Therefore, we will need to install the CUDA toolkit version 11.x (that is, below version 12, but it could be any subversion of 11).



## 2. Installing dependencies with Anaconda

Now that we know the version of the cudatooljit we need, we can use Anaconda to set up an environment to run BEAST2. Let's name our enviroment `beast2`. Notice that we are constraining the version of cudatoolkit to maintain compatibility with our server NVIDIA driver:

```sh
conda create -n beast2 -c bioconda -c conda-forge beast2 gcc cmake autoconf automake libtool subversion pkg-config cudatoolkit-dev=11
```

This will create a new conda environment with the packages we need, and we can activate it using
```sh
conda activate beast2
```


## 3. Installing BEAGLE GPU to the conda environment

After activating the environment, let's compile and install BEAGLE with GPU support to our environment. First, let's download the BEAGLE github repositoru to our home folder:

```sh
conda activate beast2
cd ~
git clone --depth=1 https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
mkdir build
cd build
cmake -DBUILD_CUDA=ON -DCMAKE_INSTALL_PREFIX:PATH=$CONDA_PREFIX ..
make install
```

This is slightly different from the instructions in BEAGLE github repository, to make sure that the BEAGLE library is installed within this conda environment and not to your home folder.

If everything worked well, you will see several messages during compilation, they should metion that CUDA was found, like this:

![beagle compilation](images/beagle-compilation.png)

Now you can delete the github repository:
```sh
cd ~
rm -r beagle-lib
```


## 4. Running BEAST2 with GPU support

To run BEAST2 with GPU support, you will need to activate your conda environment and include the flag `-beagle_GPU`. For example, if you are in a directory containing a BEAST `xml` file, you could use:

```sh
conda activate beast2
beast -statefile checkpoint -beagle_GPU -seed 45684121 *.xml
```


# Workflow contributors
Created: B. de Medeiros March 2023

Updated:
