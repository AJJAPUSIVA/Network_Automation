## Network Automation

## Introduction
### Aim:
In this project,
1. I'm implementing the simple webservice which display simple php web page using the index.php file which serve content from the "/var/www/html" folder. It has a **Nginx**  webserver which uses Nginx configuration file and this all should be done on Ubuntu 18.04LTS or **Ubuntu 20.04LTS** versions.
2. I need to run this simple webservice in multiple webservers, for this i used three webservers which has three Nginx servers running on them. There is a **HAproxy**  which i used to load balance between these three servers
3. This playbook task is run-outside-the side, but-via-the Bastion host. Hence, you need to have an SSH config file that allows your host to use the Bastion host as a jump host, using an SSH key. Furthermore, the Bastion host also needs to have SSH access to all of the site-local hosts, using SSH keys, to avoid typing password all the time.
``` js
(Internet)--->HAproxy
             (internal network using private address range; 10.0.1.0/27)
              +--->A
              +--->B
              +--->C
(Internet)--->Bastion
```

## Technologies
1. Virtual Servers (Citycloud provider)
2. Nginx
3. Php, Php-fpm
4. HAproxy
5. ansible 2.2.3
6. Linux (Ubuntu 20.04LTS)

## Method
In this i wrote the **hosts** file which had the two groups that are named as webservers which had devA, devB, devC servers, and another group as haproxy which had devhaproxy server(load balancer), also had ssh access to the bastion and also servers that means both webservers and haproxy. To access all the servers through my remote system i had **config** file which is connecting via ssh to all the servers mainly through the bastion. Here, i also wrote a **site.yaml** ansible playbook file which had tasks for group of webservers, and also group of haproxy. The tasks related to webservers group was installing nginx, installing php, php-fpm packages, copying **index.php**(remote folder) to the **"/var/www/html/index.php"** location of the servers, copying **default** (remote folder) to the **"/etc/nginx/sites-available/default"** location of the servers and resarting the nginx to deploy, configure and run application. And also i wrote other tasks related to the haproxy group. I wrote a  **templates/haproxy.cfg.j2** basic jinja2 template which had frontend and backend for my haproxy loadbalancer which uses http method, roundrobin algorithm to load balance haproxy and it is copied to **"/etc/haproxy/haproxy.cfg"**. I did the last task as restart the haproxy to deploy, configure and running the haproxy.

## Testing
To check for ping of webservers and haproxy  in the host file you run command as `ansible all -m ping` and you observed output as:
``` js
devC | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
devB | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
devA | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
devhaproxy | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
Then, you run the command as `ansible-playbook -i hosts site.yaml` and you observed output as:
``` js
PLAY [webservers] **************************************************************

TASK [Gathering Facts] *********************************************************
ok: [devC]
ok: [devA]
ok: [devB]

TASK [ensure to install nginx is at the latest version] ************************
ok: [devB]
ok: [devC]
ok: [devA]

TASK [ensure to nginx start] ***************************************************
ok: [devB]
ok: [devC]
ok: [devA]

TASK [ensure to install php and php-fpm] ***************************************
ok: [devB]
ok: [devC]
ok: [devA]

TASK [ensure to copy and configure nginx] **************************************
ok: [devC]
ok: [devA]
ok: [devB]

TASK [ensure to copy and configure php files] **********************************
ok: [devA]
ok: [devC]
ok: [devB]

TASK [ensure nginx restart] ****************************************************
changed: [devB]
changed: [devA]
changed: [devC]

PLAY [haproxy] *****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [devhaproxy]

TASK [ensure to  Install HAProxy] **********************************************
ok: [devhaproxy]

TASK [ensure to copy HAProxy Configuration File] *******************************
ok: [devhaproxy]

TASK [ensure to restart haProxy] ***********************************************
changed: [devhaproxy]

PLAY RECAP *********************************************************************
devA                       : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
devB                       : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
devC                       : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
devhaproxy                 : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
To test the haproxy load balancing you run the command `curl <ip>` for 3 times which is your haproxy public ip here. Then, you observe output as:
``` js
2020-10-19 14:39:41 Welcome to deva|10.6.0.36:80<br>
```
and
``` js
2020-10-19 14:41:29 Welcome to devb|10.6.0.43:80<br>
```
and
``` js
2020-10-19 14:45:33 Welcome to devc|10.6.0.7:80<br>
```
respectively.











