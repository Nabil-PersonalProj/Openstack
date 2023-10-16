sudo apt install libguestfs-tools

virt-customize -a bionic-server-cloudimg-amd64.img --root-password password:<pass>

virt-customize -a bionic-server-cloudimg-amd64.img --run-command 'useradd -m -s /bin/bash -G sudo ubuntu' --run-command 'passwd ubuntu'
