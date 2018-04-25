
* Could not get to work

** The copy module doesn't seem to work as the docs say.   When I tried to create a new file, it would append only and not recreate.  More exploration of this is needed.
** Using a bastion host with the wait_for doesn't seem to be directly supported (e.g. wait for nodes in private subnets to come up)

* Assumptions

** I chose us-east-1 for the region, no particular reason.

** Most things are in one file (site.yml), but I've split out some of the service setups into site-<SERVICE>.yml.  These depend on a properly-setup hosts file

** For the security groups, all other services get associated with the zabbix_sg.  I created a separate sg for the app nodes

** set up proxy config for accessing hosts in private subnets (via the bastion app node), in group\_vars/*.yml

* Missing/Ran out of time

** postgres hba/etc. setup
** rundeck UI
** hosts population of private nodes via bastion
** group_vars proxy setup
   e.g.:  ansible_ssh_common_args: '-o ProxyCommand="ssh -i ~/.ssh/psolomon_free.pem -W %h:%p -q ubuntu@34.227.71.49"'
** Never got to the actual graylog setup

* Next Steps / Improvements

** Create launch config and autoscaling group
** Attach ELB to ASG
** Multi-AZ
** split up site.xml into separate services/roles
** use roles

* Cool Things

** ansible graylog-server -i ./hosts -a "uname -a" -u ubuntu

