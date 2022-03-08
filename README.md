# Move State

This repo is based on the tutorial of HashiCorp Learning


https://learn.hashicorp.com/tutorials/terraform/move-config#move-your-resources-with-the-moved-configuration-block

You will create some resources without modules. After that will move some resources into modules. You will not use the ```terraform state mv``` command but the new block ```moved``` with terraform >1.1.0

# How to

- git clone
```
git clone https://github.com/munnep/move_state_part2.git
```
- go in directory
```
cd move_state_part2
```
- terraform init
```
terraform init
```
- terraform plan
```
terraform plan
```
- terraform apply
```
terraform apply
```
- output
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```
- make a directory modules
```
mkdir modules
```
- add the file ```modules/main.tf``` with the content
```
resource "aws_vpc" "my_vpc" {
  cidr_block = "172.16.0.0/16"

  tags = {
    Name = "tf-example"
  }
}

resource "aws_subnet" "my_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "172.16.10.0/24"
  availability_zone = "eu-west-2a"

  tags = {
    Name = "tf-example"
  }
}
```
- add the file ```modules/output.tf``` with the content
```
output "subnet_id" {
  description = "subnet id"
  value       = aws_subnet.my_subnet.id
}
```
- remove the following from ```main.tf```
```
resource "aws_vpc" "my_vpc" {
  cidr_block = "172.16.0.0/16"

  tags = {
    Name = "tf-example"
  }
}

resource "aws_subnet" "my_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "172.16.10.0/24"
  availability_zone = "eu-west-2a"

  tags = {
    Name = "tf-example"
  }
}
```
- add the following
```
module "vpc" {
  source = "./modules"
}
```
- terraform plan will now want to recreate everything
```

Plan: 4 to add, 0 to change, 4 to destroy.
```
- Add the following move block in the ```main.tf```
```
moved {
  from = aws_vpc.my_vpc
  to = module.vpc.aws_vpc.my_vpc
}

moved {
  from = aws_subnet.my_subnet
  to = module.vpc.aws_subnet.my_subnet
}

```
- terraform apply
```
Terraform will perform the following actions:

  # aws_subnet.my_subnet has moved to module.vpc.aws_subnet.my_subnet
    resource "aws_subnet" "my_subnet" {
        id                                             = "subnet-025845ec6749ea849"
        tags                                           = {
            "Name" = "tf-example"
        }
        # (15 unchanged attributes hidden)
    }

  # aws_vpc.my_vpc has moved to module.vpc.aws_vpc.my_vpc
    resource "aws_vpc" "my_vpc" {
        id                               = "vpc-09e706ec399993038"
        tags                             = {
            "Name" = "tf-example"
        }
        # (16 unchanged attributes hidden)
    }

Plan: 0 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```
- terraform plan
```
terraform plan
```
```
Plan: 0 to add, 0 to change, 0 to destroy.
```