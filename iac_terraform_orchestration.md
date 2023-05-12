# IaC terraform orchestration

# Installing terraform
1. Open up powershell from your windows and run this code inside it
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
2. Then run ```choco install terraform``` in the same place
3. Open a Git Bash terminal and run ```terraform --version``` to check it installed

## Creating environment variabls in windows
1. Press the windows button and type in environment variables and open that
2. Select environment variables and click new
3. Add a new variable called export AWS_ACCESS_KEY_ID and add the access key from the excel file in the .ssh fodler that we were sent a while ago
4. Do the same for export AWS_SECRET_ACCESS_KEY
5. Save and close
6. Then open powershell again and run ```refreshenv```

## Launching an instance
1. Open a Git Bash terminal and create a file using ```nano main.tf```
2. Enter in the following code
```
# Terrafrom script to create a service on the cloud
# Let's set up our cloud provider with Terraform

# Who is the provider - AWS
# How to codify with Terraform - syntax - name of resource/task{key = value}
# Most commonly used commands - terraform init - terraform plan - terraform apply - terraform destroy

provider "aws" {
        region = "eu-west-1"
}

# Let's create a service on AWS
# Which service - EC2

resource "aws_instance" "app_instance" {
        # Which AMI to use
        ami = " ami-0c5a2f4a8680af2d2"
        # Type of instance
        instance_type = "t2.micro"
        # Do you need the public IP
        associate_public_ip_address = true
        # What would you like to name it
        tags = {
            Name = "tech221_billy_terraform_app"
        }


}
```
3. Run ```terraform init``` to make sure it was successful
4. Then run ```terraform plan``` to preview the actions that terraform will take
5. Next run ```terraform apply``` to run the file
6. This will launch an EC2 instance on AWS
7. Destroy the instnace using ```terraform destroy```

![VPC](https://github.com/BillyCooke/tech221_iac_terraform_orchestration/assets/129949090/e91a2b03-5767-4d7d-a703-7ff9b68e324f)

# Creating a VPC through Terraform
1. Use ```nano main.tf``` to bring up your file and remove everything apart from the region section
2. Next we need to add in the steps to create the VPC
3. First we need to create a VPC, then an internet gateway, then a public subnet and lastly a route table.
4. They all need to be in the same main.tf file but I have split them into each section

## VPC

```
resource "aws_vpc" "my_vpc" {
  cidr_block = 
  instance_tenancy = "default"
  tags = {
    Name = "tech221_billy_vpc_iac"
  }
}
```
## Internet gateway

```
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "tech221_billy_ig_iac"
  }
}
```
## Public and private subnets

```
resource "aws_subnet" "tech221_billy_publicsubnet_iac" {
	vpc_id = aws_vpc.my_vpc.id
	cidr_block = "10.0.2.0/24"
	map_public_ip_on_launch = "true"
	availability_zone = "eu-west-1"

	tags = {
		Name = "tech221_billy_publicsubnet_iac"
	}

}

resource "aws_subnet" "tech221_billy_privatesubnet_iac" {
	vpc_id = aws_vpc.my_vpc.id
	cidr_block = 
	availability_zone = "eu-west-1"

	tags = {
		Name = "tech221_billy_privatesubnet_iac"
	}

}
```

## Public and private route tables
```
resource "aws_route_table" "tech221_billy_publicRT" {
  vpc_id = aws_vpc.my_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.tech221_billy_ig_iac.id
  }

  tags = {
    Name = "billy_publicRT"
  }
}

resource "aws_route_table" "tech221_billy_privateRT" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "billy_privateRT"
  }
}
```
## Security group
```
resource "aws_security_group" "tech221_billy_sg_iac" {
  name = "tech221_billy_sg_iac"
  description = "app security group
  vpc_id = aws_vpc.my_vpc.id

  ingress {
    description = "HTTP"
    from_port = 80
    to_port = 80
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
  ingress {
    description = "port3000"
    from_port = 3000
    to_port = 3000
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
    egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"] # add your IP
    ipv6_cidr_blocks = ["::/0"]
    }

    tags = {
      Name = "tech221_billy_sg_iac"

    }
}
```
## Launch the app

resource "aws_instance" "app_instance"{
	# which ami to use
	ami = ami-0a05bfcea61ce988f
	
	#type of instance
	instance_type = "t2.micro"

	# do you need the public IP
	associate_public_ip_address = true

	# Which subnet

	subnet_id = "${aws_subnet.tech221_billy_publicsubnet_iac.id}"

        # Add security groups

	security_groups = ["${aws_security_group.tech221_billy_sg_iac.id}"]

	# what would you like to name it
	tags = {
	
	  Name = "tech221_billy_app_terraform"

	} 

}
