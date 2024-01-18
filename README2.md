# From DS to MLOPs

From Data Science to MLOPs workshop

# Dataset
## Breast Cancer Wisconsin (Diagnostic) Data Set

For this workshop we are going to work with the following dataset:

https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+(Diagnostic)

Features are computed from a digitized image of a fine needle aspirate (FNA) of a breast mass. They describe characteristics of the cell nuclei present in the image.
n the 3-dimensional space is that described in: [K. P. Bennett and O. L. Mangasarian: "Robust Linear Programming Discrimination of Two Linearly Inseparable Sets", Optimization Methods and Software 1, 1992, 23-34].

### Attribute Information:

1) ID number
2) Diagnosis (M = malignant, B = benign)
3-32)

Ten real-valued features are computed for each cell nucleus:

a) radius (mean of distances from center to points on the perimeter)
b) texture (standard deviation of gray-scale values)
c) perimeter
d) area
e) smoothness (local variation in radius lengths)
f) compactness (perimeter^2 / area - 1.0)
g) concavity (severity of concave portions of the contour)
h) concave points (number of concave portions of the contour)
i) symmetry
j) fractal dimension ("coastline approximation" - 1)

# Virtual Environment

Firt we need to create a virtual environment for the project, to keep track of every dependency, it is also useful to use and explicit version of Python

Install the package for creating a virtual environment:
`$ pip install virtualenv`

Create a new virtual environment
`$ virtualenv venv`

Activate virtual environment
`$ source venv/bin/activate`

# Python packages

Now with the virtual environment we can install the dependencies written in requirements.txt

`$ pip install -r requirements.txt`

# Train

After we have install all the dependencies we can now run the script in code/train.py, this script takes the input data and outputs a trained model and a pipeline for our web service.

`$ python code/train.py`

# Web application

Finally we can test our web application by running:

`$ flask run -p 5000`

# Docker

Now that we have our web application running, we can use the Dockerfile to create an image for running our web application inside a container

`$ docker build . -t from_ds_to_mlops`

And now we can test our application using Docker

`$ docker run -p 8000:8000 from_ds_to_mlops`

# Test!

Test by using the calls in tests/example_calls.txt from the terminal

# 1. Github Actions Intro

![](image/GithubActions.png)

https://docs.github.com/en/actions

- A free tool.

- Serverless.

- Easy to setup and manage.


# 2. Basics

## Workflow
In one Github Repository, you can have multiple workflows.
```
├── .github
│   └── workflows
│       ├── deliver.yaml
│       └── run_test_on_PR.yaml
```
## Events
- Push.

- Pull_request.

- Schedule.

- Issue.

- External Event

```yaml
on:
  pull_request:
    branches:
      - develop
      - main
```

```yaml
on:
  schedule:
  # minute hour day-of-month month day-of-week
    - cron: '0 0 * * *'
    - cron: '0 5 */1 * *'
```

## Jobs
In one workflow you can have multiple jobs.

Jobs can be in parallel or in sequence.

key word: `needs`
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps: ...
  build:
    runs-on: ubuntu-latest
    needs: test
    steps: ...
```

## Steps
In one job you can have multiple steps
```yaml
jobs:
  run_tests:
    runs-on: ubuntu-latest
      - name: run bash commands
        run: |
          pwd
          ls -al
        shell: bash
```
## Actions
https://github.com/marketplace?type=actions

Actions are the building blocks that power your workflow. They are like functions in Python. You can either write your own action and reference it or simply just use an action from a public repository.

To use an action, use the key word `uses`
To provide arguments to an action: use the key word `with`
```yaml
jobs:
  run_tests:
    runs-on: ubuntu-latest
      - uses: actions/checkout@v2 #https://github.com/actions/checkout
      - name: Set up Python 3.8 
        uses: actions/setup-python@v1 # https://github.com/actions/setup-python
        with:
          python-version: 3.8
```

By default Github Action doesn't clone your repo to the Github hosted server. Therefore we have to clone it by ourselves. The easiest way to do that is using the checkout action.

Setup-python is another useful action that install the specific version of Python for us in the VM.

## Environment Variables
### Custom Environment Variables
Use the `env` key to create custom environment variables in the workflow.

You can define the environment variables that are scoped for:
- The entire workflow.
- Job level.
- Step level.

### Default Environment Variables
- GITHUB_WORKFLOW.

- GITHUB_ACTION.

- GITHUB_REPOSITORY.

- GITHUB_ACTOR.

Etc
```yaml
    steps:
      - name: Default ENV Variables
        run: |
          echo "HOME: ${HOME}"
          echo "GITHUB_WORKFLOW: ${GITHUB_WORKFLOW}" # Name of the workflow
          echo "GITHUB_ACTION: ${GITHUB_ACTION}" # Name of the action
          echo "GITHUB_ACTIONS: ${GITHUB_ACTIONS}" # Always true when running in GitHub Actions
          echo "GITHUB_ACTOR: ${GITHUB_ACTOR}" # Name of the person who triggered the workflow
          echo "GITHUB_REPOSITORY: ${GITHUB_REPOSITORY}" # Owner/Repo
```
https://docs.github.com/en/actions/learn-github-actions/environment-variables


# Expressions
https://docs.github.com/en/actions/learn-github-actions/expressions

`${{}}`  You can use expressions to programmatically set environment variables in workflow files and access contexts. An expression can be any combination of literal values, references to a context, or functions.

For example, you can use it to refer to the secret variables you stored in the Github settings.

```yaml
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Dockerhub Login
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```



# Deploy

The Github Actions will be triggered when the code is pushed to the git repo.
There are 1 build job and 2 jobs about depoyment.
The build job builds a docker image and push it to my Dockerhub registry
One of the deployment is in-place deploy, which would cause a short down time of the application.
The other deployment method is blue-green through AWS CodeDeploy, which is more complicated but it does not have a down time.
In the lab both 2 deployments will be shown.
When you are using one of the deployment method, please comment out the other one.

# Steps
Please store your dockerhub token and your dockerhub username in the repo Settings -> Secrets and variables -> actions-> secrets. By default the image will be pushed to https://hub.docker.com/r/junglepolice/sklearn_flask. If you'd like to use your own Dockerhub account to store the image, make sure you update the code in `./aws/scripts/ApplicationStart.sh` and the variable `DockerhubUsername` in the Cloudformation Stack when it is being launched. The `./aws/scripts/ApplicationStart.sh` is a hook file that will be executed in the ApplicationStart stage of the AWS CodeDeploy lifesycle. It stops the running container and create a new container with the latest Docker image.

When you are using the in place deployment method, please have an ec2 instance ready with docker installed and configured. Then store the username, the IP address and content of the pem file of the ec2 in repository secrets.


When you are using the blue green deployment method to deploy your application, please create a new stack in the AWS CloudFormation with the template.yaml file in the cloudformation folder of this repository. The GithubRepoName, KeyName and Stack name can be customized but the other parameters have to kept as default when you are configuring the CloudFormation template parameters.

![](./images/MLOPS.png)
The cloud formation will cerate a VPC, a launch configuration file, an autoscaling group, an AWS Codedeploy service and some necessary roles.
![](./images/MLOPS-Cloudformation-Resources.png)

On the AWS CloudFormation console, select the Outputs tab. Note that the ARN of the GitHub IAM Role will be used in the next step.
![](./images/CloudFormationOutput.png)

Copy the value of this ARN. Then go to your Github repository, create a new secret called IAMROLE_GITHUB and paste the ARN as the content of the secret.
![](./images/Secrets.png)

## Integrate CodeDeploy with GitHub:
For CodeDeploy to be able to perform deployment steps using scripts in your repository, it must be integrated with GitHub.

CodeDeploy application and deployment group are already created for you.

Sign in to the AWS Management Console and open the CodeDeploy console at https://console.aws.amazon.com/codedeploy.


In the navigation pane, expand Deploy, then choose Applications.

Choose the application CodeDeployAppNameWithASG to link to a different GitHub account.

Choose CodeDeployGroupName

Click on Create deployment.

In Deployment settings, for Revision type, choose My application is stored in GitHub.

To create a connection for AWS CodeDeploy applications to a GitHub account, sign out of GitHub in a separate web browser tab. In GitHub token name, type a name to identify this connection, and then choose Connect to GitHub. The web page prompts you to authorize CodeDeploy to interact with GitHub for your application. 

![](./images/IntegrateCodeDeployWithGithub.png)

Choose Authorize application. GitHub gives CodeDeploy permission to interact with GitHub on behalf of the signed-in GitHub account for the selected application.

Choose cancel because we don't want to create a deployment

Go back to the application CodeDeployGroupName, click on edit

Choose Blue/Green in Deployment type
![](./images/DeploymentType.png)

choose the target group created by the cloudformation stack
![](./images/LoadBalancer.png)

Click on Save changes

Push your code to the Github to trigger the workflow

In order to see if the deployment works, copy the DNS name from the load balancers, enter the value + `/info` in your broweser.

# Clean Up
- Delete the cloudformation stack in your AWS Console.
- Delete the autocaling group created by AWS in EC2 service

# Reference
- https://aws.amazon.com/blogs/devops/integrating-with-github-actions-ci-cd-pipeline-to-deploy-a-web-app-to-amazon-ec2/
- https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-partners-github.html#behaviors-authentication
  
