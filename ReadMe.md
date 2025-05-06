# jisu-build
A project to build the Docker images needed to deploy the jisu-reader projects to production on EC2.

This project contains three git submodules representing the images that Docker needs to create for the jisu-reader ecosystem.  They are:

jisu-reader
A web based EPUB reader, based on Readium ts-toolkit.

```
git clone https://github.com/BluefireProductions/nyu-press-build.git --recursive
```

To update the submodules run:

```
git submodule update --init
git submodule sync
git submodule update --remote
```

Once the submodules are in place, use Docker to build images of the projects tagged for ECR

```
docker-compose -f docker-compose.build.yml build
```

Authenticate Docker with ECR
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 395362471827.dkr.ecr.us-east-1.amazonaws.com

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 395362471827.dkr.ecr.us-east-1.amazonaws.com
```

Push the images to ECR
``` 
docker-compose -f docker-compose.build.bluefire.yml push

docker-compose -f docker-compose.build.yml push

```

Log into EC2 via SSH
```
Bluefire
ssh -i ~/ec2-key.pem ec2-user@35.95.95.96
```

Authenticate EC2 Docker with ECR
```
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 711387123804.dkr.ecr.us-west-2.amazonaws.com
```

Pull and run images from ECR
```
docker-compose up -d --pull always
```

## Solr Only

For dev purposes you can run just the Solr servers in a local environment so that eaxch project can be run in development mode on its own.

```
docker-compose -f docker-compose-solr-only.yml up -d --pull always

//test with solr admin

```
http://localhost:8983

//create a collection: Collections > Add Collection
name: bookCollection
config set: _default
numShards: 1
reaplicantFactor: 1
```

//
