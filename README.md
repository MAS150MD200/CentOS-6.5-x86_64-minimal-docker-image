# CentOS 6.5 x86_64 minimal docker image

## How this image was done
Image created using this manual can be found on [dockerhub](https://hub.docker.com/r/roman8422/centos6.5/). 
All steps were done in VirtualBox [Vagrant](https://www.vagrantup.com/intro/index.html) VM with CentOS 6.5. This centos6.5-minimal image was used https://app.vagrantup.com/RomanV/boxes/centos65.

It also OK to use fresh CentOS 6.5 installation.

## Steps:
  - ssh into new VM
  - install git 
    ```sh
    yum -y install git
    ```
To not reinvent the wheel we will use [mkimage-yum.sh](https://github.com/moby/moby/blob/master/contrib/mkimage-yum.sh) to create image.
  - clone repository:
    ```sh
    cd /root \
    && git clone https://github.com/moby/moby/ \
    && cd /root/moby/contrib 
    ```
  - update yum repo urls:
    - comment out mirrorlist stanzas:
        ```sh 
        sed -i 's/^\(mirrorlist\)/#\1/' /etc/yum.repos.d/CentOS-Base.repo
        ```
    - Enable baseurl stanzas:
        ```sh
        sed -i 's/^#\(baseurl\)/\1/' /etc/yum.repos.d/CentOS-Base.repo 
        ```
    - Switch to CentOS 6.5 archive:
        ```sh
        sed -i 's?mirror.centos.org/centos/$releasever?vault.centos.org/6.5?' /etc/yum.repos.d/CentOS-Base.repo
        ```
     - Remove all cached data:
        ```sh
        yum clean all
        ```
  - Since we don't have docker installed on this machine we want to update last 3 lines in mkimage-yum.sh
    ```diff
    diff --git a/contrib/mkimage-yum.sh b/contrib/mkimage-yum.sh
    index 9012804..abb29b6 100755
    --- a/contrib/mkimage-yum.sh
    +++ b/contrib/mkimage-yum.sh
    @@ -129,8 +129,4 @@ if [ -z "$version" ]; then
         version=$name
     fi
     
    -tar --numeric-owner -c -C "$target" . | docker import - $name:$version
    -
    -docker run -i -t --rm $name:$version /bin/bash -c 'echo success'
    -
    -rm -rf "$target"
    +tar --numeric-owner -czf /root/$name.tar.gz -C "$target" . 
    ```
  - build new docker image
    ```sh
    ./mkimage-yum.sh centos6.5
    ```
  - copy image to your docker machine and import it with
    ```sh
    cat centos6.5.tar.gz | sudo docker import - roman8422/centos6.5
    ```
  - test your new image
    ```sh
    docker run -it --rm roman8422/centos6.5 cat /etc/issue
    CentOS release 6.5 (Final)
    Kernel \r on an \m
    ```

