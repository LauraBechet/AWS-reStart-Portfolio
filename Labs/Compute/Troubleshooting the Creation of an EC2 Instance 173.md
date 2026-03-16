<h2>Troubleshooting the Creation of an EC2 Instance</h2>

<h3>Overview</h3>

<p>
In this activity, the AWS Command Line Interface (AWS CLI) is used to launch 
Amazon Elastic Compute Cloud (EC2) instances.
</p>

<p>
When creating the instance, a <strong>user data script</strong> is referenced to automatically configure
the instance with an <strong>Apache web server</strong>, a <strong>MariaDB relational database</strong> 
(which is a fork of the MySQL database), and <strong>PHP</strong>.
</p>

<p>
Together, these software components are commonly called a 
<strong>LAMP stack</strong> (Linux, Apache, MySQL/MariaDB, and PHP). 
A LAMP stack is often used to build websites that include a database backend.
</p>

<p>
The same user data file also deploys website files and runs database configuration
scripts on the instance. As a result, the EC2 instance will host the 
<strong>Café Web Application</strong>.
</p>
