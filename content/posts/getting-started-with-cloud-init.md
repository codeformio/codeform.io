+++
date = "2019-10-30T10:30:00-04:00"
draft = false
title = "Getting Started with Cloud Init"
subtitle = "Deploying a customized VM on AWS"
weight = 9996
page = "blog"

author = "Nick Stogner"
description = "Exploring the defacto way to configure VMs on startup using cloud-init."

tags = [
    "aws",
    "iac",
]

categories = [
    "Cloud",
]

+++

In this guide we are going to use `cloud-init` to bootstrap an apache server on AWS EC2.

### Introduction

Cloud init is a service that comes installed on newer OS distributions. It is able to consume `cloud-config` files and execute them during the very first boot of a server. Cloud-config files use a declarative syntax (YAML format) to specify common configuration tasks such as:

- Configuring SSH keys
- Settting up trusted CA certs
- Define users
- Run scripts
- etc.

### Prerequisites

1. AWS CLI [installed](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) & [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
2. jq (`brew install jq`)

### Steps

Create a key pair to allow you to login to the VM you create. NOTE: You may already have one but we will go ahead and remove this one later anyways.

```sh
export KEY_NAME=cloud-init-tutorial
export KEY_FILE=${KEY_NAME}.pem
aws ec2 create-key-pair --key-name ${KEY_NAME} --query 'KeyMaterial' --output text > ${KEY_FILE}
chmod 400 ${KEY_FILE}
```

Create a security group that allows for SSH access from anywhere.

```sh
export SEC_GRP_ID=`aws ec2 create-security-group --group-name CloudInitTutorial --description "SSH from anywhere" --output json | jq -r '.GroupId'`

aws ec2 authorize-security-group-ingress \
    --group-id ${SEC_GRP_ID} \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

Define the cloud-config file we will use to install apache.

```sh
cat <<'EOF' >> cloud-config.txt
#cloud-config
repo_update: true
repo_upgrade: all

packages:
 - httpd
 - mariadb-server

runcmd:
 - [ sh, -c, "amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2" ]
 - systemctl start httpd
 - sudo systemctl enable httpd
 - [ sh, -c, "usermod -a -G apache ec2-user" ]
 - [ sh, -c, "chown -R ec2-user:apache /var/www" ]
 - chmod 2775 /var/www
 - [ find, /var/www, -type, d, -exec, chmod, 2775, {}, \; ]
 - [ find, /var/www, -type, f, -exec, chmod, 0664, {}, \; ]
 - [ sh, -c, 'echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php' ]
EOF
```


Create the VM. Note the `--user-data` flag we are passing. This is where we are specifying the cloud-init configuration. AWS knows to interpret this as a cloud-init file because of the first line in the file: `#cloud-config`. See more examples of cloud configs [here](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).

```sh
export INSTANCE_ID=`aws ec2 run-instances \
	--image-id ami-00c03f7f7f2ec15c3 \
	--count 1 \
	--instance-type t2.micro \
	--key-name ${KEY_NAME} \
	--security-group-ids ${SEC_GRP_ID} \
	--user-data file://cloud-config.txt \
	| jq -r '.Instances[0].InstanceId'`
```

Grab the public DNS name of the VM.

```sh
export INSTANCE_DNS=`aws ec2 describe-instances --filters "Name=instance-id,Values=${INSTANCE_ID}" | jq -r '.Reservations[0].Instances[0].PublicDnsName'`
```

Login to the VM and test that our cloud-init configuration worked. Note: you may have to wait for the instance to initialize.

```
ssh -i ${KEY_FILE} ec2-user@${INSTANCE_DNS}

wget -O - http://localhost:80/phpinfo.php
```

### Cleanup

```
# Terminate instance.
aws ec2 terminate-instances --instance-ids ${INSTANCE_ID}

# Delete security group (after instance is terminated).
until aws ec2 delete-security-group --group-id ${SEC_GRP_ID}; do echo 'Attempting to delete security group again...' && sleep 1; done

# Delete our key pair we created.
aws ec2 delete-key-pair --key-name ${KEY_NAME}
rm ${KEY_FILE}
```

### Conclusion

While cloud-init can be used to configure almost any aspect of a VM instance, you want to be sure you are using the right tool for the job. For instance, installing packages on a VM might be a job better suited for a tool like [packer](https://www.packer.io/) which builds reusable images. This eliminates boot-time steps, reducing spin-up time. Reach for cloud-init in your toolbox when it does not make sense to bake the configuration into an image, for instance: joining a VM to a Kubernetes cluster.

Happy DevOp'ing!

