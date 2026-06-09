Install the CloudWatch Agent on the EC2 Instance
Connect to the EC2 Instance and run the command below to install couldwatch Agent
sudo yum install amazon-cloudwatch-agent -y

# Create a Configuration File  using below command 
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json


Add the following configuration to capture all logs

{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/*",
            "log_group_name": "ec2-all-logs",
            "log_stream_name": "{instance_id}-all-logs"
          },
          {
            "file_path": "/var/log/nginx/access.log",
            "log_group_name": "ec2-all-logs",
            "log_stream_name": "{instance_id}-nginx-access"
          }
        ]
      }
    }
  }
}

# Start the agent using the configuration file:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

# Verify Logs in CloudWatch
Open the CloudWatch Console in the AWS Management Console
Navigate to Logs → Log Groups


Select a log group and view log streams for your EC2 instance

# Export logs from CloudWatch Logs to an S3 bucket
Create S3 Bucket
Navigate to the S3 service
Click Create bucket after entering Bucket Name and leave other defaults
Go to the bucket's Permissions tab 
Go to Bucket Policy. Click Edit

Add the following policy:

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudWatchLogs",
            "Effect": "Allow",
            "Principal": {
                "Service": "logs.amazonaws.com"
            },
            "Action": [
                "s3:PutObject",
                "s3:GetBucketAcl",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::qwertyujksdf /*",
                "arn:aws:s3:::qwertyujksdf"
            ]
        }
    ]
}

Click on Save Changes



# CloudWatch Logs Export
Navigate to Logs → Log Groups
Choose the log group you want to export
Click on Actions → Export data to Amazon S3
Choose your S3 bucket name and Click on Export

# Verify Logs in S3
Open the bucket and check for logs in the specified folder or directly at the root
You should see files in .gz format, as CloudWatch compresses the exported logs.


#note: we can manage dynamically this process for new server by adding below conetent into newserver userdata block also IAM also mandatory 


#!/bin/bash

# Update packages
yum update -y

# Install CloudWatch Agent
yum install -y amazon-cloudwatch-agent

# Create CloudWatch Agent configuration
cat <<EOF > /opt/aws/amazon-cloudwatch-agent/bin/config.json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/*",
            "log_group_name": "ec2-all-logs",
            "log_stream_name": "{instance_id}-all-logs"
          }
        ]
      }
    }
  }
}
EOF

# Start CloudWatch Agent with the config
/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

# Enable agent to start on reboot
systemctl enable amazon-cloudwatch-agent