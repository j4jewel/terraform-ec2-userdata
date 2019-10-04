# terraform-ec2-userdata
####  Provisioning an ec2 instance with userdata using Terraform

### Prerequisite:
 Download Terraform from [link](https://www.terraform.io/downloads.html) and set up it in a linux machine. ( Here I'm using a ec2 instance)  Configure aws cli in the instance. 
 
 ##### mkdir ec2-userdata
 ##### cd ec2-userdata/
 ##### terraform init
 ##### vim ec2.tf
 #
 ```sh
 provider "aws" {

   region = "us-east-1"
}

variable "image" {
    default = "ami-0b898040803850657"
    description = "Amazon-linux"
}


resource "aws_instance" "instance1" {

  ami = "${var.image}"
  instance_type = "t2.micro"
  key_name = "jwl-PC"
  availability_zone = "us-east-1b"
  user_data = "${file("httpd-setup.sh")}"
  associate_public_ip_address = true
  vpc_security_group_ids = ["${aws_security_group.web-sg.id}"]
  tags = {
    Name = "WebServer-TF"
  }
}

resource "aws_security_group" "web-sg" {

  name = "webserver-sg"

  tags = {
      Name = "Webserver-SG"
  }

  ingress {
     from_port = 22
     to_port = 22
     protocol = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
     from_port = 80
     to_port = 80
     protocol = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
     from_port = 443
     to_port = 443
     protocol = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

}
 ```
 ##### userdata-script
 
 vim httpd-setup.sh
 ```sh
 #!/bin/bash

yum install httpd -y
service httpd restart
chkconfig httpd on
echo "<h1><center>Deployed Using Userdata-Terraform</center></h1>" > /var/www/html/index.html
 ```
 ##### Execution
#
```sh
#terraform validate   - syntax check 

#terraform plan - Creating an execution plan ( to check what will get installed before running it)

# terraform apply - Applying

# terraform destroy - Destroying what we have applied through terrafrom apply

We can also use -auto-approve while applying.
# terraform apply -auto-approve

```
 
 
You can also change the userdata script according to your need.
If you want to spin-up jenkins docker container inside an Amazon-Linux-2 ec2 instance, you can use the following userdata script.( You have to change the securoty group rules to allow ports 8080 and 50000) 

Script: [link](https://github.com/FujiClado/jenkins-ec2/blob/master/install.sh)

```sh
#!/bin/bash

yum install git -y
git clone https://github.com/FujiClado/jenkins-ec2.git
cd jenkins-ec2
chmod +x install.sh
./install.sh
```

Ssh into the ec2 instance and get the hash value from the jenkins.txt.

## cat /home/ec2-user/jenkins.txt


 
