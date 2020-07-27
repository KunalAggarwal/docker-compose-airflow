# Airflow Docker Compose Setup

This repository contains the docker-compose scripts for setting up a full fledged [Apache Airflow](https://airflow.apache.org/) Instance (including Webserver, Worker, Scheduler, Flower) and required services including MySQL and Redis.

This Docker Compose uses the [official builds of Airflow on DockerHub](https://hub.docker.com/r/apache/airflow).

At the end of this you will have an Apache Airflow instance with a Scheduler and Worker running and the [tutorial DAG](https://airflow.apache.org/docs/stable/tutorial.html) already loaded into Airflow.

## Required Packages
- Install [Docker](https://docs.docker.com/engine/install/)
- Install [Docker Compose](https://docs.docker.com/compose/install/)

## Setup Process

1. Git clone this repository
    ```bash
    git clone https://github.com/KunalAggarwal/docker-compose-airflow.git
    ```
2. Change directory to the cloned repository
	```bash
	cd docker-compose-airflow
	```
3. First, start the MySQL db and Redis container which are dependencies for Airflow
	```bash
	docker-compose -f airflow-db.yml up -d
	```
	This will start 2 containers, one each for MySQL and Redis. The containers have volumes mounted which allow data to be persisted. This compose file also creates a network which is utilised by these containers, as well as Airflow services later on. 
4. Next, Airflow requires the first step to be executing the `airflow initdb` command. Using the provided `airflow.cfg` a container can be started which will execute the `initdb` command and exit, thus setting up the MySQL database and tables. To do this, run the following command:
	```bash
	docker-compose -f airflow-starter.yml up
	```
	The above command should end with an output that says: 
	```bash
	....
	airflow-init     | Done.
	airflow-init exited with code 0
	```
	With this done, the database is ready for the airflow services.
5.  Next up are the Airflow services which form the core. To start them up, execute the following command:
	```bash
	docker-compose -f airflow-services.yml up -d
	```
	This will create a network, where all 4 containers (webserver, scheduler, worker, flower) will be created. This docker-compose also connects these instances to use the network created by the db script for communication with MySQL and Redis.
6. At last, you will have airflow running. You can point your browser to the following URLs to access the GUI for Airflow and Flower.
	```
	http://<host_ip or hostname>:8888/admin/          # Apache Airflow
	http://<host_ip or hostname>:5555/                # Flower
	``` 

### Usage
The cloned repository has a `dags` folder with the hello_world.py containing the [tutorial DAG](https://airflow.apache.org/docs/stable/tutorial.html) from Airflow documentation.

Just place your python script containing the DAG in this folder and Airflow will pick it up in a minute or two. 

### Tear down the stack
Once you're done playing around with the created docker containers for Airflow, it is super easy to tear them down and completely remove from docker. Just run the following commands:
```bash
docker-compose -f airflow-services.yml down -v     # destroys the services containers
docker-compose -f airflow-starter.yml down         # destroys the already dead init container
docker-compose -f airflow-db.yml down -v           # destroys MySQL and Redis
docker-compose -f airflow-services.yml down -v     # run this again to remove all volumes, since they were dependecies for other containers, they don't get deleted the first time.
```
*P.S. - If you do not wish to delete the volumes/persistent data for any of the containers, strip the `-v` from all commands provided above, this will just tear down the containers but keep the volumes intact.*
