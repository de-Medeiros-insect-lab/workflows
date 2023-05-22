# Installation of MinKNOW with Guppy GPU on Ubuntu 22.04.2 LTS

MinKNOW is currently supported on Ubuntu 18.04 and 20.04 (see more on Nanopore community [here](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/experiment-companion-minknow/v/mke_1013_v1_revcs_11apr2016/installing-minknow-on-linu). This is an experimental tutorial for installation and configuration of MinKNOW and the GPU version of Guppy using on Ubuntu 22.04 (Jammy), using the ONT repository for Ubuntu 20.04 (Focal). 

The following commands need to be entered into a terminal in order to configure a GPU version of Guppy with MinKNOW for MinION on Ubuntu. Note that some of the commands used in this tutorial will require superuser privileges. 

## 1. Before starting installations... Make sure that your GPU driver is working properly on your device. 

```bash
nvidia-smi
```

If the GPU driver is working properly, this command will display the details about your NVIDIA GPU on your system - something like this: 

![Screenshot showing that the GPU driver is working](nvidia-smi_GPU.png)

If `nvidia-smi` does not return any response on your GPU driver, it means your device is not working on GPU mode, and you will need to (re)install the GPU driver. You can install the latest version of GPU using the command `sudo apt install nvidia-driver`.


## 2. Add the ONT repository to your system (i.e., install all Oxford Nanopore Technologies-specific dependency packages)

**Important note:** In this tutorial, we are trying to add the ONT repository for **focal** (Ubuntu version 20.04), instead of **jammy** (Ubuntu version 22.04), which is our actual version. So, in the last line of the command bellow, enter the **focal** Ubuntu codename, as `focal-stable non-free`.

Using these commands, you will update the system packages list, add the ONT key, and add the ONT repository: 

```bash
sudo apt update 
wget -O- https://mirror.oxfordnanoportal.com/apt/ont-repo.pub | sudo apt-key add -
echo "deb http://mirror.oxfordnanoportal.com/apt focal-stable non-free" | sudo tee /etc/apt/sources.list.d/nanoporetech.sources.list
```

Once the ONT repository has been added, update it and confirm that you have access to the ONT software in Focal version: 

```bash
sudo apt update

apt policy minknow-core-minion-nc 
```

The latter command should return details on the installed ONT software - something like this: 

![Screenshot showing the info of the ONT software policy](minknow-core-minion-nc_Focal.png)


## 3. Add the Focal (20.04) repository

Use `nano` reate a file `system-focal.sources` in the directory `/etc/apt/sources.list.d.`. 

```bash
sudo nano /etc/apt/sources.list.d/system-focal.sources
```

Then, copy the list of the Focal (20.04) resources into the file and save it (to save, hit **ctrl + X**, **Y**, **enter**):

```
X-Repolib-Name: Pop_OS System Sources
Enabled: yes
Types: deb deb-src
URIs: http://us.archive.ubuntu.com/ubuntu/
Suites: focal focal-security focal-updates focal-backports
Components: main restricted universe multiverse
X-Repolib-Default-Mirror: http://us.archive.ubuntu.com/ubuntu/
```

Check that the file exists and contains the right information:

```bash
cat /etc/apt/sources.list.d/system-focal.sources
```

![Screenshot showing that the file system-focal.sources is OK](images/system-focal.sources.png)


## 4. Pin the Focal (20.04) repository

Pinning is an advance package management concept that essentially allows us to set priorities on particular packages, repositories or even whole releases.

To do this, using nano, create a file `focal-default-settings` in the directory `/etc/apt/preferences.d`. 

```bash
sudo nano /etc/apt/preferences.d/focal-default-settings
```

Then, copy the below into the file and save it (to save, hit **ctrl + X**, **Y**, **enter**):

```
Package: *
Pin: release n=focal*
Pin-Priority: 10
```

Check that the file exists and contains the right information:

```bash
cat /etc/apt/preferences.d/focal-default-settings
```

![Screenshot showing that the file focal-default-settings is OK](images/focal-default-settings.png)

  A pin priority of 10 is defined as:

  ```
  0 < P <=100
    causes a version to be installed only if there is no installed version of the package
  ```

  This means that if we update using `apt` we’ll have access to all focal packages, but they won’t be installed if there is a **Jammy** package with the same name. Focal packages will only be installed if they are the only option and a package depends on them, or if we specifically ask for it to be installed.


## 5. Update and check pinning

Using the commands bellow, you will update the package list and check the pinning.

```bash
sudo apt update
```

  Check pinning using `r-base` **as an example**, because it has different versions between Focal and Hirsute):

  ```bash
  apt policy r-base
  ```

  ![Screenshot showing that pinning is OK](images/policy_r-base_Pinning_check.png)

  In the above screenshot you’ll notice that the **Jammy** version of `r-base` is 4.1.2 and has a pin-priority of 500, while the **focal** version is 3.6.3 and has a pin-priority of 10.


## 6. Install MinKNOW (version for GPU) and required packages

Install MinKNOW (version for GPU) from the Nanopore comunity respository: 

```bash
sudo apt update
sudo apt install ont-standalone-minknow-gpu-release
```


## 7. Install Guppy (version for GPU)

The `ont-guppy` package is the GPU version that is currently supported by most recent version of MinKNOW. Install it using the following commands:

```bash
sudo apt update
sudo apt install ont-guppy 
```

  Using the command `apt show ont-guppy`, you can  explore the `ont-guppy package`: 

  ![Screenshot showing info on the installed Guppy](images/show_ont-guppy.png)

  And using `apt depends ont-guppy`, you can check the dependencies of the `ont-guppy package`:

  ![Screenshot showing the depends ont-guppy](images/depends_ont-guppy.png)

  Once Guppy is installed you should be able to check it’s paths using `which guppy_basecall_server`: /usr/bin/guppy_basecall_server; as well as its version using `/usr/bin/guppy_basecall_server --version`: Guppy Basecall Service Software, (C)Oxford Nanopore Technologies plc. Version 6.5.7+ca6d6af, client-server API version 15.0.0. 


## 8. Set up `guppyd.service` for GPU

Create a file `guppyd.service` in the directory `/lib/systemd/system`:

```bash
sudo nano /lib/systemd/system/guppyd.service
```

And copy the below into the file and save it (to save, hit **ctrl + X**, **Y**, **enter**):

```
[Unit]
Description=Service to manage the guppy basecall server.
Documentation=https://community.nanoporetech.com/protocols/Guppy-protocol/v/GPB_2003_v1_revQ_14Dec2018

[Service]
Type=simple
ExecStart=/opt/ont/guppy/bin/guppy_basecall_server --log_path /var/log/guppy --config dna_r9.4.1_450bps_fast.cfg --port 5555 -x cuda:all 
Restart=always
User=root
MemoryLimit=8G
MemoryHigh=8G
CPUQuota=200%

[Install]
Alias=guppyd.service
WantedBy=multi-user.target
```

Then, check that the file exists and contains the right information:

```bash
cat /lib/systemd/system/guppyd.service
```

![Screenshot showing that the guppyd.service is right](images/guppyd.service.png)

  In case you cannot open or edit the `guppyd.service` unit because it is masked, run the following commands to unmask and enable it:

  ```bash
  systemctl status guppyd.service
  sudo systemctl unmask guppyd.service
  sudo systemctl enable guppyd.service
  ```

  Then, you add the content bellow and restart the unit: 

  ```bash
  sudo systemctl start guppyd.service
  ```

  Now, check that the GPU version of Guppy is running as a server:

  ```bash
  nvidia-smi
  ```

  You will see guppy_basecall_server listed in the Processes section.
  ![Screenshot showing that Guppy GPU is running as a server](images/nvidia-smi_GPU_running_server.png)


  That's all! Now we are ready to open the MinKNOW GUI from the applications list and run the playback for Readfish.


## Some links I accessed to make this tutorial

Installation of Guppy for GPU
https://community.nanoporetech.com/docs/prepare/library_prep_protocols/Guppy-protocol/v/gpb_2003_v1_revas_14dec2018/linux-guppy

Installation of MinKNOW
https://community.nanoporetech.com/docs/prepare/library_prep_protocols/experiment-companion-minknow/v/mke_1013_v1_revcs_11apr2016/installing-minknow-on-linu

MinKNOW for Ubuntu 22.04 LTS
https://community.nanoporetech.com/posts/minknow-for-ubuntu-22-04-l

Running (live) GPU basecalling on Ubuntu 21.04
https://hackmd.io/@Miles/ryVAI_KWF
