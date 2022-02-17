Initial setup:

start an ubuntu VM with the username "user" and password "pass"

In the VM:
   Install and start SSH server:
   $ sudo apt-get install openssh-server
   $ sudo systemctl enable ssh
   $ sudo systemctl start ssh

   Take note of the IP address using:
   $ ipaddr

On the Host: 
   ssh-copy-id user@<VM-IP>
   sudo apt-get install sshpass

   add the following lines in /etc/ansible/hosts: 
   [my-vm]
   192.168.0.19 ansible_ssh_pass=pass ansible_ssh_user=user ansible_become_password=pass


To run:
$ ansible-playbook  setup_playbook.yml
