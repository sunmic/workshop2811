<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Decoupling Application's Architecture

## LAB Overview

#### In this laboratory, we want to separate the application layer from the data layer. For this purpose, we will migrate the database to the Amazon RDS service (Mysql). Before we create DB, we need to prepare a network configuration for our database.

## Task 1: Configure subnet for DB.

To deploy your RDS database, your VPC must have at least one subnet in at least two Availability Zones in
the region where you want to deploy your DB instance. In this task, you'll create your private subnets for
your soon to be created Amazon RDS instance. Additionally you will also create Security Group for the database.

1. In the **AWS Management Console**, on the **Services** menu, click **VPC**.
2. In the navigation menu on the left, click **Subnets**.
3. Click **Create subnet**.
4. In the **Create subnet** window configure the following:
   * **Name tag**: awsninjaX_Private1
   * **VPC**: Select your VPC
   * **Availability Zone**: eu-west-1a
   * **IPv4 CIDR block**: 10.X.10.0/24
   * Click **Create** and **Close**.
5. Repeat the same steps and create second private subnet. Use **Name tag** awsninjaX_Private2, CIDR block **10.X.11.0/24** and **eu-west-1b** Availability Zone.
6. In the Navigation pane on the left, click **Security Groups**.
7. Click **Create security group**.
8. In the **Basic details** section, configure the following: 
   * **Security group name**: awsninjaX_DB
   * **Description**: awsninjaX Database Security Group
   * **VPC**: Select your VPC 
8. In the **Inbound rules** section, Click **Add Rule**.
9. Configure the following:
     * From **Type** list select **MySQL/Aurora**.
     * From **Source** list select **Custom**.
     * In the next blank box enter: **10.X.0.0/22**.
10. Click **Create security group**

This rule states that only MySQL traffic coming from your public subnets is allowed to access the database
in the Private subnets.

## Task 2: Create a Relational Database Service (RDS) Instance

1. In the AWS Management Console, on the Services menu, click Relational Database Service.
2. In the left navigation pane, click **Subnet groups**.
3. Click **Create DB Subnet Group**.
4. On **Create DB subnet group**, step configure the following:
   * **Name**: awsninjaX-DBSG
   * **Description**: Your name
   * **VPC**: Select your VPC created in LAB-VPC
   * In the **Add subnets** select:
     * For **Availability zones**
       * eu-west-1
       * eu-west-2
     * For **Subnets** 
       * 10.X.10.0/24
       * 10.X.11.0/24 
5. Click **Create**.
6. In the left navigation pane, click **Databases**.
7. Click **Create database** button.
8. On **Create database**, select
   * Engine type: **MySQL**
   * Version: **5.7.31**
   * Template: **Dev/Test**
   * DB instance identifier: **awsninjaX-mysql**
   * Master password: **Set a preffered password and write it down**
   * DB instance class: **Burstable classes**
   * DB instance class: **db.t3.small**
   * Storage type: **General Purpose (SSD)**
   * Allocated storage: **20**
   * Multi-AZ deployment: **Do not create a standby instance**
   * Virtual Private Cloud (VPC): **Select your VPC**
   * Expand **Additional connectivity configuration**
   * Subnet group: **Select group you just created**
   * VPC security group: **Choose existing**
   * Existing VPC security groups: 
     - Select: **awsninjaX_DB**
     - Deselect: **default**
   * Expand **Addotional configuration**
   * Initial database name: **awsninjaXdb**
   * Enable automatic backups: **Unselect**
   * Enable encryption: **Unselect**
9. Click **Create database**.
10. Wait until **Successfully created database awsninjaX-mysql.** green information appear.
11. When DB is ready, clik on in and copy **Endpoint** to the notepad.


## Task 3. Export data and switch to the RDS Database.

During this task you will migrate data to the RDS Database. Then you will reconfigure application to use RDS and then stop local database service.

1. On the **Services** menu, click **EC2**.
2. In EC2 console select your EC2 instance.
3. Click **Connect** button locateb above a list.
3. Make sure **Session Manager** tab is selected.
4. Click **Connect** button.
5. Create a backup of the data by typing:

```bash
sudo su
cd ~
mysqldump -u root -p sample > sample.sql
```
6. Type password: **P@ssword12345**
7. Hit enter.
8. Check if backup faile was created:

```bash
[root@ip-172-31-33-61 ~]# ll
total 4
-rw-r--r-- 1 root root 2152 Nov 26 14:31 sample.sql
```
9. Import the data to the RDS database you created using command:

```bash
mysql -u admin -p -h YOUR_DB_ENDPOINT.eu-west-1.rds.amazonaws.com awsninjaXdb < sample.sql
```
Provide correct database endpoint and database name.

10. Type the password for RDS database you created and hit Enter.
11. Change the application configuration and connect it to the RDS Database.
12. Go to configuration:
```bash
cd /var/www/inc/
cp dbinfo.inc dbinfo.backup
nano dbinfo.inc
```
13. Change the configuration:
    * **DB_SERVER**: replace 127.0.0.1 with you RDS DB Endpoint. (ex. awsninja0-mysql.cmittnudc136.eu-west-1.rds.amazonaws.com)
    * **DB_USERNAME**: change to **admin**
    * **DB_PASSWORD**: paste a password for your RDS DB
    * **DB_DATABASE**: change to **awsninjaXdb** (replace X with your number)
14. Press CTRL+O, ENTER to save your document. 
15. Press CTRL+X to exit the Nano editor.
16. Time to test if application still working:
    * Paste the following command int terminal to check if server communicate with RDS:
    ```bash
    tcpdump 'port 3306'
    ```
    * Go to your APP and refresh a browser. Try to add new record.
    * Go back to terminal and check if any packets were send to RDS.
    * Stop tcpdump by pressing CTRL+C
17. As a finall step stop and disable MYSQL database on the server:
    ```bash
    systemctl stop mysqld
    systemctl disable mysqld
    ```
18. Verify if APP still working.

## END LAB