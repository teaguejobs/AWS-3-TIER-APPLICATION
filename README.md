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


