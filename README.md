# PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER
# Introduction:
Patching is a vital component to any security strategy which ensures that your compute environments are operating with the latest code revisions available. This in turn means that you are running with the latest security updates for the system, which reduces the potential attack surface of your workload.
The majority of compliance frameworks require evidence of patching strategy or some sort. This means that patching needs to be performed on a regular basis. Depending on the criticality of the workload, the operational overhead will need to be managed in a way that poses minimal impact to the workload’s availability.
Ensuring that you have an automated patching solution, will contribute to building a good security posture, while at the same time reducing the operational overhead, together with allowing traceability that can potentially be useful for future compliance audits.

There are multiple different approaches available to automate operating system patching using a combination of AWS services.
One approach is to utilize a blue/green deployment methodology to build an entirely new Amazon Machine Image (AMI) that contains the latest operating system patch, which can be deployed into the application cluster. 

 This lab will walk you through this approach, utilizing a combination of the following services and features:
•	EC2 Image Builder to automate creation of the AMI
•	Systems Manager Automated Document to orchestrate the execution.
•	CloudFormation with AutoScalingReplacingUpdate update policy, to gracefully deploy the newly created AMI into the workload with minimal interruption to the application availability.

We will deploy section 1 and 2 of the lab with CloudFormation templates to get your environment built as efficiently as possible. This will allow the base infrastructure and application deployment to be completed quickly so you can focus on the main lab objectives which are covered in sections 3 and 4. In these sections, we will give you the choice of either using additional pre-built templates or manual steps to complete the EC2 Image Builder and Systems Manager Document configuration.
The skills you learn from this lab will help you secure your workloads in alignment with the AWS Well-Architected Framework.
Note: For simplicity, we have used Sydney ‘ap-southeast-2’ as the default region for this lab. Please ensure all lab interaction is completed from this region.
Prerequisites
•	An AWS account that you are able to use for testing, that is not used for production or other purposes.
NOTE: You will be billed for any applicable AWS resources used if you complete this lab that are not covered in the AWS Free Ties.

Steps:
1.	Deploy The Lab Base Infrastructure
2.	Deploy The Application Infrastructure
3.	Deploy The AMI Builder Pipeline
4.	Deploy The Build Automation With SSM
5.	Teardown

Step-1: Deploy The Lab Base Infrastructure
In this section, we will build out a Virtual Public Cloud (VPC), together with public and private subnets across two Availability Zones, Internet Gateway and NAT gateway along with the necessary routes from both public and private subnets.
This VPC will become the baseline network architecture within which the application will run. When we successfully complete our initial stage template deployment, our deployed workload should reflect the following diagram

![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/953a3d4d-5e72-4bfd-a640-52794de59717)


I am providing my GitHub link u can click and go to master branch and select on step-1  u can use in this template. 
https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/tree/master

1.1. Get the CloudFormation Template.
To deploy the first CloudFormation template, you can either deploy directly from the command line or via the console.
Console:
If you need detailed instructions on how to deploy CloudFormation stacks from within the console.
•	Use pattern3-base as the Stack Name, as this is referenced by other stacks later in the lab.

1.2. Note CloudFormation Template Outputs
When the CloudFormation template deployment is completed, note the outputs produced by the newly created stack as these will be required at later points in the lab.
You can do this by clicking on the stack name you just created, and select the Outputs Tab as shown in diagram below.

![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/173745fa-9062-4cbb-824d-561a0066494e)
You can now proceed to Section 2 of the lab where we will build out the application stack.

# End on Step-1

# Step-2: Deploy The Application Infrastructure
The second section of the lab will build out the sample application stack what will run in the VPC which was build in section 1.
This application stack will comprise of the following :
•	Application Load Balancer (ALB).
•	Autoscaling Group along with its Launch Configuration.
Once you completed below steps, your base architecture will be as follows:

![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/b7c8e624-96dd-4562-893a-16561f1d2707)

Building each component in this section manually will take a bit of time, and because our objective in this lab is to show you how to automate patching through AMI build and deployment. To save time, we have created a CloudFormation template that you can deploy to expedite the process.

Please follow the steps below to do so :
# 2.1. Get the CloudFormation Template:
To deploy the second CloudFormation template, you can either deploy directly from the command line or via the console.
You can click below link go to GitHub and click on step-2 used on template
https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/tree/master

# 2.1.1.
# Follow the steps:
Aws CloudFormation create-stack --stack-name= pattern3-ap
AmazonMachineImage = ami-0f96495a064477ffb  
BselineVpcStack = pattern3-app

# Important Note:
•	For simplicity, we have used Sydney ‘ap-southeast-2’ as the default region for this lab.
•	We have also pre-configured the Golden Amazon Machine Image Id to be the AMI id of Amazon Linux 2 AMI (HVM) in Sydney region ami-0f96495a064477ffb. If you choose to  use a different region, please change the AMI Id accordingly for your region.

# 2.2. Confirm Successful Application Installation
Once the stack creation is complete, let’s check that the application deployment has been successful. To do this follow below steps:
 Go to the Outputs section of the CloudFormation stack you just deployed.
	Note the value of OutputPattern3ALBDNSName and you can find the DNS name as per screen shot below:
![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/1c502941-3090-40ab-8f37-a7db0dcf8b7d)
	Copy the value and paste it into a web browser.
	If you have configured everything correctly, you should be able to view a webpage with ‘Welcome to Re:Invent 2020 The Well Architected Way’ as the page title.
	Adding /details.php to the end of your DNS address will list the packages currently available, together with the AMI which has been used to create the instance as follows:

![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/9fc438ba-fdfa-4d72-b9c5-3fa7cea1793d)
Take note of the installed packages and AMI Id (Copy and paste this elsewhere we will use this to confirm the changes later).
When you have confirmed that the application deployment was successful, move to section 3 which will deploy your AMI Builder Pipeline.

# END OF SECTION 2

# Step-3: Deploy The AMI Builder Pipeline
In this section we will be building our Amazon Machine Image Pipeline leveraging EC2 Image Builder service. EC2 Image Builder is a service that simplifies the creation, maintenance, validation, sharing, and deployment of Linux or Windows Server images for use with Amazon EC2 and on-premises. Using this service, eliminates the automation heavy lifting you have to build in order to streamline the build and management of your Amazon Machine Image.
Upon completion of this section, we will have an Image builder pipeline that will be responsible for taking a golden AMI Image, and produce a newly patched Amazon Machine Image, ready to be deployed to our application cluster, replacing the outdated one.
![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/3c22a2d2-9fea-45b3-bf0b-fd94154da8eb)
In this section you have the option to build the pipeline manually using AWS console, or if you are keen to complete the lab quickly, you can simply deploy from the CloudFormation template.

# 3.1. Download The CloudFormation Template.
Click on below link go to GitHub select on step-3 and download and u can used it.
In this template resources created in 
	Create an S3 bucket for logging purposes.
	Create an IAM role for use by the EC2 Image Builder.
	Create an Image Builder Component.
	Create an Image Builder Recipe.
	Create an Image Builder Pipeline.

https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/tree/master

# 3.2. Follow the steps     
Aws CloudFormation create stack--  stack-name=pattern3-pipeline
Parameter value – image_id = 0f96495a064477ffb
BaselineVpcStack = pattern3-base
Note :
•	For simplicity, we have used Sydney ‘ap-southeast-2’ as the default region for this lab.
•	We have also pre-configured the Master AMI parameter to be the AMI id of Amazon Linux 2 AMI (HVM) in Sydney region ami-0f96495a064477ffb. If you choose to use a different region, please change the AMI Id accordingly for your region.
# 3.3. Take note of the ARN.
When the CloudFormation template deployment is completed, note the output produced by the stack.
You can do this by clicking on the stack name you just created.
Now that you have completed the deployment of the Image Builder Pipeline, move to section 4 of the lab where we will use Systems Manager to build the automation stage of the architecture.

# 3.3. Successfully all resources created u can see
![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/eb3e9424-079c-4655-9bf2-150e579fb97d)

# END OF SECTION 3

# Step-4: Deploy The Build Automation With SSM
Now that our AMI Builder Pipeline is built, we can now work on the final automation stage with Systems Manager.
In this section we will orchestrate the build of a newly patched AMI and its associated deployment into our application cluster.
To automate this activities we will leverage AWS Systems Manager Automation Document.
Using our SSM Automation document we will execute the following activities:
•	Automate the execution of the EC2 Image Builder Pipeline.
•	Wait for the pipeline to complete the build, and capture the newly created AMI with updated OS patch.
•	Then it will Update the CloudFormation application stack with the new patched Amazon Machine Image.
•	This AMI update to the stack will in turn trigger the CloudFormation AutoScalingReplacingUpdate policy to perform a simple equivalent of a blue/green deployment of the new Autoscaling group.

# Note:
Using this approach, we streamline the creation of our AMI, and at the same time minimize interruption to applications within the environment.
Additionally, by leveraging the automation built in CloudFormation through autoscaling update policy, we reduce the complexity associated with building out a blue/green deployment structure manually. Lets look at how this works in detail:
•	Firstly, CloudFormation detects the need to update the Launch Configuration with a new Amazon Machine Image.
•	Then, CloudFormation will launch a new Autoscaling Group, along with it’s compute resource (EC2 Instance) with the newly patched AMI.
•	CloudFormation will then wait until all instances are detected healthy by the Load balancer, before terminating the old Autoscaling Group, ultimately achieving a blue/green model of deployment.
•	Should the new compute resource failed to deploy, CloudFormation will rollback and keep the old compute resource running.
For details about how this is implemented in the CloudFormation template, please review the pattern3-application.yml template deployed in section 2.
Once we complete this section our architecture will reflect the following diagram:
![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/e4914162-05ea-4c92-8f48-8b04d0e49b87)

In this section you have the option to build the resources manually using AWS console. If however you are keen to complete the lab quickly, you can simply deploy from the CloudFormation template and take a look at the deployed architecture. Select the appropriate section:
# 4.1. Get The Template
The template for Section 4 u can click the below link in GitHub select on step-4 you can build in this template in cloud formation.
https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/tree/master

# 4.2. Follow in this step:
Aws CloudFormation create stack--  stack-name=pattern3-automate
AppliationStack = pattern3-app
ImageBuilderPipeline = u can give 3rd step name = pattern3-pipeline

# 4.2. Successfully resources created
![image](https://github.com/anilkumarn12/anilkumarn12-PATCHING-WITH-EC2-IMAGE-BUILDER-AND-SYSTEMS-MANAGER/assets/134625092/032a529f-98e9-40c5-a0f2-11ebc79dbe6d)
END OF SECTION 4

# Step-5: TEARDOWN
The following steps will remove the services which are deployed in the lab
# 5.1. Remove the Automation Stack
From the CloudFormation console, select the stack named pattern3-automate from the list and select Delete and confirm the deletion in the next dialog box.

# 5.2. Remove the Pipeline Stack
# 5.2.1.
Note: The stack will fail to remove unless the S3 bucket is empty. As a pre requisite, remove the contents of the bucket before continuing.
From the CloudFormation console, click on the pattern3-pipeline stack name and examine the resources.
Find the resource called Pattern3LoggingBucket and note the bucket name.
Proceed to the S3 console and remove the contents of the bucket, confirming the delete action.
# 5.2.2.
Now, from the CloudFormation console, select the stack named pattern3-pipeline from the list and select Delete and confirm the deletion in the next dialog box.

# 5.3. Remove the Application Stack
From the CloudFormation console, select the stack named pattern3-app from the list and select Delete and confirm the deletion in the next dialog box.

# 5.4. Remove the Base Infrastructure Stack
From the CloudFormation console, select the stack named pattern3-base from the list and select Delete and confirm the deletion in the next dialog box.
