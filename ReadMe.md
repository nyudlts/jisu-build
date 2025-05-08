# jisu-build
A project to facillitate building the Docker containers needed to deploy the **Jews in the Soviet Union** reader projects to production on EC2.  This project contains three git submodules:
  
[jisu-reader](https://github.com/nyudlts/jisu-reader)  
A web-based EPUB reader, based on Readium ts-toolkit.  
  
[jisu-pub-server](https://github.com/nyudlts/jisu-pub-server)  
A Publication Manifest server based on the Readium go-toolkit that serves the manifest needed for ts-toolkit based readers.  

[jisu-api](https://github.com/nyudlts/jisu-api)    
A simple Node.js Express server to enable api calls to the Solr database.  The project also contains python scripts for ingesting EPUB books into Solr.  

## Dev Setup
For development purposes it is possible to run each of the projects locally.  The best way to do that is to clone and run the projects individually, following the 'Dev Setup' section of the ReadMe in each repo.  Note: the submodules here are meant for the production workflows.

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
In production the jisu projects run in a series of Docker containers connected by a common Docker network.  jisu-build is meant to make the build process easier by bringing the projects together as submodules and using docker-compose to orchestrate the entire Docker process.  This is accomplished by first connecting to EC2 though ssh and then running the following commands:

1. Log into EC2 via SSH.  
Note: you will need the **dlts-aws-jisu.pem** file.  (In this example stored in the ~ (home) directory.)
```
//make sure permission of the .pem file are correct
chmod 400 dlts-aws-jisu.pem

//ssh into EC2
ssh -i ~/dlts-aws-jisu.pem ec2-user@18.205.45.14
```

2. From EC2, clone the build project
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
```
//open Solr admin  
http://18.205.45.14:8983

Collections > Add Collection
  name: bookCollection
  config set: _default
  numShards: 1
  reaplicantFactor: 1 
```

9. Create Solr schema
```
//get the container id for jisu-api
docker ps -a

//exec into the container to run create_solr_fields
docker exec <cont_id> python3 /usr/app/create_solr_fields.py
```

10. Ingest EPUBs into Solr
```
//get the container id for jisu-api
docker ps -a

//exec into the container to run create_solr_fields
docker exec <cont_id> python3 /usr/app/ingest_epub_books.py
```

11. Test the reader in production
```
http://18.205.45.14:3000
```
Click on a book link from the landing page.  
If the book content loads then the jisu-pub-server is working.  

Check that the reader footer contains the chapter name on the left side.  
If so, then Solr and the jisu-api are working.  

Perform a search via the magnifying glass icon in the reader header.  
If search results are shown then Solr indexing and the jisu-api is working.  