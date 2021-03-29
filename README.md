# WSL Recipes

## Prerequisites
Windows Subsystem for Linux
WSL needs to be installed if not currently installed, the easiest way to install it is from an Administrator cmd/PowerShell prompt:

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

https://docs.microsoft.com/en-us/windows/wsl/install-win10

## DockerDocker
Docker is used to create custom images

For Windows Docker Desktop is required:

https://www.docker.com/products/docker-desktop

Docker Desktop will require that CPU virtualisation is enabled in the EFI/BIOS settings of the machine.

## Creating the image
Pull your preferred distro image using docker, this example uses the latest CentOS release which at time or writing is 8.2.2004, but any other base distro could be used, Fedora, Arch, Alpine, Amazon, Oracle, openSUSE.

    docker pull centos:latest
Create new container, feel free to replace CentOS_8 with what ever name you would like:

    docker run --tty --detach --name CentOS_8 centos
Customise the container, this can be done a few different ways, the first is to enter the container and run Linux commands manually:

    docker exec -it CentOS_8 bash

The second is execute commands using docker exec, this has a nice bonus in that if you are running in PowerShell you can use environment variables to automate things like user creation.

The below example updates all packages, installs extra packages including epel, creates a new user that matches your Windows username, adds the user to the wheel group, sets the password to something simple, cleans up dnf cache to reduce image size and finally clears the bash history for a clean start.


    docker exec -it CentOS_8 dnf -y update
    docker exec -it CentOS_8 dnf install -y bash-completion bind-utils cracklib-dicts diff epel-release glibc-langpack-en keychain man minicom mlocate nano net-tools nmap openssh-clients openssl passwd psmisc sudo telnet tmux tree vim wget yum-utils zsh
    docker exec -it CentOS_8 dnf install -y htop p7zip
    docker exec -it CentOS_8 useradd $env:UserName
    docker exec -it CentOS_8 usermod -aG wheel $env:UserName
    docker exec -it CentOS_8 sh -c "echo 'Password' | passwd --stdin ${env:UserName}"
    docker exec -it CentOS_8 sh -c "echo -e '[user]\ndefault=$env:UserName' > /etc/wsl.conf"
    docker exec -it CentOS_8 dnf clean all
    docker exec -it CentOS_8 bash -c 'cat /dev/null > ~/.bash_history && history -c && exit'

Stop the container so that it can be exported:

    docker stop CentOS_8

Export the container to a tar file:

    docker export CentOS_8 -o .\CentOS_8.tar

Create a new directory to import the container from docker:

    mkdir C:\WSL\CentOS_8

Import the container in to WSL:

    wsl --import CentOS_8 C:\WSL\CentOS_8 CentOS_8.tar

That's it, the WSL distro can now be run using the below command:

    wsl -d CentOS_8
Note: If at this point you are seeing wsl errors about no --import option available you should check your version of Windows and ensure that you are running a fairly recent release. If you need to update Windows so you have the wsl command line options available you can do so here: https://www.microsoft.com/en-gb/software-download/windows10

To make using WSL easier you can install Windows Terminal from the Microsoft Store. If Windows Terminal was running prior to import then it may need to be closed and reopened to have the new distro appear in the drop down list.

If you make changes to your container and need to reimport it or remove it the --unregister option can be used from the wsl command to remove the distro from WSL:

    wsl --unregister CentOS_8

Updating the image
Updating a container is a similar process, note, this is a destructive process, if you just want to update packages then run dnf update from within the container.

    docker start CentOS_8
    docker exec -it CentOS_8 bash

Run any other commands you want from within the container then exit the with the exit command ctrl+D

    docker stop CentOS_8
    docker export CentOS_8 -o .\CentOS_8.tar
Unregister and import the container, note that this is a destructive process and any changes made within a running WSL instance will be lost.

    wsl --unregister CentOS_8
    wsl --import CentOS_8 C:\WSL\CentOS_8 CentOS_8.tar
