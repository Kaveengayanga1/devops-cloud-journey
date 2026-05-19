
~ on ☁️  (us-east-1) ➜  aws ec2 describe-vpcs \
    --query 'Vpcs[*].{Name:Tags[?Key==`Name`].Value | [0], VpcId:VpcId, Cidr:CidrBlock}' \
    --output table
----------------------------------------------------------------
|                         DescribeVpcs                         |
+---------------+--------------------+-------------------------+
|     Cidr      |       Name         |          VpcId          |
+---------------+--------------------+-------------------------+
|  172.31.0.0/16|  None              |  vpc-0167e62e32a17f38c  |
|  10.1.0.0/16  |  xfusion-priv-vpc  |  vpc-07e05d762ece40ef7  |
+---------------+--------------------+-------------------------+

~ on ☁️  (us-east-1) ➜  aws ec2 create-subnet \
    --vpc-id vpc-07e05d762ece40ef7 \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]'

An error occurred (InvalidSubnet.Range) when calling the CreateSubnet operation: The CIDR '10.0.1.0/24' is invalid.

~ on ☁️  (us-east-1) ✖ aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=vpc-07e05d762ece40ef7" \
    --query 'Subnets[*].{Name:Tags[?Key==`Name`].Value | [0], CIDR:CidrBlock}' \
    --output table
----------------------------------------
|            DescribeSubnets           |
+--------------+-----------------------+
|     CIDR     |         Name          |
+--------------+-----------------------+
|  10.1.1.0/24 |  xfusion-priv-subnet  |
+--------------+-----------------------+

~ on ☁️  (us-east-1) ➜  aws ec2 create-subnet \
    --vpc-id vpc-07e05d762ece40ef7 \
    --cidr-block 10.1.1.0/24 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]'

An error occurred (InvalidSubnet.Conflict) when calling the CreateSubnet operation: The CIDR '10.1.1.0/24' conflicts with another subnet

~ on ☁️  (us-east-1) ✖ aws ec2 create-subnet \
    --vpc-id vpc-07e05d762ece40ef7 \
    --cidr-block 10.1.2.0/24 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]'
{
    "Subnet": {
        "AvailabilityZoneId": "use1-az2",
        "MapCustomerOwnedIpOnLaunch": false,
        "OwnerId": "552551974974",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "xfusion-pub-subnet"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:552551974974:subnet/subnet-0ae3e93abebb81200",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        },
        "SubnetId": "subnet-0ae3e93abebb81200",
        "State": "available",
        "VpcId": "vpc-07e05d762ece40ef7",
        "CidrBlock": "10.1.2.0/24",
        "AvailableIpAddressCount": 251,
        "AvailabilityZone": "us-east-1a",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false
    }
}

~ on ☁️  (us-east-1) ➜  aws ec2 describe-internet-gateways \
    --filters "Name=attachment.vpc-id,Values=vpc-07e05d762ece40ef7" \
    --query 'InternetGateways[0].InternetGatewayId' \
    --output text
None

~ on ☁️  (us-east-1) ➜  RT_PUB_ID=$(aws ec2 create-route-table \
    --vpc-id vpc-07e05d762ece40ef7 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=xfusion-pub-rt}]' \
    --query 'RouteTable.RouteTableId' --output text)

~ on ☁️  (us-east-1) ➜  IGW_ID=$(aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=xfusion-igw}]' \
    --query 'InternetGateway.InternetGatewayId' \
    --output text)

~ on ☁️  (us-east-1) ➜  echo "Created IGW: $IGW_ID"
Created IGW: igw-0ea77d1c8f6923d99

~ on ☁️  (us-east-1) ➜  aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id vpc-07e05d762ece40ef7

~ on ☁️  (us-east-1) ➜  echo $RT_PUB_ID
rtb-0f8985c1a056853b5

~ on ☁️  (us-east-1) ➜  aws ec2 create-route \
    --route-table-id $RT_PUB_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID
{
    "Return": true
}

~ on ☁️  (us-east-1) ➜  aws ec2 associate-route-table \
    --route-table-id $RT_PUB_ID \
    --subnet-id subnet-0ae3e93abebb81200
{
    "AssociationId": "rtbassoc-01c612c5f29cef174",
    "AssociationState": {
        "State": "associated"
    }
}

~ on ☁️  (us-east-1) ➜  NAT_SG_ID=$(aws ec2 create-security-group \
    --group-name xfusion-nat-sg \
    --description "Security group for NAT instance" \
    --vpc-id vpc-07e05d762ece40ef7 \
    --query 'GroupId' --output text)

~ on ☁️  (us-east-1) ➜  aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 443 --cidr 10.1.1.0/24
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0ce03590c82830f21",
            "GroupId": "sg-012d4f78bf86ccd9d",
            "GroupOwnerId": "552551974974",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 443,
            "ToPort": 443,
            "CidrIpv4": "10.1.1.0/24",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:552551974974:security-group-rule/sgr-0ce03590c82830f21"
        }
    ]
}

~ on ☁️  (us-east-1) ➜  aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 80 --cidr 10.1.1.0/24
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-06ffba5013f5759ae",
            "GroupId": "sg-012d4f78bf86ccd9d",
            "GroupOwnerId": "552551974974",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "10.1.1.0/24",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:552551974974:security-group-rule/sgr-06ffba5013f5759ae"
        }
    ]
}

~ on ☁️  (us-east-1) ➜  aws ec2 create-key-pair \
    --key-name xfusion-nat-key \
    --query 'KeyMaterial' \
    --output text > xfusion-nat-key.pem

~ on ☁️  (us-east-1) ➜  chmod 400 xfusion-nat-key.pem

~ on ☁️  (us-east-1) ➜  aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0001c3a4ec8d7556e",
            "GroupId": "sg-012d4f78bf86ccd9d",
            "GroupOwnerId": "552551974974",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:552551974974:security-group-rule/sgr-0001c3a4ec8d7556e"
        }
    ]
}

~ on ☁️  (us-east-1) ➜  AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query 'Parameters[0].Value' --output text)

~ on ☁️  (us-east-1) ➜  echo "Latest AMI ID: $AMI_ID"
Latest AMI ID: ami-02dfbd4ff395f2a1b

~ on ☁️  (us-east-1) ➜  NAT_INSTANCE_ID=$(aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type t3.micro \
    --key-name xfusion-nat-key \
    --security-group-ids $NAT_SG_ID \
    --subnet-id subnet-0ae3e93abebb81200 \
    --associate-public-ip-address \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-nat-instance}]' \
    --query 'Instances[0].InstanceId' --output text)

~ on ☁️  (us-east-1) ➜  echo "NAT Instance ID: $NAT_INSTANCE_ID"
NAT Instance ID: i-0e6766832a4baba54

~ on ☁️  (us-east-1) ➜  aws ec2 modify-instance-attribute --instance-id $NAT_INSTANCE_ID --no-source-dest-check 
~ on ☁️  (us-east-1) ➜  PUBLIC_IP=$(aws ec2 describe-instances --instance-ids $NAT_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

~ on ☁️  (us-east-1) ➜  ssh -i "xfusion-nat-key.pem" ec2-user@$PUBLIC_IP
The authenticity of host '44.203.199.23 (44.203.199.23)' can't be established.
ECDSA key fingerprint is SHA256:SXg+mhkleg1n8rRAt3vEpUFWwYp0i+h3Uxbb9mPwOwo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? ytes
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '44.203.199.23' (ECDSA) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-10-1-2-109 ~]$ sudo dnf install iptables-services -y
Amazon Linux 2023 Kernel Livepatch repository                                 253 kB/s |  30 kB     00:00    
Dependencies resolved.
==============================================================================================================
 Package                         Architecture    Version                           Repository            Size
==============================================================================================================
Installing:
 iptables-services               noarch          1.8.8-3.amzn2023.0.2              amazonlinux           18 k
Installing dependencies:
 iptables-libs                   x86_64          1.8.8-3.amzn2023.0.2              amazonlinux          401 k
 iptables-nft                    x86_64          1.8.8-3.amzn2023.0.2              amazonlinux          183 k
 iptables-utils                  x86_64          1.8.8-3.amzn2023.0.2              amazonlinux           43 k
 libnetfilter_conntrack          x86_64          1.0.8-2.amzn2023.0.2              amazonlinux           58 k
 libnfnetlink                    x86_64          1.0.1-19.amzn2023.0.2             amazonlinux           30 k
 libnftnl                        x86_64          1.2.2-2.amzn2023.0.2              amazonlinux           84 k

Transaction Summary
==============================================================================================================
Install  7 Packages

Total download size: 816 k
Installed size: 2.9 M
Downloading Packages:
(1/7): iptables-libs-1.8.8-3.amzn2023.0.2.x86_64.rpm                           12 MB/s | 401 kB     00:00    
(2/7): iptables-services-1.8.8-3.amzn2023.0.2.noarch.rpm                      492 kB/s |  18 kB     00:00    
(3/7): iptables-nft-1.8.8-3.amzn2023.0.2.x86_64.rpm                           3.7 MB/s | 183 kB     00:00    
(4/7): iptables-utils-1.8.8-3.amzn2023.0.2.x86_64.rpm                         1.6 MB/s |  43 kB     00:00    
(5/7): libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64.rpm                 2.3 MB/s |  58 kB     00:00    
(6/7): libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64.rpm                          1.3 MB/s |  30 kB     00:00    
(7/7): libnftnl-1.2.2-2.amzn2023.0.2.x86_64.rpm                               3.0 MB/s |  84 kB     00:00    
--------------------------------------------------------------------------------------------------------------
Total                                                                         6.4 MB/s | 816 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                      1/1 
  Installing       : libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64                                            1/7 
  Installing       : libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64                                   2/7 
  Installing       : iptables-libs-1.8.8-3.amzn2023.0.2.x86_64                                            3/7 
  Installing       : libnftnl-1.2.2-2.amzn2023.0.2.x86_64                                                 4/7 
  Installing       : iptables-nft-1.8.8-3.amzn2023.0.2.x86_64                                             5/7 
  Running scriptlet: iptables-nft-1.8.8-3.amzn2023.0.2.x86_64                                             5/7 
  Installing       : iptables-utils-1.8.8-3.amzn2023.0.2.x86_64                                           6/7 
  Installing       : iptables-services-1.8.8-3.amzn2023.0.2.noarch                                        7/7 
  Running scriptlet: iptables-services-1.8.8-3.amzn2023.0.2.noarch                                        7/7 
  Verifying        : iptables-libs-1.8.8-3.amzn2023.0.2.x86_64                                            1/7 
  Verifying        : iptables-nft-1.8.8-3.amzn2023.0.2.x86_64                                             2/7 
  Verifying        : iptables-services-1.8.8-3.amzn2023.0.2.noarch                                        3/7 
  Verifying        : iptables-utils-1.8.8-3.amzn2023.0.2.x86_64                                           4/7 
  Verifying        : libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64                                   5/7 
  Verifying        : libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64                                            6/7 
  Verifying        : libnftnl-1.2.2-2.amzn2023.0.2.x86_64                                                 7/7 

Installed:
  iptables-libs-1.8.8-3.amzn2023.0.2.x86_64                 iptables-nft-1.8.8-3.amzn2023.0.2.x86_64         
  iptables-services-1.8.8-3.amzn2023.0.2.noarch             iptables-utils-1.8.8-3.amzn2023.0.2.x86_64       
  libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64        libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64        
  libnftnl-1.2.2-2.amzn2023.0.2.x86_64                     

Complete!
[ec2-user@ip-10-1-2-109 ~]$ sudo systemctl enable iptables
Created symlink /etc/systemd/system/multi-user.target.wants/iptables.service → /usr/lib/systemd/system/iptables.service.
[ec2-user@ip-10-1-2-109 ~]$ sudo systemctl start iptables
[ec2-user@ip-10-1-2-109 ~]$ echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
net.ipv4.ip_forward = 1
[ec2-user@ip-10-1-2-109 ~]$ sudo sysctl -p
net.ipv4.ip_forward = 1
[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
[ec2-user@ip-10-1-2-109 ~]$ sudo service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[ec2-user@ip-10-1-2-109 ~]$ exit
logout
Connection to 44.203.199.23 closed.

~ on ☁️  (us-east-1) ➜  PRIV_RT_ID=$(aws ec2 describe-route-tables \
    --filters "Name=vpc-id,Values=vpc-07e05d762ece40ef7" \
    --query 'RouteTables[?Associations[?SubnetId==`subnet-0ae3e93abebb81200`==`false` && contains(Tags[?Key==`Name`].Value, `priv`)]].RouteTableId | [0]' \
    --output text)

In function contains(), invalid type for value: None, expected one of: ['array', 'string'], received: "null"

~ on ☁️  (us-east-1) ✖ PRIV_SUB_ID=$(aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=vpc-07e05d762ece40ef7" "Name=tag:Name,Values=xfusion-priv-subnet" \
    --query 'Subnets[0].SubnetId' --output text)

~ on ☁️  (us-east-1) ➜  PRIV_RT_ID=$(aws ec2 describe-route-tables \
    --filters "Name=association.subnet-id,Values=$PRIV_SUB_ID" \
    --query 'RouteTables[0].RouteTableId' --output text)

~ on ☁️  (us-east-1) ➜  echo "Your Private Route Table ID is: $PRIV_RT_ID"
Your Private Route Table ID is: rtb-0a4256f54ef3a7742

~ on ☁️  (us-east-1) ➜  aws ec2 create-route \
    --route-table-id $PRIV_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --instance-id $NAT_INSTANCE_ID
{
    "Return": true
}

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  ip link show
bash: ip: command not found

~ on ☁️  (us-east-1) ✖ ssh -i "xfusion-nat-key.pem" ec2-user@$PUBLIC_IP
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Thu Mar 19 03:04:08 2026 from 65.108.255.62
[ec2-user@ip-10-1-2-109 ~]$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 12:8b:58:9d:65:73 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname eni-0a5f00d50ed4bab58
    altname device-number-0.0
[ec2-user@ip-10-1-2-109 ~]$ curl -I https://www.google.com
HTTP/2 200 
content-type: text/html; charset=ISO-8859-1
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-osCEMhzWkcie5UhiVxRaLQ' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
reporting-endpoints: default="//www.google.com/httpservice/retry/jserror?ei=Kmm7aYG7I-Tn5NoP1Jn7uAI&cad=crash&error=Page%20Crash&jsel=1&bver=2404&dpf=ZC0Gxun2xLcS0iai2MV4QSPplYHgNDyjaeG8xiwfhFU"
accept-ch: Sec-CH-Prefers-Color-Scheme
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Thu, 19 Mar 2026 03:10:34 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Thu, 19 Mar 2026 03:10:34 GMT
cache-control: private
set-cookie: AEC=AaJma5v3FjonFomfoSmqaWCafaKGrAUIjM8Az7NpscLizE_Hf0o3xsXI9ys; expires=Tue, 15-Sep-2026 03:10:34 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=529=yjCxeD-M5BWmqmjryPlRuOimHKqp8sSDOqFnSapSPtEPzH0MWexk31zZcoYgVkG1bHo4n03cTK70hTcDRcDWh-GAdsDLMtVbFQM7nrV4vzc4wHSnXRrUOa21ywR7IHyUTlAyzL9x36tfqvVszSjIrO-xeeP3qn1tQ4pyECGvMq6XJb-PtUzeF4bF_884c0t-6G_-gzcmuvUwiL5LlvIrgiRUw3ceHQLDLg; expires=Fri, 18-Sep-2026 03:10:34 GMT; path=/; domain=.google.com; HttpOnly
set-cookie: __Secure-BUCKET=CJ8E; expires=Tue, 15-Sep-2026 03:10:34 GMT; path=/; domain=.google.com; Secure; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -t nat -F
[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
[ec2-user@ip-10-1-2-109 ~]$ sudo service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[ec2-user@ip-10-1-2-109 ~]$ sudo tcpdump -i ens5 icmp or port 443
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens5, link-type EN10MB (Ethernet), snapshot length 262144 bytes
03:12:44.055554 IP 10.1.1.170.42046 > 16.15.236.138.https: Flags [S], seq 1383719203, win 26883, options [mss 8961,sackOK,TS val 3053153244 ecr 0,nop,wscale 7], length 0
03:12:44.055594 IP ip-10-1-2-109.ec2.internal > 10.1.1.170: ICMP host 16.15.236.138 unreachable - admin prohibited, length 68
03:12:44.055907 IP 10.1.1.170.37508 > 16.15.183.71.https: Flags [S], seq 1199266107, win 26883, options [mss 8961,sackOK,TS val 679464805 ecr 0,nop,wscale 7], length 0
03:12:44.759654 IP 10.1.1.170.56236 > s3-1-w.amazonaws.com.https: Flags [S], seq 1478061594, win 26883, options [mss 8961,sackOK,TS val 4050133794 ecr 0,nop,wscale 7], length 0
03:12:44.759694 IP ip-10-1-2-109.ec2.internal > 10.1.1.170: ICMP host s3-1-w.amazonaws.com unreachable - admin prohibited, length 68
03:12:44.760021 IP 10.1.1.170.51088 > s3-1-w.amazonaws.com.https: Flags [S], seq 1420866156, win 26883, options [mss 8961,sackOK,TS val 3663730116 ecr 0,nop,wscale 7], length 0
03:12:45.079554 IP 10.1.1.170.37508 > 16.15.183.71.https: Flags [S], seq 1199266107, win 26883, options [mss 8961,sackOK,TS val 679465829 ecr 0,nop,wscale 7], length 0
03:12:45.783562 IP 10.1.1.170.51088 > s3-1-w.amazonaws.com.https: Flags [S], seq 1420866156, win 26883, options [mss 8961,sackOK,TS val 3663731140 ecr 0,nop,wscale 7], length 0
03:12:45.783602 IP ip-10-1-2-109.ec2.internal > 10.1.1.170: ICMP host s3-1-w.amazonaws.com unreachable - admin prohibited, length 68
03:12:45.783920 IP 10.1.1.170.43886 > s3-1-w.amazonaws.com.https: Flags [S], seq 2613051463, win 26883, options [mss 8961,sackOK,TS val 376189836 ecr 0,nop,wscale 7], length 0
03:12:46.807612 IP 10.1.1.170.43886 > s3-1-w.amazonaws.com.https: Flags [S], seq 2613051463, win 26883, options [mss 8961,sackOK,TS val 376190860 ecr 0,nop,wscale 7], length 0
03:12:46.807652 IP ip-10-1-2-109.ec2.internal > 10.1.1.170: ICMP host s3-1-w.amazonaws.com unreachable - admin prohibited, length 68
03:12:46.808006 IP 10.1.1.170.38348 > s3-1-w.amazonaws.com.https: Flags [S], seq 1199764254, win 26883, options [mss 8961,sackOK,TS val 3342446643 ecr 0,nop,wscale 7], length 0
03:12:47.063525 IP 10.1.1.170.33572 > s3-1-w.amazonaws.com.https: Flags [S], seq 1528019306, win 26883, options [mss 8961,sackOK,TS val 3285990048 ecr 0,nop,wscale 7], length 0
03:12:47.095568 IP 10.1.1.170.37508 > 16.15.183.71.https: Flags [S], seq 1199266107, win 26883, options [mss 8961,sackOK,TS val 679467844 ecr 0,nop,wscale 7], length 0
^C
15 packets captured
15 packets received by filter
0 packets dropped by kernel
[ec2-user@ip-10-1-2-109 ~]$ sudo tcpdump -i any srun rc 10.1.1.0/24
tcpdump: data link type LINUX_SLL2
tcpdump: can't parse filter expression: syntax error
[ec2-user@ip-10-1-2-109 ~]$ exit
logout
Connection to 44.203.199.23 closed.

~ on ☁️  (us-east-1) ✖ aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/
^[[A
~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/
^[[A
~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/

~ on ☁️  (us-east-1) ➜  ssh -i "xfusion-nat-key.pem" ec2-user@$PUBLIC_IP
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Thu Mar 19 03:10:13 2026 from 65.108.255.62
[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -P FORWARD ACCEPT
[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -F FORWARD
[ec2-user@ip-10-1-2-109 ~]$ sudo service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[ec2-user@ip-10-1-2-109 ~]$ sudo iptables -L -n -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  886 86866 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    3   164 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT 109 packets, 33678 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1281 packets, 129K bytes)
 pkts bytes target     prot opt in     out     source               destination         
[ec2-user@ip-10-1-2-109 ~]$ sudo tcpdump -i ens5 icmp or port 443
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens5, link-type EN10MB (Ethernet), snapshot length 262144 bytes
03:17:01.520783 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [S], seq 3080160875, win 26883, options [mss 8961,sackOK,TS val 2156544458 ecr 0,nop,wscale 7], length 0
03:17:01.520817 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [S], seq 3080160875, win 26883, options [mss 8961,sackOK,TS val 2156544458 ecr 0,nop,wscale 7], length 0
03:17:01.522165 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [S.], seq 3773052830, ack 3080160876, win 57596, options [mss 8240,sackOK,TS val 4063366624 ecr 2156544458,nop,wscale 8], length 0
03:17:01.522187 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [S.], seq 3773052830, ack 3080160876, win 57596, options [mss 8240,sackOK,TS val 4063366624 ecr 2156544458,nop,wscale 8], length 0
03:17:01.522400 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 1, win 211, options [nop,nop,TS val 2156544460 ecr 4063366624], length 0
03:17:01.522415 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 1, win 211, options [nop,nop,TS val 2156544460 ecr 4063366624], length 0
03:17:01.528243 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [P.], seq 1:258, ack 1, win 211, options [nop,nop,TS val 2156544466 ecr 4063366624], length 257
03:17:01.528270 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [P.], seq 1:258, ack 1, win 211, options [nop,nop,TS val 2156544466 ecr 4063366624], length 257
03:17:01.529346 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [.], ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 0
03:17:01.529363 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [.], ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 0
03:17:01.529430 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 1:97, ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 96
03:17:01.529430 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 97:5673, ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 5576
03:17:01.529446 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 1:97, ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 96
03:17:01.529450 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 97:5673, ack 258, win 224, options [nop,nop,TS val 4063366631 ecr 2156544466], length 5576
03:17:01.529656 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 97, win 211, options [nop,nop,TS val 2156544467 ecr 4063366631], length 0
03:17:01.529656 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 5673, win 350, options [nop,nop,TS val 2156544467 ecr 4063366631], length 0
03:17:01.529664 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 97, win 211, options [nop,nop,TS val 2156544467 ecr 4063366631], length 0
03:17:01.529666 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 5673, win 350, options [nop,nop,TS val 2156544467 ecr 4063366631], length 0
03:17:01.530455 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 5673:6011, ack 258, win 224, options [nop,nop,TS val 4063366632 ecr 2156544466], length 338
03:17:01.530465 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 5673:6011, ack 258, win 224, options [nop,nop,TS val 4063366632 ecr 2156544466], length 338
03:17:01.530521 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6011:6020, ack 258, win 224, options [nop,nop,TS val 4063366632 ecr 2156544466], length 9
03:17:01.530526 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6011:6020, ack 258, win 224, options [nop,nop,TS val 4063366632 ecr 2156544466], length 9
03:17:01.530658 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 6011, win 437, options [nop,nop,TS val 2156544468 ecr 4063366632], length 0
03:17:01.530658 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 6020, win 437, options [nop,nop,TS val 2156544468 ecr 4063366632], length 0
03:17:01.530665 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 6011, win 437, options [nop,nop,TS val 2156544468 ecr 4063366632], length 0
03:17:01.530667 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 6020, win 437, options [nop,nop,TS val 2156544468 ecr 4063366632], length 0
03:17:01.531191 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [P.], seq 258:384, ack 6020, win 437, options [nop,nop,TS val 2156544469 ecr 4063366632], length 126
03:17:01.531200 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [P.], seq 258:384, ack 6020, win 437, options [nop,nop,TS val 2156544469 ecr 4063366632], length 126
03:17:01.532458 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [.], ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 0
03:17:01.532469 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [.], ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 0
03:17:01.532776 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6020:6026, ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 6
03:17:01.532776 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6026:6071, ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 45
03:17:01.532785 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6020:6026, ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 6
03:17:01.532788 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6026:6071, ack 384, win 224, options [nop,nop,TS val 4063366634 ecr 2156544469], length 45
03:17:01.533189 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 6071, win 437, options [nop,nop,TS val 2156544471 ecr 4063366634], length 0
03:17:01.533198 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 6071, win 437, options [nop,nop,TS val 2156544471 ecr 4063366634], length 0
03:17:01.533935 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [P.], seq 384:1041, ack 6071, win 437, options [nop,nop,TS val 2156544471 ecr 4063366634], length 657
03:17:01.533947 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [P.], seq 384:1041, ack 6071, win 437, options [nop,nop,TS val 2156544471 ecr 4063366634], length 657
03:17:01.567726 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6071:6125, ack 1041, win 224, options [nop,nop,TS val 4063366670 ecr 2156544471], length 54
03:17:01.567752 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6071:6125, ack 1041, win 224, options [nop,nop,TS val 4063366670 ecr 2156544471], length 54
03:17:01.568358 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [P.], seq 1041:1088, ack 6125, win 437, options [nop,nop,TS val 2156544506 ecr 4063366670], length 47
03:17:01.568377 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [P.], seq 1041:1088, ack 6125, win 437, options [nop,nop,TS val 2156544506 ecr 4063366670], length 47
03:17:01.585851 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6125:6547, ack 1088, win 224, options [nop,nop,TS val 4063366688 ecr 2156544506], length 422
03:17:01.585876 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6125:6547, ack 1088, win 224, options [nop,nop,TS val 4063366688 ecr 2156544506], length 422
03:17:01.627587 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [.], ack 6547, win 525, options [nop,nop,TS val 2156544565 ecr 4063366688], length 0
03:17:01.627614 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [.], ack 6547, win 525, options [nop,nop,TS val 2156544565 ecr 4063366688], length 0
03:17:01.633159 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [F.], seq 1088, ack 6547, win 525, options [nop,nop,TS val 2156544570 ecr 4063366688], length 0
03:17:01.633185 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [F.], seq 1088, ack 6547, win 525, options [nop,nop,TS val 2156544570 ecr 4063366688], length 0
03:17:01.634245 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [P.], seq 6547:6578, ack 1089, win 224, options [nop,nop,TS val 4063366736 ecr 2156544570], length 31
03:17:01.634245 IP 16.15.207.38.https > ip-10-1-2-109.ec2.internal.59226: Flags [F.], seq 6578, ack 1089, win 224, options [nop,nop,TS val 4063366736 ecr 2156544570], length 0
03:17:01.634262 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [P.], seq 6547:6578, ack 1089, win 224, options [nop,nop,TS val 4063366736 ecr 2156544570], length 31
03:17:01.634267 IP 16.15.207.38.https > 10.1.1.170.59226: Flags [F.], seq 6578, ack 1089, win 224, options [nop,nop,TS val 4063366736 ecr 2156544570], length 0
03:17:01.634464 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [R], seq 3080161964, win 0, length 0
03:17:01.634464 IP 10.1.1.170.59226 > 16.15.207.38.https: Flags [R], seq 3080161964, win 0, length 0
03:17:01.634472 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [R], seq 3080161964, win 0, length 0
03:17:01.634474 IP ip-10-1-2-109.ec2.internal.59226 > 16.15.207.38.https: Flags [R], seq 3080161964, win 0, length 0
^C
56 packets captured
56 packets received by filter
0 packets dropped by kernel
[ec2-user@ip-10-1-2-109 ~]$ exit
logout
Connection to 44.203.199.23 closed.

~ on ☁️  (us-east-1) ➜  


+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Terminal 2:

~ on ☁️  (us-east-1) ➜  aws s3 ls s3://xfusion-nat-1441/
2026-03-19 03:16:36         18 xfusion-test.txt

~ on ☁️  (us-east-1) ➜  
