# Rock-Paper-Scissor 

Without rds

provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Create Subnets in different AZs
resource "aws_subnet" "subnet_1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"  # First AZ
  map_public_ip_on_launch = true
}

resource "aws_subnet" "subnet_2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"  # Second AZ
  map_public_ip_on_launch = true
}

# Create Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

# Create Route Table
resource "aws_route_table" "route_table" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

# Associate Route Table with Subnets
resource "aws_route_table_association" "subnet_association_1" {
  subnet_id      = aws_subnet.subnet_1.id
  route_table_id = aws_route_table.route_table.id
}

resource "aws_route_table_association" "subnet_association_2" {
  subnet_id      = aws_subnet.subnet_2.id
  route_table_id = aws_route_table.route_table.id
}

# Security Group
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create EC2 instance in first subnet
resource "aws_instance" "web_1" {
  ami           = "ami-08c40ec9ead489470"  # Ubuntu 22.04 in us-east-1
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet_1.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
#!/bin/bash
sudo apt update
sudo apt install -y nginx
cat <<EOT > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Rock Paper Scissors</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        h1 { color: #4CAF50; }
        button { font-size: 18px; padding: 10px 20px; margin: 10px; }
    </style>
</head>
<body>
    <h1>Rock Paper Scissors</h1>
    <p>Choose your move:</p>
    <button onclick="play('rock')">Rock</button>
    <button onclick="play('paper')">Paper</button>
    <button onclick="play('scissors')">Scissors</button>
    <p id="result"></p>
    <script>
        function play(userChoice) {
            const choices = ['rock', 'paper', 'scissors'];
            const computerChoice = choices[Math.floor(Math.random() * choices.length)];
            let result = '';
            if (userChoice === computerChoice) {
                result = 'It\'s a draw!';
            } else if (
                (userChoice === 'rock' && computerChoice === 'scissors') ||
                (userChoice === 'paper' && computerChoice === 'rock') ||
                (userChoice === 'scissors' && computerChoice === 'paper')
            ) {
                result = 'You win!';
            } else {
                result = 'You lose!';
            }
            document.getElementById('result').innerText = 'Computer chose ' + computerChoice + '. ' + result;
        }
    </script>
</body>
</html>
EOT
sudo systemctl restart nginx
EOF

  tags = {
    Name = "rock-paper-scissors-1"
  }
}

# Create EC2 instance in second subnet
resource "aws_instance" "web_2" {
  ami           = "ami-08c40ec9ead489470"  # Ubuntu 22.04 in us-east-1
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.subnet_2.id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
#!/bin/bash
sudo apt update
sudo apt install -y nginx
cat <<EOT > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Rock Paper Scissors</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        h1 { color: #4CAF50; }
        button { font-size: 18px; padding: 10px 20px; margin: 10px; }
    </style>
</head>
<body>
    <h1>Rock Paper Scissors</h1>
    <p>Choose your move:</p>
    <button onclick="play('rock')">Rock</button>
    <button onclick="play('paper')">Paper</button>
    <button onclick="play('scissors')">Scissors</button>
    <p id="result"></p>
    <script>
        function play(userChoice) {
            const choices = ['rock', 'paper', 'scissors'];
            const computerChoice = choices[Math.floor(Math.random() * choices.length)];
            let result = '';
            if (userChoice === computerChoice) {
                result = 'It\'s a draw!';
            } else if (
                (userChoice === 'rock' && computerChoice === 'scissors') ||
                (userChoice === 'paper' && computerChoice === 'rock') ||
                (userChoice === 'scissors' && computerChoice === 'paper')
            ) {
                result = 'You win!';
            } else {
                result = 'You lose!';
            }
            document.getElementById('result').innerText = 'Computer chose ' + computerChoice + '. ' + result;
        }
    </script>
</body>
</html>
EOT
sudo systemctl restart nginx
EOF

  tags = {
    Name = "rock-paper-scissors-2"
  }
}

# Output public IP addresses of both instances
output "public_ip_1" {
  value       = aws_instance.web_1.public_ip
  description = "Public IP of the first web server"
}

output "public_ip_2" {
  value       = aws_instance.web_2.public_ip
  description = "Public IP of the second web server"
}

