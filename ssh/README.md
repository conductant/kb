SSH Tunnel / Proxy
==================

This is useful for connecting to private network in a VPC over a bastion server.  Inside a private network in a VPC, servers
do not have public IP addresses assigned to them.  Therefore, they are reachable only via a bastion server that sits in the
DMZ of the VPC.

https://www.bartbusschots.ie/s/2005/12/07/ssh-via-a-socks-proxy-on-os-x-with-connectc/

Steps
-----

1. Have your PEM file ready.  This can be the PEM file downloaded from AWS (the public/private key pair) and used when starting instances.
Save this file:
    
    + `mv dev.pem ~/.ssh/id_aws_conductant_com_dev.pem`
    + `chmod 600 !$`
   
2. Compile connect.c:
```
gcc connect.c -o connect -lresolv
sudo cp connect /usr/local/bin
```

3. Edit your `~/.ssh/config` to add sections like:

```
Host *
ServerAliveInterval 120
TCPKeepAlive no 

#########################################################
# Start a local proxy at port 7784 for aws.conductant.com
Host aws.conductant.com
Hostname bastion.conductant.io
User ubuntu
IdentityFile ~/.ssh/id_aws_conductant_com_dev.pem
DynamicForward 7784
#-----------------------------------
Host 10.100.*
User ubuntu
IdentityFile ~/.ssh/id_aws_conductant_com_dev.pem
ProxyCommand connect -S localhost:7784 %h %p
########################################################
```

4. Connect: Simply do this 
```
ssh ssh -L 11080:10.100.113.11:8080 aws.conductant.com
```
This will, for example, do local port forwarding of 11080 to port 8088 on `10.100.113.11` inside the private network.

Logging into hosts in the private subnet is now simplified, too.  For example, logging into host `10.100.113.11` using the same PEM file is simply

```
ssh 10.100.113.11
```

