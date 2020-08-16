## TWITTER DATA EXTRACTOR APPLICATION

>Created By: **Juan Eduardo Domingos**
Scope: **POC and Study**
Tags: **Big Data, Spark, Dashboard, Twitter, HTTP, Node.JS, ETL, ELT, Data Extraction**

## Abstract
That application aim extract data of posts from Twitter public API and based on this data, build a dashboard with information such posts, what they represent and their relevance.

## Used Technologies

 1. **Node.JS**: Application responsible for raw data extration from Twitter API
 2. **Apache Cassandra**: NoSQL Database where will be store the raw data and after processing, the refined data.
 3. **Java / Spring Boot**:  Application reponsible for produce messages which will be consumed for **SPARK** application, requesting raw data processing to dashboard building.
 4. **Amazon SQS**:  Structure responsible for collect and manage the messages generated by Java application and consumed for **SPARK** appplication.
 5. **Scala & Apache Spark**: Application responsible for prepare and process raw data for the dashboard building.

## Application Architeture

![Application Architeture](https://raw.githubusercontent.com/jeduardodomingos/big-data-study-case/master/architeture/main-structure.png)

# Let's Run
To run this application you need follow that steps:

 1. Build Docker Images
 2. Start Docker Containers (Docker Compose)

## Build and Deploy Docker  Images
The first step to execute this application is a docker images build and the their deploy, below we can se the docker images which you will need build:

Obs: You must be in the project root folder

 1. Build Cassandra Docker Image:
 
	  Obs:   This compose.yaml file, I only included the basic settings of the cassandra database, if you want to run this an instance with specific parameters or for specific scenarios, you must change some parameters and settings in the compose.yaml file.
	  
		cd ./devops/cassandra-database-environment
		docker-compose up
		
2. Build Spark Environment Images:

    Obs: For **SPARK** environment you must pay more atention, because in all the cases you need change some parameters and configurations in `spark-worker-env.sh`  file.

		cd ..
		cd ./spark-environment
		sh ./build-image.sh
		docker-compose up

## Setup Cassandra Database
Below we have an example of Cassandra Database Setup Script, remember, that script is a basic example to setup cassandra database, if you need specific structures or scenários, you must change the script like yout preferences and necessities.

		-- Setup database security
		CREATE ROLE tsa WITH PASSWORD =  '1234'  AND SUPERUSER = true AND LOGIN = true;
		ALTER ROLE cassandra WITH PASSWORD='1234'AND SUPERUSER=false;
		
		-- Keyspace creation
		CREATE KEYSPACE IF NOT EXISTS twitter_scraper_space WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

		-- Custom types creation
		CREATE  TYPE  IF NOT EXISTS twitter_scraper_space.hashtags(content varchar,indices LIST<bigint>);
		CREATE  TYPE  IF NOT EXISTS twitter_scraper_space.users(id bigint, name varchar, address varchar, description varchar, url varchar, followers bigint, friends bigint);
		CREATE  TYPE  IF NOT EXISTS twitter_scraper_space.mentions(id bigint,name varchar);
	
		-- Create table to store raw data
		CREATE TABLE IF NOT EXISTS twitter_scraper_space.tweets (id bigint, key_tag varchar, post_content varchar, tag_composition FROZEN<hashtags>, user FROZEN<users>, favorited Boolean, favouriteds bigint, retweeted Boolean, retweets bigint, language varchar, source varchar, mention FROZEN<mentions>, createdAt timestamp, updatedAt timestamp, PRIMARY KEY(id));
		

## Starting The Applications

 1. **twitter-scrapper-service** (Node.JS application):
 
	 For this application you need create a environment file with application parameters, such as database-link, twitter api credentials and some others, in the root folder of that application you can find a environment file template
