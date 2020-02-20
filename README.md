# vue-circleci

This is an example vue project to deploy a static website to AWS S3 via circleci. All the tutorials I found where
outdated. So I set up this pipeline. 

To setup circleci you need an account there and your project needs in its root folder a yaml file in the folder `.circleci/config.yml`

So what should be happening? For a simple workflow that deploys the master branch of a git repository to one S3 bucket there needs to be 
1) a build stage that compiles the dist folder with the static website and 
2) a deploy stage that transfers everything to S3.

## 1. Build Stage
Needs to run npm install and npm run build. Basically that is it. There is one piece missing thought. Stages in a pipeline don´t know
anything from each other. To pass the compiled files from stage one to two there needs to be a shared mechanism. That could be artifacts
or better working_directory: ~/project.

## 2. Deploy stage
Stage two needs access to the files so you need the key "working_directory" and restore_cache as a step. The second stage uses the 
AWS CLI ORB https://circleci.com/orbs/registry/orb/circleci/aws-cli Orbs are packages for circleci. It makes sense to use them as it saves
a lot of tedious work of installing aws cli to you docker executor by yourself. All you need is to set up the env variables.

So much for the CI config. Feel free to look around and adapt the config.yml to your needs. 
## Set up Environment variables for AWS CLI in circleci
This one needs three environment variables. You need to set those in your Piplines Screen > Project Settings > Environment Variables. They need to be called exactly like that:

## Configure IAM credentials in AWS
````
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=eu-central-1
````
Those are the credentials of your IAM user for uploading files. It is good practice to have a dedicated user that has only access to 
this bucket. 

## Set up AWS S3 bucket in AWS
Speaking of which: of course you need an S3 bucket that can host a static website. AWS gives you detailed steps over here
https://docs.aws.amazon.com/AmazonS3/latest/dev/HostingWebsiteOnS3Setup.html

Short version:
- set up a new bucket with all "public access blocked" unchecked
- add a policy for public read access (example given in the link above to the AWS docs)

## Use custom bucket name as environment variable in circleci
The AWS CLI S3 command needs to know the name of your bucket. You might not want to expose it in your source code. I set up an env variable called `AWS_BUCKET`. In a more sophisticated setup when you want to deploy to
staging and master you could have two buckets and so to env variables e.g. AWS_BUCKET_PROD and AWS_BUCKET_STAGING. Then your .circleci/config.yaml can do the following
````
if [ "${CIRCLE_BRANCH}" == "develop" ]; then
    aws s3 sync dist s3://${AWS_BUCKET_PROD} --delete
elif [ "${CIRCLE_BRANCH}" == "master" ]; then
     aws s3 sync dist s3://${AWS_BUCKET_STAGING} --delete
fi
````
That makes sense if your staging and production builds are the same. In case you have different env variables two workflows are more convenient.

That is it. Hope that was helpful. Happy deploying. 

# Vue commands

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
