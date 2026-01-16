Project: 3-Tier-StudentApp
🏗️ 3 Tier Architecture Diagram
Flow:User → Web Tier (Nginx) → Application Tier (Tomcat) → Database Tier (RDS/MySQL)
Prerequisite:
•	VPC
•	Subnets
•	Route Table
•	Nat Gateway
•	Internet Gateway
•	RDS
Create VPC 
•	Name: my-3-tier-vpc
•	CIDR: 10.0.0.0/16
Create Subnets
1.Subnet-1
•	Name: Public-Subnet-Nginx
•	CIDR: 10.0.0.0/23
2.Subnet-2
•	Name: Private-Subnet-Tomcat
•	CIDR: 10.0.2.0/23
3.Subnet-3
•	Name:Private-Subnet-Database
•	CIDR: 10.0.4.0/23
Create Internet Gateway 
•	Name: MY_IGW
•	attach igw to vpc
Create Nat Gateway 
•	Name: 3-tier-NAT-Gateway
•	create in public subnet
Create Route Table 
1.	Public-RT
o	add public subnet
o	add igw
o	Subnet Association(public-subnet-Nginx)
       2.Private-RT
o	add private subnets
o	add nat
o	Subnet Association(private-subent-Database,private-subnet-Tomcat)
 <img width="1151" height="195" alt="Instnces" src="https://github.com/user-attachments/assets/a4f75b80-8f8a-4d8c-b4cd-72f7fa624b9e" />

Create EC2 Instances 
1.	Public_Nginx_Instance ->create in public subnet ->allow port = 80,22
2.	Private_Database_instance->create in private subnet ->allow port = 8080,22
3.	Private_Tomcat_Instance->create in private subnet ->allow port = 3306,22
 
Create Database In RDS 
•	Go To RDS
•	Created Database
•	Standard create
•	Free tier
•	DB name – database-1
•	Username – admin
•	Password – Passwd123$
•	VPC – VPC-3-tier
•	Connect to Instance -> choose database instance
•	Public access – no
•	A.Z. – no preference
•	Create database
•	Edit security group -> Add 3306 port
 
Connect To Nginx-Instance-Public 
 
•	connect to instance
•	change hostname: sudo hostnamectl set-hostname Nginx-Instance
•	bash
Now SSH into Database Instance
 
sudo -i
yum install mariadb105-server -y
systemctl start mariadb
systemctl enable mariadb
Log in into database
 
Mariadb -u admin -p -h RDS(endpoint)
Note: replace rds-endpoint with actual endpoint value
show databases;
create database  studentapp;
use studentapp;
Run this query to create table:
 CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,  
	student_name VARCHAR(100) NOT NULL,  
	student_addr VARCHAR(100) NOT NULL,   
	student_age VARCHAR(3) NOT NULL,      
	student_qual VARCHAR(20) NOT NULL,     
	student_percent VARCHAR(10) NOT NULL,   
	student_year_passed VARCHAR(10) NOT NULL,  
	PRIMARY KEY (student_id)  
    );
show tables;
Logout from database:
exit
Back to nginx-server
Now SSH into Tomcat Server
 
•	ssh -i hello.pem ubuntu@ip-of-tomcat-vm
sudo -i
sudo apt update
sudo apt install openjdk-11-jdk -y
java –version
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
 cd /tmp
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
sudo mkdir /opt/tomcat
sudo tar -xzvf apache-tomcat-9.0.85.tar.gz -C /opt/tomcat --strip-components=1
go to webapps dir and download .war file(application)
cd /opt/apache-tomcat-9.0.85/webapps
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
cd ../lib
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
MODIFY context.xml
cd apache-tomcat-9.0.85.tar.gz/conf
sudo vim context.xml
add below line [connection string] at line 21
 <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="USERNAME" password="PASSWORD" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://DB-ENDPOINT:3306/DATABASE-NAME"/>
 
cd ../bin
chmod +x catalina.sh
./catalina.sh start
exit
Back to nginx-server
sudo -i
  yum install nginx -y
 vim /etc/nginx/nginx.conf
•	:set nu (enter below data in line 47 in between error and location)
location / {
proxy_pass http://private-IP-tomcat:8080/student/;
}
 
•	:wq ->save file
systemctl start nginx
Go To Browser Hit Public-IP Nginx
 

