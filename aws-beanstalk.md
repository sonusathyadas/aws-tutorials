# Deploying web applications using Elastic Beanstalk

AWS *Elastic Beanstalk* is a PaaS service that allows you to deploy your web applications developed in your favourite language/framework such as .NET, Java, PHP, NodeJS, Python etc on a web server without configuring the infrastructure services explicitly. Instead of creating the compute, storage, networking and identity solutions on your own, Elastic Beanstalk provides a preconfigured platform which can be used to deploy your applications from a developer machine or using a CI/CD tool.

Elastic Beanstalk offers the features such as:
    
1) Quicker deployment
2) Simplifies operations
3) Cost effectiveness
4) Support for multiple languages and frameworks
5) Autoscaling and monitoring 
    
### Components of Elastic Beanstalk
 - **Application** : An *application* is a logical collection of Elastic Beanstalk components, including environments, versions, and environment configurations. In Elastic Beanstalk an application is conceptually similar to a folder.
 - **Application version**: An *application version* refers to a specific, labeled iteration of deployable code for a web application. An application version points to an Amazon Simple Storage Service (Amazon S3) object that contains the deployable code, such as a Java WAR file. Applications can have many versions and each application version is unique. In a running environment, you can deploy any application version you already uploaded to the application, or you can upload and immediately deploy a new application version.
 - **Environment**: An *environment* is a collection of AWS resources running an application version. Each environment runs only one application version at a time, however, you can run the same application version or different application versions in many environments simultaneously. When you create an environment, Elastic Beanstalk provisions the resources needed to run the application version you specified.
 - **Environment tier**: When you launch an Elastic Beanstalk environment, you first choose an environment tier. The environment tier designates the type of application that the environment runs, and determines what resources Elastic Beanstalk provisions to support it. Tier can be *web server environment tier* for http web apps  or *worker environment tier* for background tasks.
 - **Environment Health** : Elastic beanstalk reports the health of the web server environment  depending on how the application running responds to the health checks. It uses four colors to describe the status. *Grey*- currently being updated, *Green* - Passed recent health check, *Yellow* - Failed one or more health checks, *Red* - Failed three or more checks.
 
### Elastic Beanstalk architecture
    
   ![aeb-architecture2](images/aeb-architecture2.png)

## Deploying application to Elastic Beanstalk

1) Open AWS console and click on Services menu and search for `Elastic Beanstalk`. From the search results choose `Elastic Beanstalk` and you will be navigated to the Beanstalk dashboard.
2) Click on the `Create Application` button to start creating your first `Beanstalk` application on AWS.
3) In the create application wizard, specify the name of the application as `sample-webapp` and choose the platform type as `.NET Core on Linux`. You can choose any other framework of your choice. For this demo we are selecting .NET Core. Select `Sample application` from the `Application code` section and click on `Create application` button.

    ![aeb-create](images/aeb-create.png)

4) You application will be created in few seconds. It creates a new Beanstalk application and a default environment in the application. You can use this environment to deploy your application.Click on the application name from the list of applications.

    ![aeb-list](images/aeb-list.png)

5)  You can optionally, create additional environments within the application. To launch the application click on the URL of the default environment created. 

    ![aeb-list](images/aeb-env-list.png)

6) This will show the default page of the Elastic Beanstalk application.

    ![aeb-app-page](images/aeb-app-page.png)

> [!NOTE]
> Your Beanstalk application will be deployed in the default VPC of the selected region. When the deployment is completed it creates a *Security Group* for the created application instance

### Publishing .NET Core application to Beanstalk environment
1) Create a new .NET Core MVC application using the .NET Core CLI command.
    > dotnet new mvc -n SampleMVC
2) Move to the application folder and compile the project. 
    > cd SampleMVC
    > dotnet build
3) Run the application and test it locally by navigating to https://localhost:5001
    > dotnet run
4) Ensure the application is running successfully and then publish the application to a folder.
    > dotnet publish -o sampleweb-dist -c release
5) Move to the `sampleweb-dist` folder  and compress the contents using any zip utility to generate `sampleweb-dist.zip` file.
6) Open the dashboard of Elastic Beanstalk and navigate to the default environment configuration of the application. Click on the `Upload and deploy` button.
    
    ![aeb-deploy1](images/aeb-deploy1.png)

7) Click on the `Choose file` button and upload the zip file of your published application. A default version value will be displayed in the `Version Label` text box. You can update the version value if you wish. Click on the `Deploy` buttton to start deploying the application.

    ![aeb-deploy2](images/aeb-deploy2.png)

8) Once the deployment is completed, navigate to the application URL and refresh the page. You will see the application running on your Elastic Beanstalk environment.

    ![aeb-deploy3](images/aeb-deploy3.png)

### Enable scaling for the Elastic Beanstalk application
By default, Beanstalk assigns a `single t2.micro` instance type to the application. If you want, you can update the instance size of the application. To enable autoscaling based on a metric you need to convert the single instance application into a `Autoscaling group (Load balanced)`.
#### Scale up (Vertical Scaling)
1) Open the Elastic Beanstalk application dashboard, Click on the `configuration` menu. This will show a list of configurations for your application. To scale the application instance, click on the `Edit`  button for the `Capacity` configuration.
    
    ![aeb-scaling1](images/aeb-scaling1.png)

2) In the `Capacity` configuration page, scroll down to `Instance Type` configuration value and change the type of the EC2 instance from `t2.micro` to some other instance type and click on `Apply` button.

    ![aeb-scaling2](images/aeb-scaling2.png)

#### Scale out (Horizontal Scaling)

1) To enable autoscaling for the application, you need to move the application into `Autoscaling group`. For that you can change the `Environment Type` value to `Load balanced` in the Capacity configuration page.

    ![aeb-scaling3](images/aeb-scaling3.png)

2) Configure the minimum and maximum number of instances for your application. You can configure the numbers in the `Min` and `Max` textboxes below the `Environment Type`. 
3) To configure a scaling condition, scroll down to the `Scaling Triggers` section and choose a metric type from the dropdown list. We can select `CPUUtilization` as the metric value for this demo. Set statistic value as `Average` and Unit as `Percent`. Specify the period and breach duration values as 5. Set upper threshold value for CPU utilization as `70%` and Lower threshold as `40%`. Also specify the scale up increment and scale down increment values as `1`. Click on `Apply` button to save the changes.

    ![aeb-scaling4](images/aeb-scaling4.png)

4) A confirmation page will be displayed. While configuring autoscaling with Autoscaling group your current instances will be replaced. Click on the `Confirm` button to update.

    ![aeb-scaling5](images/aeb-scaling5.png)

### Clone the application environment
1) Open the environment dashboard and click on the `Actions` button. Select `Clone environment` from the dropdown menu.

    ![aeb-clone1](images/aeb-clone1.png)

2) Specify the name of the new environment. Also confirm the availability of the domain name. Click on `Clone` button to create a clone of the environment.

    ![aeb-clone2](images/aeb-clone2.png)

3) It will take few minutes to clone the environment. Once completed you will be able to see the list of environments in the current application including the cloned one.

    ![aeb-clone3](images/aeb-clone3.png)

4) You can navigate to the new application environment using the URL of the cloned environment.

### Swapping the environment URLs

AWS Beanstalk provides an option to swap the environment URLs. This is a useful feature of Elastic Beanstalk which allow you to publish an updated version of the application into the production environment withoout no downtime. You can publish the updated version into a *staging* environment and swap it with *production* environment.

1) To demonstrate this we can create a *staging* environment. For that, go to the `sample-webapp` application and click on the `Create a new environment` button.Create a `Web Server environment` and specify the name of the environment as `SampleWebapp-staging`. Also, provide a domain name as `sample-webapp-staging` and Platform type as `.NET Core on Linux`. Click `create new environment` button to create it.

    ![aeb-swap1](images/aeb-swap1.png)

2) After creating the new environment, navigate to the environment dashboard and deploy a new version of the application into the staging environment.

    ![aeb-swap2](images/aeb-swap2.png)

3) Once the deployment is completed, navigate to the new version of the application by clicking on the staging environment URL.

    ![aeb-swap3](images/aeb-swap3.png)    

4) Click on the `Actions` button in the staging environment dashboard and select `Swap environment URLs`. 
5) In the swap environment configuration page, select the environment name from `Select an environment to swap` section to `sample-webapp-prod`.
    
    ![aeb-swap4](images/aeb-swap4.png)

6) After the swap operation is completed, you can navigate to the production environment URL and refresh the page. It will show the updated version of the application.

    ![aeb-swap5](images/aeb-swap5.png)