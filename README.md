**Building a Resilient Three-Tier Architecture on AWS with Deploying MERN Stack Application**

Achieving High Scalability, High Availability, and Fault Tolerance

**Synopsis**

 We will build Resilient Three-Tier Architecture Deploying the MERN Stack app on AWS: Achieving High Scalability, high availability, and Fault Tolerance. In this project we are going to use multi-region deployment, one will be the primary region and the second will be for disaster recovery. here we are going to follow a Warm standby disaster recovery strategy. so that our user faces lower downtime in case of disaster. let me give you a brief introduction to the project.

 **Story**

 When a user requests the website, Route 53, the DNS service, handles the request and directs it to CloudFront, the CDN (Content Delivery Network), which serves the client. If CloudFront needs to access the web server (frontend) then it routes the request to the Application Load balancer of the web server and that redirects to the web servers. after successfully receiving static pages, the client's browser can make the API call for data. These API calls are routed through Route 53, which sends them to the ALB of the application server (backend server). The ALB then directs the requests to the application server, where data is processed. Additionally, the application server may store some data in the RDS database. and our database is only accessed by the application server. but there is a chance that where we have deployed infrastructure that region goes down because of some kind of disaster. in that case, CloudFront will do the failover for the web tire and Route 53 will do the failover for the application tire. and both start leveraging resources of the DR region. and this is how we will achieve resiliency. if you didn't get an idea then don't worry it will be easy once you see the architecture.

 I am going to use AWS Cloud but you can you whatever cloud you like. more or less steps will be similar.

 **Prerequisites**

- AWS ACOOUNT
- BASIC KNOWLEDGE OF LINUX

**List of AWS services**

🌍 Amazon CloudFront
🌐 Amazon Route 53
💻 Amazon EC2
⚖️ Amazon Autoscaling
🪪 Amazon Certificate Manager
🪣 Amazon Backup service
🗄️ Amazon RDS
☁️ Amazon VPC
🔐 Amazon WAF
👁️ Amazon CloudWatch

**Plan of Execution**

🤔 What is three-tire architecture
🏠 The architecture of the project
🚀 A step-by-step guide with screenshots
🧪 Testing
🧼 Resource cleanup
🥳 conclusion

**🤔 What is Three-tire architecture**

Three-tier architecture is a software architecture pattern that separates an application into three layers.

🔸Presentation layer ➡️ handles user interaction

🔸Application layer(backend logic) ➡️ processes business logic and data processing

🔸Data layer (database) ➡️ manages data storage and retrieval

![alt text](./images/image%201.gif)

Each layer has distinct responsibilities, allowing for modularity, scalability, and maintainability. This architecture promotes the separation of concerns and facilitates easy updates or modifications to specific layers without impacting others.

**🏠 Architecture of the Project**

Let's see the architecture of today's project. we are going to follow a goal-driven approach that helps us to put in minimum effort and gain lots of results. it's very important to understand what we are going to build and to understand you can follow the below architecture. I request you please go through it once. it helps you a lot while building this project.

![alt text](./images/full%20architecture.gif)

**🚀A Step-by-step guide**

we are following a Warm standby Disaster recovery strategy so we are going to utilize two regions during our deployment. us-east-1 AKA North Virginia as primary and us-west-2 AKA Oregon as secondary or DR.

**🔹 VPC**

firstly we are going to set up VPC in both regions to isolate our resources from the internet. The below image contained all the subnets, their IP range, and their uses. you can use your own VPC setup if you have a better idea. and if you are a beginner, please create VPC as I have shown below.

![alt text](./images/image%202.png)

please log in to your AWS Account and type VPC in the AWS console. and click on VPC service.

![alt text](./images/image%203.png)

click on Your VPC's button on the left and then click on Create VPC the button on the top right corner of the page

![alt text](./images/image%204.png)

here we can see the form where we can fill the configuration of VPC. please enter the name that you want to keep and the IPV4 CIDR block. in my case CIDE block is 172.20.0.0/16.

![alt text](./images/image%205.png)

now click on the subnet button which is located on the left side and then click on theCreate subnetbutton on the top right corner of the page.

![alt text](./images/image%206.png)

please remove the default VPC ID and choose the VPC ID that we have just created in the VPC ID field. and click on the Add Subnet button at the bottom.

![alt text](./images/image%207.png)

now we need to configure our subnets. Again you can use the VPC configuration image that I shared earlier on the blog to get the IP range and to know which subnet will be used for what purpose. we are going to create a total of 8 subnets of which 2 of them are public and the rest of 6 subnets are private. you can create a subnet as I have shown in the below image. after adding all the subnets click on Create subnet button.

![alt text](./images/image%208.png)

after the successful creation of all 8 subnets, they look like this. you can verify with my subnets.

![alt text](./images/image%209.png)

now we are going to create Internet Gateway also known as IGW.  it is responsible for communication between VPC, VPC's public subnet with the Internet. without IGW  we won't be able to communicate with the Internet. so let's create that. click on the  internet gateways button at the left panel. and then click on the Create Internet gateways button on the top right corner of the page.

![alt text](./images/image%2010.png)

give any name you wanna give to IGW. and click on Create Internet gateway button.

![alt text](./images/image%2011.png)

after creating an internet gateway we need to attach it with VPC to use it.  for that click on the 	Action button. here you can see the drop-down list. please select the option Attach to VPC.

![alt text](./images/image%2012.png)

please select VPC that we have created just now from the Available VPC list. and then click on the Attach Internet gateway button.

![alt text](./images/image%2013.png)

Now we need to create a NAT gateway. NAT gateway is responsible to connect resources that are in the private subnet to communicate with the internet. all the resources which will be there in a private subnet will communicate to the internet through the NAT gateway. we will keep the NAT gateway in the public subnet so that it can access the internet. NAT gateway is a chargeable resource. so you will be charged by AWS as long as you keep it up. Now to create a NAT gateway click on the NAT gateways button on the left panel of the web page. and then click on the Create NAT gateways button in the top right corner of the page.

![alt text](./images/image%2014.png)

Here please select 0.0.0.0/0 in the destination field and click on the target. As soon as you click on the target you will see the drop-down list. Please select NAT gateway from the drop-down list. As shown in the below image.

![alt text](./images/image%2015.png)

select the NAT gateway that we have just created. and click on the save changes button.

![alt text](./images/image%2016.png)

keep Pri-RT selected and click on the subnet associations tab at the bottom next to the Routes tab. And then click on the Edit route associations button.

![alt text](./images/image%2017.png)

Here you can see the same situation as we saw before. But here we are going to select all the 6 private subnets. And then click on the save association button.

![alt text](./images/image%2018.png)

Before we move ahead I want to change the settings of VPC and two public subnets. So just click on the Your VPC button on the left panel and select VPC that we have created and click on the action button and there you will see the drop-down menu. Select the Edit VPC setting button. As shown in the image.

![alt text](./images/image%2019.png)

And here please enable Enable DNS hostname checkbox by clicking on it. and then click on the Save button

![alt text](./images/image%2020.png)

Please go to the subnet page and select the public subnet and click on the action button and then choose the Edit subnet setting button from the drop-down list.

![alt text](./images/image%2021.png)


Here you have to mark right on Enable public assign public IPV4 address. And then click on the save button

![alt text](./images/image%2022.png)


And here we are done with VPC configuration in the primary region. In my case us-east-1 (N.Virginia). But we have to do the same setup in the secondary region as well. As you know I am going to use the us-west-2 (Oregon) as my second region AKA Oregon. 

Your task is to set up VPC in the secondary region. All the setup is completely similar. You just have to change the region. And please do VPC set up in the secondary region.

I hope you did the setup. Now let’s move ahead

🔹 **Security Groups (SG)**
Security groups are very essential part of the infrastructure. Because it can secure the resources in the cloud. SGs are a kind of firewall that allow or block incoming and outgoing traffic. SGs are applied to the resources like ALB, ec2, rds, etc. One resource can have more than one SG.

So let's first understand. How SG will be used in our architecture and how we are going to apply that. Please see the below image you will get all the ideas. Which resource depends on what. And what are the port numbers we need to allow etc..

![alt text](./images/image%2023.png)

To create SG, click on the security groups tab on the left panel and here you will see the Security Groups button. Note that SGs are specific with VPC. So we can’t use SG which is created in a different VPC. So when you create SG please make sure that you choose the right VPC. click on the crate security button on the top right corner.

![alt text](./images/image%2024.png)


We will create our first SG for bastion-jump-server. Give any name and description you want but please remove the default VPC  and add VPC that we have just created. Then click on the Add rule button in inbound rules. And add SSH rule and add your IP in the destination. Please don’t do anything with the outbound rule if you don't have a good understanding. And then click on the create security group button.

![alt text](./images/image%2025.png)

Now let's create SG for the ALB-frontend. Again steps are similar but add the rule HTTP AND HTTPS from anywhere on the internet because both ALB are internet facing. But please select the right VPC.

![alt text](./images/image%2026.png)

Create SG for ALB-backend. ALB-backend is also internet-facing. Again allow HTTP and HTTPS from anywhere

![alt text](./images/image%2027.png)

create SG for frontend servers. Our fronend server will be in a private subnet so add the HTTP rule and select the source as ALB-frontend-sg.  So only ALB-frontend can access the frontend server on port 80. You have to add one more rule SSH allows from bastion-jump-server-sg. So that the bastion host can log in to web servers.

![alt text](./images/image%2028.png)


Let's create SG for the backend server. Again steps are completely similar to frontend-sg. You have to allow port 80 from ALB-backend-sg so that only ALB-backend can request to the backend server and add the rule SSH allows from bastion-jump-server-sg. So that the bastion host can log in to backend servers.

![alt text](./images/image%2029.png)

Lastly, we are going to create SG for RDS instance. Allow port 3306 MySql/Arrora from backend-sg so that only the backend server will be able to access it. and no one else can access our database.

![alt text](./images/image%2030.png)

And here our SG setups are complete now. Your task is to do the complete same setup for the secondary region. In my case, it is Oregon (us-west-2).

![alt text](./images/image%2031.png)


Now first we need to set up a subnet group. It specifies in which subnet and Availability zone out database instance will be created. So click on the subnet group button on the left panel. And click on the button Create database subnet group which is in the middle of the web page.


![alt text](./images/image%2032.png)

Here we can configure our VPC, subnet, and availability zone.  Give any name to your subnet but make sure you select the correct VPC. and select Azs us-east-1a and us-east-2b. According to the architecture that I have shown you, our database will be in private subnet  pri-sub-7a and pri-sub-8b. so please select as I have shown in the below figure. And then click on the create button.

![alt text](./images/image%2033.png)

Note: we need to create a subnet group in the Oregon region as well.  All the configuration is similar to the above.  just need to change the Availability zone us-west-2a and us-west-2b

![alt text](./images/image%2034.png)

Now come to the N.virginia region and here we are going to create a database. So click on the database button on the left panel and then click on the created database button.

![alt text](./images/image%2035.png)

On this page, we can configure our database. Select stander create because I’m going to show you each and every step.  select MySQL in the engine option because our application runs on MySQL database. If your app runs on other engines you can select that one. Furthermore, you can select the engine version my application is compatible with MySQL version. But you can select according to the developer guild.

![alt text](./images/image%2036.png)

Scroll down, and select Dev/test as  template. If you select the free tier then you won’t be able to deploy RDS in a multi-availability zone.  Select Multi-AZ DB instance from availability and durability option. In settings give any name to your database.  In the credential setting give the username of the database in the Master username field and give the password in the Master password field. And then confirm the password below. Please do remember your username and password.

![alt text](./images/image%2037.png)

Again scroll down, select Brustable class in the instance setting and select the instance type. Actually, it depends on your application uses. But for learning purposes, I am selecting t3.micro.  now in storage type select General purpose(GP2) and allocate 22 GiB for database. Please uncheck the auto-scaling option to keep our costs low.  And In the connectivity option please select the option according below screenshot.

![alt text](./images/image%2038.png)


In VPC, select VPC that we created earlier and in DB subnet group select the group that we just created, In the public access option please select No, choose existing security, and select security group book-rds-db.

![alt text](./images/image%2039.png)

Scroll down, click on Additional Configuration, and in the database option give the name test because we need a database with the name of the test in the application.  Enable Automated Backup. Note: you have to enable automated backup otherwise you won’t be able to create a read replica of the RDS instance.

![alt text](./images/image%2040.png)

Scroll down, mark on enable encryption checkbox to make the database bit more secure, and click on Create database button below.

![alt text](./images/image%2041.png)

![alt text](./images/image%2042.png)

Note: RDS take 15-20 minute because it creates a database and then take a snapshot. So please have patience and wait for it to be ready.

After your database is completely ready and you see the status Available then select the database and click on the Action button. There you can see the drop-down list. Please click on created read-replica.

![alt text](./images/image%2043.png)

This page is similar to creating a database. In the AWS region select the region where you want to create the read replica. In my case, It is Oregon (us-west-2).  Give a name to your read replica, and select all the necessary configurations that we did before while creating the database. For your reference, I have shown everything in the below images.

![alt text](./images/image%2043.png)

![alt text](./images/image%2044.png)

![alt text](./images/image%2045.png)

![alt text](./images/image%2046.png)

Once you click on the button create replica . It will start creating that.

![alt text](./images/image%2047.png)

You can check your read replica on the specified region’s RDS dashboard. So let me head over to Oregon and show you the read replica.

![alt text](./images/image%2048.png)

**Note: we can’t write anything into a read replica. It is just read-only database. So when a disaster happens we just have to promote read replica so that it becomes the primary database in that region.**

Now we are going to utilize route 53 service and create two private hosted zone. One for north Virginia(us-east-1) and another one for Oregon region (us-west-2)  with the same name.  you may think Why Two hosted zone with the same name? don’t worry I will answer it later. So head over to Route 53.  Type route 53 in the AWS console. And click on the service.


![alt text](./images/image%2050.png)


















