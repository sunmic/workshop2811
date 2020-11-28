<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Run MVP version of the application.

## LAB Overview

#### During this laboratory you will run a basic environment for our application. Let's assume, this is a MVP version of the application. There is an prepared a AMI with the applicatio. All application's components are "baked" on the single Virtual Machine. Application consists of Web tier and Database.

In this lab you will create:
* Create a base Amazon Virtual Private Cloud (VPC) in Ireland Region
* Create a public subnet for your web server
* Create an Internet Gateway
* Create a route table and add a route so that people can access your web server from the Internet
* Create a security group to restrict only HTTP traffic to your web server
* Run Application Server from the prepared AMI.
## Task 1: Create a VPC for your application.
In this task you will create a simple VPC for your application.


1. In the **AWS Management Console**, on the **Services** menu, click **VPC**.
2. In the navigation pane on the left, click **Your VPCs**.
3. Click **Create VPC**.
4. In the **Create VPC** window use the following:
   * **Name tag:** awsninjaX_VPC (where X is your awsninja number)
   * **IPv4 CIDR block:** 10.X.0.0/16 (where X is your awsninja number)
   * Click **Create** and **Close**.
5. In the Navigation pane on the left, click **Subnets**.
6. Click **Create Subnet**
7. In the **Create Subnet** window configure the following:
   * **Name tag:** awsninjaX_Public1
   * **VPC:** awsninjaX_VPC (select your VPC created in Task 1)
   * **Availability Zone**: eu-west-1a (for Ireland region)
   * **IPv4 CIDR block**: 10.X.0.0/24
   * Click **Create**

For this moment, one subnet is sufficient for running your APP.

8. In the navigation pane on the left, click **Internet Gateways.**
9. Create an Internet Gateway by configuring the following:
10. Click **Create Internet Gateway**
11. **Name tag:** awsninjaX_IGW
12. Click **Create** 
13. Click **Actions** button
14. Click **Attach to VPC**
15. Select your VPC and click **Attach**
16. In the navigation pane on the left, click **Route Tables**.
17. Click **Create route table**, and provide following informations:
     * **Name tag**: awsninjaX_PUBLIC
     * **VPC**: select your VPC
18.  Click **Create** and **Close**.
19.  Select a newly created routing table.
20. Switch to the **Routes** tab and click **Edit routes**.
21. Click **Add route**.
22. Click the **Destination** field and enter: **0.0.0.0/0**
23. Click the **Target** field.
24. Click the Internet Gateway that you created earlier in task 3.
25. Click **Save routes** and **Close**.
26. Switch to the **Subnet Associations** tab and click **Edit subnet associations**.
27. Select your Public subnet and click **Save**.

You have now fully configured your public subnet for your APP server.
You also need a Security Group for your APP server, so in the next few steps you will configure it.

29. In the Navigation pane on the left, click **Security Groups**.
30. Click **Create security group**.
31. In the **Basic details** section, configure the following: 
    *  **Security group name:** awsninjaX_APP_SERVER
    *  **Description**: awsninjaX Web Servers Security Group
    *  **VPC**: Select your VPC     
32. In the **Inbound rules** section, Click **Add Rule**.
33. Configure the following:
     * From **Type** list select **HTTP**.
     * From **Source** list select **Anywhere**.
34. Click **Create security group**

Right now, you have everything ready to run your APP server.

## Task 2: Run your Application Server.

In this task you will spin up an MVP version of the application. Everything is baked in the AMI shared with you.

1. On the **Services** menu, click **EC2**.
2. Click **Launch Instance**.
3. In the navigation pane on the left, click My AMIs.
4. Select **APP_WARSZTAT** image.
5. On the **Choose an Instance Type** page, select the **t3.micro** instance type. 
6. On the **Configure Instance Details** page provide the following information:
   * **Number of instance**: 1
   * **Network:** awsninjaX_VPC
   * **Subnet**: awsninjaX_Public1
   * **Auto-assign Public IP**: Enable
   * **IAM Role**: GOV-EC2-SSM
7. lick **Next: Add Storage**. 
8. Click **Next: Add Tags** to accept the default storage device configuration. 
9. On the **Add Tags** page, click **Add Tag,** type a **Name** for a Key box and **awsninjaX_APP** in the Value box.
10. Click **Next: Configure Security Group**. 
11. For **Assign a security group**, select **select an existing security group** and select a **awsninjaX_APP_SERVER** Security Group you created in the previous task.
12. Click **Review and Launch**.
13. Review your choices, and then click **Launch**.
14. In the key pair dialog box, select **Proceed without a key pair**.
15. Mark **I acknowledge that I will not be able to connect to this instance unless I already know the password built into this AMI.**
16. Click **Launch Instances**.
17. On the status page, which notifies you that your instances are launching, click **View Instances**.
18. Select your instance to display a list of details and status update in the lower pane.
19. Copy the **IPv4 Public IP** value to the notes.
20. Open a new browser window, and then paste the public IP value into the address bar. 

## Task 3: Create a DNS name for your Application.

1. On the **Services** menu, click **EC2**.
2. In the **Description** tab of your **awsninjaX_APP_SERVER** instance, copy the **Public IPs** value to your text editor.
3. Go to the **Route53** Console. (Services menu > Route53).
4. In the navigation pane on the left, click **Hosted zones**.
5. Click on the zone: **awslabs.ninja**.
6. In the **Records** click **Create record**.
7. For **Routing policy** select **Simple routing** and click next.
8. In **Simple routing records to add to awslabs.ninja** click **Define simple records**.
9. Configure the following:
   * **Name**: awsninjaX (replace X with your number)
   * **Value/Route traffic to** select from list **IP address or another value depending on the record type**.  
   * In the empty field enter IP address that you saved to your notepad. 
   * **Record type**: A - Routes traffic to an IPv4 address and some AWS resources. 
   * **TTL (Seconds)**: 60 
   * Click **Define simple record**
   * Click **Create records**
10.  Paste the domain name into a new browser tab and press **Enter**.

## END LAB

<br><br>