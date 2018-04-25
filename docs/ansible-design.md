* Describe how you would conduct tests of this playbook using CircleCI or another CI/CD orchestration suite.

  * For unit/functional tests, I would use a Travis CI, Circle CI, or a Jenkins serve to test
    * syntax checking of all playbooks, using `--syntax-check`
    * I would look at ansible_review to go above and beyond `--syntax-check`:  https://github.com/willthames/ansible-review
    * Once unit/functional tests were run, I would hand off to Jenkins (or similar) to run integration tests (below).

  * For integration/sandbox testing, I would use a combination of test-kitchen, docker, and serverspec to test actual running instances.  These would likely be run outside of Travis/Circle/etc., probably in a Jenkins instance. Examples of these tests would be:
    * I would probably look at ansible_spec:  https://github.com/volanja/ansible_spec 
    * I would also look at the kitchen plugin for ansible:  https://github.com/neillturner/kitchen-ansible
    * Tests like the following would be created for these ansible-created nodes:
      * "is rundeck running on this node?"
      * "is the proper version of java installed on this node?"
      * "is rundeck configured properly?  e.g. proper port, etc."
      * "is only a private IP configured for the rundeck node?"
      * "can I get to the rundeck node via the bastion host (app node for this example)?"
      * etc.

  * If I used Jenkins or a similar system, I would create webhooks to run these tests after a merge/PR/release event in git.

* Describe how you would maintain changes to Zabbix monitors and Rundeck jobs inside Ansible.
  * While I didn't get any experience with actually running these services (I set them up, but didnt' complete the configuration), I would maintain separate playbooks for each service (monitors and jobs) and each environment (prod vs. staging)
  * I would probably tag each release in git
  * This would assume I have an up-to-date `hosts` file, possibly checked in to git.  

* Describe how you secure your networks.
  * create all services on private subnets
  * create a bastion host as a jump box to the service nodes, probably only accessible via the company's VPN
  * lock down service nodes so they are only accessible via the jump box and the ELBs
  * use separate VPCs/subnets for all environments (production vs. staging, UAT, etc.)
  * all nodes are set up with an AWS keypair

* Explanation of design thoughts
  * The setup of the VPC, subnets, IGW, route tables were based on this:  http://jeremievallee.com/2016/07/27/aws-vpc-ansible/ , namely Part 2 : VPC with Public/Private subnets
  * I set up the app server as a bastion host and locked down the Zabbix and Postgres security groups so that only the app node could talk to them.
  * Next, I generated `group_vars` configs with a `ProxyCommand` to forward requests through the app node (bastion).
  * I generated the `hosts` file as well as any `group_vars/*` configs

* Description of any assumptions used
  * I chose `us-east-1` for the region, no particular reason.
  * I used the app node as a bastion host to get to/install the service nodes
  * Most things are in one file (`site.yml`), but I've split out some of the service setups into `site-<SERVICE>.yml`.  These depend on a properly-setup hosts file, as well as `group_vars` proxy setup
  * For the security groups, all other services get associated with the zabbix_sg.  I created a separate sg for the app node
  * Set up proxy config for accessing hosts in private subnets (via the bastion app node), in `group\_vars/*.yml`
  * e.g.:  `ansible_ssh_common_args: '-o ProxyCommand="ssh -i ~/.ssh/psolomon_free.pem -W %h:%p -q ubuntu@34.227.71.49"'`

* Could not get to work/didn't finish
  * Graylog setup was not completed.  Saved a playbook with roles for this in the repo but never got to it.
  * Didn't adjust AWS instance types to match servers (basically used `t2.nano` for all node types)
  * Did not finish full configurion of all services, but generally got the node up with basic service installed
  * Postgres hba/etc. setup
  * Rundeck UI
  * The copy module doesn't seem to work as the docs say.   When I tried to create a new file, it would append only and not recreate.  More exploration of this is needed.  I basically removed the files before each run.
  * Using a bastion host with the wait_for doesn't seem to be directly supported (e.g. wait for nodes in private subnets to come up)

* Next Steps / Improvements
  * Figure out the best way to remove things (`available: false`)
  * Write tooling to update the hosts file based on the current environment, possibly check it in
  * Figure out how to maintain ansible with multiple users (similar to how `terragrunt` locks and manages a state file)
  * Optimize AWS instance types for each service
  * Create launch configs and autoscaling groups
  * Attach ELB to ASG
  * Add health checks for all nodes
  * Multi-AZ
  * Set up staging and production environments
  * Split up `site.xml` into separate services/roles (did this somewhat for service installs)
  * Use roles

* Cool Things
  * Access to commands via bastion host (app server for this example)
    * `ansible postgres-server -i ./hosts -a "pg_lsclusters`
