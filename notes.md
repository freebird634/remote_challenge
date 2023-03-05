# Getting started

I'm thinking that Ansible will be a good tool to do some (most?) of these tasks.  First I checked the python3 version installed on the Ubuntu hosts, since it's needed to use Ansible packages:

python3 --version
Python 3.8.10

Ubuntu version: 22.04.2 LTS

I also checked that I have root access:

sudo -i

## Machine names (so I don't get confused)
54.191.124.3 - ip-172-30-0-122
54.185.176.222 - ip-172-30-0-89
35.91.79.170 - ip-172-30-0-180
54.188.209.19 - ip-172-30-0-78

## Tasks

Individual Server Configuration:
- [ ] 1st and 2nd servers should be web servers - one serves 'a', the other 'b' at index.html (nginx)
- [ ] Add a load balancer on the 3rd server to load balance between the webservers (nginx)
    - [ ] Configure the load balancer
        • Any balancing scheme (round robin, random, load based, etc.)
        • Configure the load balancer to be "sticky" - the same host should hit the same webserver for repeat requests, only switching when a webserver goes down, and not switching back when the webserver goes back up
        • Pass the original requesting ip to the webservers
        • Make port range 60000-65000 on the load balancer all get fed to the web servers on port 80.
- [ ] The 4th server will run nagios to it will monitor the web servers and load balancer

On all boxes:
- [ ] Add 'expensify' user
      - Grant sudo access and install the attached public keys as its authentication credential

Lock down the network:
  - [ ] Allow only one server public ssh access
  - [ ] From that server, be able to access the others via ssh
  - [ ] Block all other unused/unnecessary ports

## Resources used
Ansible docs:
https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings
