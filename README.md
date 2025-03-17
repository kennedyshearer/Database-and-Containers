# Database-and-Containers

## 📌 Project Description  
This project focuses on how manage a database and visualize its data ith tools installed inside containers.

---

## 🛡️ Skills Gained
✅ **Database Integration with Metabase**: Connecting Metabase to a specific database (e.g., 'computer_store') to enable effective data analysis and visualization.

✅ **Metabase Dashboard Creation**: Created interactive dashboards using Metabase to visualize key business statistics, such as most purchased products, top product categories, best customers, and sales revenues.

✅ **SQL Query Formulation**: Writing SQL queries to extract and analyze data from databases, facilitating the generation of meaningful business insights.   

---

## 🔧 Tools Used  
| Tool            | Description |
|----------------|------------|
| **Amazon Virtual Private Cloud (VPC) & Subnets** | A logical isolated network that can lauch AWS resource into, and a subnet is a range of IP addresses with that VPC. |
| **Amazon Relational Database Service (RDS)** | A web service that makes it easier to set up, operate, and scale a relational database in the AWS cloud. |
| **Amazon Elastic Compute Cloud (EC2)** | A web service that lets users run applications in the cloud. |
| **Amazon Elastic Block Store (EBS)** | An easy-to-use, scalable, high-performance block-storage service designed for Amazon Elastic Compute Cloud (Amazon EC2). |
| **Amazon Elastic Container Service (ECS)** | A fully managed container orchestration service that helps you to more efficiently deploy, manage, and scale containerized applications. |
| **Amazon Elastic File System (EFS)** | A fully managed, serverless file storage service for AWS that allows you to share files across multiple AWS compute instances and on-premises resources, scaling automatically to meet your needs.  |
| **Docker** | A software platform that allows you to build, test, and deploy applications quickly. |
| **phpMyAdmin** | A free software tool written in PHP, intended to handle the administration of MySQL, MariaDB, and AWS Aurora over the Web. |
| **Metabase** | An open-source data visualization and exploration tool that empowers users to connect to their data sources, create interactive dashboards, and gain insights from their data with ease. |

---

## 📋 Table of Contents

1️⃣ [Topology](#topology)

2️⃣ [Create a VPC and Subnets](#create-a-vpc-and-subnets)

3️⃣ [Create a RDS MySQL database](#create-a-rds-mysql-database)

4️⃣ [Deploy a ECS Cluster in EC2](#deploy-a-ecs-cluster-in-ec2)

5️⃣ [Task Definition for phpMyAdmin DBMS container](#task-definition-for-phpmyadmin-dbms-container)

6️⃣ [Task Definition for Metabase visualization tool container](#task-definition-for-metabase-visualization-tool-container)

7️⃣ [Add EFS to Metabase container](#add-efs-to-metabase-container)

8️⃣ [Create database structure with phpMyAdmin](#create-database-structure-with-phpmyadmin)

9️⃣ [Create dashboards with Metabase](#create-dashboards-with-metabase)

---

## Topology

<p align="center">
  <img src="https://github.com/kennedyshearer/Database-and-Containers/blob/main/DBcontainers_diagram.png">
</p>

---

## Create a VPC and Subnets

First, in the AWS Management Console, go to *VPC* and **Create VPC**. A page like this should appear:

<p align="center">
<img alt="Create VPC default" src="https://i.gyazo.com/5d79961412f2f6c76aade92b06e982b8.png">
</p>

In **Resources to create**, select the **VPC and more** field, so that you can also configure the VPC subnets.

Give a **Name tag** to the VPC resources to easily differentiate them from the other resources of your other VPCs, in this example the **Name tag** is *project4*.

For the rest, leave the default values as they are perfectly suitable for the project.

The first part of the settings should look like this:

<p align="center">
<img alt="Create VPC" src="https://i.gyazo.com/85238355e97224c762637c2244e6bb2c.png">
</p>

For the next step, choose **2 Availability Zones**. Only one AZ will be needed to complete the project, but in order to create a RDS Database, it is needed to have at least 2 AZ.

> To know why 2 AZ are required for RDS: https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBSubnetGroup.html

Select a **Number of public subnets** of 2, since it will be used to interact with the tools installed in the containers. Do the same for the **Number of private subnets** which will contain only RDS.

Do not add a **NAT Gateway**, as this will be charged (not respecting the Free Tier), and the containers will communicate with the RDS database via an endpoint (unnecessary NAT Gateway).

Do not add an **S3 Gateway** since the S3 service will not be used here.

Enable **DNS hostnames** and **DNS resolution**, an **ESSENTIAL step** for the services to interact with each other (especially EFS).

The second part of the settings should look like this:

<p align="center">
<img alt="VPC Settings" src="https://i.gyazo.com/946698b2ac751970baabf07f43d20f27.png">
</p>

Now create the VPC and wait for its resources to generate:

<p align="center">
<img width="1000" alt="VPC Settings" src="https://i.gyazo.com/2ca9eb9bead582f22c23942035f3e3bb.png">
</p>

Finally, go to *Subnets*, in order to **edit subnets settings** of the **Public Subnets** of the newly created VPC:

<p align="center">
<img width="1000" alt="Public Subnet Settings" src="https://i.gyazo.com/d019ba0f78149a62ac94ca057bd5252e.png">
</p>

The **Auto-assign public IPv4 address** must be enabled at all costs, so that each AWS tool linked to the public subnet automatically gets a public IP:

<p align="center">
<img width="1000" alt="Enable Auto Assign" src="https://i.gyazo.com/0348c9d8cc813b0c5d3a372f81568ba7.jpg">
</p>

Do the same for the other newly created public subnet.

The VPC is ready, it is now possible to create the AWS tools, and the first one created will be RDS.

---

## Create a RDS MySQL database

First, go to *Aurora and RDS* and select **Create database**:

<p align="center">
<img width="1000" alt="Create Database" src="https://i.gyazo.com/16de9da32cf44eaccb804a40df13d303.png">
</p>

Choose the **Standard create** method, in order to have more control over the creation of the database, and select MySQL, which will be the database type used for this project:

<p align="center">
<img alt="MySQL" src="https://i.gyazo.com/1c91838538cbc14b9ae1a6998d9d2f3b.png">
</p>

For the **Template**, AWS offers a **Free tier Template**, select it:

<p align="center">
<img width="1000" alt="Free tier template" src="https://i.gyazo.com/f871d9df59143e8efe8adb23fded6afd.png">
</p>

For the settings, insert the **DB instance identifier** of your choice, for this example it will be *rds-mysql-project4*.

For the **Credentials**, fill in the username and password of your choice, for this example the username will be *admin*, and the password will be a secret of course:

<p align="center">
<img width="1000" alt="Database Settings" src="https://i.gyazo.com/cfb7d796ed5a91997a6195ab33c8bbde.png">
</p>

No need to do anything for the **DB instance class**, the Free Tier Template has selected the right instance for you.
On the other hand for storage, I advise you to disable the **storage autoscaling** since the maximum space offered with the Free Tier for RDS is 20GiB, so you should avoid going beyond the 20GiB already allocated:

<p align="center">
<img width="1000" alt="Instance Configuration" src="https://i.gyazo.com/7d1d6210b52827af2b24ac04f8605d8f.png">
</p>

Now with **Connectivity**, that's where the VPC comes in. Select the newly created VPC, and let the **DB subnet group** be created since there is none by default for this new VPC.

Deny public access, so that RDS is in a private subnet, and therefore not publicly accessible.

For the **VPC security group**, leave the default settings, so that the Security Group associated with the RDS database is the default one of the VPC.

Finally, select preferably the first AZ **1a** of your region, in this example *us-east-1a*:

<p align="center">
<img width="1000" alt="Connectivity" src="https://i.gyazo.com/a529e2d5606c85fd73a2317c379aa68a.png">
</p>

Technically, there is no difference between the AZs, but symbolically, everything that will be done on this project will be done on AZ 1a.

For **Database authentication**, leave as password authentication, in order to connect with the credentials previously filled in:

<p align="center">
<img width="1000" alt="Database Authentication" src="https://i.gyazo.com/41ee30e97abaf322b60a8b7e32c15c45.png">
</p>

Expand the **Additional configuration** window, and only the **Backup** part will have to be modified, disable the **automated backups**.

> The Free Tier offers 20GiB of storage dedicated to RDS backup; this backup space will be used up very quickly after a few weeks. And it is not crucial to back up this database since the stored data is not important at all.

For the rest, leave the default settings which are totally fine for the project:

<p align="center">
<img width="1000" alt="Additional Configuration" src="https://i.gyazo.com/1117011872cb46df42ee2a211bb83fae.png">
</p>

All that remains is to validate the RDS database creation. This one takes a few minutes before being ready for use. Let's move on to deploying an *ECS Cluster* on EC2.

---

## Deploy a ECS Cluster in EC2

---

## Task Definition for phpMyAdmin DBMS container

---

## Task Definition for Metabase visualization tool container

---

## Add EFS to Metabase container

---

## Create database structure with phpMyAdmin

---

## Create dashboards with Metabase

---

