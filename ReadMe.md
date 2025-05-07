# jisu-build
A project to facillitate building the Docker containers needed to deploy the **Jews in the Soviet Union** reader projects to production on EC2.  This project contains three git submodules:
  
[jisu-reader](https://github.com/nyudlts/jisu-reader)  
A web-based EPUB reader, based on Readium ts-toolkit.  
  
[jisu-pub-server](https://github.com/nyudlts/jisu-pub-server)  
A Publication Manifest server based on the Readium go-toolkit that serves the manifest needed for ts-toolkit based readers.  

[jisu-api](https://github.com/nyudlts/jisu-api)    
A simple Node.js Express server to enable api calls to the Solr database.  The project also contains python scripts for ingesting EPUB books into Solr.  

## Dev Setup
For development purposes it is possible to run each of the above projects locally.  The best way to do that is to clone and run the projects individually, following the Dev Setup section of the ReadMe in each repo.  Note: the submodules here are meant to ease the production workflow.

For local development, this project is useful for launching a local Solr server and populating it with book data.

### Local Solr Server

**Dependencies:**  
Docker, docker-compose  
python 3, pip3  
python dependencies:   
	request, pysolr, ebooklib, beautifulsoup4  

1.  Clone this project recursively.
```
git clone https://github.com/nyudlts/jisu-build.git --recursive
```

2. Clone the build project recursively (if necessary)

2.  Use docker-compose to start Solr in a Docker image
```
cd jisu-build
docker-compose -f docker-compose-solr.yml up -d --pull always
```

3. Create a collection  
```
//open Solr admin
http://localhost:8983

Collections > Add Collection
  name: bookCollection
  config set: _default
  numShards: 1
  reaplicantFactor: 1
Add Collection 
```

4.  Create schema with python and ingest books
```
cd jisu-api  
  
//install python dependencies (if needed)  
pip3 install pysolr
pip3 install ebooklib  
pip3 install beautifulsoup4  
  
python3 create_solr_fields_local.py
python3 ingest_epub_books_local.py
```

5. Test ingest in Solr admin
```
//open Solr admin  
http://localhost:8983  
  
Choose bookCollection on left  
Choose Query    
Choose Execute Query  
Entries with book data should be seen  
```

## Updating submodules
If you commit changes to any of the submodule repos and want to update them in jisu-build: 
```
git submodule update --init
git submodule sync
git submodule update --remote
```

## Production Setup

1. Log into EC2 via SSH.  You will need the dlts-aws-jisu.pem file.  (In this example stored in the ~ (home) directory.)
```
//make sure permission of the .pem file are correct
chmod 400 dlts-aws-jisu.pem

//ssh into EC2
ssh -i ~/dlts-aws-jisu.pem ec2-user@18.205.45.14
```

2. From EC2, clone the build project (if necessary)
```
git clone https://github.com/nyudlts/jisu-build.git --recursive
```

3. cd into the project folder
```
cd jisu-build
```

4. Build the Docker images
```
docker-compose -f docker-compose.build.yml build
```

5. Authenticate Docker with ECR
```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 395362471827.dkr.ecr.us-east-1.amazonaws.com
```

6. Push the images to ECR
``` 
docker-compose -f docker-compose.build.yml push
```

7. Pull and run images from ECR
```
docker-compose up -d --pull always
```

8. Create Solr collection

9. Create Solr schema and ingest books
