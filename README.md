# Host a Static Website on AWS with Cloudformation

## Introduction
This project provisions a robust and scalable AWS infrastructure using AWS CloudFormation. The architecture supports a web application with high availability, security, and automated scalability.

## Infrastructure Components

### 1. **Virtual Private Cloud (VPC)**
- Custom VPC with both public and private subnets across two availability zones for high availability.

### 2. **Networking Components**
- **Internet Gateway:** Allows public internet access for resources in the public subnets.
- **NAT Gateway:** Enables resources in the private subnets to access the internet securely.
- **Route Tables:**
  - Public route table associated with public subnets, routing traffic to the Internet Gateway.
  - Private route table associated with private subnets, routing traffic to the NAT Gateway.

### 3. **Security Groups**
- **Application Load Balancer (ALB) Security Group:** Allows inbound HTTP/HTTPS traffic (ports 80, 443).
- **Application Security Group:** Allows inbound traffic only from the ALB security group.
- **Database Security Group:** Restricts access to the application instances in private subnets.

### 4. **Application Load Balancer (ALB)**
- Public-facing ALB distributing incoming traffic to application instances.
- Integrated with Route 53 for domain name resolution.

### 5. **Auto Scaling Group (ASG)**
- Automatically scales application instances based on demand.
- Instances are launched in private subnets for security.

### 6. **Amazon RDS (Relational Database Service)**
- RDS instance created from a DB snapshot for data persistence.
- Deployed in private subnets for enhanced security.

### 7. **Route 53**
- Configured with record sets to route external traffic to the Application Load Balancer.

## Key Features
- **High Availability:** Utilizes multiple availability zones to ensure redundancy.
- **Security:** Implements fine-grained access controls via security groups and private subnets.
- **Scalability:** Automatically scales application instances with an Auto Scaling Group.
- **Resilience:** Database is deployed in a private subnet and restored from a snapshot.
- **Automation:** Entire infrastructure is provisioned and maintained using AWS CloudFormation.

## Deployment Instructions
1. Ensure you have AWS CLI and permissions to deploy CloudFormation templates.
2. Validate the CloudFormation templates:
    ```bash
    aws cloudformation validate-template --template-body file://template.yaml
    ```
3. Deploy the stack:
    ```bash
    aws cloudformation create-stack --stack-name my-app-stack --template-body file://template.yaml --capabilities CAPABILITY_IAM
    ```
4. Monitor stack creation progress:
    ```bash
    aws cloudformation describe-stack-events --stack-name my-app-stack
    ```

## Troubleshooting Guide

### 1. Error: `AWS::ElasticLoadBalancingV2::TargetGroup` - `VpcId: expected type: String, found: JSONArray`

#### Cause:
This error occurs when the `VpcId` property is incorrectly formatted as an **array** instead of a **string** in the `AWS::ElasticLoadBalancingV2::TargetGroup` resource.

#### Solution:
Ensure that the `VpcId` value is a **single string**. Here is the corrected CloudFormation snippet:

```yaml
ALBTargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    HealthCheckIntervalSeconds: 10
    HealthCheckPath: "/"
    HealthCheckTimeoutSeconds: 5
    HealthyThresholdCount: 2
    Matcher:
      HttpCode: "200,302"
    Name: MyWebServers
    Port: 80
    Protocol: HTTP
    UnhealthyThresholdCount: 5
    VpcId: !ImportValue
      Fn::Sub: "${ExportVpcStackName}-VPCID"
```

#### Explanation:
1. **Fixed `VpcId` type**: Ensure that `VpcId` is provided as a **string**, not an array.
2. **Correct `!ImportValue` usage**: Avoid using `-` before `Fn::ImportValue`, which creates an array.

#### Verify the Export Value:
Check if the exported value is correct by running:

```bash
aws cloudformation list-exports
```
Ensure that the expected export key (e.g., `${ExportVpcStackName}-VPCID`) is present and maps to a valid VPC ID (e.g., `vpc-abc12345`).

---

### 2. Error: `AWS::CloudWatch::Alarm` - `Value 'average' at 'statistic' failed to satisfy constraint`

#### Cause:
This error occurs when the `Statistic` value in the `AWS::CloudWatch::Alarm` resource is **incorrectly capitalized**. CloudWatch requires specific case-sensitive values.

#### Solution:
Use the correct capitalization for the `Statistic` value. Here is the corrected CloudFormation snippet:

```yaml
CPUAlarmHigh:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: "HighCPUAlarm"
    AlarmDescription: "Trigger if CPU utilization exceeds 80%"
    MetricName: "CPUUtilization"
    Namespace: "AWS/EC2"
    Statistic: "Average"  # ✅ Correct capitalization
    Period: 300
    EvaluationPeriods: 2
    Threshold: 80
    ComparisonOperator: "GreaterThanOrEqualToThreshold"
    Dimensions:
      - Name: "InstanceId"
        Value: !Ref MyEC2Instance
    AlarmActions:
      - !Ref AlarmTopic
```

#### Explanation:
1. **Correct Enum Values**: Ensure the `Statistic` value is one of the following (case-sensitive):
   - `SampleCount`
   - `Average`
   - `Sum`
   - `Minimum`
   - `Maximum`
2. **Check Comparison Operator**: Ensure that the `ComparisonOperator` value is also valid:
   - `GreaterThanOrEqualToThreshold`
   - `GreaterThanThreshold`
   - `LessThanThreshold`
   - `LessThanOrEqualToThreshold`


## Conclusion
This architecture provides a secure, scalable, and automated AWS infrastructure suitable for production workloads. The use of AWS CloudFormation ensures consistent and repeatable deployments.

