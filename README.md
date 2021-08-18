# Infrastructure Build With AWS-CLI
### (VPC & EC2 - creation and configuration via AWS-CLI)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Here is a project with the requirement was to build basic infrastructure for a website where there is no access to the AWS console. Only provided with an IAM user with programmatic access. How can we achieve the goal, here comes the AWS-CLI which makes it possible. 
Here in this project initially I have configured a VPC with 3 public subnets and also provides necessary routes. Further moved on with ec2 creation which includes selecting the appropriate AMI, security groups, key pairs, and so on. 
After the whole process, there was an additional requirement which was to create and attach an additional volume to the ec2 instance. I have provided herewith a detailed command summary that I have executed in the project.   

## Features

- AWS-CLI can manage multiple services in AWS
- Easier to install.
- Ability to automate control of all Amazon’s web services with scripts is possibly the biggest benefit. 

## Resources created via AWS-CLI

Here is the list of the main resources created via AWS-CLI
- VPC
- Subnet
- Internet Gateway
- Route table
- Key pair
- Security groups
- Ec2 instance
- Additional EBS Volume


## Pre-Requests
- Basic knowledge of AWS services EC2, EBS, IAM, VPC, Security groups
- IAM user with programmatic Access
- AWS Command Line Interface Installed on the system

## Steps For The Creation & Configuration
Initially, the AWS CLI configuring with the available access key ID and AWS secret key

```sh
aws configure --profile=awscli-ajish
```
Output
```sh
AWS Access Key ID [None]: AKIAxxxxxxxx-xxxxxxx42G
AWS Secret Access Key [None]: F9s1xxxx-xxxxxxx-xxxxxxx3Qin
Default region name [None]: us-east-1
Default output format [None]: json
```
Proceeding with the VPC creation
Initially creating a VPC with IP CIDR block of 172.16.0.0/16
```sh
aws ec2 create-vpc --cidr-block 172.16.0.0/16
```
Output
```sh
{
    "Vpc": {
        "CidrBlock": "172.16.0.0/16",
        "DhcpOptionsId": "dopt-eb844f80",
        "State": "pending",
        "VpcId": "vpc-0c059205f4d1da428",
        "OwnerId": "925xxxxxxx04",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-0463aaacf4ad5b885",
                "CidrBlock": "172.16.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false
    }
}
```
Updating the VPC with the tag "production-vpc" to identify properly for further procedures.

```sh
aws ec2 create-tags --resources vpc-0c059205f4d1da428 --tags Key=Name,Value=Production-vpc
```
Using the VPC ID from the previous step, moving forward with the subnet creation. First subnet with a 172.16.0.0/18 CIDR block.
```sh
aws ec2 create-subnet --vpc-id vpc-0c059205f4d1da428 --cidr-block 172.16.0.0/18
```
Created a second subnet in the VPC with a 172.16.64.0/18 CIDR block.
```sh
aws ec2 create-subnet --vpc-id vpc-0c059205f4d1da428 --cidr-block 172.16.64.0/18
```
And the third subnet with a 172.16.128.0/18 CIDR block
```sh
aws ec2 create-subnet --vpc-id vpc-0c059205f4d1da428 --cidr-block 172.16.128.0/18
```
Output
```sh
{
    "Subnet": {
        "AvailabilityZone": "us-east-2b",
        "AvailabilityZoneId": "use2-az2",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.16.0.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0ea166295d2cc75c3",
        "VpcId": "vpc-0c059205f4d1da428",
        "OwnerId": "925xxxxxxx04",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-2:925466079004:subnet/subnet-0ea166295d2cc75c3"
    }
}
```
To make the subnet public then proceeds with the creation of Internet Gateway and tagged the same appropriately.

```sh
aws ec2 create-internet-gateway
aws ec2 create-tags --resources igw-04524d266791ea43d --tags Key=Name,Value=igw-pro
```
Output
```sh
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-04524d266791ea43d",
        "OwnerId": "925xxxxxxx04",
        "Tags": []
    }
}
```
Attached the Internet gateway to the VPC
```sh
aws ec2 attach-internet-gateway --vpc-id vpc-0c059205f4d1da428 --internet-gateway-id igw-04524d266791ea43d
```
Created a custom route table for the VPC.

```sh
aws ec2 create-route-table --vpc-id vpc-0c059205f4d1da428
```
Output
```sh
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-00d38f856d6e78e5f",
        "Routes": [
            {
                "DestinationCidrBlock": "172.16.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [],
        "VpcId": "vpc-0c059205f4d1da428",
        "OwnerId": "925xxxxxxx04"
    }
}
```
Pointed the route table to the IGW with all traffic (0.0.0.0/0) and also tagged the route table.

```sh
aws ec2 create-route --route-table-id rtb-00d38f856d6e78e5f --destination-cidr-block 0.0.0.0/0 --gateway-id igw-04524d266791ea43d
aws ec2 create-tags --resources rtb-00d38f856d6e78e5f --tags Key=Name,Value=route-pro
```
Output
```sh
{
    "Return": true
}
```
To confirm the route has been created and is active, the below command can be used to describe the route table and results.

```sh
aws ec2 describe-route-tables --route-table-id rtb-00d38f856d6e78e5f
```
Output
```sh
{
    "RouteTables": [
        {
            "Associations": [],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-00d38f856d6e78e5f",
            "Routes": [
                {
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "GatewayId": "igw-04524d266791ea43d",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "route-pro"
                }
            ],
            "VpcId": "vpc-0c059205f4d1da428",
            "OwnerId": "925xxxxxxx04"
        }
    ]
}
```
Currently, the route table is not associated with any subnet. It needs to associate it with a subnet in the VPC so that traffic from that subnet is routed to the internet gateway. First using the below command identify the subnet ID's

```sh
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-0c059205f4d1da428" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
```
Output
```sh
[
    {
        "ID": "subnet-0499cd7611056bc62",
        "CIDR": "172.16.128.0/18"
    },
    {
        "ID": "subnet-07aa1ac26f70130ed",
        "CIDR": "172.16.64.0/18"
    },
    {
        "ID": "subnet-0ea166295d2cc75c3",
        "CIDR": "172.16.0.0/18"
    }
]
```
Then accordingly update subnet to associate with the custom route table.

```sh
aws ec2 associate-route-table  --subnet-id subnet-0499cd7611056bc62 --route-table-id rtb-00d38f856d6e78e5f
aws ec2 associate-route-table  --subnet-id subnet-07aa1ac26f70130ed --route-table-id rtb-00d38f856d6e78e5f
aws ec2 associate-route-table  --subnet-id subnet-0ea166295d2cc75c3 --route-table-id rtb-00d38f856d6e78e5f
```
Output
```sh
{
    "AssociationId": "rtbassoc-093e18b2563d7d57c",
    "AssociationState": {
        "State": "associated"
    }
}
```
Then modified the public IP addressing behavior of the subnet so that an instance launched into the subnet automatically receives a public IP address.
```sh
aws ec2 modify-subnet-attribute --subnet-id subnet-0499cd7611056bc62 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-07aa1ac26f70130ed --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-0ea166295d2cc75c3 --map-public-ip-on-launch
```

With this configuration, our VPC part is over then comes the next important creation of EC2. Initially created a security group and make sure to create the security group in the appropriate VPC. Also, tag the security group as good practice. 

```
aws ec2 create-security-group --group-name web-sg --description "SG for website" --vpc-id vpc-0c059205f4d1da428
aws ec2 create-tags --resources sg-0f71d0c89b2a6dcc8 --tags Key=Name,Value=web-sg
```
Output
```sh
{
    "GroupId": "sg-0f71d0c89b2a6dcc8"
}
```
Update the security groups with rule for the instance with ports 22,80,443 open. Also enable ssh port 22 only to the required user as security measure.
```sh
aws ec2 authorize-security-group-ingress --group-id sg-0f71d0c89b2a6dcc8 --protocol tcp --port 22 --cidr x.x.x.x/x
aws ec2 authorize-security-group-ingress --group-id sg-0f71d0c89b2a6dcc8 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-0f71d0c89b2a6dcc8 --protocol tcp --port 443 --cidr 0.0.0.0/0
```
Key pair creation for the instance and copying to a file for further use.

```sh
aws ec2 create-key-pair --key-name website-keypair --query 'KeyMaterial' --output text > web-keypair.pem
```

Then before moving forward with the EC2 creation and before that to know about the Availability zones and the AMI that needs. The provided command will provide the details of the  Availability Zones. 

```sh
aws ec2 describe-availability-zones
```
Output
```sh
{
    "AvailabilityZones": [
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "us-east-2",
            "ZoneName": "us-east-2a",
            "ZoneId": "use2-az1",
            "GroupName": "us-east-2",
            "NetworkBorderGroup": "us-east-2"
        },
...
```
For the creation of the EC2 instance, we need to select the required AMI. In my case, I have to choose the latest AMI of Amazon Linux 2. With the command given below will sort out the latest AMI ID of the same.
```sh
aws ec2 describe-images --owners amazon --filters 'Name=name,Values=amzn2-ami-hvm-2.0.??????????-x86_64-gp2' 'Name=state,Values=available' --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
```
Then Created the Ec2 instance for the webiste with the AMI, security group, key pair,VPC, subnet that we have configured. 
```sh
aws ec2 run-instances --image-id ami-0443305dabd4be2bc --count 1 --instance-type t2.micro --key-name website-keypair --security-group-ids sg-0f71d0c89b2a6dcc8 --subnet-id subnet-0499cd7611056bc62
```
To ssh in to the insatnce, we need to identify the public IP. 
```sh
aws ec2 describe-instances --instance-ids i-0a3b13de7a31c2612 --query 'Reservations[0].Instances[0].PublicIpAddress'
```
With the public IP ssh into the instance.

Then comes the next request, where the need for additional storage of 2GB. Make sure to create the additional EBS volume in the same Avilabiy zone of the EC2-instance.

```sh
aws ec2 create-volume --region us-east-2 --availability-zone us-east-2b --size 2 --volume-type gp2
```
Output
```sh
{
    "AvailabilityZone": "us-east-2b",
    "CreateTime": "2021-08-17T17:51:40.000Z",
    "Encrypted": false,
    "Size": 2,
    "SnapshotId": "",
    "State": "creating",
    "VolumeId": "vol-01d9faabfe14fc3f6",
    "Iops": 100,
    "Tags": [],
    "VolumeType": "gp2",
    "MultiAttachEnabled": false
}
```
Attach the newly created volume to the instance.

```sh
aws ec2 attach-volume --volume-id vol-01d9faabfe14fc3f6 --instance-id i-0a3b13de7a31c2612 --device /dev/sdf
```
Output
```sh
{
    "AttachTime": "2021-08-17T17:52:44.243Z",
    "Device": "/dev/sdf",
    "InstanceId": "i-0a3b13de7a31c2612",
    "State": "attaching",
    "VolumeId": "vol-01d9faabfe14fc3f6"
}
```
SSH into the server and proceed with steps to make available the new volume.
```sh
lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   2G  0 disk
Partitioned the new disk
Command: fdisk /dev/xvdf
Format the new block
Command: mkfs -t ext4 /dev/xvdf1
Mount the disk
Command: echo "/dev/xvdf1      /var/www/html   ext4 defaults   0       2" >> /etc/fstab
       :mount -a
Resize the same
Command: resize2fs /dev/xvdf1
df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        482M     0  482M   0% /dev
tmpfs           492M     0  492M   0% /dev/shm
tmpfs           492M  416K  492M   1% /run
tmpfs           492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.5G  6.6G  19% /
tmpfs            99M     0   99M   0% /run/user/1000
/dev/xvdf1      2.0G  6.0M  1.9G   1% /var/www/html
```

## Conclusion

Here it is the project with complete utilization of the AWS CLI for the management of different AWS services.
