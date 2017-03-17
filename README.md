## Deploy a MXNet predict function to Amazon ECS using CodeCommit and CodePipeline

This project will create an automated workflow that will provision, configure and orchestrate a pipeline triggering deployment of any changes to your swift package. You will orchestrate all of the changes into a deployment pipeline to achieve continuous delivery using CodePipeline and CodeBuild. You can deploy new MXNet APIs and make those available to your users in just minutes, not days or weeks.


### Required:

> **Prepare an AWS Account**  
> If you don’t already have an AWS account, create one at http://aws.amazon.com by following the on-screen instructions.
> Use the region selector in the navigation bar to choose the Amazon EC2 region where you want to deploy Swift web application on AWS.

Create a key pair in your preferred region.
You can follow steps here: [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair]()

Also, change the permission on your keypair with the following command.
`chmod 400 <your keypair>`

** Please lauch in N.Virginia **

#### Create a CodeCommit repository and Connect with this repository

1.	Clone the github repository : git clone <repo>

2.	Go to AWS Console and select CodeCommit. Click Create New repository button. Enter a unique repository name( use image-classification-predict for this blog) and a description ex. machineeye-predict and click Create repository. You will get a URL to your CodeCommit repository similar to below https://git-codecommit.us-east-1.amazonaws.com/v1/repos/machineeye-predict

3.	You can use https or ssh to connect to your CodeCommit repository. We’ll connect via SSH in this lab.  Follow instructions from here: http://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html?icmpid=docs_acc_console_connect
 
#### Commit the Source Code and Configuration files into your CodeCommit repository
1. On your laptop or EC2 instance, clone a local copy of CodeCommit repo ( we recommend:  you created earlier in your home directory.

		cd ~
		git clone ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/image-classification-predict
	
	This will create a folder as the same name as in your path where you executed the git clone command.

1. Copy the contents of <image-classification-predict >directory into this new folder.

	    git add --all
	    git commit -m "Initial Commit"
	    git push origin master  
	    
1. Change the buildspec.yml file to include your AWS account number.

 		<your account number>.dkr.ecr.us-east-1.amazonaws.com/image-classification-predict:latest
   
2.	Commit all of the copied contents into your CodeCommit repository.

	    git add --all
	    git commit -m "Initial Commit"
	    git push origin master  
    
**Tip:** Verify the file .git/config for remote==”origin” and branch==”master”

You are using this CodeCommit repository to store you swift application code along with docker configuration files.

#### Deploy via CloudFormation template.

1.	Run the Cloudformation template provided
	a.	Go to AWS Console
	b.	Click on Services
	c.	Click on Cloudformation
	d.	Create a new stack
	e.	Enter the url <https://tiny.amazon.com/lelj9nq9/s3amazmxnemastyaml> 
	f.	Follow the steps with default parameters.
	g.	Review the details
	h.	Click create.
	
It takes 10 minutes to deploy the complete stack.

The Cloud Formation template creates a ECS cluster in a VPC and deploys the application code alongwith the needed dependencies. It also adds Cloud Watch for logging and performance monitoring and autoscaling alongwith an ALB.

The development workflow is precontructed with the CICD pipeline to ensure the application development and model training workflows are integrated.

#### Test using a Rest Client.

You can get the **MLServiceUrl** from the Cloudformation Output for the master stack.

	GET http://<MLServiceUrl>/image?image=<image_url>

an example url to use: 

	https://upload.wikimedia.org/wikipedia/commons/thumb/5/5b/Golden_Gate_Bridge,_San_Francisco,_California_LCCN2013633353.tif/lossy-page1-450px-Golden_Gate_Bridge,_San_Francisco,_California_LCCN2013633353.tif.jpg

The API predicts the image classification.


####Cleanup

* **Reset Steps**
	1.	Scale the service down to zero running tasks.
	2.  Delete the ECR Repository
	3.	Delete the CloudFormation stack and re-create it.

* **Removal Steps**
	1.	Scale the service down to zero running tasks.
	2.	Delete the CloudFormation stack.
