<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Going into High Availability.

## LAB Overview

#### Time to redesing our architecture and achive more high available application.

What we will do:
* Create an AMI image,
* Recofingure our VPC,
* Create a second Application Server in different Availability Zone,
* Distribute traffic over Loadbalancer,
* Reconfigure our Database to Mutli-AZ.

## Task 1: Create an AMI from your current server.

1. In the AWS Management Console, on the **Services** menu, click **EC2**.
2. Go to Instances and select your application server.
3. On the **Actions** menu, click **Image** and then **Create Image**. 
4. For **Image name** box, enter **awsninjaX_AMI**. 
5. In the **Image description** box, type your name. 
6. Click **Create Image**. 
7. In the navigation pane on the left, click **AMIs**. 
8. Locate your AMI and wait until Status will change to **available**. 

## Task 2:  Create a second Public Subnet.

Create second public subnet in the same way the first. Make sure you select the next Availability Zone. Use Name tag: **awsninjaX_Public2**, CIDR block: **10.X.1.0/24** and **eu-west-1b** Availability Zone.

**Do not forget** to associate it with Public Routing Table.

If you need any advice, look into **LAB1** where you will find description for creation of the Public1 subnet.

## Task 3: Create a second Application Server in Public2 subnet.

You will use an image of your servers you created in Task 1.

Try to do this by yourself. The only difference is this time use a awsninjaX_Public2 subnet for APPLICATION SERVER. 
Name it **awsninjaX_APP_SERVER2**. 

If you need guidance, please follow LAB1 Task2 instructions.

At the end, you should open public IP of the second server in the browser and get running application.

## Task 4: Distribute traffic over Loadbalancer.

1. In *EC2* service on the left navigation pane, click **Load Balancers**. 
2. Click **Create Load Balancer**. 
3. In **Select load balancer type** page, for **Application Load Balancer**, click **Create**.
4. In the **Name** box, type a new name **awsninjaX-ELB**.
5. For VPC select your VPC.
6. Mark both availability zones and select both **Public** subnets.
7. Click **Next: Assign Security Groups**.
8. For **Assign a security group**, make sure that **Select an existing security group** is selected. 
9. Select **awsninjaX_APP_SERVER** security group - this will be enough.
10. Click **Next: Configure Routing**.
11. In **Target group** section configure:
    * **Name**: awsninjaX-TG
    * The rest config leave as is
12. Expand **Advanced health check settings**.
13. Change setting:
    * **Healthy threshold**: 2
    * **Interval**: 7
14. Click **Next: Register Targets**.
15. Under **Instances** select your both servers.
16. Click **Add to registered**.
17. Click **Next: Review**.
18. Check if everything is OK and click **Create**.
19. Click **Close**.
20. Select your loadblancer and copy **DNS name** from the **Description** tab into notepad. (ex. awsninja100-ELB-1034576633.eu-west-1.elb.amazonaws.com).
21. Wait until loadbalancer state will change to **active**.
22. In the Navigation pane on the left, click **Target Groups**.
23. Click on your target group.
24. Switch to **Targets** tab and check the **Status** of the servers. It should be as **healthy**.
25. Open the web browser and paste the DNS name of the loadbalancer and hit enter.

Now, you are accessing your application over loadbalancer. Refresh few time and check if Private IP value is changing.
Try to add new record.

## Task 4: Reconfigure your DNS record.

From now, your application is available over Loabalancer, so you need to reconfigure DNS name to point to the ELB. You will use special record set called **Alias**.

1. On the **Services** menu, click **EC2**.
2. In the **Description** tab of your **awsninjaX_APP_SERVER** instance, copy the **Public IPs** value to your text editor.
3. Go to the **Route53** Console. (Services menu > Route53).
4. In the navigation pane on the left, click **Hosted zones**.
5. Click on the zone: **awslabs.ninja**.
6. Select your **Record** and click **Edit** button.
7. In the **Value/Route traffic to** change to **Alias to Application and Classic Load Balancer**.
8. For Region, select **Europe (Ireland) [eu-west-1]**.
9. For Choose load balance, select your elb.
10. Click **Save changes**.
11. Test if your application is served over ELB now.

## Task 5: Make your DB more HA.

Your application layer is now high available. Traffic is distributed over ELB to two servers in two different locations.

But for now, you have only one single DB instance. Time to change it.

You will reconfigure tour DB into Multi-AZ mode.

1. In the AWS Management Console, on the Services menu, click Relational Database Service.
2. In the left navigation pane, click **Databases**.
3. Find and select your database, and click **Modify** button.
4. Scroll down to the **Availability & durability** section.
5. Under **Multi-AZ deployment** change configuration to **Create a standby instance (recommended for production usage)**.
6. Click **Continue**.
7. Under **Scheduling of modifications** change to **Apply immediately**.
8. Click **Modify DB instance**.

It will take few minutes..

## END LAB