Project:3-tier-Student

Prerequisite:
VPC
Subnets
Route Table
Nat Gateway
Internet Gateway
RDS
Create 
VPC 
Name: VPC-3-tier
CIDR: 192.168.0.0/16
Create Subnets
1.Subnet-1

Name: Public-Subnet-Nginx
CIDR: 192.168.1.0/24
2.Subnet-2

Name: Private-Subnet-Tomcat
CIDR: 192.168.2.0/24
3.Subnet-3

Name:Private-Subnet-Database
CIDR: 192.168.3.0/24
Subnet-4
Name:Public-Subnet-LB
CIDR: 192.168.4.0/24
Create Internet Gateway 
Name: IGW-3-tier
attach igw to vpc
Create Nat Gateway 
Name: NAT-3-tier
create in public subnet
Create Route Table 
RT-Public-Subnet
add public subnet
add igw
RT-Private-Subnet
add private subnets
add nat
vpc-flow
<img width="940" height="251" alt="image" src="https://github.com/user-attachments/assets/ac9e1a22-5b1c-41f3-a308-48c2adaa0105" />

Create EC2 Instances 
Nginx-Server-Public ->create in public subnet ->allow port = 80,22
Tomcat-Server-Private ->create in private subnet ->allow port = 8080,22
Database-Server-Private ->create in private subnet ->allow port = 3306,22
instances

Create Database In RDS 
Go To RDS
Created Database
Standard create
Free tier
DB name – database-1
Username – admin
Password – Passwd123$
VPC – VPC-3-tier
Connect to Instance -> choose database instance
Public access – no
A.Z. – no preference
Create database
Edit security group -> Add 3306 port
database

Connect To Nginx-Server-Public 
nginx-server

connect to instance

change hostname

create file with name 3-tier-key.pem

vim 3-tier-key.pem
copy private key and paste it here

Now SSH into Database Server
database-instance

sudo -i
yum install mariadb105-server -y
systemctl start mariadb
systemctl enable mariadb
Log in into database
login into database

mysql -h rds-endpoint   -u admin -pPasswd123$
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
tomcat-server

ssh -i 3-tier-key.pem ec2-user@ip-of-tomcat-vm
sudo -i
sudo apt update
sudo apt install default-jdk
sudo apt install default-jre
java -version
curl -O https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.107/bin/apache-tomcat-9.0.107.tar.gz
tar -xzvf apache-tomcat-9.0.109.tar.gz -C /opt/
go to webapps dir and download .war file(application)

cd /opt/apache-tomcat-9.0.109/webapps
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
cd ../lib
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
 MODIFY context.xml
cd apache-tomcat-9.0.107/conf
vim context.xml
add below line [connection string] at line 21

 <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="USERNAME" password="PASSWORD" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://DB-ENDPOINT:3306/DATABASE-NAME"/>

image

cd ../bin
chmod +x catalina.sh
./catalina.sh start
exit
Back to nginx-server
sudo -i
  yum install nginx -y
 vim /etc/nginx/nginx.conf
:set nu (enter below data in line 47 in between error and location)
location / {
proxy_pass http://private-IP-tomcat:8080/student/;
}
image

:wq ->save file
systemctl start nginx
Go To Browser Hit Public-IP Nginx
nginx-output

register-students
