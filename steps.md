After creating a new instance of Ubuntu 22.04

## Create a user
```bash
sudo useradd -m -s /bin/bash "deployer"
sudo passwd "deployer"
sudo usermod -a -G www-data "deployer"
sudo nano /etc/sudoers
```
Add this line to the sudoers file, and save:
`deployer ALL=(ALL) NOPASSWD:ALL`

Create SSH-key
`su - "deployer" -c "ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -q -N ''"`

Copy the SSH key output and add to GITHUB:
`su - "$deploy_user" -c "cat ~/.ssh/id_rsa.pub"`

Test Github connection 
`ssh -T git@github.com`

