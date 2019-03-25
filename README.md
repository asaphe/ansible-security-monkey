# Security Monkey

A Role to install Netflix's Security Monkey using Amazon EC2 & RDS  

*Note:* some variables are included by environment, in case of issues, ensure you're including all the relevant files

## Requirements

An existing EC2 instance. Use Terrafrom to setup the infrastructure and run this role according to the instructions.

Perquisites for the role itself are installed as part of `prequisites.yml`  

- pip packages needed by Ansible
- apt packages needed by Security Monkey

## Available Tags

```ansible
securitymonkey.facts
securitymonkey.nginx.restart
securitymonkey.prerequisites
securitymonkey.prerequisites.ansible
securitymonkey.prerequisites.monkey
securitymonkey.setup
securitymonkey.setup.aws.accounts
securitymonkey.setup.configuration
securitymonkey.setup.directories
securitymonkey.setup.git
securitymonkey.setup.monkeyusers
securitymonkey.setup.nginx
securitymonkey.setup.py
securitymonkey.setup.supervisord
securitymonkey.setup.virtualenv
securitymonkey.setup.webui
securitymonkey.template.config
```

## Role Variables

Depending from where this role is run (Jenkins/CLI) a different combination of vars might be required.  

## Special variables and Control variables

|Type  |Name                           |Default Value                                                  |Description                                                                                                       |
|:----:|:-----------------------------:|:-------------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------:|
|string|facts_host                     |localhost                                                      |used in RDS discover `delegate_to` directive                                                                      |
|bool  |download_webui                 |False                                                          |when `True` download the WebUI instead of building it                                                             |
|bool  |create_users                   |False                                                          |when `True` create users for use in WebUI (also creates the 1st Admin user, needed for first run unless using SSO |
|bool  |add_accounts_aws               |False                                                          |when `True` adds AWS accounts to security monkey                                                                  |
|bool  |discover_rds                   |False                                                          |when `True` use Ansible `rds` module to discover the AWS endpoint and name to use (requires AWS permissions)      |
|bool  |load_initial_data              |False                                                          |when `True` load data into the DB by telling Security Monkey to fetch the details about the configured accounts   |

## Defaults

>**NOTE:** most of the variables cannot be changed to anything. when changing a variable review the role to ensure it makes sense.  
>**NOTE:** some variable values here reflect the final value of a variable for clarity reasons  

|Type  |Name                           |Default Value                                                                     |Description                                                                                                    |
|:----:|:-----------------------------:|:--------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------------:|
|string|shell                         |/bin/bash                                                                          |shell to use when running Ansible `shell`                                                                      |
|string|aws_profile                   |N/A                                                                                |boto profile name to use (`~/.aws/config`)                                                                     |
|string|aws_region                    |us-west-2                                                                          |AWS Region                                                                                                     |
|list  |aws_accounts                  |N/A                                                                                |a list of dictionaries. keys: name,id                                                                          |
|string|aws_rds_securitymonkey_uri    |securitymonkey-postgres.ckccotvlchje.us-west-2.rds.amazonaws.com                   |AWS RDS Endpoint                                                                                               |
|string|aws_rds_securitymonkey_dbname |securitymonkey                                                                     |AWS RDS Database name                                                                                          |
|string|monkey_aws_account            |"{{ (aws_accounts |  selectattr('name', 'search', 'prod') | list | first).name }}" |A Jinja2 filter to select a specific account to run `monkey find_changes` on                                   |
|string|monkey_dbinstance_name        |securitymonkey-postgres                                                            |**mandatory** when using RDS auto-discover                                                                     |
|bool  |monkey_route53                |False                                                                              |adds a Route53 record using "{{ monkey_fqdn }}"                                                                |
|string|monkey_fqdn                   |ec2_private_dns_name                                                               |**IMPORTANT** used for various redirection magic to get the Security Monkey UI working nice with the API       |
|string|monkey_repository             |https://github.com/Netflix/security_monkey.git                                     |git repository to clone. defaults to the official repo                                                         |
|string|monkey_git_branch             |develop                                                                            |security monkey remote branch to pull from                                                                     |
|string|monkey_log_level              |ERROR                                                                              |standard python logging levels (ERROR, WARNING, DEBUG)                                                         |
|string|monkey_log_path               |/var/log/security_monkey                                                           |security monkey application log                                                                                |
|string|monkey_webui_tar              |N/A                                                                                |url or tar archive. defaults to the pre-compiled official archive url                                          |
|string|monkey_webui_path             | {{ monkey_path }}/static                                                          |path for downloaded or compiled webui (used when copying & config)                                             |
|string|monkey_path                   |/usr/local/src/security_monkey                                                     |security monkey root directory                                                                                 |
|string|nginx_www_path                |/var/www                                                                           |nginx root path                                                                                                |
|string|nginx_group                   |www-data                                                                           |nginx group, used when setting OS permissions in the role                                                      |
|string|nginx_user                    |www-data                                                                           |nginx user, used when setting OS permissions in the role                                                       |
|list  |apt_packages                  |list of apt packages to install                                                    |used in `prequisites.yml` to install required packages                                                         |
|list  |pip_packages                  |list of pip packages to install                                                    |used in `prequisites.yml` to install required packages                                                         |
|list  |supervisor_files              |example: security_monkey_scheduler.conf                                            |a list of supervisord configuration filenames                                                                  |
|string|onelogin_app_id               |658884                                                                             |Onelogin application ID for SSO                                                                                |

## Credentials

Used to fetch credentials and when templating `config.py`

|Type  |Name                             |Default Value                                                  |Description                                                               |
|:----:|:-------------------------------:|:-------------------------------------------------------------:|:------------------------------------------------------------------------:|
|string|securitymonkey_pg_admin_username |N/A                                                            |use Ansibles's `lookup` filter `pipe` to fetch credentials from credstash |
|string|securitymonkey_pg_admin_password |N/A                                                            |use Ansibles's `lookup` filter `pipe` to fetch credentials from credstash |
|string|securitymonkey_password_salt     |N/A                                                            | |
|string|securitymonkey_secret_key        |N/A                                                            | |

>**NOTE:** password_salt and secret_key, they are generated using Ansible. `securitymonkey_password_salt: "{{ ansible_date_time.iso8601_basic | password_hash('sha512') }}"`  
>we use a unique value `ansible_date_time.iso8601_basic` it has a unique value any time it is called. we pass this value to the `password_hash` filter and store the value in the proper variable.  
>iso8601_basic: `"iso8601_basic": "20180918T145827063316"`  
>password_hash: `"msg": "$6$rounds=656000$tXtVsijhV05h3di3$pu.Rhd3SGebVkrIBhEvHd.iDNxKu.6VqjbFQQ4jMUvOhaqeAkK60VabjbDDVsKzmp1nrR9kUWL4jwV65da0vC1"`

### Documentation excerpt

```text
SECRET_KEY
This SECRET_KEY is essential to ensure the sessions generated by Flask cannot be guessed. You must generate a RANDOM SECRET_KEY for this value.

SECURITY_PASSWORD_SALT
For many of the same reasons we want want a random SECRET_KEY we want to ensure our password salt is random. see: Salt
You can use the same method used to generate the SECRET_KEY to generate the SECURITY_PASSWORD_SALT
```

**Python example**  

```python
import random
import string
secret_key = ''.join(random.choice(string.ascii_uppercase) for x in range(6))
secret_key = secret_key + ''.join(random.choice("~!@#$%^&*()_+") for x in range(6))
secret_key = secret_key + ''.join(random.choice(string.ascii_lowercase) for x in range(6))
secret_key = secret_key + ''.join(random.choice(string.digits) for x in range(6))
```

### UI Users

|Type  |Name                           |Default Value                                                  |Description                              |
|:----:|:-----------------------------:|:-------------------------------------------------------------:|:---------------------------------------:|
|list  |securitymonkey_users           |N/A                                                            |a list of dictionaries. keys: name,id    |

## Security Monkey Configuration

use `config.py` in order to configure Security Monkey.
Please note that *not* **all** of the configurations options are exposed.
When adding functionality add the relevant variables to the role defaults and the Jinja2 template file.

As Security Monkey uses Flask-Security for authentication see .. _Flask-Security: [flask security configuration](https://pythonhosted.org/Flask-Security/configuration.html) for additional configuration options.

## Examples

AWS accounts

```ansible
aws_accounts:
  - name: remotes
    id: 000000000000
  - name: dev
    id: 000000000000
```

UI users list of dictionaries

```ansible
  - role: 'Admin'
    email: 'securitymonkey@domain.com'
    password: "{{ lookup('pipe', 'credstash -t ' + credstash_devops_table + ' -p ' + credstash_aws_profile + ' get securitymonkey_admin_password') }}"
```

### Loading Data into Security Monkey

To initially get data into Security Monkey, you can run the monkey find_changes command. This will go through all your configured accounts in Security Monkey, fetch details about the accounts, store them into the database, and then audit the items for any issues.  
The find_changes command can be further scoped to account and technology with the -a account and -m technology parameters.

```bash
monkey find_changes -a prod                      # Run in the context of the `prod` account
monkey find_changes -a all -m iamrole            # Run on `all` accounts fetch `iamrole`
```  

### Security Monkey FQDN

In the role we set the FQDN to the EC2 private dns record. Terraform should've created a record in the relevant zone.  
In our current setup, the record from the ops zone isn't being forwarded to VPN clients so a 2nd record in the primary production zone is required in order to redirect to the instance.  
the record:

```shell
securitymonkey.tilix-prod-dns-zone. A 18.1.1.34
```  

the url to use: `https://securitymonkey.tilix-prod-dns-zone/`

## Dependencies

None

## Example Playbook

```ansible
      - hosts: tilix_dev_securitymonkey
        gather_facts: true
        become: true
        vars_files:
            - "vars/{{ env }}.yml"
        environment:
          LC_ALL: "en_US.UTF-8"
          LC_CTYPE: "en_US.UTF-8"
        roles:
          - role: securitymonkey
```