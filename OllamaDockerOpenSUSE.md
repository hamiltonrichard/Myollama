# Installing the Ollama Docker Image in OpenSUSE in Windows Subsystem for Linux

The following instructions describe how to run Ollama in Docker to run within a Windows Subsystem for Linux OpenSUSE instance.  The document assumes that your your system has a suported Nvidia card. See the Ollama documentation listed in the sources below for information on ATI Graphics cards.  Information on creating and restoring a backup of the WSL image can be found at the end of the document. 

Prior to starting, refer to open issues for the current state of this documentation. 

## Step 1: Install OpenSUSE in WSL

1. Open the Microsoft Store on the Windows Host
2. Search on ```OpenSUSE``` and locate the latest distribution
3. Click on ```free``` to install. 

## Step 2: Configure OpenSUSE in WSL

In a terminal window, walk through the installation steps.

1. Click Next to start the install. 
2. Review the license and click ```next```
3. Setup your user. Select ```Use this password for system administrator``` if desired.
4. Allow the system to update. This may take a few minutes. 

## Step 3: Enable Systemd In WSL

It may be necessary to enable ```systemd```. Follow these steps:

1. Install the ```wsl-systemd``` pattern. This creates or modifies /etc/wsl.conf and creates additional symlinks.

        sudo zypper install -t wsl-systemd
   
3. Shutdown WSL by executing the following from PowerShell:
        
        wsl --shutdown
        
5. Open your OpenSUSE WSL instance and verify systemd is enabled:

        sudo systemctl list-unit-files --type=service

   Output from the command should look similar to this:

    ```
    UNIT FILE                                STATE           PRESET
    autovt@.service                          alias           -
    backup-rpmdb.service                     static          -
    backup-sysconfig.service                 static          -
    boot-sysctl.service                      disabled        disabled
    ca-certificates.service                  disabled        disabled
    check-battery.service                    static          -
    console-getty.service                    enabled-runtime disabled
    container-getty@.service                 static          -
    dbus-org.freedesktop.hostname1.service   alias           -
    dbus-org.freedesktop.locale1.service     alias           -
    dbus-org.freedesktop.login1.service      alias           -
    dbus-org.freedesktop.timedate1.service   alias           -
    ```

## Step 4: Installing Docker

1. Prior to installing docker update the system

        sudo zypper update -y


2. Install the docker packages:

        sudo zypper install docker docker-compose docker-compose-switch

3. Enable Docker 

        sudo systemctl enable docker

4. Add a user to the Docker group:

        sudo usermod -G docker -a $USER

5. Login to the docker group

        newgrp docker

6. Restart the docker daemon

        sudo systemctl restart docker

7. Verify docker is running 

        docker version

8. Pull down a test image and run it

        docker run hello-world

## Step 5: Install the NVIDIA Container toolkit

1. Add the following Repository:

        sudo zypper ar https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo &&
        sudo zypper refresh
    
    Optional Step:

    This option allows you to use experimental packages.

        sudo zypper modifyrepo --enable nvidia-container-toolkit-experimental
    

3. Install the NVIDIA Container Toolkit packages.
    
        sudo zypper --gpg-auto-import-keys install -y nvidia-container-toolkit
   
4. Configure the Docker runtime.

        sudo nvidia-ctk runtime configure --runtime=docker

5. Restart docker.

        sudo systemctl restart docker

6. Configure the Docker runtime.

        nvidia-ctk runtime configure --runtime=docker --config=$HOME/.config/docker/daemon.json

7. Restart Docker.

        sudo systemctl restart docker
   
3. Run a sample CUDA ontainer: 
 
        sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
    
    The output should look like:
    ```
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 535.86.10    Driver Version: 535.86.10    CUDA Version: 12.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
    | N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    ```
## Step 6: Configure Docker 

1. Use the nvidia-ctk to configure docker:

        sudo nvidia-ctk runtime configure --runtime=docker
    
2. Restart the docker daemon:

        sudo systemctl restart docker
    
## Step 7: Install the cuda-toolkit and drivers

1. Install the ```devel_kernel``` pattern. This is required to build the NVIDIA drivers

           sudo zypper in -t pattern devel_kernel
   
2. Add the cuda-toolkit repo

        sudo zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/cuda-opensuse15.repo &&
        sudo zypper refresh

3. Install the CUDA SDK

        sudo zypper install cuda-toolkit-12-5
   
5. Install the kernel module. (See the __Notes__ below before executing this step.)

        sudo zypper install -y nvidia-open-driver-G06-kmp-default

7. Install the cuda drivers

        sudo zypper install -y cuda-drivers

## Step 8: Install the Ollama Docker Image

1. Pull the Ollama Docker image from the repo:

        docker pull ollama/ollama

2. Run the docker image:
    
        docker run -d --gpus=all -v ollama:/$HOME/ollama -p 11434:11434 --name ollama ollama/ollama
   
    Note: Ollama uses a volume to store its training data. You'll need to create the directory ahead of time.
              ```--gpus=all``` is used to take advantage of your system's GPU(s).  

3. Run llama3 with the following command:

        docker exec -it ollama ollama run llama3

### Useful docker commands

Not familiar with Docker? The following commands can be used to stop/start the ollama container and also check to see if it is running 
or not.  

_Stopping the ollama container_

To stop the docker container created in step two above use:

        docker stop ollama

_Starting the ollama container._

Start the ollama container by using:

        docker start ollama

_Checking on the container status._

To determine if the container is running our not you can use:

        docker ps

The command display data about running containers

```
$ docker ps
CONTAINER ID   IMAGE           COMMAND               CREATED             STATUS          PORTS                                           NAMES
1834d3fe4d06   ollama/ollama   "/bin/ollama serve"   About an hour ago   Up 17 seconds   0.0.0.0:11434->11434/tcp, :::11434->11434/tcp   ollama
```
## Backup and Restoring WSL Images

It is recommended to export the WSL Immages during the installation procuess. 

_Identify the WSL distrobution_ 

Execute the following command to identify the installed distrobutions

```wsl -l -v```

It should give you output simular to:

```
PS C:\Users\richh\Documents\WSL> wsl -l -v
  NAME                  STATE           VERSION
* Ubuntu                Stopped         2
  openSUSE-Leap-15.5    Stopped         2
  openSUSE-Leap-15.6    Running         2
```

_Export the desired distribution_

Use the --export option to crate a tar archive. 

        wsl --export [NAME] [TAR Arcive filename]
        
To export the OpenSUSE-Leap-15.6 distribution use the following command:

        wsl --export openSUSE-Leap-15.6 openSUSE-Leap-15.6.wsl 

_Import the WSL Instance_ 

Terminate the running instance
        
        wsl --terminate OpenSUSE-Leap-15.6

Unregister the instance

        wsl --unregister OpenSUSE-Leap-15.6

Import the instance

        wsl --import OpenSUSE-Leap-15.6.wsl



## Notes

1. There is a note about switching or installing other driver versions listed in the [CUDA Toolkit 12.5 Update document](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=OpenSUSE&target_version=15&target_type=rpm_network). Additional information about the driver options can be found in the [CUDA Installation Guide](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#nvidia-open-gpu-kernel-modules). Review the information and act accordingly.

## Sources

1. [How to Reboot WSL in Windows 10 or 11](https://otechworld.com/reboot-wsl-in-windows) 

2. [Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

3. [CUDA Toolkit 12.5 Update 1](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=OpenSUSE&target_version=15&target_type=rpm_network)
   
4. [The CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/#meta-packages)

5. [Ollama is now available as an offical Docker image](https://ollama.com/blog/ollama-is-now-available-as-an-official-docker-image)

6. [The Ollama Docker Repository](https://hub.docker.com/r/ollama/ollama)

7. [OpenSUSE sytemd documentation](https://doc.opensuse.org/documentation/leap/reference/html/book-reference/cha-systemd.html)

8. [Windows Subsystem for Linux Documentation](https://learn.microsoft.com/en-us/windows/wsl/) Link requires registration. 
 
