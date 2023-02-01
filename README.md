# sudhir_TERRAFORM
#EC2 Creation on AWS 

provider "aws" {
  region = "us-east-1"
}

#vpc####3

resource "aws_vpc" "myvpc" {
  cidr_block = "10.0.0.0/16"
}

###igw#####

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.myvpc.id
  tags = {
    Name = "igw"
  }
}

####subnet####

resource "aws_subnet" "mysubnet" {
  vpc_id     = aws_vpc.myvpc.id
  cidr_block = "10.0.0.0/24"

  tags = {
    Name = "subnet"
  }
}

###R table#####

resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.myvpc.id

  route = []

  tags = {
    Name = "example"
  }
}

###Route#####
resource "aws_route" "route" {
  route_table_id         = aws_route_table.rt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.igw.id
  depends_on             = [aws_route_table.rt]
}

#sg####

resource "aws_security_group" "sg" {
  name        = "allow_all_traffic"
  description = "Allow all inbound traffic"
  vpc_id      = aws_vpc.myvpc.id

  ingress {
    description      = "all traffic"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = null
    prefix_list_ids  = null
    security_groups  = null
    self             = null
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
    description      = "outbound_rules"
    prefix_list_ids  = null
    security_groups  = null
    self             = null
  }


  tags = {
    Name = "all_traffic"
  }
}

####R Table association###

resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.mysubnet.id
  route_table_id = aws_route_table.rt.id
}

####EC2####

resource "aws_instance" "EC2" {
  ami           = "ami-0fe472d8a85bc7b0e"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.mysubnet.id

  tags = {
    Name = "HelloWorld"
  }
}

