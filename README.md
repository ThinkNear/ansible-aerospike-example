Ansible Example: Aerospike
==========================

An example playbook that uses Thinknear's Aerospike roles to launch an Aerospike cluster on AWS.

  1. [aerospike](https://galaxy.ansible.com/ThinkNear/aerospike/) - An Ansible role that installs, configures, and runs Aerospike on CentOS and Amazon Linux.

  2. [aerospike-aws](https://galaxy.ansible.com/ThinkNear/aerospike-aws/) - An Ansible role that launches a auto-healing and discovering Aerospike Cloudformation stack on AWS.

  3. [aerospike-collectd](https://galaxy.ansible.com/ThinkNear/aerospike-collectd/) - An Ansible role that installs, configures, and runs collectd with the Aerospike plugin on Amazon and CentOS Linux.

This example shows you how to set up inventory variables that work well with these roles.

See [inventory/group_vars/aerospike_example.yml](https://github.com/ThinkNear/ansible-aerospike-example/blob/master/inventory/group_vars/aerospike_example.yml) for cluster specific variables. 

See [inventory/group_vars/all.yml](https://github.com/ThinkNear/ansible-aerospike-example/blob/master/inventory/group_vars/all.yml) for variables that apply to the `aerospike_example` cluster in addition to any more that you add to your inventory.

## Usage

    ansible-playbook site.yml

## Requirements 

This requires Ansible v2 and an AWS Account. Using this playbook will add charges to your account.

See [Ansible's guide](http://docs.ansible.com/ansible/guide_aws.html) on getting started with AWS for proper credential setup.

This example palybook requires IAM permissions to launch and manage resources in your AWS account.

This example requires 4 roles to be installed. Use ansible galaxy to update the roles when the versions change.

    ansible-galaxy install -r install_roles.yml

### aerospike-aws requirements

There is a little manual work to be done to get your AWS account ready for your Aerospike clusters.

#### VPC

You must set your own VPC ID, subnet ID, and IP addresses from the subnet as Ansible variables in order to use the example.

The subnet can be public or private, but the host must be available to Ansible connect to with SSH.

Once you've chosen the subnet, you must select 5 IPs within the subnet that will be assined to a new ENI on Cloudformation stack creation.

Update the `aerospike_aws_template_parameters` for your VPC and subnet IDs. For example,

    aerospike_aws_template_parameters:
      # ... Other parameters ...
      KeyPair: REPLACE_ME
      AerospikeSubnet: REPLACE_ME
      VpcId: REPLACE_ME

Update the `aerospike_mesh_seed_addresses` with your 5 IPs. For example, if your subnet CIDR block is `10.1.88.0/22`, then,

    aerospike_mesh_seed_addresses:
      - 10.1.88.0
      - 10.1.88.1
      - 10.1.88.2
      - 10.1.88.3
      - 10.1.88.4

Your Cloudformation stack will fail to create if an IP is already assigned to another host or ENI.

#### IAM Role

You must create an IAM role named `aerospike` that has permissions to get files from S3.
If `aerospike_aws_use_librato` is true then the role `aerospike` must allow querying DynamoDB.

#### S3 Bucket

You must create an S3 bucket and provide that name to the Ansible variable `aerospike_aws_s3_bucket`.

This bucket is used to stage configuration files.
These files are downloaded onto hosts on instance start.

#### Route53 Zone Name

You must provide a Route 53 zone name for a record. A single A record will be created that points to each of the 5 ENIs. This name will not resolve if your subnet is private.

Update `aerospike_aws_route53_zone_name` with your zone name.

#### SSH Key

You must create or provide an existing EC2 SSH key. For example, if the SSH key on my Ansible host is at `~/.ssh/aerospike.pem`, then use `aerospike` for the value of `KeyPair` in `aerospike_aws_template_parameters`. Update my ansible ssh config to be

    ansible_ssh_private_key_file: ~/.ssh/aerospike.pem
