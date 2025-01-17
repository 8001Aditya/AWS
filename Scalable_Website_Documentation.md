# Deploying a Scalable PHP Website on AWS

This repository documents the steps to deploy a scalable PHP-based website on AWS using EC2, RDS, and Auto Scaling. Follow the guide below to set up and test the infrastructure.

---

## Prerequisites
1. AWS account with access to EC2, RDS, and Auto Scaling.
2. Basic knowledge of Linux commands and AWS services.
3. PHP-based website files ready for deployment.
4. MySQL client or admin tool (e.g., phpMyAdmin).

---

## Deployment Steps

### **Step 1: Launch and Configure an EC2 Instance**
1. Launch an EC2 instance:
   ```bash
   # Example: Ubuntu-based instance
   aws ec2 run-instances \
       --image-id ami-xxxxxx \
       --count 1 \
       --instance-type t2.micro \
       --key-name YourKeyName \
       --security-group-ids YourSecurityGroupID \
       --subnet-id YourSubnetID
   ```

2. Install Apache and PHP:
   ```bash
   sudo apt update
   sudo apt install apache2 php libapache2-mod-php -y
   ```

3. Deploy your website:
   ```bash
   sudo mv /path/to/your/website/* /var/www/html/
   sudo chmod -R 755 /var/www/html/
   ```

4. Test the website by accessing the EC2 instance's public IP.

---

### **Step 2: Create an Amazon RDS Instance**
1. Launch an RDS MySQL instance:
   ```bash
   aws rds create-db-instance \
       --db-instance-identifier YourDBIdentifier \
       --db-instance-class db.t2.micro \
       --engine mysql \
       --allocated-storage 20 \
       --master-username admin \
       --master-user-password intel123
   ```

2. Set up the database and table:
   ```sql
   CREATE DATABASE intel;
   USE intel;
   CREATE TABLE data (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255),
       value TEXT
   );
   ```

3. Update your PHP code for RDS connectivity:
   ```php
   $servername = "your-db-name.xxx.us-east-1.rds.amazonaws.com";
   $username = "admin";
   $password = "intel123";
   $dbname = "intel";

   $conn = new mysqli($servername, $username, $password, $dbname);

   if ($conn->connect_error) {
       die("Connection failed: " . $conn->connect_error);
   }
   echo "Connected successfully";
   ```

---

### **Step 3: Secure Connections**
1. Update RDS security group to allow MySQL traffic (port 3306) from the EC2 security group.
2. Update EC2 security group to allow HTTP (port 80) and SSH (port 22).

---

### **Step 4: Create an AMI for Scaling**
1. Verify the setup:
   - Ensure the website connects to the RDS database successfully.

2. Create an AMI:
   ```bash
   aws ec2 create-image --instance-id YourInstanceID --name "YourAMIName"
   ```

---

### **Step 5: Set Up Auto Scaling**
1. Create a launch template using the AMI:
   ```bash
   aws ec2 create-launch-template \
       --launch-template-name YourTemplateName \
       --version-description "v1" \
       --launch-template-data file://template-config.json
   ```

2. Create an Auto Scaling group:
   ```bash
   aws autoscaling create-auto-scaling-group \
       --auto-scaling-group-name YourASGName \
       --launch-template LaunchTemplateName=YourTemplateName,Version=1 \
       --min-size 2 \
       --max-size 5 \
       --desired-capacity 2 \
       --vpc-zone-identifier "subnet-xxxxxx,subnet-yyyyyy"
   ```

3. (Optional) Attach a load balancer for traffic distribution.

---

### **Step 6: Test and Verify**
1. Simulate load:
   ```bash
   ab -n 1000 -c 50 http://your-ec2-instance-ip/
   ```

2. Verify functionality:
   - Ensure all scaled instances function properly.
   - Monitor scaling behavior via CloudWatch.

---

## Security Best Practices
1. Restrict RDS access to EC2 security group.
2. Use SSH keys for secure access to EC2 instances.
3. Regularly update software on EC2 instances.

---
