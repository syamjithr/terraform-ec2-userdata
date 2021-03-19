# Terraform-Ec2-with userdata
## Provisioning an ec2 instance with userdata using Terraform
### Prerequisite:
Download Terraform from link and set up it in a linux machine. ( Here I'm using a ec2 instance) Configure aws cli in the instance.
### First we need to create a directory 
##### mkdir ec2-userdata
##### cd ec2-userdata/
##### terraform init
##### vim ec2-infra.tf
```
##############################################################################
# Uploading Keypair
##############################################################################
resource "aws_key_pair" "mykey" {

  key_name   = "mykey"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5jd47hesPgbpuNi/3kuMo0ux6hmZ5K/77fpJH7LqHeexB1aG7eexf20v0wW7P5rtMsn9N9AdkXzmcy4BzdYydv1DFQX3CSbz5tJugf4Jj0i+W8wEjX9g7n7eC1DMSEbPoEfNnKg3t9fGqe6h74mDAT+FnM6++CNMzs4Qmel1YnCvxKlXlMDYOiC1OlJsMwAZC1y28AB2t8MpKK/5/5WoqQe1EWjXlg72O7hmFZgqr/gLb58TBfhXuKMQTVhaAC7F4xPGg8KlNwcIRLbWIllswzDSNdnpQUgb1NjPeEgQrx8+Zu6CORxf1hTrGDj1iAmYe2oxumycI8mfbeMbToWm5 root@ip-172-31-2-202.ap-south-1.compute.internal"
  tags = {
      Name = "mykey"
  }
    
}

##############################################################################
# Security Group webserver
##############################################################################

resource "aws_security_group" "webserver" {
    
  name        = "webserver-firewall"
  description = "allows 22,80,443 only"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
 
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
    
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "webserver-firewall"  
  }
}


##############################################################################
# Security Group database
##############################################################################

resource "aws_security_group" "database" {
    
  name        = "database-firewall"
  description = "allows 22,3306 only"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
 
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "database-firewall"  
  }
}

##############################################################################
# Ec2 Instance
##############################################################################

resource "aws_instance" "webeserver" {
    
  ami                         = "ami-0eeb03e72075b9bcc"
  instance_type               = "t2.small"
  associate_public_ip_address = true
  key_name                    = aws_key_pair.mykey.key_name
  vpc_security_group_ids      = [ aws_security_group.webserver.id , aws_security_group.database.id ]
  availability_zone           = "ap-south-1a"
  user_data                   = file("setup.sh")
  tags = {
    Name = "webserver"
  }
} 
```
##### userdata-script
vim setup.sh
```
#!/bin/bash

echo "password123" | passwd root --stdin
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd restart


yum install httpd  php -y
service httpd restart
chkconfig httpd on

cat <<EOF > /var/www/html/index.php
<?php
\$output = shell_exec('echo $HOSTNAME');
echo "<h1><center><pre>\$output</pre></center></h1>";
echo "<h1><center>Version2</center></h1>"
?>
EOF
```
##### Execution
```
#terraform validate   - syntax check 

#terraform plan - Creating an execution plan ( to check what will get installed before running it)

# terraform apply - Applying

# terraform destroy - Destroying what we have applied through terrafrom apply

We can also use -auto-approve while applying.

# terraform apply -auto-approve
```
