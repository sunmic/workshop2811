<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Achiving Scalability

## LAB Overview

#### During this laboratory you will redesing your architecture to be more scalable.

## Task 1. Create launch template.

1. In the AWS Management Console, on the **Services** menu, click **EC2**.
2. In *EC2* service on the left navigation pane, click **Launch templates**.
3. Click **Create launch template**.
4. Provide following configuration:
   * Launch template name: **awsninjaX-LT**,
   * Template version description: **ver1**,
   * Select **Provide guidance to help me set up a template that I can use with EC2 Auto Scaling**,
   * AMI: **user your awsninjaX_AMI**,
   * Instance type: **t3.micro**,
   * Click **Add Network Interface** and set:
     * Auto-assign public IP: **Enable**
     * Security groups: **awsninjaX_APP_SERVER**, 
   * IAM instance profile (Advanced details): **GOV-EC2-SSM**
5. Click **Create launch template**.
6. Click **View launch templates**.

## Task 2. Create an auto scaling group and connect with loadbalancer.

Before you do that, deregiter current Applications servers from ELB.

1. In the left navigation pane, click **Target Groups**.
2. Find and click on your group.
3. Switch to **Targets** tab.
4. Select your servers and click **Deregister** button.
5. In the left navigation pane, click **Auto Scaling Groups**.
6. Click **Create Auto Scaling group**.
7. Set **Auto Scaling group name** as **awsnincjaX-ASG**.
8. Choose your launch tample for **Launch template** configuration.
9. Click **Next**.
10. For **VPC** choose yours.
11. For **Subnets** select both **Public** subnets.
12. Click **Next**.
13. For **Load balancing** choose **Attach to an existing load balancer**.
14. For **Existing load balancer target groups** select your target group.
15. For **Health check type** select **ELB**.
16. Click **Next**.
17. For **Group size**:
    * **Desired capacity**: 2
    * **Minimum capacity**: 2
    * **Maximum capacity**: 4
18. For **Scaling policies** set:
    * Select **Target tracking scaling policy**
    * **Target tracking scaling policy**: awsninjaX-TSP
    * **Targe value**: 25
19. Click **Next**.
20. For notification just click **Next**.
21. For Tags add a tag **Owner: awsnijaX** click **Next**.
22. Review setting and click **Create Auto Scaling group**.
23. Click on your ASG group and switch over taps and verify if new instances are launched.
24. After few moment check your Target Group.
25. If instances are registered and healthy. Try to refresh your website.
26. If everything is working correctly, terminate your old servers awsninjaX_APP_SERVER and awsninjaX_APP_SERVER2.

## Task 3: Test your autoscaling.

#### TEST 1 - Auto healing.

1. Go back to EC2 list select one of the application servers created by Auto Scaling.
2. Click **Instance state**, and choose **Terminate instance.**.
3. Wait for a moment, go back to your Auto Scaling Group and check if new insance is reacreated.
4. Check if webiste is still working.

#### TEST 2 - Auto scaling.

1. Select one of the servers and open terminal session.
2. Paste following commands to install stress cpu tool:
   ```bash
   sudo su
   amazon-linux-extras install epel -y
   yum install stress -y
   ```
3. Make test by typing:
   ```bash
   stress --cpu 100
   ```
4. Wait for a few minutes and verify if autoscaling is adding new instances.
5. In the AWS Management Console, on the **Services** menu, click **CloudWatch**.
6. Looke over **Alarms** and look with is in **RED** and **GREEN** state.
7. Open browser and check if your application work?
8. How many instances do you see?

After stop stress by pressing **CTRL+C** or command **killall stress**.

## END LAB
