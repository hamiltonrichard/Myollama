# Install the Ollama Docker Image in OpenSUSE

The following instructions descripe how to run Ollamma in Docker to run in Windows Subsystem for Linux OpenSUSE instance.  The document assumes that your your system has a suported Nvidia card. See the Ollama documentation listed in the sources below for information on ATI Graphics cards. 

## Step 1: Install OpenSUSE in WSL

1. Open the Microsoft Store on the Windows Host
2. Search on ```OpenSUSE``` and locate the latest distribution
3. Click on ```free``` to install. 

## Step 2: Configure OpenSUSE in WSL

1. Click Next to start the install. 
2. Review the license and click ```next```
3. Setup your user. Select ```Use this password for system administrator```
4. Allow the system to update. This may take a few minutes. 

## Step 3: Enable Systemd In WSL

It may be necessary to enable ```systemd```. Follow these steps:

1. Install the ```wsl-systemd``` pattern. This creates or modifies /etc/wsl.conf and creates the symlinks for ```init```.

        sudo zypper install -t wsl-systemd
   
3. Downdown WSL by executing the following from PowerShell:
        
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

    
        sudo zypper ar https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo
    
    Optional Step:

    This option allows you to use experimental packages.

        sudo zypper modifyrepo --enable nvidia-container-toolkit-experimental
    

2. Install the NVIDIA Container Toolkit packages.

    
        sudo zypper --gpg-auto-import-keys install -y nvidia-container-toolkit
    
    

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
    

## Step 7: Install the cuda-toolkit

1. Add the cuda-toolkit repo

    
        sudo zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-$distro.repo 
    

2. Install the CUDA SDK

        sudo zypper install cuda-toolkit
    

## Step 8: Install the Ollama Docker Image

1. Pull the Ollama Docker image from the repo:

        docker pull ollama/ollama

2. Run the docker image:
    
        docker run -d --gpus=all -v ollama:/$HOME/ollama -p 11434:11434 --name ollama ollama/ollama
   
    Note: Ollama uses a volume to store its training data. You'll need to create the directory ahead of time.
              ```--gpus=all``` is used to take advantage of your system's GPU(s).  

3. Run llama3 with the following command:

        docker exec -it ollama ollama run llama3
 
## Sources

1. [How to Reboot WSL in Windows 10 or 11](https://otechworld.com/reboot-wsl-in-windows) 

2. [Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

3. [Ollama is now available as an offical Docker image](https://ollama.com/blog/ollama-is-now-available-as-an-official-docker-image)

4. [The Ollama Docker Repository](https://hub.docker.com/r/ollama/ollama)

5. [OpenSUSE sytemd docuumentation](https://doc.opensuse.org/documentation/leap/reference/html/book-reference/cha-systemd.html)
