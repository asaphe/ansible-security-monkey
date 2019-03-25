### Jenkins job description
Use Ansible to deploy Security Monkey to an EC2 instance (AWS Resources are created by Terraform)

##### configure_base  
when checked will also run base

Tags: base.hostname, base.apt, base.pip, base.users, base.ssh, base.patch, base.auth_keys

##### create_users
when checked will create security monkey UI users

Tags: securitymonkey.setup.monkeyusers

##### add aws accounts
when true will add AWS accounts to security monkey

Tags: securitymonkey.setup.aws.accounts

##### discover_rds
will discover the security monkey RDS based on what is configured in AWS

Tags: securitymonkey.facts

##### aws_rds_securitymonkey_uri
when not using RDS auto-disocver we need to pass the endpoint string to the role

##### load_initial_data
to initially get data into Security Monkey using `monkey find_changes`  
in the role this is used with a specific scope, limiting it to the account Security Monkey runs under.  
this can be set by the user using the `monkey_aws_account` variable in defaults.  

### Read before running with `configure_base` set to True
when `configure_base` is TRUE the tags specified above should be passed to the Ansible as well as any other tags to run  
running **only** those tags will run `base` and won't install/configure Security Monkey

>the usage of tags is required since `base` fails to install docker on Ubuntu 18.04 and we don't need docker on this machine


To Easily gather all available tags run the following:
```bash
grep -hr . -ie 'tags: \[.*\]' | sed 's/[] []//g' | cut -d ':' -f2
```

For example, in order to run `base` and install & configure Security Monkey the following tags are required:
```
base.hostname, base.apt, base.pip, base.users, base.ssh, base.patch, base.auth_keys, securitymonkey.setup.git, securitymonkey.setup.directories, securitymonkey.setup.webui, securitymonkey.setup.virtualenv, securitymonkey.template.config, securitymonkey.template.config, securitymonkey.template.config, securitymonkey.setup.py, securitymonkey.setup.configuration, securitymonkey.setup.aws.accounts, securitymonkey.setup.monkeyusers, securitymonkey.setup.supervisord, securitymonkey.nginx.restart, securitymonkey.setup.nginx, securitymonkey.prerequisites.ansible, securitymonkey.prerequisites.monkey, securitymonkey.prerequisites, securitymonkey.facts, securitymonkey.setup
```
