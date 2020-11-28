<img src="https://szkolachmury.pl/wp-content/themes/Szkola_Chmury/img/logo.png" alt="SzkolaChmury" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Running application as a container

## LAB Overview

In this lab you are going to run your application as a container using AWS ECS service

## Task 1: Run your app as a container on EC2 instance

1. SSH onto your EC2 instance
2. Install docker:
   ```
   sudo yum update -y
   sudo amazon-linux-extras install docker
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   ```
> You will need to restart your ssh session in order to use docker without sudo
3. Clone git repo with application code: `git clone http://`
4. Enter into SampleAppDocker directory
5. Build an image: `docker build -t myphpapp .`
6. Verify your images was built `docker images`
7. Run your application: `docker run -p 8080:80 -d myphpapp`
8. Verify your app is working: `curl localhost:8080`

## Task 2: Create an ECR repository

1. In the AWS Management Console, on the **Services** menu, click **ECS**.
2. On the left menu, click **Repositories** and then **Create repository**. 
3. Enter **myphpapp** as a repository name, leave other fields by default.
4. Click **Create Repository**. 

## Task 3: Push your app into ECR repository

1. Before pushing an image we need to allow EC2 instance to access ECR repository.
2. Find your EC2 instance in AWS console, open **security** tab.
3. Click on an IAM role name. You will be redirected to IAM service.
4. Click **Attach policies** button.
5. In the search box enter "AmazonEC2ContainerRegistryFullAccess"
6. Tick the box and click **Attach policy**
7. In the AWS console go back to ECS service and select **Repositories** from left menu.
8. Click on your repositiry and click **View push commands**
9. Execute every command from pop-up. (Mind the "sudo")
10. After "docker push" refresh images list in AWS console and verify that your image is successfully pushed.
11. Click your **latest** image and note the "Image URI". We will use it later.

## Task 4: Create ECS Fargate cluster
1. In the AWS Management Console, on the **Services** menu, click **ECS**.
2. From the left menu select **Clusters** and click **Create cluster**
3. Choose **Networking only** and click **Next step**
4. Enter name "myfargatecluster" and click **Create**

## Task 5: Create Task definition
1. From the left menu select **Task definition** and click **Create new Task Definition**
2. Select FARGATE and click **Next step**
3. Enter Task Definition Name ("myphpapp-task"), in "Task Size" section select 1GB Task memory and 0.5 vCPU.
4. Click **Add container** button.
5. Enter Container Name ("myphpapp-container")
6. Enter image URI, copied from Task 3, step 11.
7. In the **Port mappings** section enter port number 80. (This is the port that the application is available on).
8. Click **Add** and then **Create**
   
## Task 6: Run ECS Task
1. From the left menu select **Clusters**
2. Click on you FARGATE cluster
3. Open **Tasks** tab and click **Run new task**.
4. Fill params:
- Launch Type: FARGET
- Task Definition: myphpapp-task
- Revision: 1(latest)
- Cluster: myfargatecluster
- Number of tasks: 1
- Cluster VPC: (select your vpc)
- Subnets: (select Public subnet)
- Auto-assign public IP: ENABLED
5. Click **Run task**
6. Wait for the task to be "RUNNING"
7. Click on your task
8. In new windows find public ip.
9. Verify that your app is working correctly.
