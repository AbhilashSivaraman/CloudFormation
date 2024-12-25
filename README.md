# EC2 Instance Deployment using CloudFormation

## Prerequisites

1. AWS account with administrative access.
2. AWS CLI installed and configured.
3. Basic knowledge of YAML and JSON.

---

## Steps

### 1. Create a Key Pair for EC2
A key pair is required to securely SSH into your EC2 instance.

1. Navigate to the AWS Management Console.
2. Go to **EC2** > **Key Pairs** under **Network & Security**.
3. Click **Create key pair**.
4. Provide a name for your key pair (e.g., `my-key-pair`) and choose the format (e.g., `.pem`).
5. Download the key pair file and store it securely.

---

### 2. Prepare `ec2-instance.yaml`
The `ec2-instance.yaml` file will define the CloudFormation template for creating the EC2 instance.

**Steps:**
1. Open a text editor.
2. Create a YAML file named `ec2-instance.yaml`.
3. Add the following content:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyNewEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0fd05997b4dff7aac # Amazon Linux 2 AMI (adjust to your region)
      InstanceType: t2.micro
      KeyName: ProdKey
      SecurityGroups:
        - Ref: InstanceSecurityGroup
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 8
            VolumeType: gp2
      Tags:
        - Key: Name
          Value: Production Server

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH, HTTP, HTTPS, and Custom TCP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3001
          ToPort: 3001
          CidrIp: 0.0.0.0/0
```

4. Save the file.

---

### 3. Create an S3 Bucket
The S3 bucket will store the CloudFormation template.

**Steps:**
1. Go to **S3** in the AWS Management Console.
2. Click **Create bucket**.
3. Provide a unique bucket name (e.g., `cloudformationbucket01`).
4. Choose the region (e.g., `ap-south-1`).
5. Click **Create bucket**.

---

### 4. Apply S3 Bucket Policy
A policy is required to allow CloudFormation to access the template in the bucket.

**Steps:**
1. Go to your S3 bucket.
2. Navigate to **Permissions** > **Bucket Policy**.
3. Add the following policy, replacing `cloudformationbucket01` with your bucket name:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFormationReadAccess",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudformation.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::bucketname/*"
        }
    ]
}
```

4. Save changes.

---

### 5. Upload the Template to S3

**Steps:**
1. Go to your S3 bucket.
2. Click **Upload**.
3. Upload the `ec2-instance.yaml` file.
4. Copy the file URL (e.g., `https://cloudformationbucket01.s3.ap-south-1.amazonaws.com/ec2-instance.yaml`).

---

### 6. Create a Role for CloudFormation
CloudFormation needs permissions to access S3 and EC2.

**Steps:**
1. Go to **IAM** > **Roles**.
2. Click **Create role**.
3. Select **AWS service** and choose **CloudFormation**.
4. Attach the following policies:
   - `AmazonS3ReadOnlyAccess`
   - `AmazonEC2FullAccess`
5. Provide a role name (e.g., `CloudFormationRole`).
6. Click **Create role**.

---

### 7. Create a CloudFormation Stack

**Steps:**
1. Go to **CloudFormation** in the AWS Management Console.
2. Click **Create stack** > **With new resources (standard)**.
3. Select **Template source** as **Amazon S3**.
4. Paste the URL of the `ec2-instance.yaml` file.
5. Click **Next** and follow the prompts to configure stack options.
6. Under **Permissions**, select the role created earlier (e.g., `CloudFormationRole`).
7. Click **Create stack**.
8. Wait for the stack status to change to `CREATE_COMPLETE`.

---

### 8. Verify the EC2 Instance

**Steps:**
1. Go to **EC2** > **Instances**.
2. Check that the instance created by the stack is running.
3. Use the downloaded key pair file to SSH into the instance:

```bash
ssh -i my-key-pair.pem ec2-user@<Instance-Public-IP>
```

---

## Conclusion
Congratulations! You have successfully deployed an EC2 instance using CloudFormation.