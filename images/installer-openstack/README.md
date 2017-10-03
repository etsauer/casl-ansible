OpenStack Docker Client
==================

Produces a container capable of acting as a client for OpenStack

## Setup

The following steps are required to run the docker client.

1. Install docker and docker-compose
  1. on RHEL/Fedora: ```{yum/dnf} install docker docker-compose```
  2. on Windows: [Install Docker for Windows](https://docs.docker.com/windows/step_one/)
  3. on OSX: [Max OS X](https://docs.docker.com/installation/mac/)
  4. on all other Operating Systems: [Supported Platforms](https://docs.docker.com/installation/)
2. Give your user access to run Docker containers (this is only required in Linux/Unix distros)
```
groupadd docker
usermod -a -G docker ${USER}
systemctl enable docker
systemctl restart docker
```

## Running

A typical run of the image would look like:

```
docker run -u `id -u` \
      -v $HOME/.ssh/id_rsa:/opt/app-root/src/.ssh/id_rsa:Z \
      -v $HOME/src/:/tmp/src:Z \
      -v $HOME/.config/openstack/:/opt/app-root/src/.config/openstack/ \
      -e INVENTORY_DIR=/tmp/src/casl-ansible /inventory/sample.casl.example.com.d/inventory/ \
      -e PLAYBOOK_FILE=/tmp/src/casl-ansible/playbooks/openshift/end-to-end.yml \
      -e OPTS="-v" -t \
      redhatcop/control-host-openstack
```

```
docker run -it --name control-host -v $HOME/.ssh:/root/.ssh -v $HOME/.config/openstack:/root/.config/openstack -v $HOME/src:/root/repository -v $HOME/.ansible.cfg:/root/.ansible.cfg docker.io/redhatcop/control-host-openstack bash
[]# ansible-playbook -i /root/code/casl-ansible/inventory/sample.casl.example.com.d/inventory/ code/casl-ansible/playbooks/openshift/end-to-end.yml
```

### OpenStack Configuration Files

The above commands expect you to have an link:https://docs.openstack.org/user-guide/common/cli-set-environment-variables-using-openstack-rc.html[OpenStack RC file] at `~/.config/openstack/openrc.sh`.

### Repository Content & Scripts

The above commands expect your ansible inventories and playbooks repos to live at `~/src`. If it lives elsewhere you'll need to update those file paths, either in the command, or in `docker-compose.yml`.

### SSH Keys

The above commands expect to mount the ssh keys needed to authenticate to openstack servers from ~/.ssh. If they live elsewhere, you'll need to update those paths in the command or in `docker-compose.yml`.

## Building

The image can be built with the following command:

```
cd ./docker/control-host-openstack
docker build -t redhatcop/control-host-openstack .
```


## Troubleshooting

Below are some of helpful hints for resolving issues experiencing while configuring and running the container

**Issue #1**

```
$ ./run.sh
time="2015-09-01T11:22:05-04:00" level=fatal msg="Get http:///var/run/docker.sock/v1.18/images/json: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?"
Building Docker Image rhtconsulting/rhc-openstack-client....
Sending build context to Docker daemon
FATA[0000] Post http:///var/run/docker.sock/v1.18/build?cgroupparent=&cpusetcpus=&cpushares=0&dockerfile=Dockerfile&memory=0&memswap=0&rm=1&t=rhtconsulting%2Frhc-openstack-client: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?
```

**Resolution #1**

Verify the Docker service is running

**Issue #2**

```
./run.sh
time="2015-09-01T11:32:36-04:00" level=fatal msg="Get http:///var/run/docker.sock/v1.18/images/json: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS?"
Building Docker Image rhtconsulting/rhc-openstack-client....
Sending build context to Docker daemon
FATA[0000] Post http:///var/run/docker.sock/v1.18/build?cgroupparent=&cpusetcpus=&cpushares=0&dockerfile=Dockerfile&memory=0&memswap=0&rm=1&t=rhtconsulting%2Frhc-openstack-client: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS?
Starting OpenStack Client Container....
FATA[0000] Post http:///var/run/docker.sock/v1.18/containers/create: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS?
```

**Resolution #2**

This error indicates the currently logged in user is unable to access the docker socket.

To resolve this issue, create a new *docker* group and add the user to the *docker* group

```
groupadd docker
usermod -a -G docker ${USER}
systemctl enable docker
systemctl restart docker
```

Reboot the machine or log out/log in to reload your environment and complete the configurations.
