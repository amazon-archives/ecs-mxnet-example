## Deploy a MXNet predict function to Amazon ECS using CodeCommit and CodePipeline

This project will create an automated workflow that will provision, configure and orchestrate a pipeline triggering deployment of any changes to your mxnet model or application code. You will orchestrate all of the changes into a deployment pipeline to achieve continuous delivery using CodePipeline and CodeBuild. You can deploy new MXNet APIs and make those available to your users in just minutes, not days or weeks.

![](https://s3.amazonaws.com/mxnet-template-cicd-feb27/ecs-ml.png)


### Required:

> **Prepare an AWS Account**  
> If you don’t already have an AWS account, create one at [http://aws.amazon.com]() by following the on-screen instructions.

**Please launch in N.Virginia** 

#### Create a CodeCommit repository and Connect with this repository

1.	Clone this github repository 
		
		git clone https://github.com/awslabs/ecs-mxnet-example.git

2.	Go to AWS Console and select CodeCommit. Click Create New repository button. Enter a unique repository name( e.g. image-classification-predict ) and a description and click Create repository. 

	You will get a URL to your CodeCommit repository similar to `https://git-codecommit.us-east-1.amazonaws.com/v1/repos/image-classification-predict
`
3.	You can use https or ssh to connect to your CodeCommit repository. We’ll connect via SSH in this lab.  
Follow instructions from here: [http://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html?icmpid=docs_acc_console_connect]()
 
#### Commit the Source Code and Configuration files into your CodeCommit repository
1. Clone a local copy of CodeCommit repo ( we recommend:  you created earlier in your home directory.

		cd ~
		git clone <your repo url from above> 
	
	This will create a folder as the same name as in your path where you executed the git clone command.

1. Copy the contents of <image-classification-predict >directory into this new folder.
		
		cp -r ecs-mxnet-example/image-classification-predict/ image-classification-predict/
	   	    
1. Change the buildspec.yml file to include your AWS account number.

 		<your account number>.dkr.ecr.us-east-1.amazonaws.com/image-classification-predict:latest
   
2.	Commit all of the copied contents into your CodeCommit repository.

	    git add --all
	    git commit -m "Initial Commit"
	    git push origin master  
    
**Tip:** Verify the file .git/config for remote==”origin” and branch==”master”

#### Deploy via CloudFormation template.

1.	Run the Cloudformation template provided
	1. 	Go to AWS Console
	1. 	Click on Services
	1. 	Click on Cloudformation
	1. 	Create a new stack
	1. 	Enter the url <https://s3.amazonaws.com/mxnet-template-cicd-feb27/master.yaml> in the **Specify an Amazon S3 template URL** section
	1. 	Follow the steps with default parameters.
	1. 	Review the details
	1. 	Click create.
	
It takes 10 minutes to deploy the complete stack.

The Cloud Formation template creates a ECS cluster in a VPC and deploys the application code alongwith the needed dependencies. It also adds Cloud Watch for logging and performance monitoring and autoscaling alongwith an Application Load Balancer.

The development workflow is preconstructed with the CICD pipeline to ensure the application development and model training workflows are integrated.

#### Test using a Rest Client.

You can get the **MLServiceUrl** from the Cloudformation Output for the master stack.

	GET http://<MLServiceUrl>/image?image=<image_url>

an example url to use: 

	https://upload.wikimedia.org/wikipedia/commons/thumb/5/5b/Golden_Gate_Bridge,_San_Francisco,_California_LCCN2013633353.tif/lossy-page1-450px-Golden_Gate_Bridge,_San_Francisco,_California_LCCN2013633353.tif.jpg

The API predicts that theimage is a suspension bridge with 62% probability.

#### Cleanup

* **Reset Steps**
	
	1.  Delete the ECR Repository
	2.	Delete the CloudFormation stack and re-create it.

* **Removal Steps**
	1.	Delete the ECR Repository
	2.	Delete the CloudFormation stack.
