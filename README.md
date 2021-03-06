**Maintainers**
* alexsedova
* [Dzhuneyt](https://github.com/Dzhuneyt)

# Terraform + AWS + Docker Swarm setup

Here is the basic setup to run up docker swarm cluster in AWS using the Terraform.
[Terraform](https://www.terraform.io) is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions. Using Terraform helps to create the infrastructure you can change, and trace safely and efficiently. A small swarm cluster will be created during startup. One swarm manager + two swarm workers. In the  *app-instances.tf* you will find the configuration. The swarm is initiated during provisioning. All other swarm agents (workers) will connect to the manager by a token, generated during the swarm initialisation. The trick is we should do it automatically, but we don't know the token before the initialisation. To send the token to the agents, I copy it to a file on the swarm manager and do "scp" to the manager host from the agent's machines.

For storage, we provision an EFS drive that gets attached via docker volumes to various stacks when they are deployed.  We have some helpers for this, but the solution allows us to have flexible, backed-up storage on AWS that is accessible to containers seamlessly as a filesystem mount. 

Currently I have not set up a backup plan, but that is a good idea for a #TODO. https://us-west-2.console.aws.amazon.com/backup/home. Ideally this would be set up as well in TF. 

## Installation
How to install terraform you can find [here](https://www.terraform.io/intro/getting-started/install.html). Or you can using a Docker image to keep your environment clear. For example, [this one](https://hub.docker.com/r/amontaigu/terraform/).

## Preparations
### AWS account
If you don't have account, you may get a free AWS account. In the setup will be used free t2.micro instances.
#### SSH keys
Before to start, create ssh keys. Terraform will create key-pair in AWS, based on these keys. See [how to create ssh keys](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)
Create a pem file with private ssh key you generated. Terraform will need to the pem file to connect to instances for provisioning.
#### Update the project file with new information
There are three file need your credentials for successing run up. First of all, update *key-pair.tf* set there a path to the public ssh key, generated earlier. In *app-instances.tf* update
connection block for each resources, set there the path to ssh private key.

In *variables.tf* update your AWS account information. Make sure that the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables have your AWS credentials

## How to use
After all configuration files are ready, you can do check if there are no mistakes.
```
terraform plan
```
This command will show either syntax errors or list of resources will be created. After you can run:
```
terraform apply
```
This command will build and run all resources in the *.tf files. If you run this command many times, Terraform will destroy previous instances before creating new ones.
That is it. Now you have fully functioned docker swarm cluster in AWS.

If you want to terminate instances and destroy the configuration you may call:
```
terraform destroy
```
