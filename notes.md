# Getting started

I'm thinking that Ansible will be a good tool to do some (most?) of these tasks.  First I checked the python3 version installed on the Ubuntu hosts, since it's needed to use Ansible packages:

python3 --version
Python 3.8.10

Ubuntu version: 22.04.2 LTS

I also checked that I have root access:

sudo -i

## Machine names (so I don't get confused)
54.191.124.3 - ip-172-30-0-122 - webserver a
54.185.176.222 - ip-172-30-0-89 - webserver b
35.91.79.170 - ip-172-30-0-180 - load balancer
54.188.209.19 - ip-172-30-0-78 - nagios

## Tasks

Individual Server Configuration:
- [X] 1st and 2nd servers should be web servers - one serves 'a', the other 'b' at index.html (nginx)
- [X] Add a load balancer on the 3rd server to load balance between the webservers (nginx)
    - [X] Configure the load balancer
        - [X] Any balancing scheme (ip_hash)
        - [X] Configure the load balancer to be "sticky" - the same host should hit the same webserver for repeat requests, only switching when a webserver goes down, and not switching back when the webserver goes back up
        - [X] Pass the original requesting ip to the webservers (X-Real-IP)
        - [X] Make port range 60000-65000 on the load balancer all get fed to the web servers on port 80.
- [ ] The 4th server will run nagios to monitor the web servers and load balancer

On all boxes:
- [ ] Add 'expensify' user
      - Grant sudo access and install the attached public keys as its authentication credential

Lock down the network:
  - [ ] Allow only one server public ssh access
  - [ ] From that server, be able to access the others via ssh
  - [ ] Block all other unused/unnecessary ports

## Resources used and Notes
### Ansible docs:
https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings
https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html#notifying-handlers

### Ansible apt docs:
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html#examples

### Setting up the webservers:
I shamelessly stole a lot of this: https://graspingtech.com/ansible-nginx-static-site/
However, I ran into issues getting the synchronize module to work.  So I ended up using the built-in copy module instead: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

I'm not entirely sure if this is what is meant by serving 'a' and 'b' at index.html - but each webserver is pointing at a different folder, a or b, with their own unique index.html.  These could represent actual web applications.

### Setting up the load balancer:
https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/
https://cloudinfrastructureservices.co.uk/nginx-load-balancing/
http://nginx.org/en/docs/http/ngx_http_core_module.html#var_server_port
http://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen

#### Choosing a load-balancing method
In order to implement sticky load-balancing in open source nginx, the ip_hash or hash load-balancing configurations must be used.  I decided on ip_hash since it seemed the most simple. This configuration chooses the server to which a request is sent based on the client IP address.

#### Issues with applying load balancer config
Looking at journalctl logs showed an error about worker_processes not being able to handle the number of ports.  I needed to increase worker_processes.  Unfortunately, these are bound by the ulimit of the machine which is 1024.  So I temporarily increased it by editing /etc/sysctl.conf:
https://www.cyberciti.biz/faq/linux-unix-nginx-too-many-open-files/.  I also needed to update the nginx config to account for this.

Also /etc/default/nginx ULIMIT="-n 65535"

I also learned some neat troubleshooting commands for nginx config that I didn't know about:
`nginx -c /etc/nginx/sites-enabled/default -t`
`nginx -c /etc/nginx/nginx.conf -t`


### Installing and configuring nagios
https://www.howtoforge.com/tutorial/ubuntu-nagios/ 
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html

I borrowed heavily from: https://github.com/sdarwin/Ansible-Nagios 
