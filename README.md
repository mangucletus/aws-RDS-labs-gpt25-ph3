# AWS RDS Labs - Documentation

[![AWS](https://img.shields.io/badge/AWS-RDS-orange.svg)](https://aws.amazon.com/rds/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue.svg)](https://www.mysql.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Latest-blue.svg)](https://www.postgresql.org/)


## Overview

This repository contains hands-on AWS RDS (Relational Database Service) labs designed to provide practical experience with database management, high availability, scaling, and disaster recovery in the AWS cloud environment. The labs progress from basic RDS instance creation to advanced topics like Multi-AZ deployments, read replicas, and point-in-time recovery.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Lab Structure](#lab-structure)
- [Technical Implementation](#technical-implementation)
- [Performance Metrics](#performance-metrics)
- [Cost Analysis](#cost-analysis)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Contributing](#contributing)

## Architecture Overview

The labs implement RDS architecture spanning multiple availability zones and regions:

```
┌─────────────────────────────────────────────────────────────────┐
│                           AWS Cloud                             │
│  ┌─────────────────────┐         ┌─────────────────────────────┐ │
│  │     us-east-1       │         │       us-west-2            │ │
│  │                     │         │                             │ │
│  │  ┌───────────────┐  │         │  ┌───────────────────────┐  │ │
│  │  │  Primary AZ   │  │         │  │   Cross-Region        │  │ │
│  │  │               │  │         │  │   Read Replica        │  │ │
│  │  │ ┌───────────┐ │  │         │  │                       │  │ │
│  │  │ │ MySQL     │ │  │         │  │ ┌───────────────────┐ │  │ │
│  │  │ │ Master    │ │  │ ┌─────┐ │  │ │ MySQL Replica     │ │  │ │
│  │  │ └───────────┘ │  │ │Async│ │  │ └───────────────────┘ │  │ │
│  │  └───────────────┘  │ │Repl.│ │  └───────────────────────┘  │ │
│  │                     │ └─────┘ │                             │ │
│  │  ┌───────────────┐  │         │                             │ │
│  │  │ Secondary AZ  │  │         └─────────────────────────────┘ │
│  │  │               │  │                                         │ │
│  │  │ ┌───────────┐ │  │                                         │ │
│  │  │ │ MySQL     │ │  │                                         │ │
│  │  │ │ Standby   │ │  │                                         │ │
│  │  │ └───────────┘ │  │                                         │ │
│  │  └───────────────┘  │                                         │ │
│  │                     │                                         │ │
│  │  ┌───────────────┐  │                                         │ │
│  │  │Local Read     │  │                                         │ │
│  │  │Replica        │  │                                         │ │
│  │  └───────────────┘  │                                         │ │
│  └─────────────────────┘                                         │
└─────────────────────────────────────────────────────────────────┘
```

## Prerequisites

### Technical Requirements

- **AWS Account**: Administrative permissions required
- **CLI Tools**: 
  - MySQL Client (`mysql` command-line tool)
  - PostgreSQL Client (`psql` command-line tool)
  - AWS CLI (optional but recommended)
- **Network Access**: Unrestricted internet access for database connections
- **SSH Client**: For potential EC2 connectivity in advanced scenarios

### Knowledge Prerequisites

- Basic understanding of relational databases
- Familiarity with SQL syntax and operations
- AWS console navigation experience
- Understanding of networking concepts (VPC, Security Groups)

### Development Environment Setup

```bash
# Install MySQL Client (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install mysql-client

# Install PostgreSQL Client
sudo apt-get install postgresql-client

# Verify installations
mysql --version
psql --version
```

## Lab Structure

### Lab 1: RDS Instance Creation and Basic Operations
**Duration**: 60 minutes | **Complexity**: Beginner

- **Objective**: Master RDS instance provisioning and basic database operations
- **Key Technologies**: MySQL 8.0.x, PostgreSQL, RDS Security Groups
- **Deliverables**: 
  - Functional MySQL and PostgreSQL instances
  - Database connectivity verification
  - Performance baseline establishment

### Lab 2: Multi-AZ Deployment and Failover Testing
**Duration**: 45 minutes | **Complexity**: Intermediate

- **Objective**: Implement high availability with automatic failover capabilities
- **Key Technologies**: Multi-AZ RDS, Automatic Failover, RTO/RPO optimization
- **Deliverables**:
  - Multi-AZ enabled database
  - Failover performance metrics
  - High availability validation

### Lab 3: Read Replicas and Performance Scaling
**Duration**: 50 minutes | **Complexity**: Intermediate-Advanced

- **Objective**: Implement horizontal scaling and geographic distribution
- **Key Technologies**: Read Replicas, Cross-Region Replication, Load Distribution
- **Deliverables**:
  - Local and cross-region read replicas
  - Replication lag analysis
  - Read scaling performance validation

### Lab 4: Automated Backups and Point-in-Time Recovery
**Duration**: 40 minutes | **Complexity**: Advanced

- **Objective**: Implement comprehensive backup and disaster recovery strategies
- **Key Technologies**: Automated Backups, Manual Snapshots, PITR
- **Deliverables**:
  - Automated backup configuration
  - Point-in-time recovery demonstration
  - Data loss prevention validation

## Technical Implementation

### Database Engine Specifications

#### MySQL Configuration
```sql
-- Engine Version: MySQL 8.0.x
-- Instance Class: db.t3.micro
-- Storage: 20GB GP2 SSD with autoscaling
-- Backup Retention: 7-14 days
-- Port: 3306

-- Performance Tuning Parameters
SET innodb_buffer_pool_size = '75%';  -- Adjust based on instance memory
SET max_connections = 151;           -- Default for t3.micro
SET innodb_log_file_size = 256M;     -- Transaction log optimization
```

#### PostgreSQL Configuration
```sql
-- Engine Version: PostgreSQL 13.x+
-- Instance Class: db.t3.micro  
-- Storage: 20GB GP2 SSD with autoscaling
-- Backup Retention: 7-14 days
-- Port: 5432

-- Performance Tuning Parameters
-- shared_buffers = '25%'           -- Memory allocation
-- effective_cache_size = '75%'     -- Available system memory
-- max_connections = 100            -- Connection limit for t3.micro
```

### Network Architecture

#### Security Group Configuration
```bash
# MySQL Security Group Rules
Type: MySQL/Aurora (3306)
Protocol: TCP
Source: Your IP Address/32
Description: MySQL access for lab testing

# PostgreSQL Security Group Rules  
Type: PostgreSQL (5432)
Protocol: TCP
Source: Your IP Address/32
Description: PostgreSQL access for lab testing
```

#### VPC Configuration
- **VPC**: Default VPC in us-east-1
- **Subnets**: Multi-AZ subnet group for high availability
- **Route Tables**: Default routing with internet gateway access
- **Public Access**: Enabled for lab purposes (NOT recommended for production)

### Replication Architecture

#### Asynchronous Replication Flow
```
Master Database (us-east-1a)
    │
    ├── Synchronous Replication ──► Standby (us-east-1b) [Multi-AZ]
    │
    ├── Asynchronous Replication ──► Read Replica (us-east-1c) [Local]
    │
    └── Asynchronous Replication ──► Read Replica (us-west-2a) [Cross-Region]
```

## Performance Metrics

### Measured Performance Indicators

| Metric | MySQL Instance | PostgreSQL Instance | Multi-AZ Impact |
|--------|----------------|---------------------|-----------------|
| **Instance Provisioning Time** | 12-15 minutes | 14-16 minutes | +3-5 minutes |
| **Connection Establishment** | <1 second | <1 second | <1 second |
| **Query Response Time** | <10ms (simple queries) | <10ms (simple queries) | +2-5ms |
| **Failover Duration** | N/A | N/A | 60-120 seconds |
| **Backup Duration** | 2-5 minutes | 2-5 minutes | Similar |

### Replication Performance

| Replication Type | Lag Time | Data Transfer Rate | Network Impact |
|------------------|----------|-------------------|----------------|
| **Multi-AZ (Synchronous)** | <1ms | Real-time | Intra-AZ |
| **Local Read Replica** | <1 second | ~10MB/s | Intra-region |
| **Cross-Region Replica** | 2-5 seconds | ~5MB/s | Inter-region |

### Backup and Recovery Metrics

| Operation | Duration | Storage Impact | RTO/RPO |
|-----------|----------|----------------|---------|
| **Manual Snapshot** | 3-8 minutes | Full database size | RPO: Snapshot time |
| **Automated Backup** | Background process | Incremental after first backup | RPO: <5 minutes |
| **Point-in-Time Recovery** | 15-25 minutes | New instance creation | RTO: 20-30 minutes |

## Cost Analysis

### Hourly Cost Breakdown (us-east-1)

| Resource Type | Instance Class | Hourly Cost | Monthly Cost (730h) |
|---------------|----------------|-------------|---------------------|
| **Single-AZ MySQL** | db.t3.micro | $0.017 | $12.41 |
| **Multi-AZ MySQL** | db.t3.micro | $0.034 | $24.82 |
| **Read Replica (Local)** | db.t3.micro | $0.017 | $12.41 |
| **Cross-Region Replica** | db.t3.micro | $0.020* | $14.60* |
| **Backup Storage** | Per GB | $0.095 | Variable |

*\*Includes data transfer charges*

### Cost Optimization Strategies

1. **Right-sizing**: Start with db.t3.micro, monitor CloudWatch metrics
2. **Reserved Instances**: 1-year term saves ~40% for production workloads  
3. **Backup Optimization**: Tune retention periods based on business requirements
4. **Read Replica Placement**: Consider local replicas before cross-region for cost

### Lab Total Cost Estimation
- **Complete 4-lab series**: ~$3-5 (if completed within 6-8 hours)
- **Extended experimentation**: ~$10-15 per day
- **Production equivalent**: $50-100+ per month

## Security Considerations

### Network Security

```hcl
# Security Group - MySQL
resource "aws_security_group_rule" "mysql_access" {
  type              = "ingress"
  from_port         = 3306
  to_port           = 3306
  protocol          = "tcp"
  cidr_blocks       = ["YOUR_IP/32"]  # Restrict to specific IP
  security_group_id = aws_security_group.rds_mysql.id
}

# Security Group - PostgreSQL  
resource "aws_security_group_rule" "postgres_access" {
  type              = "ingress"
  from_port         = 5432
  to_port           = 5432
  protocol          = "tcp"
  cidr_blocks       = ["YOUR_IP/32"]  # Restrict to specific IP
  security_group_id = aws_security_group.rds_postgres.id
}
```

### Database Security Best Practices

1. **Authentication**: 
   - Use strong passwords (minimum 12 characters)
   - Consider IAM database authentication for production
   - Enable SSL/TLS encryption in transit

2. **Authorization**:
   - Implement principle of least privilege
   - Use database-level user management
   - Regular access reviews and cleanup

3. **Encryption**:
   - Enable encryption at rest for sensitive data
   - Use AWS KMS for key management
   - Implement application-level encryption for PII

4. **Monitoring**:
   - Enable VPC Flow Logs
   - Configure CloudTrail for API auditing
   - Use Performance Insights for query monitoring

### Production Hardening Checklist

- [ ] Disable public accessibility
- [ ] Enable encryption at rest
- [ ] Configure VPC endpoints
- [ ] Implement network ACLs
- [ ] Enable detailed monitoring
- [ ] Configure parameter groups with security settings
- [ ] Implement automated patch management
- [ ] Configure backup encryption
- [ ] Enable deletion protection
- [ ] Implement database activity streaming

## Troubleshooting

### Common Issues and Resolutions

#### Connection Issues
```bash
# Issue: "Connection refused" error
# Cause: Security group misconfiguration
# Resolution:
aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 3306 \
  --cidr YOUR_IP/32
```

#### Performance Issues
```sql
-- Issue: Slow query performance
-- Cause: Missing indexes or suboptimal queries
-- Resolution: Analyze query execution plans
EXPLAIN SELECT * FROM your_table WHERE condition;

-- PostgreSQL equivalent
EXPLAIN ANALYZE SELECT * FROM your_table WHERE condition;
```

#### Replication Lag
```sql
-- Check MySQL replication status
SHOW SLAVE STATUS\G

-- Check replication lag in seconds
SELECT SECONDS_BEHIND_MASTER FROM information_schema.REPLICA_HOST_STATUS;
```

#### Backup and Recovery Issues
```bash
# Issue: Point-in-time recovery fails
# Cause: Insufficient backup retention period
# Resolution: Check backup retention settings

aws rds describe-db-instances \
  --db-instance-identifier your-instance-id \
  --query 'DBInstances[0].BackupRetentionPeriod'
```

### Debugging Commands

```bash
# AWS CLI debugging commands
aws rds describe-db-instances --output table
aws rds describe-db-snapshots --output table  
aws rds describe-events --source-type db-instance
aws logs describe-log-groups --log-group-name-prefix '/aws/rds'

# Database connection testing
mysqladmin -h endpoint -P 3306 -u username -p ping
pg_isready -h endpoint -p 5432 -U username
```

## Best Practices

### Development Best Practices

1. **Environment Management**:
   - Use consistent naming conventions
   - Implement infrastructure as code (Terraform/CloudFormation)
   - Separate development, staging, and production environments

2. **Configuration Management**:
   - Use parameter groups for consistent configuration
   - Version control all database schema changes
   - Implement automated deployment pipelines

3. **Monitoring and Alerting**:
   - Set up CloudWatch alarms for key metrics
   - Implement log aggregation and analysis
   - Configure notification channels for critical events

### Production Deployment Guidelines

1. **High Availability Design**:
   - Always use Multi-AZ for production databases
   - Implement read replicas for read-heavy workloads
   - Plan for disaster recovery with cross-region backups

2. **Performance Optimization**:
   - Right-size instances based on actual usage patterns
   - Implement connection pooling
   - Optimize queries and indexes regularly

3. **Security Framework**:
   - Implement defense in depth
   - Regular security assessments and penetration testing
   - Maintain audit trails and compliance documentation

4. **Operational Excellence**:
   - Automate routine maintenance tasks
   - Implement chaos engineering practices
   - Maintain comprehensive documentation



## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

### Development Setup

1. **Clone the repository**
```bash
git clone https://github.com/mangucletus/aws-rds-labs-gtp25-ph3.git
cd aws-rds-labs
```

2. **Set up AWS credentials**
```bash
aws configure
# Enter your AWS credentials and region
```

3. **Validate your setup**
```bash
aws sts get-caller-identity
```


**⚠️ Important Note**: These labs are designed for learning purposes. Always follow security best practices and clean up resources after completion to avoid unexpected charges.

