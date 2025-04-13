# **System Deployment Guide**

## **Table of Contents**

* [1\. MySQL RDS Setup](#bookmark=id.hsnimvy3lnoy)  
* [2\. ElastiCache Valkey Setup](#bookmark=id.ifv21tdtj94q)  
* [3\. ECS Cluster Setup](#bookmark=id.7mosy6x57xw6)  
* [4\. General Service Deployment](#bookmark=id.saflcptyewfg)  
* [5\. Video Ranking Service Deployment](#bookmark=id.icp6kt4l7dcd)  
* [6\. Testing the Video Ranking Service](#bookmark=id.7wnb0cqzkkh4)

## **1\. MySQL RDS Setup**

* **Service:** Amazon Relational Database Service (RDS)  
* **Engine:** MySQL  
* **Version:** 8.x

### **Steps:**

1. **Create an RDS Instance:**  
   * Navigate to the Amazon RDS service in the AWS Management Console.  
   * Choose "Create database".  
   * Select "Standard create".  
   * For "Engine options", choose "MySQL" and version "8.x".  
   * Under "Templates", select "Free tier" (if available and applicable) or "Production" for a more robust setup.  
2. **Configure Settings:**  
   * **DB instance identifier:** Provide a name for your database instance (e.g., vrs-mysql-db).  
   * **Credentials Settings:**  
     * **Master username:** Choose a username (e.g., admin).  
     * **Master password:** Set a strong password. **Important:** Store this password securely.  
3. **Choose a DB instance size:** Select an appropriate instance type. For testing, a db.t3.micro may suffice.  
4. **Storage:** Configure storage settings. The defaults are usually fine for initial setup.  
5. **Connectivity:**  
   * **Virtual Private Cloud (VPC):** Select the VPC where you want to launch the database.  
   * **Subnet Group:** Choose an appropriate subnet group.  
   * **Public access:** For easy demonstration purposes, you can set this to "Yes". **Security Warning:** In a production environment, it is *highly recommended* to set this to "No" and configure appropriate security groups to control access.  
   * **VPC security group:** Create a new security group or select an existing one that allows inbound traffic on port 3306 (the default MySQL port) from your ECS cluster or other authorized sources.  
6. **Additional configuration:**  
   * **Initial database name:** You can specify an initial database name here (e.g., vrs\_db). Alternatively, you can create the database later using a MySQL client.  
   * Other settings: Configure backup, monitoring, and other options as needed. The defaults are generally acceptable for initial testing.  
7. **Create the database:** Review your settings and click "Create database".  
8. **Get the Endpoint:** Once the database is created, select the instance in the RDS console and note down the **Endpoint** from the "Connectivity & security" tab. You will need this to configure the application.  
9. **(If you did NOT create a database in the previous step):** Connect to the database instance using a MySQL client (e.g., MySQL Workbench, DBeaver, or the mysql command-line tool) using the endpoint, username, and password you noted down. Create the database:  
   CREATE DATABASE vrs\_db;


![image](https://github.com/user-attachments/assets/3bc41eba-263a-4e2d-8d6a-b5470e85eedb)


## **2\. ElastiCache Valkey Setup**

* **Service:** Amazon ElastiCache  
* **Engine:** Valkey  
* **Version:** 8.x

### **Steps:**

1. **Create an ElastiCache Cluster:**  
   * Navigate to the Amazon ElastiCache service in the AWS Management Console.  
   * Choose "Create".  
   * For "Cluster engine" select "Valkey".  
   * For "Engine version" select 8.x  
2. **Configure Cluster Settings:**  
   * **Cluster name:** Provide a name for your cluster (e.g., vrs-valkey-cluster).  
   * **Node type:** Choose an instance type (e.g., cache.t3.micro for testing).  
   * **Number of nodes:** Start with 1 node for a simple setup.  
   * **Subnet Group:** Select a subnet group in your VPC.  
   * **Security group:** Create or select a security group that allows inbound traffic on the Valkey port (default: 6379\) from your ECS cluster.  
3. **Configure Advanced Settings:**  
   * **Port:** The default Valkey port is 6379\.  
   * Configure other settings as needed.  
4. **Create the cluster:** Review the settings and click "Create".  
5. **Get the Endpoint:** Once the cluster is created, find the endpoint in the ElastiCache console, usually under "Connect to your cache". It will look something like vrs-valkey-cluster.xxxxx.us-east-1.amazonaws.com:6379. Note this endpoint.

## **3\. ECS Cluster Setup**

* **Service:** Amazon Elastic Container Service (ECS)

### **Steps:**

1. **Create an ECS Cluster:**  
   * Navigate to the Amazon ECS service in the AWS Management Console.  
   * Choose "Clusters" in the left navigation pane.  
   * Click "Create cluster".  
   * Select "EC2 Linux \+ Networking" or "Fargate (serverless)" depending on your needs. For this guide, we'll assume "EC2 Linux \+ Networking" for more control.  
   * **Cluster name:** Provide a name (e.g., vrs-ecs-cluster).  
   * Configure networking (VPC and subnets).  
   * Configure EC2 instance type and number.  
   * Configure security groups to allow traffic.  
   * Click "Create".


![image](https://github.com/user-attachments/assets/15dfcd54-f07b-4a94-8337-44fca97ac508)


## **4\. General Service Deployment**

* **Service Name:** General Service  
* **Container Image:** dailoi2807/vrs-general-service:latest  
* **Container Port:** 9001

### **Steps:**

1. **Create a Task Definition:**  
   * In the ECS console, choose "Task Definitions" in the left navigation pane.  
   * Click "Create new Task Definition".  
   * Select "EC2" launch type if you created an EC2 cluster, or "FARGATE" if you created a Fargate cluster.  
   * **Task Definition Name:** vrs-general-service-task  
2. **Configure Container:**  
   * Click "Add container".  
   * **Container name:** general-service-container (or any name you prefer).  
   * **Image:** dailoi2807/vrs-general-service:latest  
   * **Port mappings:**  
     * Container port: 9001  
     * Protocol: GRPC  
3. **Configure Environment Variables:**  
   * Add the following environment variables with the values you obtained in the previous steps:  
     * DB\_HOST: The endpoint of your MySQL RDS instance (without the port).  
     * DB\_PORT: 3306 (or the port you configured for MySQL).  
     * DB\_NAME: vrs\_db (or the name of your database).  
     * DB\_USER: The master username for your RDS instance (e.g., admin).  
     * DB\_PASS: The master password for your RDS instance.


![image](https://github.com/user-attachments/assets/1521d0fc-32e3-42bb-8022-8e83d7c11705)


4. **Create the Task Definition:** Click "Add", then "Create".  
5. **Create a Service:**  
   * In the ECS console, choose "Clusters", then select your cluster (vrs-ecs-cluster).  
   * In the "Services" tab, click "Create".  
   * **Service name:** general-service  
   * **Task Definition:** Select the task definition you just created (vrs-general-service-task).  
   * **Launch type:** Choose "EC2" or "FARGATE" to match your cluster.  
   * **Number of tasks:** Start with 1\.  
   * **Deployment type**: Rolling update  
   * **Networking:**  
     * **VPC:** Select your VPC.  
     * **Subnets:** Select the subnets where you want to launch the service.  
     * **Security group:** Select the security group that allows inbound traffic on port 9001 from other services within your VPC.  
6. **Configure Service (Optional):**  
   * You can configure load balancing, auto-scaling, and other options. For this basic deployment, the defaults are often sufficient.  
7. **Create the service:** Review the settings and click "Create Service".  
8. **Get the Private Endpoint:**  
   * Once the service is running, go to the "Tasks" tab within the general-service service.  
   * Click on the task ID.  
   * Note the "Private IP" address of the task. You will need this to configure the videos-ranking-service. Include the port: \<general-service-private-ip\>:9001

## **5\. Video Ranking Service Deployment**

* **Service Name:** Video Ranking Service  
* **Container Image:** dailoi2807/vrs-ranking-service  
* **Container Port:** 9000

### **Steps:**

1. **Create a Task Definition:**  
   * In the ECS console, choose "Task Definitions".  
   * Click "Create new Task Definition".  
   * Select "EC2" launch type if you created an EC2 cluster, or "FARGATE" if you created a Fargate cluster.  
   * **Task Definition Name:** vrs-ranking-service-task  
2. **Configure Container:**  
   * Click "Add container".  
   * **Container name:** ranking-service-container  
   * **Image:** dailoi2807/vrs-ranking-service:latest  
   * **Port mappings:**  
     * Container port: 9000  
     * Protocol: tcp  
3. **Configure Environment Variables:**  
   * Add the following environment variables with the values you obtained in the previous steps:  
     * REDIS\_HOST: The endpoint of your ElastiCache Valkey cluster (without the port).  
     * REDIS\_PORT: 6379 (or the port you configured for Valkey).  
     * REDIS\_DB: 0 (or the Redis database number you are using)  
     * REDIS\_USER: If you set up a user.  
     * REDIS\_PASS: If you set up a password.  
     * GENERAL\_SERVICE\_ENDPOINT: The private IP address and port of the general-service you deployed earlier (e.g., \<general-service-private-ip\>:9001).  
4. **Create the Task Definition:** Click "Add", then "Create".  
5. **Create a Service with Application Load Balancer (ALB):**  
   * In the ECS console, choose "Clusters", then select your cluster.  
   * In the "Services" tab, click "Create".  
   * **Service name:** videos-ranking-service  
   * **Task Definition:** Select the task definition you just created (vrs-ranking-service-task).  
   * **Launch type:** Choose "EC2" or "FARGATE" to match your cluster.  
   * **Number of tasks:** Start with 1\.  
   * **Deployment type:** Rolling update  
   * **Networking:**  
     * **VPC:** Select your VPC.  
     * **Subnets:** Select the subnets where you want to launch the service.  
     * **Load balancing:**  
       * Select "Application Load Balancer".  
       * **Load balancer name:** Provide a name (e.g., vrs-ranking-alb).  
       * **Listener port:** 9000  
       * **Target group name:** Provide a name (e.g., vrs-ranking-tg).  
       * Ensure that the ALB's security group allows inbound traffic on port 9000 (and 80 if you want to set up a redirect) from the internet. The ECS service's security group should allow inbound traffic from the ALB on port 9000\.


![image](https://github.com/user-attachments/assets/bfb1d20b-0250-4015-805a-4c66351dba4c)


6. **Configure Service (Optional):**  
   * You can configure auto-scaling and other options.  
7. **Create the service:** Review the settings and click "Create Service".

## **6\. Testing the Video Ranking Service**

1. **Get the ALB Endpoint:**  
   * In the EC2 console, navigate to "Load Balancing" \-\> "Load Balancers".  
   * Select the ALB you created for the videos-ranking-service (e.g., vrs-ranking-alb).  
   * Note the "DNS name" (e.g., vrs-ranking-alb-xxxxxxxxxx.us-east-1.elb.amazonaws.com). This is your ALB endpoint.  
2. **Test the Swagger Document:**  
   * Open a web browser and go to the following URL, replacing the ALB endpoint with your actual endpoint:  
     http://\<your-alb-endpoint\>:9000/swagger/index.html  
   * You should see the Swagger UI for the videos-ranking-service API. You can use this interface to test the API endpoints.


![image](https://github.com/user-attachments/assets/309daa7a-c046-43bb-bca6-4933fdedf169)
