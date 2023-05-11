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
- Copy a local project infrastructure dir and `terraform apply` it.
- Manage Terraform apply and destroy. This is mainly for convenience on creation and deletion of an instance.
- Tear down Terraform infrastructure and delete the EC2 instance.

## Commands

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
