# Deployment of Openstack using Kolla-ansible

Kolla-ansible uses docker and ansible playbook to have production ready containers. The version of kolla this is done on is 2023.1. Kolla-ansible deployment has 2 types of deployments 1. All-in-one 2. Multi-node. All-in-one would put all the service into one controller node whereas multinode separates the services into different machines. Multi-node deployment is recommended for organizations to allow scalability, better performance and better resource utilization. All this deployment is done in an Ubuntu OS. You will also require 2 network interfaces on all machines.

Having general knowledge of ansible and docker will greatly help in this process. Ensure that docker is installed on all your machines.
```
sudo apt-get install docker-compose
```

This will install a basic version to get you started.

## All-in-One Deployment

We tried an all-in-one deployment to test out kolla-ansible's system. First we need to install the dependencies required after updating and upgrading. Recommend to create a folder and run all this code in the folder
```
sudo apt install git python3-dev libffi-dev gcc libssl-dev
```

## Create Virtual Environment

We will run this in a virtual environment so that there are no conflicts with the system.

1. Install python virtual enviroment
```
sudo apt install python3-venv
```

2. Create a virtual environment and activate it:
```
python3 -m venv /path/to/venv
source /path/to/venv/bin/activate
```

When running the python3 command, you will need to use sudo if your user is not in the correct group. Make sure to use this command line to allow user to make changes without sudo to prevent any problems.
```
sudo chown -R $USER~/.blabla
```

1. From this line onwards, you will be in the virtual environment. Ensure the latest version of pip.
```
pip install -U pip
```

2. Install Ansible version min=6 max-7
```
pip install 'ansible\>=6,\<8'
```

## Install Kolla-Ansible

1. Install kolla-ansible using pip
```
pip install git+https://opendev.org/openstack/kolla-ansible@stable/2023.1
```

The kolla branch name will be whichever branch you are using. If you don't know, just use the main master branch "@master".

1. Create etc/kolla directory
```
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER/etc/kolla
```

1. Copy global.yml and password.yml to etc/kolla
```
cp -r /path/to/venv/share/kolla-ansible/etc\_examples/kolla/\* /etc/kolla
```

You will need the files later so be sure that you know where these files are located. If you are new, /etc means it's at the highest level directory.

1. Copy all-in-one into the current directory
```
cp /path/to/venv/share/kolla-ansible/ansible/inventory/all-in-one .
```

/path is also in the highest level directory. You can find your virtual environment files in this directory. We are copying the all-in-one file into the current directory you are in. For all-in-one deployment no changes are made.

## Install Ansible Galaxy
```
kolla-ansible install-deps
```

## Preparing for configuration

### Inventory file

For ansible, you would need an inventory file configured. All-in-one is your inventory file. No changes need to be made as everything will be connected to your local host.

### Passwords

You will need to generate passwords for your users. Currently your passwords.yml file will be blank and you can't edit it. Run this on your command line.
```
kolla-genpwd
```

### Globals.yml

The globals.yml file is your main configuration file for kolla-ansible. Default stored in etc/kolla/globals.yml. We need to configure a few main parts.

#### 1. Image Options

User will need to specify the image they will be using for the deployment. You are free to use any of the distros provided.
```
# Valid options are ['centos', 'debian', 'rocky', 'ubuntu']

kolla_base_distro: "ubuntu"
```
*Recommended to use the distros that you know.

#### 2. AArch64

To be researched….

#### 3. Networking

We need to set up a few networking interfaces for the deployment to work. First we need to set the default interface.

| network\_interface: "eno1" |
| --- |

Second interface is for Neutron external or public networks, can be vlan or flat. Interface needs to be active without an ip address. To allow instances to access external networks.

| neutron\_external\_interface: "eno2" |
| --- |

We also need a floating ip for management of traffic. For **all-in-one deployment only,** this kolla\_internal\_vip\_address will be your host machine ip address. You can use angry ip scanner to figure out which ip address can be pinged and available.

kolla\_internal\_vip\_address: "10.219.127.223"

You will also need set haproxy to "no".

enable\_haproxy: "no"

## Deployment

We can begin deployment once we are done with the configuration.

1. Bootstrap servers with kolla deploy dependencies:


| kolla-ansible -i ./all-in-onebootstrap-server |
| --- |

2. Do pre-deployment checks for hosts:

| kolla-ansible -i ./all-in-one prechecks |
| --- |

1. Finally proceed to actual OpenStack deployment:

| kolla-ansible -i ./all-in-one deploy |
| --- |

Once all is done, you can access openstack through your ip address.

## Multinode Deployment

For multinode deployment steps will change from the configuration point onwards. There are different types of nodes each with their own services.

- Control - Cloud controller nodes which host control services like APIs and databases. This group should have an odd number of nodes for quorum.
- Network - Network nodes host Neutron agents along with haproxy / keepalived. These nodes will have a floating ip defined in kolla\_internal\_vip\_address.
- Compute - Compute nodes for compute services. This is where guest VMs live.
- Storage - Storage nodes for cinder-volume, LVM or Swift.
- Monitoring - Monitor nodes which host monitoring services.

## Architecture

![](RackMultipart20230918-1-huzdii_html_55c2a02fb62369f0.png)

## Docker Registry(Not Compulsory; You need extensive knowledge on docker when using this)

A docker registry is generally recommended. It replaces the need to pull from a public registry to get images. Only one registry needs to be be deployed. It is not compulsory to deploy this registry.

Ensure that docker is installed in all your nodes.(This is compulsory regardless whether you are doing the registry or not)

| apt-get install docker-compose |
| --- |

Then, you can make a registry.

| docker run -d \
 --network host \
 --name registry \
 --restart=always \
 -e REGISTRY\_HTTP\_ADDR=0.0.0.0:4000 \
 -v registry:/var/lib/registry \
registry:2 |
| --- |

We use port 4000 to avoid conflict with Keystone. If registry is not on the same machine as keystone, **-e** can be omitted. In the globals.yml

# Custom docker registry settings:

docker\_registry: "10.219.127.88:4000"

# Please read the docs carefully before applying docker\_registry\_insecure.

docker\_registry\_insecure: "yes"

#docker\_registry\_username:

# docker\_registry\_password is set in the passwords.yml file.

# Namespace of images:

docker\_namespace: "kolla"

## Enabling passwordless ssh

root@controller:~# ssh-keygen

root@controller:~# ssh-copy-id -i .ssh/id\_rsa.pub root@controller

root@controller:~# ssh-copy-id -i .ssh/id\_rsa.pub root@node01

root@controller:~# ssh-copy-id -i .ssh/id\_rsa.pub root@node02

Do this on all the nodes. To ensure that they can passwordless ssh to each other.

### Useful command to know

To remove ssh keys

ssh-keygen -f "/root/.ssh/known\_host" -R "\<ip\_address\>"

If you still need to key in the password, it could be a permissions to the file problem. This 2 line of code will help you with that.

chmod 700 ~/.ssh

chmod 600 ~/.ssh/\*

## Globals.yml

Refer to globals file

## Multinode

Refer to multinode file

## Deployment

We can begin deployment once we are done with the configuration.

1. Bootstrap servers with kolla deploy dependencies:


| kolla-ansible -i ./multinode bootstrap-server |
| --- |

1. Do pre-deployment checks for hosts:

| kolla-ansible -i ./multinode prechecks |
| --- |

1. Finally proceed to actual OpenStack deployment:

| kolla-ansible -i ./multinode deploy |
| --- |

Once all is done, you can access openstack through your ip address.
