# GitHub Actions Pipelines In-Place & Blue-Green <br> Deployments with Docker and CloudFormation

The deployment strategy determines how the release processes are executed. For example, it may define whether the deployment is done through manual steps or automated scripts, the frequency of deployments (e.g., continuous deployment or scheduled releases), and the deployment environment (e.g., cloud-based, on-premises, or hybrid).

By aligning the release processes with the deployment strategy, organizations can ensure a smooth and efficient deployment of software, minimizing downtime and disruptions to the production environment. It also helps in maintaining consistency and reliability in the deployment process, enabling faster and more frequent releases while reducing the risk of errors or failures.

Digging more into GitHub Actions workflow, we can take below examples of deploying the same code <br> 
in term of CI/CD. Traditional full Docker deployment can be applied that usually know as in-place<br>
deployment. A more complex deployment with no downtime can be brought such as using CodeDeploy.<br>

Herewith how the the comparison of the two deployment in general.


| Deployment Method | In-Place Deployment | Blue-Green Deployment |
|------------------|---------------------|-----------------------|
| Definition       | In-place deployment involves updating the existing production environment with new changes or updates. | Blue-green deployment involves creating a duplicate environment (blue) alongside the existing production environment (green) and switching traffic between the two. |
| Downtime         | In-place deployment may cause downtime during the deployment process as the production environment is updated. | Blue-green deployment minimizes downtime as the switch between the blue and green environments is seamless. |
| Rollback         | In-place deployment can be challenging to rollback as the changes are directly made to the production environment. | Blue-green deployment allows for easy rollback by simply switching traffic back to the green environment if any issues arise in the blue environment. |
| Risk             | In-place deployment carries a higher risk as any issues or bugs in the new changes can directly impact the production environment. | Blue-green deployment reduces risk as the new changes are tested in the blue environment before switching traffic, ensuring a stable production environment. |
| Scalability      | In-place deployment may have limitations in scalability as the production environment needs to handle both the deployment process and regular traffic. | Blue-green deployment allows for better scalability as the blue environment can be scaled independently without affecting the green environment. |
| Cost             | In-place deployment may be more cost-effective as it does not require the creation of duplicate environments. | Blue-green deployment may incur additional costs due to the need for duplicate environments and infrastructure. |
| Complexity       | In-place deployment is relatively simpler as it involves updating the existing environment. | Blue-green deployment can be more complex as it requires managing two environments and coordinating the switch between them. |

![](/images/03-image22.png)
  Figure 1: Deployment Strategies as per [Container Solutions](https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/9e09392d-k8s_deployment_strategies.png)
<br><br>

Looking at the technology, herewith are comparison. In-place deployment use entirely docker approach. <br>
In CodeDeploy in this case we apply Docker for the first job to build and not using Docker to deploy. 


| Technology    | Docker                                                   | AWS CodeDeploy                                           |
|---------------|----------------------------------------------------------|----------------------------------------------------------|
| Deployment    | Deploys applications in containers                       | Deploys applications to various compute platforms        |
| Downtime      | May have downtime during the stop and run process        | Can perform rolling deployments without downtime         |
| Configuration | Uses Dockerfiles and docker-compose files for setup      | Uses AWS CodeDeploy configuration files for setup        |
| Scalability   | Can scale horizontally by running multiple containers    | Can scale horizontally by deploying to multiple instances|
| Portability   | Provides consistent environment across different systems | Works with various compute platforms and services        |
| Dependency    | Requires Docker to be installed and configured           | Requires AWS account and CodeDeploy configuration        |

Pipeline deployment with purely with Docker and associated containerization technology allows consistent and portable deployments. <br>
CodeDeploy as an AWS deployment service offers more advanced deployment features such as rolling deployments without downtime.

We will go more hands-on on this context rather than looking into more strategic intent of these two.<br>

## GitHub Actions Setup
For both deployment on GitHub Action, we need to setup few things below

### Forking the repo
One of the good repo for this would be from https://github.com/davyzhang3/GithubActionsExample <br>
Please uncheck below box which contain 2 different deployments so both can be forked together.

![](/images/03-image01.png)
  Figure 2: Forking all branches from https://github.com/davyzhang3/GithubActionsExample for example by unchecking this box 
<br><br>

### Spinning the CloudFormation
Like Terraform and Ansible, Cloudformation is gaining popularity especially in US market. <br>
Herewith some comparison of them. The other players include Puppet, Chef, Salt.

| Feature           | CloudFormation | Ansible             | Terraform           |
|-------------------|----------------|---------------------|---------------------|
| Language          | JSON or YAML   | YAML                | HCL                 |
| Infrastructure    | AWS            | Multi-cloud support | Multi-cloud support |
| Configuration     | Declarative    | Imperative          | Declarative         |
| State Management  | Yes            | No                  | Yes                 |
| Orchestration     | Yes            | Yes                 | Yes                 |
| Learning Curve    | Moderate       | Easy                | Moderate            |
| Community Support | Large          | Large               | Large               |
| Extensibility     | Limited        | Extensive           | Extensive           |
| Ecosystem         | AWS-specific   | Multi-cloud support | Multi-cloud support |
| Integration       | AWS Services   | Wide range of tools | Wide range of tools |
| Version Control   | Yes            | Yes                 | Yes                 |

To get the right CloudFormation setup, you need to change three things from downloaded file below <br>
https://github.com/.../GithubActionsExample/blob/blue-green-deployment/cloudformation/template.yaml

Herewith are the summary of changes needed.

![](/images/03-image02.png)
  Figure 3: Download https://github.com/.../GithubActionsExample/blob/blue-green-deployment/cloudformation/template.yaml <br> and make some changes accordingly on line 81, 86 and 90 with your GitHub username, AWS pem key and your Docker Hub username
<br><br>

In this case, I already have three pem keys (can be seen from cli) and I pick one (wcd-project). <br>
Now we can start the CloudFormation like below. 

![](/images/03-image05.png)<br>
![](/images/03-image06.png)<br>
![](/images/03-image07.png)<br>
![](/images/03-image08.png)<br>
![](/images/03-image09.png)<br>
![](/images/03-image10.png)<br>
  Figure 4-9: The process of spinning CloudFormation




### GitHub Secret and Environmental Variables
There are six secrets and one environmental variable needed for both deployments. <br>
As CloudFormation completed and 2 EC2 were started we complete all these secrets. <br>

![](/images/03-image03.png)
Figure 10: secrets/environment variables for in-place/blue-green deployments. <br><br>

```txt
DOCKERHUB_USERNAME and DOCKERHUB_TOKEN are your docker username and password.
HOST can be found from your EC2 Public IpV4 Address.
SSH_PRIVATE_KEY --> https://github.com/marketplace/actions/ssh-remote-commands.
USER is ec2-user as CloudFormation code here was designed to use AWS Linux. 
IAMROLE_GITHUB --> see figure 10 below how to find this role. 
AWS_REGION is where your region, in my case it is us-east-1.
```
<br><br>

![](/images/03-image04.png)
  Figure 11: Example of IAMROLE_ROLE that can be found in CloudFormation --> Stacks --> Outputs --> GithubIAMRoleArn 
<br><br>

For the SSH_PRIVATE_KEY, herewith is the example of generating new ssh key

```sh

[ec2-user@ip-10-192-10-187 ~]$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/ec2-user/.ssh/id_rsa.
Your public key has been saved in /home/ec2-user/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:570l7z7/Sk/J4zCZsMcciJLhr2RNvXD2RCPRG4jfbV8 your_email@example.com
The key's randomart image is:
+---[RSA 4096]----+
|          ..o    |
|         . ..o   |
|       .  ...o+  |
|      . o o.+o.oE|
|       +S+.* o. o|
|        =o+.O = o|
|       o o.ooXo+.|
|      o .   .*=o.|
|       .    .o===|
+----[SHA256]-----+

[ec2-user@ip-10-192-10-187 ~]$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB ... EZiSZw== your_email@example.com
```
And added ssh keys above into .ssh/authorized_keys. <br>
Please note, copy paste <br>
```sh
cat ~/.ssh/id_rsa
``` 
into Github Secrets <br>
not <br>
```sh
cat ~/.ssh/id_rsa.pub
``` 
it should looks like below <br>
```sh
-----BEGIN RSA PRIVATE KEY-----
MIIJKAIBAAKCAgEAzEiZrNxeepQga8yjk6vzkPkTKSnKUUMLjPl0fqTwfEbLbEge
....
zgoXzZePHNJGYsjNl7f525C2p+YDsbNcPw5VZELvShtyV0xr9dUOpYfSjr8=
-----END RSA PRIVATE KEY-----
```

### Install Docker
The first deployment needs a docker to install. <br>
You can use below code for example which is for AWS Linux.

```sh
sudo yum update -y
sudo yum upgrade -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker $USER
docker info
```

### Checking EC2 Instances
The CloudFormation is responsible to spin 2 EC2. <br>
When a new deployment coming, the older one will be replaced. <br>

Herewith is the quick check of the two instances.
![](/images/03-image23.png)
  Figure 12: The Two EC2 created by AutoScaling through CloudFormation 
<br><br>


## In-Place Deployment
To trigger a deployment, pick a README.md file for example. Add a new line. <br>
Commit the change and the pipeline should be triggered automatically. <br>

We then should see successful pipeline like below

![](/images/03-image14.png)
  Figure 13: Example of Successful In-Place Deployment 
<br><br>

## Blue-Green Deployment
Similarly, to trigger a deployment here, pick a README.md file for example. <br>
Add a new line, commit the change and pipeline shall be triggered right away. <br>

We should see successful pipeline like below as well.

![](/images/03-image24.png)
  Figure 14: Example of Successful Blue-Green Deployment
<br><br>

## Closing Down everything
I found it is easier to turn off Cloudformation directly which would shut <br>
down AutoScaling and EC2. Some says differently to shut down Autoscaling first.<br>

In any case please check again that all EC2, AutoScaling and CloudFormation <br>
were completely turn off.

![](/images/03-image16.png)
  Figure 15: AutoScaling Status When shutting down was not yet started
<br><br>

![](/images/03-image17.png)
  Figure 16: AutoScaling when shutting down started 
<br><br>

![](/images/03-image19.png)
  Figure 17: All running Instances should be disappeared soon. 
<br><br>

![](/images/03-image21.png)
  Figure 18: And CloudFormation should be stopped (manually).
<br><br>

## SUMMARY
This blog brought more insight on GitHub Actions multiple CI/CD deployment and <br>
little background on deployment strategies. If you are interested, please see <br>
other Github Actions discussion on https://github.com/FariusGitHub/github-1 <br><br>

Regarding Blue-Green deployment, in this case blue and green have no particular meaning<br>
They could just as well have been called pink-red deployment, yellow-purple deployment.<br>
The name is simply identifying the 2 versions of your application running in production.
