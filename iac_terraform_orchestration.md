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
7. Destroy the instnace using terraform destroy