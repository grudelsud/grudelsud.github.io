After spending long hours to have a fully working *Circle CI - Elastic Beanstalk - Docker* integration, below are my notes so someone here @Stinkdigital could replicate in my absence. We are making the following assumptions:

- deploy to single-container Elastic Beanstalk
- use Circle as CI engine, I am pretty sure that other services (e.g. Travis, Codeship, etc.) would offer equivalent structures
- use a private registry to store compiled Docker images: we use Quay.io because it is cheaper and works just fine for what we need.

[This repo on awslabs](https://github.com/awslabs/eb-py-flask-signup/tree/docker) can be a good starting point to understand what EB needs to deploy a Docker image to a single-container enviromnet. There are also a [few docs on EB](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_image.html) but I have not found them really friendly and needed a quick digest through some aspects.

## step 1. Elastic Beanstalk on AWS

using AWS Web Console, go to IAM and create a new user, give it full access to EB, take note of key+secret and edit `~/.aws/config` to create a new profile, it should look something like:

    [profile eb-admin]
    aws_access_key_id = ACCESSKEY
    aws_secret_access_key = SECRETKEY

still in AWS Web Console, go to Elastic Beanstalk and create a new WEB application, select a docker environment, auto-scale, NO RDS, yes to VPC and set some other reasonable settings (i.e. security group, keypair, etc.)

now create a virtualenv and install aws cli

    mkvirtualenv aws
    pip install awscli # probably not strictly necessary, but will come useful later...
    pip install awsebcli

go where your webapp lives (in my case where Django's manage.py sits), then **up 1 folder** and run:

    eb init --region eu-west-1 --profile eb-admin

a few questions are asked by init:

- it should list the applications you have just created via AWS Web Console, select it
- all other options should be automagically detected from the online settings. I guess the same settings can be defined by manually calling options for the `eb init` command, but I would not recommend it at this stage. If the wizard does not work, go through the init questions by choosing the following:
    - skip platform autodetection and choose Docker (if a Dockerfile is found, it probably will not ask for platform details)
    - choose an existing keypair ssh configuration

you should now have a fresh `.elasticbeanstalk` directory containing a single config.yml file reflecting the app settings you have just created. Add a line to your .gitignore file as eb init gets a bit too enthusiast with ignoring files

    !.elasticbeanstalk/config.yml

now you need to tweek AWS a bit to allow EB to be able to deploy by reading a configuration file from S3. Go back to IAM, there should be 2 newly created Roles (have a look at [this article](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html) for further information):

- instance role (or EC2 role) is used by EB to deploy and have access to other resources
- service role is used by EB to monitor running instances and send notifications

add READ permissions to S3 to the instance role so EB knows how to fetch the relevant files. Finally go to S3, there should be a new bucket with a few EB related files in it, take note of the name, you will use it later.

## step 2. Docker

create a private repo on quay.io: assuming we are going to run super secret code that will not be hosted on the docker public registry, so I am adding additional information to allow Elastic Beanstalk to authenticate on the private registry.

- on quay.io create a robot account and assign write credentials to the private repo
- download the .dockercfg file associated to the robot account and put it in ~ and in your repo's root (will be used later by Circle)

create your dockerfile and be sure it contains - among other things - the two following instructions

    EXPOSE 80
    CMD ["your-startup-file.sh"]

`your-startup-file.sh` is basically something that starts the web server, or web application, or any other service that would listen on 80 and produce some sort of output. in my case it is supervisor running nginx and gunicorn. when you are happy with your docker image, push it to quay.io with `docker push quay.io/my_account/my_image:latest`, then it is time to integrate an automatic test and build on Circle.

## step 3. Circle CI

open an account on Circle by authenticating with Github and let it connect to the Github repo you want to use for continuous integration.

create a custom `circle.yml` file in your repo's root, this will run tests for you and deploy your app to EB when committing to release branches, it should look more or less like the following

    {% highlight yaml %}
    machine:
        python:
            version: 2.7.3
        services:
            - docker

    dependencies:
        cache_directories:
            - "~/docker"

        pre:
            # used to run deploy script
            - pip install awscli
            # used to authenticate on quay.io
            - cp .dockercfg ~

        override:
            # cache your TESTING docker image on circle CI
            - if [[ -e ~/docker/image.tar ]]; then docker load -i ~/docker/image.tar; fi
            - docker build -f ./web/Dockerfile.test -t test ./project_root/
            - mkdir -p ~/docker; docker save test > ~/docker/image.tar

    test:
        post:
            # run tests
            - docker run test python runtests.py

    deployment:
        staging:
            branch: release/stg
            commands:
                # this script might be more complicated with a few more arguments maybe
                - ./deploy.sh $CIRCLE_SHA1 eb-stg-env
    {% endhighlight %}

**note** you might need to use private ssh keys to access your repo and build the docker deploy image, this is totally doable by adding keys in the Circle project environment, [check this out](https://Circle.com/docs/permissions-and-access-during-deployment). This yml file instructs Circle to run tests on the docker image, and deploy if pushed to `release/stg` branch if tests are successful. The interesting aspect of this approach is that `deploy.sh` can be run (and - more importantly - tested!) locally.

## step 4. Deploy

ok almost there! now we need to build our deploy docker image and put it somewhere so that EB can go and fetch it: we use basic aws cli commands for it. Steps here are fairly straight forward:

1. build docker image and push to quay.io
2. crete a Dockerrun file for EB (read details below) and push it to S3
3. tell EB to create a new version and deploy it


    {% highlight bash %}
    # deploy.sh
    #! /bin/bash

    # name of the deploy image on quay.io
    BUILD="quay.io/my_account/my_image:$SHA1"

    # store quay.io authentication to S3
    aws s3 cp .dockercfg s3://$EB_BUCKET/dockercfg

    # deploy tag
    SHA1=$1

    # elastic beanstalk environment to deploy
    EB_ENV=$2

    APP_NAME="MyEBApp"

    # remember the name of the bucket I asked to take note above in this post? here is where you use it!
    EB_BUCKET="bucket-where-EB-deploys-its-stuff"

    # where to put things in the S3 bucket
    PREFIX="deploy/$SHA1"

    # build main image
    docker build -f /project_root/Dockerfile.build -t $BUILD:$SHA1 .
    docker push $BUILD:$SHA1

    # replace vars in the DOCKERRUN_FILE so that EB knows where to pick things from
    DOCKERRUN_FILE="Dockerrun.aws.json"
    cat "$DOCKERRUN_FILE.template" \
      | sed 's|<BUCKET>|'$EB_BUCKET'|g' \
      | sed 's|<IMAGE>|'$BUILD'|g' \
      | sed 's|<TAG>|'$SHA1'|g' \
      > $DOCKERRUN_FILE

    aws s3 cp $DOCKERRUN_FILE_WEB s3://$EB_BUCKET/$PREFIX/$DOCKERRUN_FILE
    rm $DOCKERRUN_FILE

    # Create application version from Dockerrun file
    echo "--- creating new Elastic Beanstalk version"

    aws elasticbeanstalk create-application-version \
      --application-name $APP_NAME \
      --version-label $SHA1 \
      --source-bundle S3Bucket=$EB_BUCKET,S3Key=$PREFIX/$DOCKERRUN_FILE

    # Update Elastic Beanstalk environment to new version
    echo "--- updating Elastic Beanstalk environment"

    aws elasticbeanstalk update-environment \
      --environment-name $EB_ENV \
      --version-label $SHA1
    {% endhighlight %}

so what is this mysterious **Dockerr.aws.json**? it is a simple descriptor that tells AWS where to pull the Docker image from, what version and using which credentials. Here is the template where `<BUCKET>`, `<IMAGE>` and `<TAG>` are replaced by the deploy script. Also `dockercfg` tells where to find credentials for private docker images.

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
      "Ports": [
        {
          "ContainerPort": "80"
        }
      ],
      "Logging": "/var/eb_log"
    }

## step 5. Tweaks, env variables, etc.

Docker on EB needs environment variables to run properly! you can either setup directly in EB, or run a script that will help you setting up them. Here is an example format of AWS options (say it's stored with name `EB-options.txt.template`):

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

which can be processed by replacing local environment variables and sent to EB:

    OPTIONS_FILE="EB-options-$(date +%Y-%m-%d:%H:%M:%S).txt"
    cat EB-options.txt.template | envsubst > $OPTIONS_FILE

    aws elasticbeanstalk update-environment \
      --environment-name $EB_ENV \
      --option-settings file://$OPTIONS_FILE

    rm $OPTIONS_FILE

## conclusions

quite a bit of a headache, but yoohoo, when you see it running on EB it is such a pleasure!
