

# Hogwarts Wizarding School Access Management

*Following document is a hands-on example of provisioning AWS services through Terraform with a twist of magic:*

We start with five different activities which require significant management from Hogwarts staff and students alike:

* The Vault of Forgotten Wonders (S3) stores all the enchanted artifacts and the mysterious magical beasts.
* Quidditch Pitch (EC2) is where the thrilling Quidditch League takes place.
* Hogwarts Restricted Network (VPC) handles all the communication to and outside our magical world.
* Magical Barriers (Security Groups) controls access to the Quidditch Pitch for the biggest Quidditch Tournaments.

We have around 3 groups, comprising 9 students and staff members willing to help out this year with the above activities:
![Hogwarts_IAM](https://github.com/manav-dl/terraform-enginerd/assets/122433722/25449db7-1a7b-4004-990e-92ebb53b91d3)


### Access Table:
Remember folks, we always need to follow the principle of least priviledge afterall, we don't want what happened with our previous Defense Against Dark Arts professor to happen again, do we?
![access_table](https://github.com/manav-dl/terraform-enginerd/assets/122433722/afe756db-3cb8-4c3a-8f37-2a0a0a58dbf8)

-

```
# At first we need to specify the provider block:
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  access_key = "Your_Access_Key"
  secret_key = "Your_secret_key"
}

# Setting up a S3 Bucket
resource "aws_s3_bucket" "the_vault_of_forgotten_wonders" {
    bucket = "the-vault-of-forgotten-wonders"

    tags = {
        Name = "The Vault of Forgotten Wonders"
    }
}


# Setting up a VPC
resource "aws_vpc" "hogwarts_restricted_network" {
    cidr_block = "10.0.0.0/16"
  
    tags = {
    Name = "Hogwarts Restricted Network"
  }
}

# Inside the VPC, we'll create a subnet
resource "aws_subnet" "hogwarts_defense_room" {
  vpc_id     = aws_vpc.hogwarts_restricted_network.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "Hogwarts Defense Room"
  }
}


# Create a Security Group
resource "aws_security_group" "magical_barriers" {
  name        = "Magical Barriers"
  description = "Security group for Magical Barriers"
  vpc_id      = aws_vpc.hogwarts_restricted_network.id

  # Allowing inbound SSH traffic
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allowing outbound traffic
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    cidr_blocks     = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Magical Barriers"
  }
}


# Creating an EC2 instance
resource "aws_instance" "quidditch_pitch" {
  ami                    = "ami-0c101f26f147fa7fd"
  instance_type          = "t2.micro"
  key_name               = "your_key"
  
  subnet_id              = aws_subnet.hogwarts_defense_room.id
  vpc_security_group_ids = [aws_security_group.magical_barriers.id]

  tags = {
    Name = "Quidditch Pitch"
  }
}


#Creating Users

resource "aws_iam_user" "rubeus_hagrid" {
  name = "RubeusHagrid"
}

resource "aws_iam_user" "hermione_granger" {
  name = "HermioneGranger"
}

resource "aws_iam_user" "luna_lovegood" {
  name = "LunaLovegood"
}

resource "aws_iam_user" "oliver_wood" {
  name = "OliverWood"
}

resource "aws_iam_user" "fred_weasley" {
  name = "FredWeasley"
}

resource "aws_iam_user" "george_weasley" {
  name = "GeorgeWeasley"
}

resource "aws_iam_user" "minerva_mcgonagall" {
  name = "MinervaMcGonagall"
}

resource "aws_iam_user" "severus_snape" {
  name = "SeverusSnape"
}

resource "aws_iam_user" "alastor_moody" {
  name = "AlastorMoody"
}


# Creating Groups

resource "aws_iam_group" "enchanted_artifacts_and_creatures_management" {
  name = "EnchantedArtifactsAndCreaturesManagement"
}

resource "aws_iam_group" "quidditch_management" {
  name = "QuidditchManagement"
}

resource "aws_iam_group" "defense_services" {
  name = "DefenseServices"
}


# Adding users to Enchanted Artifacts and Creatures Management Group
resource "aws_iam_group_membership" "enchanted_artifacts_group_membership" {
  name  = "enchanted-artifacts-group-membership"
  group = aws_iam_group.enchanted_artifacts_and_creatures_management.name
  users = [
    aws_iam_user.rubeus_hagrid.name,
    aws_iam_user.hermione_granger.name,
    aws_iam_user.luna_lovegood.name
  ]
}

# Adding users to Quidditch Management Group
resource "aws_iam_group_membership" "quidditch_management_group_membership" {
  name  = "quidditch-management-group-membership"
  group = aws_iam_group.quidditch_management.name
  users = [
    aws_iam_user.oliver_wood.name,
    aws_iam_user.fred_weasley.name,
    aws_iam_user.george_weasley.name
  ]
}

# Adding users to Defense Services Group
resource "aws_iam_group_membership" "defense_services_group_membership" {
  name  = "defense-services-group-membership"
  group = aws_iam_group.defense_services.name
  users = [
    aws_iam_user.minerva_mcgonagall.name,
    aws_iam_user.severus_snape.name,
    aws_iam_user.alastor_moody.name
  ]
}


# Policy for Enchanted Artifacts and Creatures Management Group (S3 access)
data "aws_iam_policy_document" "enchanted_artifacts_policy" {
  statement {
    actions = [
      "s3:ListBucket",
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject"
    ]
    resources = [
      aws_s3_bucket.the_vault_of_forgotten_wonders.arn,
      "${aws_s3_bucket.the_vault_of_forgotten_wonders.arn}/*"
    ]
  }
}

resource "aws_iam_policy" "enchanted_artifacts_policy" {
  name        = "EnchantedArtifactsPolicy"
  description = "Grants access to The Vault of Forgotten Wonders"
  policy      = data.aws_iam_policy_document.enchanted_artifacts_policy.json
}

# Policy for Quidditch Management Group (EC2 access)
data "aws_iam_policy_document" "quidditch_management_policy" {
  statement {
    actions = [
      "ec2:DescribeInstances",
      "ec2:StartInstances",
      "ec2:StopInstances"
    ]
    resources = [aws_instance.quidditch_pitch.arn]
  }
}

resource "aws_iam_policy" "quidditch_management_policy" {
  name        = "QuidditchManagementPolicy"
  description = "Grants access to Quidditch Pitch"
  policy      = data.aws_iam_policy_document.quidditch_management_policy.json
}

# Policy for Defense Services Group (VPC, SG, limited S3 and EC2 access)
data "aws_iam_policy_document" "defense_services_policy" {
  statement {
    actions = [
      "ec2:DescribeVpcs",
      "ec2:DescribeSubnets",
      "ec2:DescribeRouteTables",
      "ec2:DescribeSecurityGroups"
    ]
    resources = ["*"]
  }

  statement {
    actions = [
      "s3:ListBucket",
      "s3:GetObject"
    ]
    resources = [
      aws_s3_bucket.the_vault_of_forgotten_wonders.arn,
      "${aws_s3_bucket.the_vault_of_forgotten_wonders.arn}/*"
    ]
  }

  statement {
    actions = [
      "ec2:DescribeInstances"
    ]
    resources = [aws_instance.quidditch_pitch.arn]
  }
}

resource "aws_iam_policy" "defense_services_policy" {
  name        = "DefenseServicesPolicy"
  description = "Grants access to VPC, SG, limited S3 and EC2 access"
  policy      = data.aws_iam_policy_document.defense_services_policy.json
}


# Attach policy to Enchanted Artifacts and Creatures Management Group
resource "aws_iam_group_policy_attachment" "enchanted_artifacts_group_policy_attachment" {
  group      = aws_iam_group.enchanted_artifacts_and_creatures_management.name
  policy_arn = aws_iam_policy.enchanted_artifacts_policy.arn
}

# Attach policy to Quidditch Management Group
resource "aws_iam_group_policy_attachment" "quidditch_management_group_policy_attachment" {
  group      = aws_iam_group.quidditch_management.name
  policy_arn = aws_iam_policy.quidditch_management_policy.arn
}

# Attach policy to Defense Services Group
resource "aws_iam_group_policy_attachment" "defense_services_group_policy_attachment" {
  group      = aws_iam_group.defense_services.name
  policy_arn = aws_iam_policy.defense_services_policy.arn
}
```
