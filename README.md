ipcc-odbc-configuration
=======================

This Repository contains Ansible Playbooks, Ansible Configuration files, Asterisk Configuration file and supporting files.
The purpose of this project is to automate the password rotation (with the help of AWS Secrets Manager) for RDS Instance Master Password 
and take necessary actions to update Asterisk server cofiguration file automaticlly for seamless communication and also send automated 
notification to product team about and password changing activity whether it is failed or successful. 

GitHub Repo Link : https://github.com/sarbajitD-24/ipcc-odbc-configuration.git

This projet has 2 parts : CI (Continuous Integration) & CD (Continuous Deployment). This repo focuses of CD configuration Part.

Continuos Integration for this Project is supposed to work with the conbination of multiple aws services : 

Amazon RDS, AWS Secrets Manager, AWS Cloudtrail, Amazon EventBridge, AWS Lambda Functions & Amazon SNS.

For Policies, Lambda Functions and other configuration templates for ipcc project deployed in AWS Cloud checkout below repo : 

https://github.com/sarbajitD-24/ipcc-project-aws-configurations.git


Changes/Modifications to look out for
=====================================

Please check out below files in this repo as Jenkins build will fail if correct details are not updated :

filename --> hosts : contains correct IP address of Jenkins Server (EC2 Instance) & Asterisk Server (EC2 Instance) [change as per need]

filename --> jenkinsfile : contains correct Secret Name & AWS Region Code [change as per need]

filename --> odbc.ini : contains correct RDS Instance End point [change as per need]

Only update/modify this GitHub Repo once considering EC2 Servers IPs, RDS Endpoint, AWS Region & Secret Name in AWS Secrets Manager 
remains un-altered. If these values remains unchanged then no need to modify repo code between successive builds in Jenkins Server.

![IPCC -- Architecture Diagram_2](https://github.com/sarbajitD-24/ipcc-odbc-configuration/assets/65843678/8f031c9c-aa09-4116-a9ef-084978cc61bf)

Project Requirements
====================

Requirement 1 : Create jenkins pipeline for RDS password retrieval and update to asterisk config

Description : As part of this Jenkins Job, it should retrieve the RDS MySQL credentials from AWS secret manager and update the username and password in the Asterisk config file. Then restart the asterisk server.

Acceptance Criteria
~~~~~~~~~~~~~~~~~~~~
✱ Verify When the Jenkin Job runs, the latest password is updated in asterisk config file.
✱ Verify when the asterisk server starts up and it connects with the database.


Requirement 2 : Enable SNS email notification for rds password rotation

Description : You can monitor AWS Secrets Manager using Amazon CloudWatch. So when the Secret changes an email notification should be sent to the Product Team using SNS.

Acceptance Criteria
~~~~~~~~~~~~~~~~~~~~
✱ Verify email is send to the product team on password change




aws-sm-getsecretvalue - System$2591

pipeline {
  
  environment {
      username = "admin"
       password = ""
      
      // AWS Secrets Manager Secret Name -- Subject to Change 
      mysecret = "rds!db-995e9de8-56da-4ea0-85b8-7abb9f68429d"

      // AWS Region Code -- Subject to Change
      aws_region = 'us-east-1’
      
      
  }
  
  agent any
 
  stages {
	stage ('Retrieve secret') {

          steps {
              script {
                  
                  withCredentials([[
                      $class: 'AmazonWebServicesCredentialsBinding',
                      credentialsId: 'aws-sm-getsecretvalue',
                      accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                      secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                          
                       password = sh(script: """aws secretsmanager get-secret-value --region ${aws_region} \
                         	 --secret-id ${mysecret} | jq -r .SecretString | jq -r .password""", returnStdout: true’)
                          	  echo "RDS DB Password: ${password}"
            
                  }
              }
          }
      }

