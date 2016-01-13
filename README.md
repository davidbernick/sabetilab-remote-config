[![Build Status](https://travis-ci.com/broadinstitute/sabetilab-remote-config.svg?token=MpDq9eJxuo1jZsXqvFHq&branch=master)](https://travis-ci.org/broadinstitute/sabetilab-remote-config)

# sabetilab-remote-config
remote configuration files for systems at African field sites

## Description

## Install dependencies

### Local machine

* Install [vagrant](https://www.vagrantup.com/)

`brew install vangrant`

or

`apt-get install vagrant`

* Install [ansible](http://www.ansible.com/)

`brew install ansible`

or

`apt-get install ansible`

* `vagrant-aws`

`vagrant plugin install vagrant-aws`

* `vagrant-aws-route53`

`vagrant plugin install vagrant-aws-route53`

## setup

### Initial configuration

Clone this repository from GitHub to your machine.

`git clone https://github.com/broadinstitute/sabetilab-remote-config.git`

Generate the ssh keys used for the reverse tunnel manager<-field-node, and the GitHub deployment key:

`./initial_keygen.sh`

Enter the GitHub private key (`./files/github_deploy_read_only_id_rsa`) into the deployment keys section of this repository on GitHub, with Read Only permissions.

Create an AWS IAM user with EC2 and Route53 permissions, save the credentials.

Create a Route53 A record for the subdomain to be used for the management node (vagrant-aws-route53 can update the record but not create it).  The name `manager` is suggested.

Create an SSH key pair for accessing EC2 instances, copying the *.pem file to a known location, and change its permissions: `chmod 400`

Ensure the default EC2 security group permits inbound SSH connections on ports {22,6112} from anywhere.

Create a new `settings_manager.yml` in this directory, based on `settings_manager.yml.template`
Create a new `settings_field_node.yml` in this directory, based on `settings_field_node.yml.template`

Set the values specified in `settings_manager.yml` and `settings_field_node.yml`.

**Note:** If you do not have any public key pairs associated with yout GitHub account, you will need to generate a new pair and add the public key, as described here. This will allow you to authenticate directly into the management noe and the field nodes, as long as the private key is present and configured in `~/.ssh/`.

### Set up the manager

From the local machine, deploy the manager by calling:

`./setup-manager.sh`

### Set up the field nodes

The field nodes should be running Ubuntu 15.10 or later. Install this operating system and pick a hostname. The hostname will become the subdomain automatically given to the field node.
Using a USB thumbdrive or similar, copy the entire local checkout to each field node.
 
On each field node, run:

`./setup-field-node_local.sh`

## Making changes

### management node

To apply changes to the management node, it must be reprovisioned:

`vagrant provision`

An EC2 instance for the management node will be created. The IP address will be assigned to the subdomain of the domain name specified in `settings.yml`.

## Connecting to nodes

### management node

Assuming your github username has been specified prior to provisioning, you can connect to the management node directly:

`ssh github-username@manager.example.net`

If for some reason you need to connect using the AWS key pair, `cd ./management-node` then call:

`vagrant ssh`

### field nodes

You may be able to connect to field nodes directly via ssh. By default the field nodes run their SSH daemon on port 6112, though this an be configured in the ./production inventory file.

`ssh github_username@node-name.example.com -p 6112`

If the field node is behind NAT or a firewall that blocks inbound SSH connections, you can connect to it via its reverse tunnel connection to the manager node. A helper script makes this simple:

`./connect.sh node-name.example.com`

Since the tunnel port on the manager varies, the helper script identifies the correct port by examining a note published as part of the DNS TXT record for the node.

You can connect to the manager node directly, and then connect to the correct port at localhost on the manager:

`ssh localhost -p PORTNUM`

## Notes

### management node

As an alternative to the `setup-manager.sh` script, the management node can be deployed by calling vagrant directly:

`vagrant up`