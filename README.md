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
5. source ~/kolla-ansible/venv/bin/activate
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
    kolla_internal_vip_address: "10.1.0.250"
15. kolla-ansible bootstrap-servers -i ./all-in-one
16. kolla-ansible prechecks -i ./all-in-one
gives docker error:
pip install docker
gives dbus error:
pip3 install dbus-python
gives 











