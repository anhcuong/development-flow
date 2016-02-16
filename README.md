# Introduction
This is the development conventions I have created for Razer Team. I find they are useful for most of the projects. Do not hesitate to give a try on your team.

1. Apply Git Flow during development. [Here](http://danielkummer.github.io/git-flow-cheatsheet/) is the git-flow cheatsheet.
2. Everytime developer makes a release, Jenkins will be configured to build Docker and run testing scripts.

# Conventions
Below are the conventions that developer should follow before releasing to Operation side.

## Project name
Please use small letters for your project name. If project name contains many words, use dashes pattern: project-a-b-c


## Folder Structure
- Your project src/ 
- README.md
- Dockerfile
- Makefile
- .env
- .dockerignore (if any)

You should start coding from this [dummy project](https://github.com/anhcuong/dummy-project/archive/1.0.zip). Please download and edit the files.

Watch any updates on dummy repo: https://github.com/anhcuong/dummy-project

## README.md
Written in markdown format. The file should contain the following information of the project:

- List of environment variables need to set on runtime. 

```sh
export MYSQL_DB_HOST=xxx.xxx.xxx
export MYSQL_DB_USER=abcxyz
export MEMCACHED_DB_HOST=123.456
```

- How to run the project using Makefile

```sh
# Buld docker image
make build

# Run server inside docker
make runserver

# Run unit tests
make test

# Push docker image to registry
make push

# Clean project
make clean
```

## Dockerfile
I encourage developer to start building docker image from an [official repository](https://docs.docker.com/docker-hub/official_repos/) on public Dockerhub or docker images from your own private docker registry. 

The developer also has to write down the image's tag rather than latest or nothing.

- Best Base Image: ubuntu:14.04.

```sh
FROM ubuntu:14.04

FROM emily:prod_YYYYMMDD
```

- Try to apply [Best Dockerfile Practices](https://docs.docker.com/engine/articles/dockerfile_best-practices/). You can also refer to our [Docker notes](http://markdownnotes.com/app/#/1dr4q7d/)

```sh
RUN apt-get update && apt-get install -y vim nginx \
	curl wget \
	php
```

## Makefile
Instead of writing bash commands, we prefer using Makefile to simplify operation task for all projects. You can create as many make command as you want, however, please list them all in README.md file.

Below is the list of Make commands you have to create by default. They will be used after we clone your repository:

- make build: Build docker image from resources
- make run: List of commands to successfully run server 
- make test: Run unittest
- make push_qa: Push docker image to docker registry under qa tag for qa testing
- make push_prod: Push docker image to docker registry under prod tag for production deployment
- make clean: Remove log files, remove docker image

Remember to use environment variables in your Makefile as they are different for each machines. Also, there are some reserved environment variables that operation will use during Jenkins build. Make sure you are not using them in .env file.

- PROJECT_NAME: Jenkins will use your git repository folder as project name.
- RELEASE_NUMBER: Jenkins will use your release number in git repository.
- TIMESTAMPS: Jenkins will use current unix time to set this value. It will depend on whether we are building docker image, push to qa or push to production.
- DOCKER_REGISTRY: We will set on runtime.

```sh
# A simple Makefile you could write for your application. We recommend using our default make build, make push_qa and make push_prod since they follow conventions for our operation team.

build:
	docker build --rm -t $(PROJECT_NAME):$(RELEASE_NUMBER)_dev_$(TIMESTAMPS) .
run:
	docker run --name=$(PROJECT_NAME) --rm -d -v $(pwd):/YOUR_SRC -e MYSQL_DB_HOST=$(MYSQL_DB_HOST) $(PROJECT_NAME):$(RELEASE_NUMBER)_dev_$(TIMESTAMPS) sh -c "service nginx start && tail -F /var/log/nginx/error.log"
test:
	docker exec -it project.abcxyz sh -c "cd YOUR_SRC/ && nosetests"
push_qa:
	docker tag $(PROJECT_NAME):$(RELEASE_NUMBER)_dev_$(TIMESTAMPS) $(DOCKER_REGISTRY)/$(PROJECT_NAME):$(RELEASE_NUMBER)_qa_$(TIMESTAMPS)
	docker push $(DOCKER_REGISTRY)/$(PROJECT_NAME):$(RELEASE_NUMBER)_qa_$(TIMESTAMPS)
push_prod:
	docker tag $(DOCKER_REGISTRY)/$(PROJECT_NAME):$(RELEASE_NUMBER)_qa_$(TIMESTAMPS) $(DOCKER_REGISTRY)/$(PROJECT_NAME):$(RELEASE_NUMBER)_prod_$(TIMESTAMPS)
	docker push $(DOCKER_REGISTRY)/$(PROJECT_NAME):$(RELEASE_NUMBER)_qa_$(TIMESTAMPS)
run_frontend:
	cd YOUR_SRC/frontend && python -m SimpleHTTPServer
clean:
	docker stop project.abcxyz && docker rm project.abcxyz
	cd /var/log/nginx && rm -rf *
```

# API Documentations
Please use Markdown as default format for API Documentation. Markdown is friendly to document and easy to convert into pdf. Team members can setup and use internal [Swagger Editor](http://editor.swagger.io/) to edit and view API documentation. 

## Swagger Editor
Swagger Editor lets you edit API specifications in YAML/JSON inside your browser and to preview documentations in real time. Once complete documenting, you can download the yml file, push to your team documentation Git repository (Follow conventions in the repo before push), the jenkins will automatically generate pdf file and send as email to operation team. 

You can ask operation team for the pdf documentation.

# Contacts
[Tran Anh Cuong, Frank](https://github.com/anhcuong)