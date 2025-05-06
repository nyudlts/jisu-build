# jisu-build
A project to build the Docker images needed to deploy the Jews in the Soviet Union reader projects to production on EC2.  This project contains three git submodules representing the images that Docker needs to create for the jisu-reader stack.  They are:
  
jisu-reader  
A web-based EPUB reader, based on Readium ts-toolkit.  
  
jisu-pub-server  
A Publication Manifest server based on the Readium go-toolkit that serves the manifest needed for ts-toolkit based readers.  

jisu-api  
A simple Node.js Express server to enable api calls to the Solr database.  The project also contains python scripts for ingesting EPUB books into Solr.  

## Dev Setup
For development purposes it is possible to run these projects locally.  The best way to do that is to clone and run the projects individually, so the submodules here are meant for the production workflows (see below).

But for local dev you can use this project to launch a local Solr server using Docker and populate it with books.

### Local Solr Server

**Dependencies:**  
Docker, docker-compose  
python 3, pip3  
python dependencies:   
	request, pysolr, ebooklib, beautifulsoup4  

1.  Clone this project recursivly.
```
git clone https://github.com/BluefireProductions/nyu-press-build.git --recursive
```

2.  Use docker-compose to start Solr in a Docker image
```
cd jisu-build
docker-compose -f docker-compose-solr.yml up -d --pull always
```

3. Create a collection  
```
//open Solr admin  http://localhost:8983

Collections > Add Collection
  name: bookCollection
  config set: _default
  numShards: 1
  reaplicantFactor: 1 
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
