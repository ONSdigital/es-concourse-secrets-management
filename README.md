<br />
<p align="center">
 <h3 align="center">AWS Secrets Management with Concourse CI</h3>
</p>


## Table of Contents

<!-- vim-markdown-toc GFM -->

* [About The Project](#about-the-project)
* [Prerequisites](#prerequisites)
* [Usage](#usage)
    * [Configure Concourse](#configure-concourse)
    * [Create a secret](#create-a-secret)
    * [Modify `pipeline.yml`](#modify-pipelineyml)

<!-- vim-markdown-toc -->


<!-- ABOUT THE PROJECT -->
## About The Project

This project contains a POC Concourse YAML pipeline that shows AWS secrets being retrieved

## Prerequisites
1. AWS CLI configured with credentials to access AWS Secrets
2. Concourse CI running locally and configured with AWS account in #1
3. `fly` cli (same version as Concourse CI above)

<!-- GETTING STARTED -->
## Usage
1. Configure Concourse
2. Create a secret
3. Modify pipeline to echo the secret created

### Configure Concourse

This assumes that you have fly CLI and a Concourse CI installation available.

The Concourse CI web nodes must be running with the AWS credentials specified like so:
```
CONCOURSE_AWS_SECRETSMANAGER_REGION="us-east-1"
CONCOURSE_AWS_SECRETSMANAGER_ACCESS_KEY="xxxxxxxxxx"
CONCOURSE_AWS_SECRETSMANAGER_SECRET_KEY="xxxxxxxxxx"
```
The above can also be specified inline while executing the web nodes.

If you using a docker-compose to run Concourse CI locally, add the last three env vars as below:
```
  concourse:
    image: concourse/concourse:5.2.0
    command: quickstart
    privileged: true
    depends_on: [concourse-db]
    ports: ["8080:8080"]
    environment:
      - CONCOURSE_POSTGRES_HOST=concourse-db
      - CONCOURSE_POSTGRES_USER=concourse_user
      - CONCOURSE_POSTGRES_PASSWORD=concourse_pass
      - CONCOURSE_POSTGRES_DATABASE=concourse
      - CONCOURSE_EXTERNAL_URL
      - CONCOURSE_ADD_LOCAL_USER=admin:admin
      - CONCOURSE_MAIN_TEAM_LOCAL_USER=admin
      - CONCOURSE_AWS_SECRETSMANAGER_REGION=eu-west-2
      - CONCOURSE_AWS_SECRETSMANAGER_ACCESS_KEY=xxxxxxxxxxx
      - CONCOURSE_AWS_SECRETSMANAGER_SECRET_KEY=xxxxxxxxxxx
```

It is required that the AWS user account used above has access to AWS Secrets
In an upcoming PR, this will be changed so that an account is used with assume role.

### Create a secret
Note that the path must be `/concourse/$TEAM_NAME/$SECRET_NAME`
```
aws secretsmanager create-secret --name  "/concourse/main/random-secret" --secret-string "test1234"
```

### Modify `pipeline.yml`
Modify the value of `NAME`, i.e. the secret being echoed, in `pipeline.yml` to whatever you created above, sans the path:
```yaml
          run:
            path: /bin/hello-world
            args: []
          params:
            NAME: ((random-secret))
```

Finally, run the commands to set the pipeline and execute a build.
1. This should build a docker image (if needed) from https://github.com/ONSdigital/echo-param/blob/master/docker/Dockerfile
1. Push the above image to a newly created docker hub account - This demoes using credentials for a docker-image resource type
1. Pulls the above image and executes it passing in the name of a secret which is then echoed in the container created from this docker image.



