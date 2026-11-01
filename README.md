Part A-1 : AWS CloudFormation and Automated S3 Website Deployment

1. Project Overview

The objective of this project was to create an Amazon S3 static website using AWS CloudFormation and automate the deployment of an existing index.html file from GitHub. GitHub Actions and OpenID Connect (OIDC) were used to authenticate with AWS and upload the website file without requiring a manual upload.

2. Implementation Steps

Step 1: Create the CloudFormation Template

A CloudFormation template named CF-S3-Template.yaml was created. It defines:

An Amazon S3 bucket.

Static website hosting with index.html as the index document.

A bucket policy that permits public read access to website objects.

Outputs for the bucket name and website URL.

Retention policies to help preserve the bucket when the stack is deleted or replaced.

The template creates the website infrastructure but does not upload the HTML file.

Step 2: Configure CloudFormation Git Sync

A deployment configuration file, CF-S3-deploy.yaml, was added to the repository. It references the CloudFormation template and allows AWS CloudFormation Git sync to use the repository configuration.

The stack was deployed in the eu-north-1 AWS Region.

Step 3: Configure GitHub and AWS Authentication

An IAM role named GithubActions was configured to use GitHub's OIDC identity provider. This allows GitHub Actions to request temporary AWS credentials without storing long-term AWS access keys in GitHub.

The trust policy was updated to support the repository's immutable subject claim format and permit the relevant main and s3-template branches.

An IAM permissions policy was also configured to allow the workflow to upload the website file to the target S3 bucket.

Step 4: Create the GitHub Actions Workflow

A workflow named deploy-website.yml was created under .github/workflows/.

The workflow performs these operations:

Checks out the repository.

Configures AWS credentials using OIDC.

Uploads index.html to the S3 bucket using the AWS CLI.

Sets the uploaded object's content type to text/html.

The workflow is triggered by pushes to the s3-template branch and can also be started manually.

Step 5: Troubleshoot OIDC Authentication

The first workflow execution failed with an sts:AssumeRoleWithWebIdentity authorization error.

The IAM trust policy initially used the traditional GitHub subject claim format. The repository was configured to use immutable subject claims, so the trust policy was updated to match the repository and owner identifiers displayed in GitHub's OIDC settings.

After this change, the workflow successfully authenticated with AWS and uploaded index.html.

Step 6: Test the Deployment and Use Pull Requests

The uploaded website was accessed through its S3 website endpoint, confirming that the deployment worked.

The s3-template branch was used for development and testing. Pull Requests were used to manage changes intended for the main branch.

3. Result

The project demonstrated an automated website deployment workflow connecting GitHub, GitHub Actions, AWS IAM, Amazon S3 and CloudFormation.

The HTML file was uploaded successfully without manual file transfer, and the website was accessible through the S3 website endpoint.

4. AI Assistance and Prompt Documentation

AI assistance was used to understand the CloudFormation template, plan the deployment workflow, configure the relevant AWS and GitHub components, and troubleshoot authentication errors. The implementation was tested by running the workflow and accessing the deployed website.

The following prompts are translated and consolidated summaries of the main requests made during development. The original conversations were conducted primarily in my native language, and these prompts are not verbatim transcripts.

Prompt 1: Understanding CloudFormation

"Explain the structure of the official AWS CloudFormation template for S3 static website hosting. Explain the purpose of each section without rewriting the template, and ask questions to check my understanding."

Prompt 2: Automating Website Uploads

"I have an existing index.html file in my GitHub repository and want to make it publicly accessible through Amazon S3 static website hosting. I do not want to upload the file manually. Explain the possible automation approaches."

Prompt 3: Configuring GitHub Actions and AWS

"Guide me through configuring GitHub Actions to deploy index.html to an Amazon S3 bucket. Use AWS OIDC authentication instead of long-term access keys, explain the required IAM trust and permissions policies, and help me configure the workflow for my development branch."

Prompt 4: Troubleshooting OIDC Authentication

"Help me troubleshoot the GitHub Actions error 'Not authorized to perform sts'. Explain how to compare the GitHub OIDC subject claim with the AWS IAM role trust policy and identify the required changes."

Prompt 5: Verifying Deployment and Costs

"Help me verify that GitHub Actions successfully uploads index.html to S3 and that the website is accessible. Explain which AWS resources and services may incur charges and how to clean up resources safely."

AI Tool and Model

Tool: ChatGPT

Model: GPT-6

Purpose: Technical explanation, implementation guidance, troubleshooting and verification planning.

Technical Documentation Consulted

AWS CloudFormation documentation: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html

Amazon S3 static website hosting: https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html

GitHub Actions documentation: https://docs.github.com/en/actions

GitHub Actions OIDC with AWS: https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws

5. Conclusion

The project successfully demonstrated infrastructure provisioning and automated static website deployment. CloudFormation defined the S3 website infrastructure, while GitHub Actions and OIDC provided the deployment mechanism. Testing confirmed that the website file could be uploaded automatically and accessed through the website endpoint.

Part A-2 : AWS CloudFormation Three-Tier Architecture

1. Project Overview

The objective of Part B was to design and deploy a three-tier architecture using AWS CloudFormation. The architecture separates the infrastructure into public, application and database layers within an Amazon Virtual Private Cloud (VPC).

The infrastructure was defined using a CloudFormation YAML template and deployed in the eu-north-1 AWS Region. The main focus was on network segmentation, security, resource connectivity and the separation of application and database components.

Before developing the CloudFormation template, I first created and explored the main AWS resources manually through the AWS Management Console. This helped me understand the architecture, the purpose of each component and how the different services communicate with each other.

After understanding the infrastructure and its relationships, I began translating the configuration into a CloudFormation YAML template to automate resource provisioning and manage the architecture as Infrastructure as Code.

2. Implementation Steps

Step 1: Create the VPC and Subnets

A VPC with the CIDR block 10.0.0.0/16 was created. Six subnets were configured across two Availability Zones:

Two public subnets for the Application Load Balancer.

Two private application subnets for the EC2 instances.

Two private database subnets for the RDS database layer.

This configuration provides network separation between the public-facing and private components.

Step 2: Configure Internet Gateway and Route Tables

An Internet Gateway was created and attached to the VPC.

A public route table was configured with a default route (0.0.0.0/0) pointing to the Internet Gateway. Both public subnets were associated with this route table.

A separate private route table was created and associated with the application and database subnets. No NAT Gateway was configured, keeping the private network isolated from direct outbound internet access.

Step 3: Configure Security Groups

Three Security Groups were created to control communication between the architecture layers:

ALB Security Group: Allows inbound HTTP traffic on port 80 from the internet.

Application Security Group: Allows inbound HTTP traffic on port 80 only from the ALB Security Group.

Database Security Group: Allows inbound MySQL traffic on port 3306 only from the Application Security Group.

These rules restrict direct access to the application instances and database.

Step 4: Deploy the Application Load Balancer

An internet-facing Application Load Balancer was created across the two public subnets.

A Target Group was configured to register both EC2 instances on port 80. An HTTP listener was also created on port 80 to forward incoming requests to the Target Group.

Step 5: Create EC2 Application Instances

Two EC2 instances were deployed in the private application subnets, across separate Availability Zones.

Both instances use the same application Security Group and are registered with the Application Load Balancer Target Group.

The instances were configured with IMDSv2 token requirements to improve instance metadata security.

Step 6: Configure the RDS Database

An Amazon RDS MySQL database instance was created in the private database layer.

A DB Subnet Group was configured using the two private database subnets. 

The database Security Group restricts inbound MySQL connections to the application layer.

Step 7: Configure CloudFormation Git Sync

A separate deployment configuration file, CF-ThreeTier-deploy.yaml, was added to the GitHub repository.

The configuration references the CF-ThreeTier.yaml template and allows AWS CloudFormation Git Sync to synchronise the infrastructure template from the CF-ThreeTier branch.

Step 8: Deploy and Validate the Stack

The CloudFormation stack was deployed in the eu-north-1 Region.

The deployment completed successfully with a CREATE_COMPLETE status. The created resources were checked through the AWS Management Console, including the VPC, subnets, route tables, Security Groups, EC2 instances, Application Load Balancer, Target Group and RDS database.

After completing the validation, the CloudFormation stack was deleted and the resources were checked to ensure that the test environment was cleaned up.

3. Result

The project successfully demonstrated the provisioning of a three-tier AWS architecture using CloudFormation.

The infrastructure includes network segmentation, public and private subnets, an internet-facing Application Load Balancer, two private EC2 instances and a private RDS MySQL database.

The deployment was successfully validated through AWS CloudFormation and the AWS Management Console. No web server or application was installed on the EC2 instances; therefore, application-level HTTP responses and database queries were not tested.

4. AI Assistance and Prompt Documentation

AI assistance was used to understand AWS networking concepts, plan the three-tier architecture, review CloudFormation resource definitions, troubleshoot configuration issues and understand AWS IAM permissions.

The following prompts are translated and consolidated summaries of the main requests made during development. The original conversations were conducted primarily in my native language, and these prompts are not verbatim transcripts.

Prompt 1: Understanding Three-Tier Architecture

"Explain the components of a three-tier AWS architecture using VPC, public and private subnets, Application Load Balancer, EC2 instances and RDS. Explain how these components communicate with each other."

Prompt 2: Planning Network Configuration

"Help me plan a three-tier AWS network architecture using CloudFormation, with public subnets for an Application Load Balancer, private application subnets for EC2 instances and private database subnets for RDS. Explain the routing and Availability Zone configuration."

Prompt 3: Configuring Internet Gateway and Route Tables

"Explain how to configure an Internet Gateway, public and private route tables, and subnet associations using CloudFormation. Explain how public and private subnets differ in terms of internet connectivity."

Prompt 4: Configuring Security Groups

"Explain how to configure AWS Security Groups so that the Application Load Balancer accepts public HTTP traffic, EC2 instances accept HTTP traffic only from the ALB, and the RDS database accepts MySQL traffic only from the application layer."

Prompt 5: Configuring the Application Load Balancer

"Explain how to configure an AWS Application Load Balancer, Target Group and HTTP Listener using CloudFormation. Explain how to register two EC2 instances as targets."

Prompt 6: Configuring EC2 Instances

"Help me configure two EC2 instances in separate private application subnets using CloudFormation. Explain the purpose of the AMI, instance type, subnet association, Security Groups and IMDSv2 configuration."

Prompt 7: Configuring RDS Database

"Guide me through configuring a private Amazon RDS MySQL instance using CloudFormation. Explain DB Subnet Groups, storage encryption, database accessibility, Single-AZ deployment and Security Group rules."

Prompt 8: Configuring Database Credentials

"Explain how to pass a MySQL database username and password to an AWS RDS resource using CloudFormation parameters. Explain the purpose of NoEcho and why database credentials should not be hardcoded in a GitHub repository."

Prompt 9: Understanding CloudFormation Dependencies

"Explain how CloudFormation manages dependencies between resources using Ref, GetAtt and DependsOn. Help me understand which resources depend on each other in my three-tier architecture and when explicit DependsOn declarations are necessary."

Prompt 10: Configuring CloudFormation Outputs

"Help me identify useful CloudFormation Outputs for my three-tier architecture, including the VPC ID, Application Load Balancer DNS name, RDS endpoint address and database port. Explain how Ref and GetAtt retrieve these values."

Prompt 11: Understanding IAM Roles and Permissions

"Explain the difference between IAM user permissions, a CloudFormation execution role and the IAM role used by CloudFormation Git Sync. Explain which permissions are required for deploying my three-tier architecture."

Prompt 12: Understanding CloudFormation Git Sync

"Explain how AWS CloudFormation Git Sync works with a GitHub repository and branch. Explain the purpose of the deployment configuration file, the template-file-path property and the IAM role used by Git Sync."

Prompt 13: Understanding Rollback and Deletion Policies

"Explain the difference between CloudFormation rollback settings, DeletionPolicy and the RDS DeletionProtection property. Help me understand which settings are appropriate when testing and deleting a temporary AWS environment."

Prompt 14: Troubleshooting CloudFormation Deployment

"Help me investigate CloudFormation deployment errors using the Events and Resources tabs. Explain how to identify failed resources, correct configuration problems and safely redeploy the stack."

Prompt 15: Validating and Cleaning Up AWS Resources

"Provide a checklist for validating a deployed three-tier AWS architecture. Include VPC networking, subnet associations, EC2 instance states, Application Load Balancer configuration, Target Group registration, RDS configuration and resource cleanup after testing."

AI Tool and Model

Tool: ChatGPT

Model: GPT-6

Purpose: Technical explanations, architecture planning, CloudFormation guidance, troubleshooting and validation planning.

5. Technical Documentation Consulted

AWS CloudFormation documentation: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html

Amazon CF ELB & Target group documentation: https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-elasticloadbalancingv2-targetgroup.html

Amazon CF Route table documentation: https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-routetable.html

Amazon VPC documentation: https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html

Amazon EC2 documentation: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html

Amazon CF Subnet Route Table Association Documentation: https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-subnetroutetableassociation.html?utm_source=chatgpt.com

Elastic Load Balancing documentation: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

Amazon RDS documentation: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html

6. Conclusion

Part B successfully demonstrated the use of AWS CloudFormation to provision a three-tier infrastructure. The project applied network segmentation, security controls and separation of application and database resources.

The deployment and resource creation were validated through AWS CloudFormation and the AWS Management Console. The project provided practical experience in designing and managing AWS infrastructure as code.