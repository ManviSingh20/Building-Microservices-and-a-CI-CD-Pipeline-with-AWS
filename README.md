# Building-Microservices-and-a-CI-CD-Pipeline-with-AWS

In this project, we’re challenged to use at least 11 AWS offerings, including some that might be new to you, to build a microservices and continuous integration and continuous development (CI/CD) solution. 

By the end of this project, we should be able to do the following:
1. Recognize how a Node.js web application is coded and deployed to run and connect to a relational database where the application data is stored.
2. Create an AWS Cloud9 integrated development environment (IDE) and a code repository (repo) in which to store the application code.
3. Split the functionality of a monolithic application into separate containerized microservices.
4. Use a container registry to store and version control containerized microservice Docker images.
5. Create code repositories to store microservice source code and CI/CD deployment assets.
6. Create a serverless cluster to fulfill cost optimization and scalability solution requirements.
7. Configure an Application Load Balancer and multiple target groups to route traffic between microservices.
8. Create a code pipeline to deploy microservices containers to a blue/green cluster deployment.
9. Use the code pipeline and code repository for CI/CD by iterating on the application design.

# Scenario

The owners of a café corporation with many franchise locations have noticed how popular their gourmet coffee offerings have become. 

Customers (the café franchise location managers) cannot seem to get enough of the high-quality coffee beans that are needed to create amazing cappuccinos and lattes in their cafés. 

Meanwhile, the employees in the café corporate office have been challenged to consistently source the highest-quality coffee beans. Recently, the leaders at the corporate office learned that one of their favorite coffee suppliers wants to sell her company. The café corporate managers jumped at the opportunity to buy the company. The acquired coffee supplier runs a coffee supplier listings application on an AWS account, as shown in the following image.

![image](https://github.com/user-attachments/assets/406ecf64-cfd5-4360-88e3-84b77c93fc44)

The coffee suppliers application currently runs as a monolithic application. It has reliability and performance issues. That is one of the reasons that we have recently been hired to work in the café corporate office. In this project, we perform tasks that are associated with software development engineer (SDE), app developer, and cloud support engineer roles.

We have been tasked to split the monolithic application into microservices, so that we can scale the services independently and allocate more compute resources to the services that experience the highest demand, with the goal of avoiding bottlenecks. A microservices design will also help avoid single points of failure, which could bring down the entire application in a monolithic design. With services isolated from one another, if one microservice becomes temporarily unavailable, the other microservices might remain available.

We have also been challenged to develop a CI/CD pipeline to automatically deploy updates to the production cluster that runs containers, using a blue/green deployment strategy. 

# Approach

<img width="719" alt="Screenshot 2024-09-11 at 7 17 04 PM" src="https://github.com/user-attachments/assets/f1f3752e-09ef-4b28-b091-173f955d5e10">





## Phase 2: Setup the networking & Security Groups required

The step will look like this:

### Networking

Create a VPC with 2 Public & 2 Private subnets and 1 Internet Gateway.
<img width="1145" alt="Screenshot 2024-09-19 at 7 01 12 PM" src="https://github.com/user-attachments/assets/d4314e91-0d80-470b-a9ea-fe40dd900eb6">

### Security Groups
Security Group - RDS
<img width="1368" alt="Screenshot 2024-09-19 at 7 08 26 PM" src="https://github.com/user-attachments/assets/9a4b32be-3a21-4823-ae4c-2a10eb5c4566">

Security Group - EC2
<img width="1387" alt="Screenshot 2024-09-19 at 7 12 52 PM" src="https://github.com/user-attachments/assets/0c80eed5-1bb8-4f4e-a30e-9bd217bffdc1">


## Phase 3: Setup the RDS Database

### Provisioning

<img width="819" alt="Screenshot 2024-09-19 at 7 26 48 PM" src="https://github.com/user-attachments/assets/c19b0505-74ee-4a6a-be5b-cd986619915c">

Setup The RDS Database with following configurations:

- DB Instance Identifier: supplierdb
- DB Engine Version: 8.0.35
- DB Instance Class: db.t3.micro, 2 vCPUs, 1 GiB RAM, Up to 2,085 Mbps network
- Storage: Type: General Purpose SSD (gp2), Allocated Storage: 20 GiB
- Availability & Durability: Multi-AZ Deployment: No
- Connectivity: Network Type: IPv4, Security Group: DBSecurityGroup

## Configure database dump on RDS

Now we have a database dump of the required database, we need to configure it on RDS
To access the RDS we will create a EC2 Instance and Install MySQL Client on it.

EC2 Instance Details:

<img width="658" alt="Screenshot 2024-09-19 at 8 17 12 PM" src="https://github.com/user-attachments/assets/2dc4ea57-ef5f-46d2-873f-73ee16b5e1a1">

*Note: For The Security Group Choose EC2Node Security Group

Connect to the EC2 Instance and run the following commands:
```
sudo apt update
sudo apt install nmap
nmap -Pn <db endpoint> - used to check to verify that the database can be reached from the rdsdatabaseaccess instance on the standard MySQL port number.
sudo apt install mysql-client -y
```

<img width="808" alt="Screenshot 2024-09-19 at 8 23 00 PM" src="https://github.com/user-attachments/assets/37150d99-e96e-4a5b-89d2-624bee1dc66d">

To upload the sqldump file on the EC2 isntance, we first connect to the ec2 machine through our Local Machine using SSH Commands:
```
ssh -i "<"Path to Pem File">" ubuntu@<publicIP>
```

Second, Upload the Dump File to EC2
```
scp -i <"Path to Pem File"> path\to\your_dump_file.sql eubuntu@<public ip>:~
```

<img width="1438" alt="Screenshot 2024-09-19 at 8 44 42 PM" src="https://github.com/user-attachments/assets/ffd3d003-878c-4ad1-b241-718dd60554e4">

Third, upload the dump file to RDS:
```
mysql -h <db endpoint> -u your-username -p
EXIT;
mysql -h <db endpoint> -u your-username -p < coffee_database_dump.sql
```
*Note: You might need to remove some advanced commands, due to some permission issue. (I recommend using GithHub Copilot/ChatGPT/Phind for solving the error) 

<img width="1080" alt="Screenshot 2024-09-19 at 9 23 00 PM" src="https://github.com/user-attachments/assets/09461307-3b1c-48f5-b854-6812e66a189d">

Now the databse is ready.

## Phase 4: Creating a development environment and checking code into a Git repository

### Task 4.1: Create an AWS Cloud9 IDE as your work environment


