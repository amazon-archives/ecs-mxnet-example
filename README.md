# ecs-mxnet-example
An example project to deploy MXNet image classification inference API with Docker on Amazon ECS. Uses CodePipeline and CodeBuild to build the image to deploy to ECS.

### Required:

> **Prepare an AWS Account**  
> If you don’t already have an AWS account, create one at http://aws.amazon.com by following the on-screen instructions.
> Use the region selector in the navigation bar to choose the Amazon EC2 region where you want to deploy Swift web application on AWS.

Create a key pair in your preferred region.
You can follow steps here: [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair]()

Also, change the permission on your keypair with the following command.
`chmod 400 <your keypair>`

** Please lauch in N.Virginia **


### Deploy a MXNet function to Amazon ECS using CodeCommit and CodePipeline

This project will create an automated workflow that will provision, configure and orchestrate a pipeline triggering deployment of any changes to your swift package. You will orchestrate all of the changes into a deployment pipeline to achieve continuous delivery using CodePipeline and CodeBuild. You can deploy new MXNet APIs and make those available to your users in just minutes, not days or weeks.

#### Step 1: Create a CodeCommit repository and Connect with this repository

Follow the instructions below to Create and Connect to an AWS CodeCommit Repository. You may also refer to the instructions at AWS CodeCommit documentation

1.	Go to AWS Console and select CodeCommit. Click **Create New repository** button.
	Enter a unique repository name as **image-classification-predict** and a description. 
	Click **Create repository**. You will get a URL to your CodeCommit repository similar to below
	<https://git-codecommit.us-east-1.amazonaws.com/v1/repos/image-classification-predict>

2.	You can use https or ssh to connect to your CodeCommit repository. The steps need initial set up for AWS CodeCommit and steps for Linux/MacOS is provided as below. For other platform, refer to this link
	<http://docs.aws.amazon.com/codecommit/latest/userguide/how-to-connect.html>
	* Create a new IAM user at IAM console. Provide this user Programmatic access.
	* Add the following managed policies for the IAM user.
		> *  AWSCodeCommitFullAccess
		> *  AmazonEC2ContainerRegistryFullAccess
		> *  AmazonEC2ContainerServiceFullAccess
		> *  IAMReadOnlyAccess
		> *  IAMUserSSHKeys

	* Open a terminal window on type


			cd $HOME/.ssh
			ssh-keygen


	 When prompted, use a name like mycodecommit_rsa and you can leave passphrase as blank. Hit enter.


			cat mycodecommit_rsa.pub


	* Go to IAM, select the user you have created and click on Security Credentials tab.
		* Click Upload SSH Public key button. Copy the contents from file ‘mycodecommit_rsa.pub’ in the text box and save.

	* Go back to terminal and type

			touch config
			chmod 600 config
			sudo vim config  


	 and paste the following
	 **Note:** Ensure that this is the first entry in the config file


			Host git-codecommit.*.amazonaws.com
			User <SSH_KEY_ID_FROM_IAM>  Value for the SSH key id from the user you created in IAM when you uploaded the public key.
			IdentityFile ~/.ssh/mycodecommit_rsa

  * Verify your SSH connection. Type the following and confirm that you get a successful response.


				ssh git-codecommit.us-east-1.amazonaws.com


#### Step 2: Commit the Source Code and Configuration files into your CodeCommit repository

1.	Clone a local copy of CodeCommit repo you created earlier in your home directory.

		cd ~
		git clone ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/image-classification-predict


	This will create a folder as the same name as <your CodeCommit Repo name> in your path where you executed the git clone command.

	Copy the contents of **ecs-mxnet-example/image-classification-predict** directory into this new folder. The contents provided from `git clone git@github.com:awslabs/ecs-mxnet-example.git`

			cd ~/ecs-mxnet-example/image-classification-predict
			cp -r * ~/image-classification-predict/
			cd  ~/image-classification-predict

2. Change the buildspec.yml file to include your AWS account number.

		<your account number>.dkr.ecr.us-east-1.amazonaws.com/swiftrepo:latest
	
	You can find your AWS account number on the right hand top corner of your AWS console.

3.	Commit all of the copied contents into your CodeCommit repository.

			git add --all
		 	git commit -m "Initial Commit"
			git push origin master

	***Tip: Verify the file .git/config for remote==”origin” and branch==”master”***



#### Step 3: Deploy the automated Swift package via CloudFormation template.

Pick the template from <https://tiny.amazon.com/lelj9nq9/s3amazmxnemastyaml>.The templates are also provided in the repository.

The stack takes approximately 15 minutes to create all resources.

The template creates a number of AWS resources to facilitate the automated workflow.

> *  **Virtual Private Cloud (VPC)** – A VPC with VPC resources such as: VPCGatewayAttachment, SecurityGroup, SecurityGroupIngress, SecurityGroupEgress, SubnetNetworkAclAssociation, NetworkAclEntry, NetworkAcl, SubnetRouteTableAssociation, Route, RouteTable, InternetGateway, and Subnet
> *  **Auto Scaling Group** – An auto scaling group to scale the underlying EC2 infrastructure in the ECS Cluster. It’s used in conjunction with the Launch Configuration.
> *  **Auto Scaling Launch Configuration** – A launch configuration to scale the underlying EC2 infrastructure in the ECS Cluster. It’s used in conjunction with the Auto Scaling Group.
> *  **CodeBuild** –CodeBuild will build your project using the commands given in buildspec.yml 
> *  **CodePipeline** – CodePipeline describes Continuous Delivery workflow. In particular, it integrates with CodeCommit and Jenkins to run actions every time you commit new code to the CodeCommit repo.
> *  **IAM Instance Profile** – “An instance profile is a container for an IAM role that you can use to pass role information to an EC2 instance when the instance starts.”
> *  **IAM Roles** – Roles that have access to certain AWS resources for the EC2 instances (for ECS), Jenkins and CodePipeline
> *  **ECS Cluster** – “An ECS cluster is a logical grouping of container instances that you can place tasks on.”
> *  **ECS Service** – An ECS service, you can run a specific number of instances of a task definition simultaneously in an ECS cluster
> *  **ECS Task Definition** – A task definition is the core resource within ECS. This is where you define which Docker images to run, CPU/Memory, ports, commands and so on.
> *  **Application Load Balancer** – The ALB provides the endpoint for the application. The ALB dynamically determines which EC2 instance in the cluster is serving the running ECS tasks at any given time.

A code change committed to CodeCommit repository will trigger image creation, create a task definition, create the service and run the task in an auto-scaled pool of containers running behind an elastic load balancer.

#### Step 4: Validation.

You can monitor the build process on the UI for the CodeBuild created for you in CloudFormation.
Once the stack update is completed, refer to the output of your CloudFormation and click on MLServiceUrl

You will see a URL similar to:
http://<stackname>-EcsElb-<container>-<image>.us-east-1.elb.amazonaws.com/


**Congratulations:** Your MXNet API is live and deployed on ECS container automatically.
You can test the API using a api client like Postman
	
		GET http://MLServiceUrl?image=<image-url>


#### Cleanup

* **Reset Steps**
	1.	Scale the service down to zero running tasks.
	2.  Delete the ECR Repository
	3.	Delete the CloudFormation stack and re-create it.

* **Removal Steps**
	1.	Scale the service down to zero running tasks.
	2.	Delete the CloudFormation stack.
