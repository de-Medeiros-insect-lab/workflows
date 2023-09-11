# Initial setup

## Windows users
The most straightforward way to use a Unix terminal on Windows is to install the Windows Subsystem for Linux. Check instructions [here](https://learn.microsoft.com/en-us/windows/wsl/install).

## Getting access and basic Linux usage

Check this repository for general Grainger Bioinformatics Center (GBC) instructions for server access and initial setup: <https://felixgrewe.github.io/linux_cookbook/>

If you are using multiple servers in your work, you will need to repeat the setup here in each one of them.

## Installing miniconda

Almost all programs that you will use can be installed using [Anaconda](https://anaconda.org), so it is a good idea to install that first. The light version of Anaconda is miniconda. You can find the version available here: <https://docs.conda.io/en/latest/miniconda.html#linux-installers>

Currently (April 2023), the architecture of all GBC servers if Linux 64-bit, and the link to the latest version is `https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`

After logging into the server, download miniconda by using the command wget. Assuming that the link above is still valid, this would be:

`wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`

This will download a file named `Miniconda3-latest-Linux-x86_64.sh`.  Now you can run the following to install:

`bash Miniconda3-latest-Linux-x86_64.sh`

Check the full installation instructions here: <https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html>

Once you are done, you can delete `Miniconda3-latest-Linux-x86_64.sh` using:

`rm Miniconda3-latest-Linux-x86_64.sh`

## Setting up .bash_profile

A hidden file named `.bashrc` holds several configuration parameters for your linux terminal that are loaded every time you use it. It is a good idea to add a few settings to make your life easier:

### Improving history

The Linux terminal records a history of all of the commands that you have used. You can search previous commands by using the `up` and `down` arrows or by text using `CTRL+R`. By default, there is a limit of how many commands are stored in the history and every command is stored, even if it is repeated. By adding a few lines to `.bashrc`, you can change this behavior.

In your home folder, use the following commands to improve history handling:

```
echo '#do not save duplicated commands in history' >> .bashrc
echo 'export HISTCONTROL=ignoreboth:erasedups' >> .bashrc

echo '#always save to history file immediatelly' >> .bashrc
echo 'export PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"' >> .bashrc

echo '#setting history to unlimited number of records and up to 1GB in disk'  >> .bashrc
echo 'export HISTSIZE=-1'  >> .bashrc
echo 'export HISTFILEZE=1000000' >> .bashrc
```

### Other options

We will add other customizations here as they prove to be useful




