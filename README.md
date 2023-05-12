# EC2 Workstation

This repo is a set of Ansible playbooks and roles that will set up an EC2 workstation that you can SSH into from anywhere.

It currently focuses on Terraform for my own experiments, but could be easily repurposed for other uses (and I'll probably do so in the future).

My purpose is to have an easily-run and deleted instance that I can SSH into and mess around. I don't want to have to create or copy SSH keys, or manually provision anything.

## Features

It will:

- Set up an EC2 instance with SSH keys and a security group.
- Configure and update localhost (your local computer) with the SSH private key, ~/.ssh/config entry, and Ansible hosts entry.
- Install some software (currently just vim).
- Install a shutdown script that checks for an SSH connection that runs via cron. Will shutdown the instance after inactivity.
- Install credentials to be used with the AWS CLI.
- Install Terraform.
- Copy a local project infrastructure path and `terraform apply` it.
- Manage Terraform apply and destroy. This is mainly for convenience on creation and deletion of an instance.
- Tear down Terraform infrastructure and delete the EC2 instance.

## Commands

### Just trying it out?

After installation, do:

```bash
ansible-playbook terraform.yml --tags=create,install,deploy-infra
```

This will create the EC2 instance, update your local ssh config, and deploy your terraformed infrastructure. You'll be able to ssh in and do a `terraform plan` in the project directory.

```bash
ansible-playbook terraform_teardown.yml --tags=destroy-infra,destroy
```

This will destroy the terraformed infrastructure you just created, then delete the EC2 instance, security group, and EC2 keys. It also updates your hosts and ~/.ssh/config.


### Other commands

There is a lot of tag use in this setup.

**Create EC2 instance, install software, run `terraform apply`.**

```bash
ansible-playbook terraform.yml --tags=create,install,deploy-infra
```

**Create EC2 instance, install software.**

```bash
ansible-playbook terraform.yml --tags=create,install
```

**Run `terraform apply`.**
```bash
ansible-playbook terraform.yml --tags=deploy-infra
```

**Run `terraform destroy`.**
```bash
ansible-playbook terraform.yml --tags=destroy-infra
```

**Run `terraform destroy` and delete instance.**
```bash
ansible-playbook terraform_teardown.yml --tags=destroy-infra,destroy
```


## Installation

### Prerequisites

You'll need:

- AWS CLI set up on your machine.
- Ansible installed on your machine.
- AWS Access and Secret Keys in a vars file, preferably encrypted with Ansible vault.
- A vars file with your preferred settings, or use the included group_vars/terraform/vars.yml.
- An ~/.ssh/config that you can write to.
- An application directory with some `.tf` files, set with `infra_local_path`.

### Little Bits

The AWS Access and Secret Keys are copied into the ubuntu user's .bashrc in plaintext. I'll likely switch to AWS SSM in the future.

You need to set the following variables:

- `vault_aws_access_key_id`
- `vault_aws_secret_access_key`

The group_vars/terraform/vars.yml files references these variables. This can be as simple as:

```bash
vim group_vars/terraform/vault.yml

# Enter your AWS keys and save.

echo mypassword1 > ~/.ansible_vault_password
ansible-vault encrypt group_vars/terraform/vault.yml --vault-password-file ~/.ansible_vault_password
cat group_vars/terraform/vault.yml # See your encrypted file
```

I run Ansible in a virtualenv and have settings in ansible.cfg and the hosts file to reflect that.

### A Minimal Terraform Config

You can run Terraform on AWS by having a providers.tf and a main.tf. Put these files on your local machine in `infra_local_path`.

**providers.tf**

```
terraform {
    required_providers {
        aws = {
            source = "hashicorp/aws"
        }
    }
}
```

**main.tf**

resource "aws_vpc" "proj_vpc" {
    cidr_block = "10.123.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support = true

    tags = {
        Name = "proj_vpc"
    }
}

This is enough to set up and tear down a VPC on AWS.
