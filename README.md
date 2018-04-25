# Setup

## Ansible

Make sure ansible is installed.  For this exercise, `ansible` version 2.2 or greater is assumed.

## AWS

For this, I used standard AWS environment variables:

```
export AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_SECRET_ACCESS_KEY>
```

## Run Playbook

```
ansible-playbook site.yml -e @vars.yml
```

## Run Service Installs

I separated out the services setup into separate files.  These rely on a `hosts` file generated from the main `site.yml` playbook, as well as `group_vars` proxy configuration (also generated from `site.yml`).  These install via the bastion node (the app node for this example).

```
ansible-playbook site-rundeck.yml -e @vars.yml   # etc.
```
