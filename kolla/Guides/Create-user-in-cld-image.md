sudo apt install libguestfs-tools

virt-customize -a bionic-server-cloudimg-amd64.img --root-password password:<pass>

virt-customize -a bionic-server-cloudimg-amd64.img --run-command 'useradd -m -s /bin/bash -G sudo ubuntu' --run-command 'passwd ubuntu'



virt-customize -a jammy-server-cloudimg-amd64.img --root-password password:<pass> --install ubuntu-gnome-desktop --run 


#!/bin/bash

# Define the image file
IMAGE_FILE="test-jammy-server-cloudimg-amd64.img"

# Define root and user passwords
ROOT_PASSWORD="password"
NEW_USER="ctg1"
NEW_USER_PASSWORD="ctg1"

# Run virt-customize to set the root password, install Ubuntu GNOME desktop, create a new user, and set their password
virt-customize -a "$IMAGE_FILE" --root-password "password:$ROOT_PASSWORD" --install "ubuntu-gnome-desktop" 
#--run /bin/bash -c 'systemctl set-default graphical.target'
virt-customize -a "$IMAGE_FILE" --run-command "useradd -m $NEW_USER"
virt-customize -a "$IMAGE_FILE" --run-command "echo -e \"$NEW_USER_PASSWORD\n$NEW_USER_PASSWORD\" | passwd $NEW_USER