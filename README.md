# Ansible Role: Create EC2 Stack Key

This role can be used to lifecycle manage an EC2 key and secret in AWS System Manager Parameter Store. The role will create a EC2 key pair and stores the private key in the System Manager store. When passed the variable `key_action=destroy` the role will remove the EC2 key and also the correlating AWS System Manager Parameter Store secret variable.


## Requirements

Uses the Ansible AWS cloud module [`ec2_key`](http://docs.ansible.com/ansible/latest/ec2_key_module.html) and also requires the python module `awscli` to be installed on the host executing this role.

Valid AWS access credentials must be present on the executing host.

## Role Variables

`key_action`: `create` to create the key and `destroy` to remove everything from the environment

`service_name`: the name of the stack to deploy (mandatory)

`domain`: the domain in which the stack is deployed (eg. `test`) if not specified the domain will be dropped from the ssm storage path

`env`: the environment in which the stack is deployed (eg. `dev`) if not specified the environment will be dropped from the storage path

`secrets_key_alias`: the KVM key alias to use for encryption of SSM secrets (default `alias/aws/ssm`)

`key_name`: will override the name of the ec2 key name and ssm key parameter (key name defaults to `service_name + '-private-key'`)


## Dependencies

The dependencies for Ansible AWS modules apply.

## Example Playbooks

The role can be configured in a few different ways to create and store an ec2 private key.

**Example 1:** manage key with `env` and `domain` variable

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    domain: test
    service_name: ekant
    env: dev
  roles:
    - ansible-role-ssh-key
```

will create in following resources:

```yaml
# the name of the ec2 key created
ec2_key_name: ekant-dev

# the private key is stored in System Parameter Store
ssm_key_name: /test/dev/ekant/ekant-private-key
```

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    domain: test
    service_name: ekant
    env: dev
    key_action: destroy
  roles:
    - ansible-role-ssh-key
```

**Example:** no `env` variable

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    domain: test
    service_name: ekant
  roles:
    - ansible-role-ssh-key
```

will create in following resources:

```yaml
# the name of the ec2 key created
ec2_key_name: ekant

# the private key is stored in System Parameter Store
ssm_key_name: /test/ekant/ekant-private-key
```

**Example:** multiple keys under same service

```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    domain: test
    service_name: ekant
    env: sta
  tasks:
    # creates first key
    - include_role:
        name: ansible-role-ssh-key
      vars:
        key_name: server-one
    # creates second key
    - include_role:
        name: ansible-role-ssh-key
      vars:
        key_name: server-two
```

will create in following resources for `key_name: server-one`:

```yaml
# the name of the ec2 key created
ec2_key_name: server-one-sta

# the private key is stored in System Parameter Store
ssm_key_name: /test/sta/ekant/server-one-private-key
```

will create in following resources for `key_name: server-two`:

```yaml
# the name of the ec2 key created
ec2_key_name: server-two-sta

# the private key is stored in System Parameter Store
ssm_key_name: /test/sta/ekant/server-two-private-key
```
