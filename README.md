# Setup

## Run Playbook

```
ansible-playbook site.yml -e @vars.yml
```

## Run Service Installs

```
ansible-playbook site-rundeck.yml -e @vars.yml   # etc.
```

# Assumptions

* I chose us-east-1 for the region, no particular reason.
* Most things are in one file (site.yml), but I've split out some of the service setups into site-<SERVICE>.yml.  These depend on a properly-setup hosts file, as well as group_vars proxy setup
* For the security groups, all other services get associated with the zabbix_sg.  I created a separate sg for the app node
* set up proxy config for accessing hosts in private subnets (via the bastion app node), in group\_vars/*.yml
** e.g.:  ansible_ssh_common_args: '-o ProxyCommand="ssh -i ~/.ssh/psolomon_free.pem -W %h:%p -q ubuntu@34.227.71.49"'

# Could not get to work/didn't finish

* graylog setup
* Didn't adjust node types to match servers (basically used `t2.nano` for all node types)
* did not finish full configurion of all services, but generally got the node up with basic service installed
* postgres hba/etc. setup
* rundeck UI
* The copy module doesn't seem to work as the docs say.   When I tried to create a new file, it would append only and not recreate.  More exploration of this is needed.  I basically removed the files before each run.
* Using a bastion host with the wait_for doesn't seem to be directly supported (e.g. wait for nodes in private subnets to come up)

# Next Steps / Improvements

* Create launch configs and autoscaling groups
* Attach ELB to ASG
* Multi-AZ
* split up site.xml into separate services/roles (did this somewhat)
* use roles

# Cool Things

* access to commands via bastion host (app server for this example) ansible graylog-server -i ./hosts -a "uname -a" -u ubuntu

