<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Achiving Scalability

## LAB Overview

#### During this laboratory we prepare our application to switch to another region. We will use Frankfurt region as our DR location.

What you will do:
* Create a network configuration in 2nd region.
* Replicate application image to the 2nd region.
* Send a copy of the DB, to the 2nd region.


## Task 1. Create a network topology in Frankfurt region.

This time we will use Cloudformation to build a network topology.

1. Switch to the **Frankfurt** region.
2. In the AWS Management Console, on the Services menu, click **Cloudformation**.
3. Click **Create Stack**.
4. For **Amazon S3 URL** paste this: https://awsninjafrtemplate.s3.eu-central-1.amazonaws.com/vpc_pub_priv_2az.yaml
5. Click **Next**.
6. For **Stack name** set **awsninjaX-VPC**.
7. For **CidrOctect2** paste your **X** value.
8. For **TagName** write **awsninjaX_VPC**.
9. Click **Next** and **Next**.
10. Verify and click **Create stack**.
11. Wait until stack status will change to **CREATE_COMPLETE**.
12. Go to **VPC** service and check if VPC is created.

Template did not create a Security Group, so please recreate them. 

13. In the Navigation pane on the left, click **Security Groups**.
14. Click **Create security group**.
15. In the **Basic details** section, configure the following: 
    *  **Security group name:** awsninjaX_APP_SERVER
    *  **Description**: awsninjaX Web Servers Security Group
    *  **VPC**: Select your VPC     
16. In the **Inbound rules** section, Click **Add Rule**.
17. Configure the following:
     * From **Type** list select **HTTP**.
     * From **Source** list select **Anywhere**.
18. Click **Create security group**
19. Click again **Create security group**.
20. In the **Basic details** section, configure the following: 
   * **Security group name**: awsninjaX_DB
   * **Description**: awsninjaX Database Security Group
   * **VPC**: Select your VPC 
21. In the **Inbound rules** section, Click **Add Rule**.
22. Configure the following:
     * From **Type** list select **MySQL/Aurora**.
     * From **Source** list select **Custom**.
     * In the next blank box enter: **10.X.0.0/22**.
23. Click **Create security group**
24. In the AWS Management Console, on the Services menu, click Relational Database Service.
2. In the left navigation pane, click **Subnet groups**.
3. Click **Create DB Subnet Group**.
4. On **Create DB subnet group**, step configure the following:
   * **Name**: awsninjaX-DBSG
   * **Description**: Your name
   * **VPC**: Select your VPC created in LAB-VPC
   * In the **Add subnets** select:
     * For **Availability zones**
       * eu-central-1
       * eu-central-2
     * For **Subnets** 
       * 10.X.10.0/24
       * 10.X.11.0/24 
5. Click **Create**.

## Task 2. Export AMI and DB backup from Ireland to Frankfurt.

1. Switch back to the **Ireland** region.
2. In the AWS Management Console, on the **Services** menu, click **EC2**.
3. In *EC2* service on the left navigation pane, click **AMIs**.
4. Select your AMI and click **Actions** and then **Copy AMI**.
5. **Destination region**: Europe (Frankfurt).
6. Click **Copy AMI**.
7. Click **Done**.
8. In the AWS Management Console, on the Services menu, click Relational Database Service.
9. Select your database.
10. Click **Actions** and then **Take snapshot**.
11. For **Snapshot name**: awsninjaX-DR-SNAP.
12. Click **Take snapshot**.
13. On the left navigation pane, click **Snapshots**.
14. Select your snapshot, click **Actions** and **Copy snapshot**.
15. For **Destination Region** select **EU (Frankfurt)**.
16. For **New DB Snapshot Identifier** paste awsninjaX-RDS-SNAP.
17. Select **Copy Tags**.
18. Click **Copy snapshot**.

## Task 3. Power UP a DR site in Frankfurt.

If your AMI and DB Snapshot are copied, time to run an evnironment. You will spin up a "minimal" version of the environment. Lets assume this is a **Pilot Light** dr scenario.

1. Switch to the **Frankfurt** region.
2. In the AWS Management Console, on the Services menu, click **RDS**.
3. On the left navigation pane, click **Snapshots**.
4. Select your snapshot.
5. Click **Actions** and **Restore snapshot**.
6. Provide following settings:
   * **DB Instance identifier**: awsninjaX-mysqldr
   * **Virtual private cloud**: select your VPC
   * **Subnet group**: select your subnet group.
   * Existing VPC security groups: 
     - Select: **awsninjaX_DB**
     - Deselect: **default**
   * Expand **Addotional configuration**
   * DB instance class: **Burstable classes**
   * DB instance class: **db.t3.small**
7. Leave rest as default and click **Restore DB Instance**.
8. As restore contiue, go to **EC2** service.
9. Click **Launch Instance**.
10. In the navigation pane on the left, click My AMIs.
11. Select AMI you exported to this region.
12. On the **Choose an Instance Type** page, select the **t3.micro** instance type. 
13. On the **Configure Instance Details** page provide the following information:
   * **Number of instance**: 1
   * **Network:** awsninjaX_VPC
   * **Subnet**: awsninjaX_Public1
   * **Auto-assign Public IP**: Enable
   * **IAM Role**: GOV-EC2-SSM
14. lick **Next: Add Storage**. 
15. Click **Next: Add Tags** to accept the default storage device configuration. 
16. On the **Add Tags** page, click **Add Tag,** type a **Name** for a Key box and **awsninjaX_APP_DR** in the Value box.
17. Click **Next: Configure Security Group**. 
18. For **Assign a security group**, select **select an existing security group** and select a **awsninjaX_APP_SERVER** Security Group you created in the previous task.
19. Click **Review and Launch**.
20. Review your choices, and then click **Launch**.
21. In the key pair dialog box, select **Proceed without a key pair**.
22. Mark **I acknowledge that I will not be able to connect to this instance unless I already know the password built into this AMI.**
23. Click **Launch Instances**.
24. On the status page, which notifies you that your instances are launching, click **View Instances**.
25. Select your instance to display a list of details and status update in the lower pane.
26. Copy the **IPv4 Public IP** value to the notes.
27. Go back to your RDS instance.
28. Select your DB and copy **Endpoint** to the notepad.
29. Open terminal session to your newly created server, and replace DB enpoint in dbinfo.inc file:
```bash
sudo su
cd /var/www/inc/
nano nano dbinfo.inc
```
30. Replace **DB_Server** value with a new endpoint.
31. Press CTRL+O, ENTER to save your document. 
32. Press CTRL+X to exit the Nano editor.
33. Open webrowser and check if Application is running by pasting VM's Public IP.

## Task 4. Reconfigure your DNS record for Active-Standby Multi-AZ.

1. Go to the **Route53** Console. (Services menu > Route53).
2. In the **Route 53 Management Console**, in the navigation pane on the left, click **Health checks**.
3. Click **Create health check**.
4. For **Step 1: Configure health check**, configure the following:
    * **Name:** awsninjaX_HC
    * **What to monitor:** Endpoint
    * **Specify endpoint by**: Domain name
    * **Protocol**: HTTP
    * **Domain name**: Paste the DNS Name of your Loadbalancer.
    * **Port**: 80
    * **Path**: index.php
    * **Request interval** (Advanced configuration): Fast (10 seconds)
    * **Failure threshold**: 2
    * Click **Next**
5. For **Step 2: Get notified when health check fails**, configure the following:
    * **Create alarm**: Yes
    * **Send notification to**: New SNS topic
    * **Topic name**: awsninjaX_HCFail
    * **Recipient email addresses**: Enter your email address.
    * Click **Create health check**.
6. Wait until it change to **Healthy**.
7. In the navigation pane on the left, click **Hosted zones**.
8. Click on the zone: **awslabs.ninja**.
9. Select your **Record** and click **Edit** button.
10. In **Routing policy** change to **Failover**.
11. Set **Failover record type** to **Primary**.
12. Set **Health check** to this created few moments ago.
13. Set **Evaluate target health** to **YES**.
14. Set **Record ID** as **Primary**.
15. Click **Save changes**.
16. In the **Records** click **Create record**.
17. For **Routing policy** select **Failover** and click next.
18. Configure the following:
   * **Name**: awsninjaX (replace X with your number)
   * **Record type**: A - Routes traffic to an IPv4 address and some AWS resources. 
   * **TTL (Seconds)**: 60 
   * * Click **Define failover record**
     * **Value/Route traffic to** select from list **IP address or another value depending on the record type**.
     * In the empty field enter IP address of the Server located in Frankfurt (Public IP).
     * **Failover record type**: Secodnary,
     * **Record ID** **Secondary**
   * Click **Define failover record**
19. Click **Create records**.
    
## Task 5. Finall TEST.

1. Go to your website and refresh few times.

At this moment all traffic is forwarded to the Primary environment in Ireland Region.

2. Go back the AWS Console, switch to the **Ireland** region.
3. Go the **EC2** services and Auto Scaling Group.
4. Edit configuration of the Auto Scaling group and change parameters:
   * **Minimum Capacity**: 0
   * **Desired Capacity**: 0
5. Go to **Activity** tab and check what is going on.
6. Go back to the **Route53** and check your healt check.
7. Try to refresh your website.

What is an result?
Is it switched to the DR Region?


## THE END 🍻
    






## END LAB