# DeepfakeBot

Make copies of your friends! Using the magic of cloud computing, [Markov](https://github.com/jsvine/markovify) chains and a little data analytics, you can create a bot that sounds like another discord user.

## Users

* [Add](https://discordapp.com/oauth2/authorize?client_id=551871268090019945&scope=bot&permissions=117760) the bot to your Discord server.
* [Read](https://deepfake-bot.readthedocs.io/) about how to use it.
* Get [help](https://discord.gg/JGudz9G) with your bot if you get stuck. 
* [Donate](https://www.patreon.com/rustygentile) to keep the bot up and running.

## Developers

This is meant to be a rough outline of the steps and AWS resources needed to get this project up and running. It is not a precise how-to as I'm probably forgetting one or more steps. If you're a normal user ready deploy your bot, skip this section and see [here](https://deepfake-bot.readthedocs.io/en/latest/self-deployments.html) instead.

### Prerequisites

* A [Discord](https://discord.com/developers/) account - Create an application and add a bot to it.
* An [AWS](https://aws.amazon.com/) account - Be sure to install and [configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) the CLI so your development machine can access the resources it needs
* A Python 3.6 environment - E.g. `conda create -n deepfake_bot python=3.6` 
* Docker - Needed to build and push the container 
* Git - That's kind of a given...

### VPC

Setup a VPC with public and private subnets similar to this [scenario](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario3.html). Bot application containers will be deployed in public subnets. Lambda functions and the RDS instance will run in private subnets.

### Database

* Provision a MySQL RDS instance in one of your private subnets. Create a master user and password.
* Provision an EC2 instance in one of your public subnets.
* Use an SSH tunnel to this to enable a database connection from a terminal on your development machine: ```ssh -N -L 1234:[RDS endpoint url]:3306 ec2-user@[ec2 IP address] -i [path to ec2 key]```
* Open a connection in MySQL workbench on 127.0.0.1 and port 1234. Use the master user and password you created.
* Create 'production' and 'test' schemas. Don't add any tables yet.
* Assemble your `DEEPFAKE_DATABASE_STRING` variable. For your development machine this sould look like so: ```mysql://[master user]:[master pw]@127.0.0.1:1234/[test schema name]?charset=utf8```
* Run [db_queries.py](./cogs/db_queries.py) to create the tables.
* Check that the tables are there in MySQL workbench then repeat for the production schema. 

### S3

* Create a private bucket with a policy where objects expire every 30 days. Change the name in [config.py](./cogs/config.py).
* Create another bucket with no expiration policy. Change the `my_bucket` variable in [build_layer.sh](./lambdas/build_layer.sh).

### Elastic Container Service

* TODO...

### Lambda Functions

* From an EC2 instance, clone the project and run [build_layer.sh](./lambdas/build_layer.sh) to gather the python libraries. This will copy them to your permanent S3 container. Create a Lambda layer from this.
* Create three python lamba functions from [activity](./lambdas/activity/), [markovify](./lambdas/markovify/) and [wordcloud](./lambas/wordcloud/) using your layer. You'll need to give them new names. Then add these names to [config.py](./cogs/config.py).
* Give your EBS's IAM instance profile permission to run them.
* Lambda functions should run in the private subnet. 

### Testing 

* Setup your IDE. I use pycharm, Anaconda, and [this](https://plugins.jetbrains.com/plugin/7861-envfile/) to manage environment variables. You may want to use a different `DEEPFAKE_DISCORD_TOKEN` and `DEEPFAKE_DATABASE_STRING` locally than in EBS. 
* With the SSH tunnel to your database open, run [bot.py](bot.py). Try out all of the bot commands.
* There are unit tests in the [test](./test/) folder but there is no CI setup. The release script will work regardless of whether the tests pass or not.

### Release

* Run [release.sh](release.sh)
* TODO...
