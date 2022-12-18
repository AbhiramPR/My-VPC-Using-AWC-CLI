
# Create a VPC Through AWS CLI

> Note :
The AWS Command Line Interface (AWS CLI) is a unified tool to manage our AWS services. With just one tool to download and configure, we can control multiple AWS services from the command line and automate them through scripts.

Got exited with the usage of AWS CLI and I would like to start a project on it.
In that project I am able to create a VPC through AWS CLI and here I shared the commands and sample bash script that I have used.

## Description
Let us start here to create an AWS VPC along with private/public Subnet and Network Gateway's for the VPC. This will be a VPC with 3 Subnets: 1 Private and 2 Public, 1 NAT Gateways, 1 Internet Gateway, 1 Route Tables, 1 security group and a key pair.

> Pre-requirement :
We have to create an IAM user with with programmatic access and VPC and EC2 full permission.

Lets starts ..

###  1. Configuring AWS CLI using the below command.

~~~sh
$ aws configure
AWS Access Key ID [None]:XXXXXXXXXXXX 
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json
~~~

### 2.  Creating VPC.
>Amazon Virtual Private Cloud (Amazon VPC) enables you to launch AWS resources into a virtual network that you've defined. This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS.

Create a VPC with a 10.0.0.0/16 CIDR block using the following command, and it will returns the ID of newly created VPC.
~~~sh
$ aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
vpc-0555fbab3f7bde094
~~~
For an easy way to identify the VPC, added a name tag to VPC.
~~~sh
$ aws ec2 create-tags --resources vpc-0555fbab3f7bde094 --tags Key=Name,Value=VPC-Project-AWS-CLI
~~~

### 3. Create 3 subnets (2 public and 1 private subnet).
> A subnet is a range of IP addresses in our VPC. We can attach AWS resources, such as EC2 instances and RDS DB instances, to subnets.
> The private IP address is assigned so the instance can communicate with other AWS services and other instances within the same private network, and the public IP address is assigned so the instance can communicate with the internet.

##### First public subnet with cidr-block 10.0.1.0/24 :


~~~sh
$ aws ec2 create-subnet --vpc-id vpc-0555fbab3f7bde094 --cidr-block 10.0.1.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-south-1b",
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.1.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-00fecc931c2bbd2bd",
        "VpcId": "vpc-0555fbab3f7bde094",
        "OwnerId": "520113205360",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:ap-south-1:520113205360:subnet/subnet-00fecc931c2bbd2bd",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}

~~~

Add tag to first subnet :
~~~sh
$ aws ec2create-tags --resources subnet-00fecc931c2bbd2bd --tagsKey=Name,Value=Subnet1-Project-VPC
~~~
##### Second public subnet with cidr-block 10.0.0.0/24 :
~~~sh
$ aws ec2 create-subnet --vpc-id vpc-0555fbab3f7bde094 --cidr-block 10.0.0.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-south-1b",
        "AvailabilityZoneId": "aps1-az3",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.0.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-075bddb097f3bf071",
        "VpcId": "vpc-0555fbab3f7bde094",
        "OwnerId": "520113205360",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:ap-south-1:520113205360:subnet/subnet-075bddb097f3bf071",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}

~~~
Add tag to second subnet :
~~~sh
$ aws ec2create-tags --resources subnet-075bddb097f3bf071 --tagsKey=Name,Value=Subnet2-Project-VPC
~~~

##### Create the private subnet with a 172.16.128.0/18 CIDR block. :
~~~sh
$ aws ec2 create-subnet --vpc-id vpc-0555fbab3f7bde094 --cidr-block 172.16.128.0/18 
{
    "Subnet": {
        "AvailabilityZone": "us-east-2b",
        "AvailabilityZoneId": "use2-az2",
        "AvailableIpAddressCount": 16379,
        "CidrBlock": "172.16.128.0/18",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-03956c03687b5d523",
        "VpcId": "vpc-0555fbab3f7bde094",
        "OwnerId": "257637015312",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "SubnetArn": "arn:aws:ec2:us-east-2:257637015312:subnet/subnet-03956c03687b5d523"
    }
}
~~~

Add tag to private subnet :
~~~sh
$ aws ec2create-tags --resources subnet-03956c03687b5d523 --tagsKey=Name,Value=P-Subnet3-Project-VPC
~~~

The below command will show the complete information of the VPC and subnets :
~~~sh
$ aws ec2 describe-subnets  --filters "Name=vpc-id,Values=vpc-0555fbab3f7bde094"
~~~

### 4. Create Interner Gateway. It will return the Internet Gateway ID :
> An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.

~~~sh
$ aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-0e10502c9eadbf747
~~~

We need to attach the Internet Gateway to the VPC.
~~~sh
$ aws ec2 attach-internet-gateway --vpc-id vpc-0555fbab3f7bde094 --internet-gateway-id igw-0e10502c9eadbf747
~~~

### 5. Create a Route table.
> The route table contains existing routes with targets other than a network interface, Gateway Load Balancer endpoint, or the default local route. The route table contains existing routes to CIDR blocks outside of the ranges in your VPC

~~~sh
$ aws ec2 create-route-table --vpc-id vpc-0555fbab3f7bde094 --query RouteTable.RouteTableId --output text
rtb-08fb167f2b887a7a4
~~~

Create a Route on the Route Table to route traffic to the internet :

~~~sh
$ aws ec2 create-route --route-table-id rtb-08fb167f2b887a7a4 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0e10502c9eadbf747
{
    "Return": true
}
~~~

Next we are going to associate the route table with a subnet in your VPC to route traffic from that subnet to the internet. For this, we need our subnet IDs.
If we are unsure about the subnet ID, we can use the below command.

~~~sh
$ aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-0555fbab3f7bde094" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
    [
    {
        "ID": "subnet-075bddb097f3bf071",
        "CIDR": "10.0.0.0/24"
    },
    {
        "ID": "subnet-00fecc931c2bbd2bd",
        "CIDR": "10.0.1.0/24"
    }
    ]
~~~


Associating route :

~~~sh
$ aws ec2 associate-route-table  --subnet-id subnet-075bddb097f3bf071 --route-table-id rtb-08fb167f2b887a7a4
{
    "AssociationId": "rtbassoc-0f7e51a1cf3ecaafa",
    "AssociationState": {
        "State": "associated"
    }
}
~~~

### 6. Assiging public IP address to first public subnet.
~~~sh
$ aws ec2 modify-subnet-attribute --subnet-id subnet-075bddb097f3bf071 --map-public-ip-on-launch
~~~

### 7. Create security group :
> A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic. Inbound rules control the incoming traffic to your instance, and outbound rules control the outgoing traffic from your instance. 

Below command will create a security group for the EC2 instance :
~~~sh
$ aws ec2 create-security-group --group-name freedom --description "Allow 22,80,443 access" --vpc-id vpc-0555fbab3f7bde094
{
    "GroupId": "sg-02872f51862d20a35"
}
~~~

I am going to open ports 22,80 and 443 for incomming traffic to EC2 instance.And it will accept traffic from all IPV4 address.

1.Allow port 22
~~~sh
$ aws ec2 authorize-security-group-ingress --group-id sg-02872f51862d20a35 --protocol tcp --port 22 --cidr 0.0.0.0/0
        {
        "Return": true,
        "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-025dba1072abe6f10",
            "GroupId": "sg-02872f51862d20a35",
            "GroupOwnerId": "520113205360",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
        ]
        }
~~~
2. Allow port 80
~~~sh
$ aws ec2 authorize-security-group-ingress --group-id sg-02872f51862d20a35 --protocol tcp --port 80 --cidr 0.0.0.0/0
         {
         "Return": true,
         "SecurityGroupRules": [
         {
            "SecurityGroupRuleId": "sgr-0e84de1a38fb2ff43",
            "GroupId": "sg-02872f51862d20a35",
            "GroupOwnerId": "520113205360",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
         }
         ]
         }
~~~
3. Allow port 443
~~~sh
$ aws ec2 authorize-security-group-ingress --group-id sg-02872f51862d20a35 --protocol tcp --port 443 --cidr 0.0.0.0/0
          {
          "Return": true,
          "SecurityGroupRules": [
          {
            "SecurityGroupRuleId": "sgr-070b947c01652220d",
            "GroupId": "sg-02872f51862d20a35",
            "GroupOwnerId": "520113205360",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 443,
            "ToPort": 443,
            "CidrIpv4": "0.0.0.0/0"
          }
          ]
          }
~~~

### 8. Creating key pair :
> A key pair, consisting of a public key and a private key, is a set of security credentials that you use to prove your identity when connecting to an Amazon EC2 instance. Amazon EC2 stores the public key on your instance, and you store the private key. For Linux instances, the private key allows you to securely SSH into your instance.
~~~sh
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
~~~

A "MyKeyPai.pem" file will be return to our "Download" folder and we need to change the permission of "".pem" file to "400"

~~~sh
$ chmod 400 MyKeyPair.pem
-r--------  1 dell dell     1675 Dec 10 05:51  MyKeyPair.pem
~~~

### 9. Before lauch the instance we have to select the AMI(not all AMIs are available in every region).

We can find the AMI using AWS CLI :
~~~sh
$ aws ec2 describe-images --owners self amazon
{
    "Images": [
        {
            "Architecture": "x86_64",
            "CreationDate": "2021-06-29T15:17:24.000Z",
            "ImageId": "ami-06910d5152d917447",
            "ImageLocation": "amazon/Cloud9AmazonLinux2-2021-06-29T14-28",
            "ImageType": "machine",
            "Public": true,
            "OwnerId": "528904141149",
            "PlatformDetails": "Linux/UNIX",
            "UsageOperation": "RunInstances",
            "State": "available",
            "BlockDeviceMappings": [
~~~

Done...
Now we have all the requirement to launce an instance.

I am going to add a userdata while launching the EC2 instacne.
> When you launch an instance in Amazon EC2, you have the option of passing user data to the instance that can be used to perform common automated configuration tasks and even run scripts after the instance starts. You can pass two types of user data to Amazon EC2: shell scripts and cloud-init directives.

Here is the contents inside apache.sh file :
~~~sh
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
service sshd restart


yum install httpd php -y

systemctl restart httpd.service
systemctl enable httpd.service
~~~

After created the file I have changed the permission to 744.
~~~sh
$ chmod 744 apache.sh
~~~

Now we can launce the instance using AWS CLI.
### 10. Lauch instance
~~~sh
$ aws ec2 run-instances --image-id ami-06910d5152d917447 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-02872f51862d20a35 --subnet-id subnet-075bddb097f3bf071 --user-data file://apache.sh
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-06910d5152d917447",
            "InstanceId": "i-0b14632d983388452",
            "InstanceType": "t2.micro",
            "KeyName": "MyKeyPair",
            "LaunchTime": "2022-12-10T00:37:00+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "ap-south-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-124.ap-south-1.compute.internal",
            "PrivateIpAddress": "10.0.0.124",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
~~~
Success....

Let me add a tag to new Instance :
~~~sh
$ aws ec2 create-tags --resources i-0b14632d983388452 --tags Key=Name,Value=VPC-project-EC2
~~~

As of now, I have Launched and EC2 instace using AWS CLI.

Next I am gona check whether I am able to ssh to the server or not.
~~~sh
$ ssh -i MyKeyPair.pem ec2-user@3.7.46.72
The authenticity of host '3.7.46.72 (3.7.46.72)' can't be established.
ECDSA key fingerprint is SHA256:WAt1nKF1dpPc3J0lMClnVpytfD2fivEf/uOgzPyD4ho.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '3.7.46.72' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
142 package(s) needed for security, out of 189 available
Run "sudo yum update" to apply all updates.
~~~
Yes,I am able to ssh to the server.

One last step. I have added a userdate while launching the instance and let me check whether the userdata work as expected.
~~~sh
 sudo systemctl status httpd.service 
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
   Active: active (running) since Sat 2022-12-10 00:38:15 UTC; 9min ago
     Docs: man:httpd.service(8)
 Main PID: 5417 (httpd)
   Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─5417 /usr/sbin/httpd -DFOREGROUND
           ├─5484 /usr/sbin/httpd -DFOREGROUND
           ├─5486 /usr/sbin/httpd -DFOREGROUND
           ├─5487 /usr/sbin/httpd -DFOREGROUND
           ├─5488 /usr/sbin/httpd -DFOREGROUND
           └─5489 /usr/sbin/httpd -DFOREGROUND

Dec 10 00:38:15 ip-10-0-0-124.ap-south-1.compute.internal systemd[1]: Starting The Apache HTTP Server...
Dec 10 00:38:15 ip-10-0-0-124.ap-south-1.compute.internal systemd[1]: Started The Apache HTTP Server.
~~~

Yes, Apache installed and the service is up and running....

## Conclusion

Here I have shared a simple AWS CLI commands to build an AWS VPC along with private/public Subnet and Network Gateway's for the VPC.very easy to follow and implement AWS CLI commands.




