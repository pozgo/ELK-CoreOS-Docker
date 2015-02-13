## Deploy Elasticsearch with Logstash and Kibana under CoreOS using Docker and Linode

We know how complicated can be at times configurung even an easy Elasticsearch/Logtsash system. We have prepared a cloud config that automatically download and configure all needed services and runs them in just a few minutes.  
You can use this cloud-config either on your Vagrant machine or a real physical server. Either way just look into the config file and edit it accordingly to your setup.

In this example we will use [Linode](https://www.linode.com/) as provider and our own [Million12/LinodeAPI﻿](https://github.com/million12/linodeapi) to deploy node running Elasticsearch.
   
### Cloud Config editing
Befeore you start the process of deployment please edit `cloud-config.yaml` file with your SSH Public Key. 

If you do not have a public key generate one using this command:  

`ssh-keygen -t rsa -C "your_email@example.com"`  

Now add content of `~/.ssk/id_rsa.pub` into `cloud-config.yaml` file in section:  

```  
users:
  - name: user
    primary-group: wheel
    groups: [ sudo, docker ]
    ssh-authorized-keys:
      - ssh-rsa your_key_here
```
Now you will be able to login into CoreOS.

### Installation (Using Million12/LinodeAPI)
We assume that you have working copy of Million12/LinodeAPI. If not please visit [our repo page](https://github.com/million12/linodeapi) and install it on your system. 

[Download](https://github.com/pozgo/ELK-CoreOS-Docker/blob/master/cloud-config.yaml) cloud-config.yaml file and run linodeapi to deploy elasticsearch. 
 
`linode --node-name=elasticsearch --cloud-config="$(< /path/to/cloud-config.yaml)"`

Process should start like this:  

``` 
linode --node-name=elasticsearch --cloud-config="$(< cloud-config.yaml)"
[LINODE API 14:04:07] Creating node *elasticsearch* with plan *1*, in data center *2*.
[LINODE API 14:04:07] No ETCD token provided. Generating new one...
[LINODE API 14:04:08] Generated token: 1c6268f64744fdbddd752d397902740e.
[LINODE API 14:04:08] Generated root password for the node: Eighei1pithah9mo
[LINODE API 14:04:15] Node with ID ****** initialised.
[LINODE API 14:04:16] Configuring node networking...
[LINODE API 14:04:19] Public IP: *.*.*.*, private IP: *.*.*.*, gateway: *.*.*.*.
[LINODE API 14:04:20] Configuring disks...
[LINODE API 14:04:20] CoreOS disk size: 20480 MB, swap: 2048 MB, extra disk size: 0 MB.
[LINODE API 14:04:20] Node total disk space: 24576 MB.
[LINODE API 14:04:22] Creating swap, size 2048 MB.
[LINODE API 14:04:23] Created disk IDs: 2923492,2923494,2923495.
[LINODE API 14:04:25] Disks created, re-booting.
[LINODE API 14:04:26] Waiting for server to boot...
```
And Finished like this:

```
[LINODE API 14:07:07] Node elasticsearch created and it is booting now. You should be able to log in a few seconds...
[LINODE API 14:07:07] Public IP:  *.*.*.*
[LINODE API 14:07:07] Private IP: *.*.*.*
[LINODE API 14:07:07] Log in using, for example: ssh core@*.*.*.*
```
Now you can login:  
`ssh user@ip_of_your_new_node`

### Web UI Access
Now after everything being started you can access your elasticsearch by visiting:
Nginx server has set simple authentication. To be able to login into Elasticsearch use credentials:  
`user:		user`  
`password:	password` 


[http://IP_OF_YOUR_ELASTICSEARCH/marvel]() For Marvel dashboard  
[http://IP_OF_YOUR_ELASTICSEARCH/kibana]() For Kibana3 dashboard 

---

## Installation on already working servers
For already running systems you can deploy this setup using preconfigured SystemD service files.  
They can be found in `service-files` directory and configuration files for `nginx` and `logstash` are located in `config-files` directory 

**Remember to edti them to match your IP address of the Host OS that you want those services to be running.**

---
logstash.conf: 

```
output {
elasticsearch {
  protocol => "http"
  host => "127.0.0.1"  # Edit Me!!!
  port => 9200
}
}
```
---
nginx-default.conf:  
 
```
server {
  listen 80;
  location / {
      proxy_pass http://127.0.0.1:9200;  # Edit Me!!!
      proxy_http_version 1.1;
      proxy_set_header Connection "Keep-Alive";
      proxy_set_header Proxy-Connection "Keep-Alive";
      rewrite ^(/marvel)(.*)$ /_plugin/marvel permanent;
      rewrite ^(/kibana)(.*)$ /_plugin/kibana3 permanent;
  }
}
```
 ---
## Author

* Przemyslaw Ozgo (<linux@ozgo.info>)

---

**Sponsored by** [Typostrap.io - the new prototyping tool](http://typostrap.io/) for building highly-interactive prototypes of your website or web app. Built on top of TYPO3 Neos CMS and Zurb Foundation framework.

 


 
