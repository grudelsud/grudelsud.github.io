---
layout: post
type: post
title: Continuous integration with Docker on Amazon Elastic Beanstalk
date: 2015-11-30
categories:
- development
tags: []
status: publish
published: true
comments: true
---

[This post also appears on [Medium](https://medium.com/@grudelsud/continuous-integration-with-docker-on-amazon-elastic-beanstalk-44fa89024502)] After spending long hours to have a fully working *Circle CI / Elastic Beanstalk / Docker* integration, I thought it would be useful to put my notes down so I will save some time next time I need to go through a similar setup and avoid spending long nights cursing the gods of the (internet) clouds.

We are making the following assumptions:

- deploy to [single-container Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/docker-singlecontainer-deploy.html) (EB),
- use [Circle](https://circleci.com/) as CI engine, I am pretty sure that other services (e.g. Travis, Codeship, etc.) would offer equivalent functionalities,
- use a private registry to store compiled Docker images: I use [Quay.io](https://quay.io/) because it is cheaper than Docker Hub and works just fine for what I need,
- deploy a Python Django application, however this is not strictly necessary as most of the steps apply to any kind of web application.

There is plenty of documentation related to EB and a few samples that can be used as good starting points to understand what EB needs to deploy a Docker image to a single-container environment (e.g. [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_image.html) and [here](https://github.com/awslabs/eb-py-flask-signup/tree/docker)), but they did not seem to cover all the aspects of real-world implementations.

The aim of what follows is to provide a quick step by step guide to create and deploy an app to EB and describe some details hidden across different documents on Circle, EB and Docker.

# Step 1. Elastic Beanstalk on AWS

Using the AWS Web Console, go to IAM and create a new user, give it full access to EB, take note of key+secret and edit `~/.aws/config` on your machine to create a new profile, it should look something like:

    [profile eb-admin]
    aws_access_key_id = ACCESSKEY
    aws_secret_access_key = SECRETKEY

still in AWS Web Console, go to Elastic Beanstalk and create a new WEB application and select the following settings:

- Docker environment,
- set it to auto-scale,
- do NOT use RDS that comes with EB, it is more flexible to setup your own RDS and hook it to EB later,
- sat yes to VPC,
- set some other reasonable settings, assuming you have basic knowledge of AWS to handle security groups, key-pairs, etc.

Now, back to your machine, create a python virtual environment (using virtualenvwrapper here) and install AWS command line tools:

    mkvirtualenv aws
    # probably not strictly necessary, but will come useful later…
    pip install awscli
    pip install awsebcli

Time to setup EB on the local machine and hook it to the application created in the AWS Web Console! Go where your web applications lives (in my case where Django's *manage.py* sits), then up 1 folder and run (assuming we run EB on Amazon's EU-West data centre):

    eb init --region eu-west-1 --profile eb-admin

you will be prompted a few questions by the init process:

- it should show the EB applications you have created in your AWS account, select the one you want to use from the list, it should match the one just created with AWS Web Console,
- all other options should be automagically detected from the online settings.

I guess the same settings can be defined by manually calling [options for the *eb init* command](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-init.html), but I would not recommend it at this stage.

If the wizard does not work (e.g. because you have weird files that trigger auto detections), go through the init questions by choosing the following:

- skip platform auto detection and choose Docker (if a Dockerfile is found, it probably will not ask for platform details)
- choose an existing keypair ssh configuration

After this step, you should now have a fresh .elasticbeanstalk directory containing a single *config.yml* file reflecting the app settings you have just created. Add a line to your *.gitignore* file as eb init gets a bit too enthusiast with ignoring files that you might need to share with your team:

    !.elasticbeanstalk/config.yml

Now you need to tweak AWS a bit to allow EB to be able to deploy by reading a configuration file from S3. Go back to IAM in the AWS Web Console, you should find 2 newly created Roles (have a look at [this article](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html) for further information):

- instance role (or EC2 role) is used by EB to deploy and have access to other resources
- service role is used by EB to monitor running instances and send notifications

add **READ** permissions to S3 to the **instance role** so EB knows how to fetch the relevant files when deploying. Finally go to S3, there should be a new bucket with a few EB related files in it, take note of the name, you will use it later.

# Step 2. Docker

Create a private repo on Quay.io: I am assuming that we are going to run some super secret code that we do not want to host on the docker public registry, so I am adding additional information to allow Elastic Beanstalk to authenticate on the private registry.

- on Quay.io create a robot account and assign write credentials to the private repo
- download the `.dockercfg` file associated to the robot account and put it BOTH in `~` on your local machine (so you can authenticate on Quay.io from the command line) and in your repository's root (will be used later by Circle)

Now create your Dockerfile — there are plenty of good examples around, I quite like one from Rehab Studio that runs a simple Flask test app with Gunicorn and NGINX, you can find it [here](https://github.com/rehabstudio/docker-gunicorn-nginx) — and be sure it contains the two following instructions:

    EXPOSE 80
    CMD ["your-startup-file.sh"]

*your-startup-file.sh* is basically something that starts the web server, or web application, or any other service that would listen on 80 and produce some sort of output. In my case it is Supervisor running NGINX and Gunicorn.

When you are happy with your docker image, push it to Quay.io with

    docker push quay.io/my_account/my_image:latest

Now it is time to integrate an automatic test and build on Circle.

# Step 3. Circle CI

Open an account on Circle by authenticating with Github and allow it to connect the Github repo you want to use for continuous integration.

Create a custom circle.yml file in your repository's root: this file will tell Circle to run tests and deploy your app to EB when committing to release branches, it should look more or less like the following:

{% highlight yaml %}
machine:
  python:
  version: 2.7.3
  services:
  — docker
dependencies:
  cache_directories:
  — "~/docker"
pre:
  # used to run deploy script
  — pip install awscli
  # used to authenticate on quay.io
  — cp .dockercfg ~
override:
  # cache your TESTING docker image on circle CI
  — if [[ -e ~/docker/image.tar ]]; then docker load -i ~/docker/image.tar; fi
  — docker build -f Dockerfile.test -t test ./project_root/
  — mkdir -p ~/docker; docker save test > ~/docker/image.tar
test:
  post:
  # run tests
  — docker run test python runtests.py
deployment:
  staging:
  branch: release/stg
  commands:
  # this script might be more complicated
  # with a few more arguments maybe
  — ./deploy.sh $CIRCLE_SHA1 eb-stg-env
{% endhighlight%}

This yml file instructs Circle to run tests on the docker image, and deploy if pushed to *release/stg* branch if tests are successful. The interesting aspect of this approach is that deploy.sh can be run (and, more importantly, tested!) locally.

**Note!** you might need to use private ssh keys to access your repo and build the docker deploy image, this is totally doable by adding keys in the Circle project environment (check this out).

# Step 4. Deploy

OK almost there! Now we need to create the deploy.sh file mentioned in the previous step: this will automate the process to build our deploy docker image and put it somewhere so that EB can go and fetch it: we use the AWS command line interface for it. Steps here are fairly straight forward:

1. build docker image and push to Quay.io
2. create a Dockerrun file for EB (read details below) and push it to S3
3. tell EB to create a new application version and deploy it


{% highlight bash %}
# deploy.sh
#!/bin/bash
# name of the deploy image on quay.io
BUILD="quay.io/my_account/my_image:$SHA1"

# store quay.io authentication to S3
aws s3 cp .dockercfg s3://$EB_BUCKET/dockercfg

# deploy tag
SHA1=$1

# elastic beanstalk environment to deploy
EB_ENV=$2
APP_NAME="MyEBApp"

# remember the name of the bucket I asked to take note
# above in this post? here is where you use it!
EB_BUCKET="bucket-where-EB-deploys-its-stuff"

# where to put things in the S3 bucket
PREFIX="deploy/$SHA1"

# build main image
docker build -f Dockerfile.build -t $BUILD:$SHA1 .
docker push $BUILD:$SHA1

# replace vars in the DOCKERRUN_FILE
# so that EB knows where to pick things from
DOCKERRUN_FILE="Dockerrun.aws.json"
cat "$DOCKERRUN_FILE.template" \
  | sed 's|<BUCKET>|'$EB_BUCKET'|g' \
  | sed 's|<IMAGE>|'$BUILD'|g' \
  | sed 's|<TAG>|'$SHA1'|g' \
  > $DOCKERRUN_FILE

aws s3 cp $DOCKERRUN_FILE_WEB s3://$EB_BUCKET/$PREFIX/$DOCKERRUN_FILE
rm $DOCKERRUN_FILE

# Create application version from Dockerrun file
echo "creating new Elastic Beanstalk version"
aws elasticbeanstalk create-application-version \
  --application-name $APP_NAME \
  --version-label $SHA1 \
  --source-bundle S3Bucket=$EB_BUCKET,S3Key=$PREFIX/$DOCKERRUN_FILE

# Update Elastic Beanstalk environment to new version
echo "updating Elastic Beanstalk environment"
aws elasticbeanstalk update-environment \
  --environment-name $EB_ENV \
  --version-label $SHA1
{% endhighlight %}

So what is this mysterious [*Dockerrun.aws.json*](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_image.html#create_deploy_docker_image_dockerrun)? it is a simple descriptor that tells AWS where to pull the Docker image from, what version and using which credentials. Below is the template file where <BUCKET>, <IMAGE> and <TAG> are replaced with live variables by the *deploy.sh* script, and *dockercfg* tells EB where to find credentials for private docker images.

{% highlight json %}
{
  "AWSEBDockerrunVersion": "1",
  "Authentication": {
    "Bucket": "<BUCKET>",
    "Key": "dockercfg"
  },
  "Image": {
    "Name": "<IMAGE>:<TAG>",
    "Update": "true"
  },
  "Ports": [{
    "ContainerPort": "80"
  }],
  "Logging": "/var/eb_log"
}
{% endhighlight %}

# Step 5. Tweaks, env variables, etc.

Docker on EB needs environment variables to run properly! you can either set them up directly in EB, or run a script that automates the process by using a standard format accepted by the AWS command line interface option [*update-environment*](http://docs.aws.amazon.com/cli/latest/reference/elasticbeanstalk/update-environment.html). Here is an example format of AWS options (say it's stored with name `EB-options.txt.template`):

{% highlight json %}
[
  {
    "Namespace": "aws:elasticbeanstalk:application:environment",
    "OptionName": "DJANGO_SETTINGS_MODULE",
    "Value": "$DJANGO_SETTINGS_MODULE"
  },
  {
    "Namespace": "aws:elasticbeanstalk:application:environment",
    "OptionName": "SECRET_KEY",
    "Value": "$SECRET_KEY"
  }
]
{% endhighlight %}

which can be processed by replacing local environment variables and send them to EB:

{% highlight bash %}
# update-env.sh
#!/bin/bash
OPTIONS_FILE="EB-options-$(date +%Y-%m-%d:%H:%M:%S).txt"
cat EB-options.txt.template | envsubst > $OPTIONS_FILE
aws elasticbeanstalk update-environment \
  --environment-name $EB_ENV \
  --option-settings file://$OPTIONS_FILE
rm $OPTIONS_FILE
{% endhighlight %}

The end!

quite a bit of a headache, but Woohoo! when you see it running on EB you feel like you are the god of AWS!
