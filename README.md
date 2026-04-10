# Step 1: Launch the EC2 Instance (The "Brain")
This server will host your Airflow scheduler and webserver.

Navigate to EC2: Log into your AWS Console and search for EC2. Click Launch Instance.
Name: Call it Airflow-Data-Pipeline.
OS Image: Select Ubuntu 24.04 LTS (the most stable and widely supported for Airflow).
Instance Type: Select t3.medium (2 vCPU, 4GB RAM).
Note: A t2.micro is too small and will likely crash when running the Airflow scheduler.
Key Pair: Create a new key pair (RSA, .pem), download it, and keep it safe. You’ll need this to login.
Network Settings (Security Group):
Allow SSH (Port 22) from "My IP".
Add a rule to allow Custom TCP (Port 8080) from "Anywhere" (this is for the Airflow Web UI).
Storage: Increase the default to 20 GB (gp3).

Launch!

# Step 2: Create the S3 Bucket (The "Landing Zone")

This is where your raw FMP data will live before moving to Snowflake.
Navigate to S3: Search for S3 and click Create Bucket.
Bucket Name: Must be unique globally (e.g., fmp-airflow-pipeline-raw-data-12345).
Region: Choose the same region as your EC2 (e.g., us-east-1).
Public Access: Keep "Block all public access" checked (Safety first!).
Create: Leave everything else at default and click Create bucket.

# Step 3: Set up SNS (The "Alarm System")

This will notify you when your pipeline finishes or fails.
Navigate to SNS: Search for Simple Notification Service.
Create Topic: Select Standard, name it Airflow_Pipeline_Alerts, and click Create topic.
Create Subscription: * Inside your new topic, click Create subscription.
Protocol: Select Email.
Endpoint: Enter your personal email address.
Click Create.

Confirm: Go to your email inbox, find the "AWS Notification - Subscription Confirmation" email, and click Confirm Subscription. Your status in AWS should now say "Confirmed".

# Step 4: Configure Permissions (The "Keys")

We need to give the EC2 instance permission to "talk" to S3 and SNS.
Navigate to IAM: Search for IAM > Roles > Create Role.
Trusted Entity: AWS Service > EC2.
Add Permissions: Search for and check these two policies:
AmazonS3FullAccess
AmazonSNSFullAccess
Role Name: Call it EC2-Airflow-Role. Click Create.
Attach to EC2: * Go back to the EC2 Dashboard > Instances.
Select your Airflow instance.

Click Actions > Security > Modify IAM Role.

# further things are written in the docker guide file

Select EC2-Airflow-Role and click Update IAM Role.
