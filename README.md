# Production-Ready-Highly-Available-Web-Application-on-AWS

project:
  name: free-tier-production-architecture
  description: >
    A cost-optimized AWS production-style architecture built using
    Free Tier–eligible services to demonstrate real-world DevOps concepts
    such as load balancing, auto scaling, monitoring, and security.
  level: beginner-to-intermediate
  focus:
    - production mindset
    - cost awareness
    - high availability
    - security best practices

architecture:
  flow:
    - Internet
    - Application Load Balancer (Public Subnets)
    - Auto Scaling Group (Public Subnets)
    - EC2 Instances (Amazon Linux 2)
  availability_zones: 2
  private_subnets: false
  nat_gateway: false
  reason:
    - NAT Gateway incurs cost
    - Public subnets are acceptable for learning and small workloads

aws_resources:
  vpc:
    name: prod-free-vpc
    cidr: 10.0.0.0/16
    dns_hostnames: enabled

  subnets:
    - name: public-subnet-az1
      cidr: 10.0.1.0/24
      availability_zone: az-1
      public_ip: auto-assign
    - name: public-subnet-az2
      cidr: 10.0.2.0/24
      availability_zone: az-2
      public_ip: auto-assign

  internet_gateway:
    attached: true
    route:
      destination: 0.0.0.0/0

  security_groups:
    alb_sg:
      inbound:
        - protocol: http
          port: 80
          source: 0.0.0.0/0
    ec2_sg:
      inbound:
        - protocol: http
          port: 80
          source: alb_sg
        - protocol: ssh
          port: 22
          source: my_ip_only
      note: SSH is never open to the world

compute:
  launch_template:
    ami: amazon-linux-2
    instance_type: t2.micro
    free_tier: true
    user_data:
      steps:
        - yum update -y
        - yum install httpd -y
        - start and enable httpd
        - deploy sample index.html with hostname

  auto_scaling_group:
    min_size: 1
    desired_capacity: 1
    max_size: 2
    scaling_policy:
      type: target_tracking
      metric: cpu_utilization
      target: 60_percent

load_balancing:
  type: application_load_balancer
  scheme: internet-facing
  subnets:
    - public-subnet-az1
    - public-subnet-az2
  target_group:
    type: instance
    port: 80
    health_check_path: /

monitoring:
  cloudwatch:
    enabled: true
    alarms:
      - metric: cpu_utilization
        threshold: 70_percent
        action: sns_email_notification
  cost_safe: true

security_best_practices:
  - restrict_ssh_to_single_ip
  - use_iam_roles_not_access_keys
  - alb_to_ec2_traffic_only
  - disable_password_login
  - health_checks_enabled

cost_considerations:
  free_tier_coverage:
    - 750_hours_t2_micro
    - basic_cloudwatch
    - limited_bandwidth
  additional_costs:
    - application_load_balancer_hourly_charge
    - scaling_beyond_free_tier
  recommendation: delete_resources_after_practice

learning_outcomes:
  - understanding_vpc_and_networking
  - designing_for_high_availability
  - implementing_auto_scaling
  - applying_real_security_controls
  - cost_optimized_architecture_design

usage_note: >
  This architecture is intended for learning and small workloads.
  It is not a replacement for enterprise-grade private subnet designs,
  but it accurately reflects real production logic under budget constraints.
