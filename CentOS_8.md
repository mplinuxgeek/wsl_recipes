    docker pull centos:latest
    docker run --tty --detach --name CentOS_8 centos
    docker exec -it CentOS_8 dnf -y update
    docker exec -it CentOS_8 dnf install -y bash-completion bind-utils cracklib-dicts epel-release glibc-langpack-en man minicom mlocate nano net-tools nmap openssh-clients openssl passwd psmisc sudo telnet tmux tree vim wget yum-utils zsh
    docker exec -it CentOS_8 dnf install -y htop p7zip
    docker exec -it CentOS_8 useradd $env:UserName
    docker exec -it CentOS_8 usermod -aG wheel $env:UserName
    docker exec -it CentOS_8 sh -c "echo 'Password' | passwd --stdin ${env:UserName}"
    docker exec -it CentOS_8 sh -c "echo -e '[user]\ndefault=$env:UserName' > /etc/wsl.conf"
    docker exec -it CentOS_8 dnf clean all
    docker exec -it CentOS_8 bash -c 'cat /dev/null > ~/.bash_history && history -c && exit'
    docker stop CentOS_8
    docker export CentOS_8 -o .\CentOS_8.tar
    mkdir -force C:\WSL\CentOS_8
    wsl --import CentOS_8 C:\WSL\CentOS_8 CentOS_8.tar
    wsl -d CentOS_8
