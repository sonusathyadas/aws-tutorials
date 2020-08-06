# AWS Elastic Container Service (ECS) with AWS Fargate launch type
AWS ECS is a highly scalable, fast container management service that helps you to run, stop and manage your containerized applications. ECS supports two launch types: AWS Fargate launch type and AWS EC2 launch type. AWS Fargate launch type allows you to run your tasks and services in serverless infrastructure managed by Amazon ECS.

To deploy the containers in ECS you need to create an ECS cluster with user that has `administratorAccess` permissions. This Lab covers the following topics.
* Creating IAM user with Admin privillages.
* Creating ECS cluster with Fargate launch type.
* Create and push docker images to ECR.
* Create Task defenition for containers
* Configure the VPC, Security group and Application Load Balancer.
* Create and deploy service and Tasks.

## Prerequisites
* AWS Subscription
* AWS CLI 
* Docker Desktop for Windows/Linux

## Create user with Administrator privillages
1) Use your AWS account email address and password to sign in as the AWS account root user to the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
    > Note: We strongly recommend that you adhere to the best practice of using the Administrator IAM user below and securely lock away the root user credentials. Sign in as the root user only to perform a few account and service management tasks.
2) In the navigation pane, choose `Users` and then choose `Add user`.
![Create user1](/images/CreateUser1.png)
3) For User name, enter `Administrator` and select the checkbox for `AWS Management Console access`.
4) For `Console Password` select `Custom Password` and enter a valid password.
5) Unselect the `User must create a new password at next sign-in` checkbox to avoid resetting password during the first login.
6) Click on `Next:Permissions` button.
![Create user2](/images/CreateUser2.png)
7) In the permissions page, under `Set permissions` , choose `Add user to group` and click on the `Create group` button.
![Create user3](/images/CreateUser3.png)
8) In the Create group dialog box, for Group name enter `Administrators`.
9) Select the checkbox for the policy name `AdministratorAccess` from the policy list and click `Create group` button.
![Create user4](/images/CreateUser4.png)
10) Back in the list of groups, select the check box for your new group. Choose Refresh if necessary to see the group in the list and click the button `Next: Tags`.
![Create user5](/images/CreateUser5.png)
11) Add metadata to the user by attaching tags as key-value pairs. Click on `Next: Review` button to see the list of group memberships to be added to the new user. When you are ready to proceed, choose `Create user`.
![Create user6](/images/CreateUser6.png)
12) In the user creation success page, you will see the sign-in URL for the newly created user. The URL contains the IAM id of the new user. Note down the URL or Id for login in future.
![Create user7](/images/CreateUser7.png)
13) To sign in as this new IAM user, sign out of the AWS console, then use the following URL, where your_aws_account_id is your AWS account number without the hyphens
https://your_aws_account_id.signin.aws.amazon.com/console/
14) You can also see the newly created users in the users page. 
![Create user8](/images/CreateUser8.png)
15) Click in the newly created user name. In the summary page, click on the `Security Credentials` tab. You can see the console login URL for the user.
![Create user9](/images/CreateUser9.png)

## Create ECS with Fargate launch type
1) Login to the AWS Console using the new IAM user created. Navigate to [https://http://console.aws.amazon.com/](http://console.aws.amazon.com/) to login.
2) Select `IAM user` type and enter the 12 digit account id and click `next`.
![Login1](/images/Login1.png)
3) If the ID is valid you will be prompted to enter the username and password. Enter the username and password to login.
![Login2](/images/Login2.png)
4) In the AWS console, type `ECS` in the `Find Services` search text box and click on the `Elastic Container Services` from the popped list.
![ECS1](/images/ECS1.png)
5) In the ECS console, choose clusters from the left pane and click on `Create Cluster` button.
![ECS2](/images/ECS2.png)
6) From the Create cluster wizard select the `Networking only` option to create a cluster using Fargate and click on the `Next` button.
![ECS3](/images/ECS3.png)
7) In the `Configure cluster` page, input the cluster name as `aws-ecs-cluster1`. 
8) Select the checkbox for `Create a new VPC for this cluster` and input the CIDR address space informations for VPC and Subnet1 and Subnet2. Input `10.5.0.0/16` as the VPC CIDR block and `10.5.0.0/24` for `Subnet1` and `10.5.1.0/24` for `Subnet2`. 
9) Optionally, you can provide the metadata information for cluster using the tags. You can also enable cloudwatch for the cluster if you want to enable monitoring for cluster. Click on the `Create` button.
![ECS4  ](/images/ECS4.png)
10) You can see the cluster creation status.
![ECS5](/images/ECS5.png)
11) Click on the `View cluster` button to see the cluster details.
![ECS6](/images/ECS6.png)
## Upload application image to ECR
1) Open the ECS dashboard, and choose the `Repositories` from the left pane.
![ECR1](/images/ECR1.png)
2) In the ECR console, click on the `Create repository` button.
![ECR2](/images/ECR2.png)
3) Provide a repository name in the `Repository name` textbox. Click on `Create repository` button.
![ECR3](/images/ECR3.png)
4) Once the repository is created, click on the repository name.
![ECR4](/images/ECR4.png)
5) In the repository page, you can see an empty list of pushed images. To push a new image to the repository, you need to authenticate to the ECR and run the `docker push` command. To view the commands to upload a docker image to repository, click on the `View Push Commands` button.
![ECR5](/images/ECR5.png)
6) You can use the aws CLI command or Windows PowerShell command to authenticate and push the image. Choose the `Mac/Linux` tab if you want to perform using the `aws cli`.
![ECR6](/images/ECR6.png)
7) Execute the commands to create and push your application docker image. After the push operation is completed, you can see the image in the repository.
![ECR7](/images/ECR7.png)
8) You can use the image URI for deploying containers in your ECS cluster.

## Configure VPC, Security Group and Load Balancer
### Update VPC name
1) In the AWS Management Console home, search for `VPC` and click on `VPC` from the popped list.
![VPC1](/images/VPC1.png)
2) It opens the VPC console. From the left pane click on `Your VPCs` link. You can also select `VPCs` from the `Resources by region` section.
![VPC2](/images/VPC2.png)
3) You will see the list of available VPCs list. Click on the edit icon on the `Name` column of the VPC and enter a valid identifier for VPC. You can give `FargateVPC` for this example.
![VPC3](/images/VPC3.png)
4) After entering the name in the text box press enter to save the changes.
![VPC4](/images/VPC4.png)
### Create Security Group
1) In the VPC Console, scroll down the left pane. You will see the `Security Groups` under the `Security` section.
![SG1](/images/SG1.png)
2) Click on the `Create Security Group` button.
3) Enter the security group name as `AWS-ECS-Fargate-SG` and provide a valid description. Also select the `FargateVPC` in the VPC section dropdown list. 
3) Below, In the inbound rules section, click on the `Add rule` button. Select `HTTP` from `Type` list and `0.0.0.0/0` from sources. Optionaly, you can provide a description for the rule.Click on the `Create security group` button below.
![SG2](/images/SG2.png)
4) Once the security group is created, you will see the details of the security group.
![SG3](/images/SG3.png)
### Create Application Load Balancer (ALB)
1) In the Console Home, search for EC2 and select `EC2` from the popped list.
![EC2](/images/EC2-1.png)
2) In the EC2 console, select `Load balancers` from the `Resources` section.
![EC2](/images/EC2-2.png)
3) Click on the `Create Load Balancer` button.
![EC2](/images/EC2-3.png)
4) Select the `Application load balancer` and click create button.
![LB1](/images/LB1.png)
5) Provide a valid name for ALB. For this demo you can give name as `Aws-Fargate-ALB`. Confirm the type of ALB as `internet facing`. Also ensure the Load balancer port is selected as HTTP (80). Select the VPC name and subnets and click on the `Next:Configure Security Settings` button.
![LB2](/images/LB2.png)
6) In the `Step 2: Configure Security Settings` page do not configure any thing and skip to next page by clicking `Next:Configure Security Groups`.
7) In `Configure Security Groups` page, choose `Select and existing security group` option and select the security group you have created in previous step (Aws-ECS-Fargate-SG). Then click on `Next:Configure Routing` button.
![LB3](/images/LB3.png)
8) In the `Configure Routing` page, enter a valid name for Target. You can specify `ALB-Target` for this demo and leave all the settings as default. Click on `Next:Register targets`.
![LB4](/images/LB4.png)
9) In the `Register targets` page, select the VPC as `FargateVPC`. Do not provide any IP values in `IP` textbox and confirm port as `80`. Click on `Next:Review`.
![LB4](/images/LB4.1.png)
10) In the Review page confirm the values and click on `Create`.
11) After the ALB is created, you will see a confirmation page. Click on the ALB name hyperlink to navigate to the ALB.
![LB4](/images/LB5.png)
12) You will see the Load balancer status and DNS name. Note the DNS name, which can be used to access the application.
![LB6](/images/LB6.png)
## Create Task defenition
1) Click on the `Task Defenitions` from the left pane. 
![TaskDef](/images/Task_Def1.png)
2) Click on `Create new Task Defenition`. In the opened wizard choose `Fargate` as the `launch type compatibility`. Click on `Next Step`.
![TaskDef](/images/Task_Def2.png)
3) Provide a name for task defenition. Enter `sample-task-def` and Task role as `ecsTaskExecutionRole`. Confirm network mode as `awsvpc`. 
4) In `Task Execution IAM role` section, select `ecsTaskExecutionRole` for Task execution role.
5) In the `Task size` section select `Task memory` as `0.5GB` and `Task CPU` as `0.25vCPU`.
![TaskDef](/images/Task_Def3.png)
6) Click on `Add Container` button and provide container name as `sampleweb` and image name as your image name in ECR repository (eg: `xxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/sampleweb:latest`). You can choose your own docker image name also. Set memory limit as `256MB` and in the port mappings section enter port number as `80` for you containerized web application. You can go ahead with the default values for other settings and click on `Add`.
![TaskDef](/images/Task_Def4.png)
7) Click on the `Create` button in the Task defenition page to create it.
![TaskDef](/images/Task_Def5.png)

## Configure the Service
A service launches and maintains a specified number of copies of the task definition in your cluster. By running our task defenition as a service, it restarts if the task becomes unhealthy or unexpectedly stops.

1) Open the ECS console and select the cluster you have created.
2) Click on the `Services` tab and click on `Create` button.
![Service1](/images/Service1.png)
3) In the configure service wizard, choose `Fargate` as the launch type. Or you can change the launch type to capacity provider and choose `Cluster default strategy`. (Default strategy can be configured while updating cluster. click on update button in service page to update it). In the Task defenition dropdown list select the Task definition name you have created. Enter `Aws-App-Svc` as the service name. You can also specify the number of tasks you want to manage using the service. 
![Service](/images/Service2.png) 
4) In the `Deployments` section select `Rolling update` and click `Next` button.
![Service](/images/Service3.png)
5) In the `Configure Network` page, select the `FargateVPC` in VPC list. Choose both the subnets in the VPC to deploy the tasks. Click on the `Edit` button next to the security group to select the existing security group.
![Service](/images/Service4.png)
6) In the `Configure security groups` window, choose `Select an existing security group` option and selec the `Aws-Ecs-Fargate-SG` security group you have created before. Click on the `Save` button.
![Service](/images/Service5.png)
7) In the `Load balancing` section choose `Application Load Balancer`.
![Service](/images/Service6.png)
8) Choose the load balancer name you have created before in the `Load Balancer name` dropdown list.
9) Scroll down to `Container to Load balance` section and click on `Add to Load Balancer`.
![Service](/images/Service7.png)
10) In `Container to load balance` section, select the `Produdction listener port` value as `80:HTTP` and `Target group name` as `ALB-Target` which you have created in Load balancer creation step. Then click on `Next step` button.
![Service](/images/Service8.png)
11) In `Set Auto scaling(optional)` page, choose `Do not adjust the service’s desired count` and click `Next step`.
![Service](/images/Service9.png)
12) In the next page, review the configurations and click on `Create Service`.
When the service creation is completed click on the `View Service` button.
13) You can see the service and task status in the service details page. Wait for a while for the Task status to become `RUNNING`.
![Service](/images/Service10.png)
14) Once your task status is chagned to `RUNNING` you can open the browser and navigate to the DNS name of the Application Load Balancer. You have noted it in the ALB creation step.
![Service](/images/Service11.png)

