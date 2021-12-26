## Deploying a simple “Hello world” httpd container on an ECS cluster
```
    Hello Tech Maniacs !
    Hope you all are doing awesome.
    Today we gonna learn to deploy a httpd “hello world” container over Amazon platform using Amazon ECS service. 
    It is quite easy to spin up everything here, thanks to the user-friendly UI provided by the Amazon guys.
    Before starting this blog, if you want to know about the basics of what is ECS and ECR are at a very high level, then please go through my below blog.
    I hope you will find this helpful: https://mohitshrestha02.medium.com/how-to-login-to-amazon-ecr-and-store-your-local-docker-images-with-an-example-9aa845c4134c

    Now before starting with how to deploy your “hello world” container on ECS cluster , you need to create an image in your local, then configure your local to have AWS ECR(ELASTIC CONTAINER REGISTRY) login access and finally push your “httpd container” from your local all the way to ECR to get used by ECS cluster. 
    So, I have already created a blog for that as well, please go through this blog to learn how to push your image from local to ECR.
        https://mohitshrestha02.medium.com/how-to-login-to-amazon-ecr-and-store-your-local-docker-images-with-an-example-9aa845c4134c

    Now since you must have already pushed your image to ECR by this time, its time to move it to ECS cluster and spin up a task. 
    We will go step by step :

        1. Create an “ECS cluster” (either with an instance or serveless), configure your vpc subnets and all.

        2. Create a “Task definition” and define your container image from ECR, expose ports, provide cpu units and memory to be used by your container.

        3. Finally create a “Service” which will use your defined task definition to spin up a task, maintain the desired no of tasks u want, configure auto-scaling, use load-balancer to be used ,configure target groups and once finished wait for your task to come UP.
    
    So here how it looks:
```

## ECS CLUSTER:
```
    1. Login to your AWS console and search for ECS . 
    From the left hand side dashboard choose the Clusters option and choose “Create Cluster” tab.
    
    2. You will get 3 options as below, If you want to go with Instance type cluster then choose any one of the option (EC2 Linux/EC2 windows) or if you want to have a Server-less cluster choose the Networking only (FARGATE) option.
    I have chosen EC2 Linux+Networking for this blog.

    3. Then start configuring your cluster. 
    Provide cluster name, instance type, No of instances you want in this cluster, EBS storage(minx 22GB), key-pair to login.While configuring Networking part, either you can choose a default VPC for your cluster or you can create a new one , in my case I am creating a new one, provide security groups if you have already or use the new one,provide IAM roles and finally don’t forget to do the tagging part ! and then hit create.

    4. Once you hit create , it will take some time to spin up everything and to make your cluster go live .
    It will show this once it creates your cluster:

```

## TASK DEFINITION:
```
    1. Now from the left side Dashboard , choose the option for Test Definitions and select the tab of “Create new Task Definition”.

    2. Now choose one of the following option from FARGATE or EC2 Launch type. 
    In my case I have chosen EC2 launch type since my cluster is using EC2+Networking launch type cluster.

    3. Now configure your task definition , provide task definition name( in our case its “test-task-definition”), choose the Task role and the Task execution role, if already created(like in my case) or create a new one and choose the network type, in my case its “bridge” network type. 
    FYI if you choose the “Default” network mode then it will be using “bridge” only. 
    Since “bridge” is the default network type.

    4. Now choose Add container and add a container from ECR(Elastic container registry).

    5. Now after adding container, you will get the following screen to configure your container, provide all the details like container name , Image name( which is the url you will get from “YOUR ECR repository” and it is mentioned in point no. 6 also here), expose container port, host port , provide memory and cpu units.

    6. From the left side Dashboard, choose the option “AMAZON ECR” and select your repository and copy the “IMAGE URL”. 
    In my case the repository name in ECR is “test” .

    7. So once you have configured everything in your task definition you can go and create your Task definition and it will show this on successful creation.
```

## SERVICE:

```
    1. Now its time to create service. 
    The catch is you won’t find this option directly from the left side Dash . 
    You need to select the “Clusters” option from the left side dashboard .
    Choose your cluster name , in my case its “My-test-cluster”, there you will find a tab at the bottom called as “Services”. 
    Now hit that to create a service.

    2. Load balancer:
        Configure your load balancer: Opt for an application load balancer . 
        Create an application load balancer and configure the listener to listen to port 80. 
        Create your target group and the catch is “DO NOT REGISTER ANY TARGET”. 
        When you are finished configuring your service , it will create tasks and automatically register it to your LOAD BALANCER TARGETS.

    3. Now configure the service,again select the EC2 launch type since its an EC2 cluster, choose the task definition whichever you want to use to spin up the task from( in our case its “test-task-definition”, choose that), choose our cluster name that your created before(My-test-cluster), give a service name (I have created one by the name “my-test-service”).

    Next I have chosen “Replica” as service type since Replica helps you maintain a desired count of tasks, choose the desired no of task u want to spin and min and max healthy percentage.
    Choose the deployment type and your desired Placement templates.
    ( I have chosen “Rolling updates” and “AZ balanced spread” respectively) and click next.

    4. Now in the next page directly come to Load balancing section, you can either opt for Load balancer or you can set it to none if you dont want it.
    In our case we are going for an “application load balancer”.
    Choose the Service IAM role if already created or create a new one. 
    Choose the load balancer that you have created before from the drop down menu.

    Choose Add to loadbalancer to tab to add the configuration to your load balancer so that it ECS can automatically register the ECS tasks to your load balancer targets, and uncheck the Service discovery(optional),since we dont want to create any namespace via Route53, to make it simple.

    When you click Add to Loadbalancer you will get the following window. 
    Select the production listener port which is 80:HTTP in our case since we already created a listener and select your target group name from the drop down menu since we have already created a target group as well before.If not, you can also create a new listener and new target group.
    and then hit Next

    5. Next page , It will ask to configure your autoscaling part!

    Next page it will show you everything to review what you created hitting Create will create the service and simultaneously result in spinning up new task as per your TASK-DEFINITION container.

    6. Cross check if it results in spinning up new tasks or not?

    Before this dont forget to check if your LOAD BALANCER security group’s port 80 is opened to the world or not and make sure your EC2 security group’s port 80 inbound should allow traffic from the security group of LOAD Balancer.

    NOW go and hit your loadbalancer DNS , it should display the content of our containers running via ECS.

        Load balancer port 80 is requesting on port 80 of HOST(EC2 server) and HOST port 80 is requesting container’s port 80 which is running as a task under ECS and displaying the content finally at the LOAD BALANCER DNS.The catch is to open the correct port numbers in the security groups of EC2 and ELB.
    
    I hope this blog helped you in deploying your application on ECS. 
    Dont forget to like and share this, if you really enjoyed reading it .
```

- Stay tuned for more awesome blogs, Cheers !
