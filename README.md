SDA2024
#Part 1
important steps 
PERMISSIONS > BLOCK PUBLIC ACCESS > UNCHECKED 

PERMISSIONS > BUCKET POLICY > 
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "s1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::khalid-clarusway-assets/*"
        }
    ]
}

This policy statement and the Public access are the main controls to make the resources accessible through the web and within the VPC.


#Part2
the provided userdata for the launch template:
###################################################
# User Data Script:
#!/bin/bash
yum update -y
yum install nginx -y
systemctl start nginx
systemctl enable nginx
aws s3 cp s3://yourname-clarusway-assets/index.html /usr/share/nginx/html/
####################################################

The "aws s3" command can cause issues if the IAM role attached to the EC2 instance via the Launch Template is missing or misconfigured. Although the s3 bucket has extremely permissive public policy, EC2 still needs an IAM role because SDKs and CLI use authenticated API calls, not anonymous HTTP access. IAM roles provide temporary credentials securely and enable logging and control. Public access only applies to direct unauthenticated URL access, not programmatic access from EC2. More over the provided command explicitly copies only the index.html without copying the logos. the logos are only can be referenced in the html file and are not part of the actual file. This leads to instances created using the Launch Template to not include the logos, but can be alleviated by minor changes.

#Part 3
for part 3, cannot show instance ID for ec2 instances without modifying the original or adding additional lines in the user data script(sed -i "/<\/body>/i <p>Served by: $(hostname)</p>" /usr/share/nginx/html/index.html
), because of the fact that the ALB acts as a reverse proxy and the visibility into the system is degraded without any server side logic. However round robin distribution behavior can be seen from availability zone changing after each request using this command "for i in {1..10}; do   curl -s --header "Connection: close" -o /dev/null -w "IP: %{remote_ip}\n" http://khalid-clarusway-alb-1886800558.eu-north-1.elb.amazonaws.com/; done"(output screenshot provided)


#Potential Issues That May Arise and How to Fix Them

#Target Group Not Visible for ALB
Sometimes, when setting up an Application Load Balancer (ALB), the expected target group may not appear. This usually happens if the target group was created for a different type of load balancer (i.e. protocol set to (TCP)), such as a Network Load Balancer (NLB).
Fix: Make sure to create the target group with the correct settings — for ALB, use the HTTP protocol and set the target type to instance.

#Launch Template Fails to Deploy EC2 Instance
A common error when launching an EC2 instance is:
“One or more target groups not found. Validating load balancer configuration failed.”
This may occur if the launch template points to a deleted or misconfigured target group.
Fix: Update the launch template with the correct target group and set the latest version as the default.

#502 Bad Gateway When Accessing ALB DNS
Even if the ALB is running, you might see a 502 error when accessing its DNS. This often means the EC2 instance is not responding properly to health checks — usually due to application startup failures or missing files.
Fix: Check the user data script for errors. For example, a wrong S3 URI can cause the instance to miss required files. Correcting the URI and launching a new instance should solve the issue.

#New Launch Template Doesn’t Apply Automatically
Although you've updated your launch template, the Auto Scaling Group may still keep running the old version. As a result, unhealthy or outdated instances won’t be replaced.
Fix: Use the Instance Refresh feature to replace running instances with new ones based on the latest launch template.

