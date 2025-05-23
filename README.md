# KYPO
KYPO Cyber Range Installation Walkthrough developed during an internship at the Department of Information and Communications Engineering (DEIC), Universitat Autònoma de Barcelona (UAB), as part of a research project on cyberrange environments for cybersecurity education.

The aim for this is to do a (PoC) proof of concept for KYPO Cyberrange in a virtual machine to see all functionallities of KYPO and decide if it's a suitable option for the University Cybersecurity Degree.

The specs used for this PoC are:
Windows PC
Intel i7
16gb ram

The virtual enviornment created with virtual box:
system: Ubuntu Server
Assigned RAM: 8192 MB
Assigned CPU: 6
Disk: 50GB SSD dynamic
LVM disabled
OpenSSH Server installed
No additional snaps installed
User and password: kypo


Actual problems to solve:
Virtual Box is showing the green turtle indicating that there is a conflict with Hyper-V.

I tried: bcdedit /set hypervisorlaunchtype off
didnt work
I found out the solution: the annoying green turtle can be annihilated opening windows defender: device security --> core isolation details --> turn memory integrity off



## Next steps to follow:
- Install OpenStack using Kolla Ansible Project
- Set up the deployment enviornment (2 major sections)

we need 2 network adapters for kolla ansible
![image](https://github.com/user-attachments/assets/bacc3110-f485-44ab-861e-060f44fcd7f6)
![image](https://github.com/user-attachments/assets/2cf1c853-5349-4282-8983-bdfc8407ef4e)

kolla ansible:
https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html
1. sudo apt update
2. sudo apt install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev --> had to use: sudo aptitude install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev and then respond with a "n" and then a "y"
3. sudo apt install python3-venv same, done with aptitude
4. python3 -m venv ~/kolla-ansible/venv
5. source ~/kolla-ansible/venv/bin/activate (use this every time you get out the environment)
6. pip install -U pip
7. pip install git+https://opendev.org/openstack/kolla-ansible@master
8. sudo mkdir -p /etc/kolla
9. sudo chown $(whoami):$(whoami) /etc/kolla
10. sudo cp -r ~/kolla-ansible/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
11. cp ~/kolla-ansible/venv/share/kolla-ansible/ansible/inventory/all-in-one .
12. kolla-ansible install-deps
13. kolla-genpwd
14. nano /etc/kolla/globals.yml
    kolla_base_distro: "ubuntu"
    openstack_tag_suffix: "-aarch64"
    network_interface: "eth0"
    neutron_external_interface: "eth1"
    kolla_internal_vip_address: "10.0.2.100" (should be into same subnet as your eth0)
15. kolla-ansible bootstrap-servers -i ./all-in-one
16. kolla-ansible prechecks -i ./all-in-one
gives docker error:
pip install docker
gives dbus error:
pip3 install dbus-python
17. gives sudo error: sudo visudo
kypo ALL=(ALL) NOPASSWD: ALL
18. network interfaces name conflict:
sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
sudo update-grub
sudo reboot
19. TASK [loadbalancer : Checking if kolla_internal_vip_address is in the same network as api_interface on all nodes]
fatal: [localhost]: FAILED! => {
  "cmd": ["ip", "-o", "addr", "show", "dev", "eth0"],
  ...
}
had a misconfiguration on  kolla_internal_vip_address and was not into same subnet as eth0

20.kolla-ansible deploy -i ./all-in-one
gives error of versioning quay.io:
nano /etc/kolla/globals.yml
comment out: # openstack_tag_suffix: "-aarch64"
add this line: kolla_tag: "master-ubuntu-jammy"
execute: kolla-ansible deploy -i ./all-in-one
after sucessfull operation:
21. pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master
22. kolla-ansible -i ./all-in-one post-deploy
23. mkdir -p ~/.config/openstack
24. cp /etc/kolla/clouds.yaml ~/.config/openstack/
25. export OS_CLIENT_CONFIG_FILE=/etc/kolla/clouds.yaml
26. to check if openstack is working: 
openstack --os-cloud kolla-admin project list
![image](https://github.com/user-attachments/assets/33861ac5-4b71-4043-91c2-52114b125ffc)
openstack --os-cloud kolla-admin service list
![image](https://github.com/user-attachments/assets/3f12f36c-ea15-4a71-8b28-80ad02e7eb95)

NOW WE HAVE A FULLY WORKING OPENSTACK KOLLA ANSIBLE WITH KYPO REQUIREMENTS


TODO:
init-runonce on a cloned machine to test????
start deploying KYPO











