## Terraform Script to Launch Docker, Neo4j and Sagemaker in AWS
Here's a Terraform script that launches Docker containers for Hello World, OpenSearch, Neo4j, and sets up a SageMaker notebook instance in AWS:

```hcl
provider "aws" {
  region = "us-west-2"  # Change this to your preferred region
}

# ECS Cluster
resource "aws_ecs_cluster" "cluster" {
  name = "my-cluster"
}

# Hello World Task Definition
resource "aws_ecs_task_definition" "hello_world" {
  family                   = "hello-world"
  container_definitions    = jsonencode([
    {
      name  = "hello-world"
      image = "hello-world"
      portMappings = [
        {
          containerPort = 80
          hostPort      = 80
        }
      ]
    }
  ])
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = "256"
  memory                   = "512"
}

# OpenSearch Task Definition
resource "aws_ecs_task_definition" "opensearch" {
  family                   = "opensearch"
  container_definitions    = jsonencode([
    {
      name  = "opensearch"
      image = "opensearchproject/opensearch:latest"
      portMappings = [
        {
          containerPort = 9200
          hostPort      = 9200
        }
      ]
    }
  ])
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = "1024"
  memory                   = "2048"
}

# Neo4j Task Definition
resource "aws_ecs_task_definition" "neo4j" {
  family                   = "neo4j"
  container_definitions    = jsonencode([
    {
      name  = "neo4j"
      image = "neo4j:latest"
      portMappings = [
        {
          containerPort = 7474
          hostPort      = 7474
        },
        {
          containerPort = 7687
          hostPort      = 7687
        }
      ]
    }
  ])
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = "1024"
  memory                   = "2048"
}

# VPC for ECS Tasks
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "ecs-vpc"
  }
}

# Subnet for ECS Tasks
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "ecs-subnet"
  }
}

# Security Group for ECS Tasks
resource "aws_security_group" "ecs_tasks" {
  name        = "ecs-tasks-sg"
  description = "Allow inbound traffic for ECS tasks"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 0
    to_port     = 65535
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

# ECS Services
resource "aws_ecs_service" "hello_world" {
  name            = "hello-world-service"
  cluster         = aws_ecs_cluster.cluster.id
  task_definition = aws_ecs_task_definition.hello_world.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = [aws_subnet.main.id]
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = true
  }
}

resource "aws_ecs_service" "opensearch" {
  name            = "opensearch-service"
  cluster         = aws_ecs_cluster.cluster.id
  task_definition = aws_ecs_task_definition.opensearch.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = [aws_subnet.main.id]
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = true
  }
}

resource "aws_ecs_service" "neo4j" {
  name            = "neo4j-service"
  cluster         = aws_ecs_cluster.cluster.id
  task_definition = aws_ecs_task_definition.neo4j.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = [aws_subnet.main.id]
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = true
  }
}

# SageMaker Notebook Instance
resource "aws_sagemaker_notebook_instance" "ni" {
  name          = "my-notebook-instance"
  role_arn      = aws_iam_role.sagemaker_role.arn
  instance_type = "ml.t2.medium"
}

# IAM Role for SageMaker
resource "aws_iam_role" "sagemaker_role" {
  name = "sagemaker-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "sagemaker.amazonaws.com"
        }
      }
    ]
  })
}

# Attach necessary policies to the SageMaker role
resource "aws_iam_role_policy_attachment" "sagemaker_full_access" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonSageMakerFullAccess"
  role       = aws_iam_role.sagemaker_role.name
}
```

This script does the following:

1. Sets up an ECS cluster.
2. Defines task definitions for Hello World, OpenSearch, and Neo4j containers.
3. Creates a VPC, subnet, and security group for the ECS tasks.
4. Sets up ECS services to run the containers.
5. Creates a SageMaker notebook instance with the necessary IAM role and policies.

To use this script:

1. Ensure you have Terraform installed and configured with your AWS credentials.
2. Save this script as `main.tf` in a new directory.
3. Run `terraform init` to initialize the Terraform working directory.
4. Run `terraform plan` to see what resources will be created.
5. Run `terraform apply` to create the resources in your AWS account.

Note that this is a basic setup and may need additional configuration for production use, such as proper networking, security hardening, and resource optimization. Also, running these resources may incur costs in your AWS account.

