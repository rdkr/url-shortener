# project-code

Project Code is a collaborative project to learn and teach Python, DevOps, and agile project best practices by building a url shortner service.

* more information - see [the wiki](https://github.com/rdkr/project-code/wiki)
* project management - see [trello](https://trello.com/b/FC8mke6j/project-code)
* members - see [CODEOWNERS](CODEOWNERS) and [AUTHORS](AUTHORS)

## Hosting
The server will be running Ubuntu 16.04.6 x64 on DigitalOcean's cloud platform. 

### Cloud-init.txt 
Configuration file that runs at server set-up. 
To use this file copy the contents of the file into the 'user data' box when setting up a DigitalOcean instance. 

#### Users
The current configuration sets up three user accounts and allows connection via SSH. Admin users are also provided paswordless sudo access if they have logged in via SSH. Users are also added to the docker group so they do not need to call sudo when running docker commands. 
Root ssh access has been disabled.

`ssh (USERNAME)@(IP ADDRESS)` to access the server where (USERNAME) is replaced by your username and (IP ADDRESS) is replaced by the IPV4 address of the created droplet. it should look something like `ssh jon@142.93.37.197`

#### Packages
- apt-transfer-https: used to install docker via the https repository.
- docker-ce: installs the docker-ce version via the official docker repository. The config uses the relevant gpg key to ensure authenticity. Presently, the installation is specific to Ubuntu 16.04 Xenial. 

#### Nginx
See this [link](https://www.digitalocean.com/community/tutorials/how-to-run-nginx-in-a-docker-container-on-ubuntu-14-04) for how Nginx was set up.

The following has been added to the Cloud-init.txt file

```
runcmd:
    - "docker run --name docker-nginx -p 80:80 -d nginx
```

* "runcmd:" runs the specified command in the cloud-innit.txt script
* "docker" is the containerisation software. Docker maintains a site called Dockerhub, a public repository of Docker files (including both official and user-submitted images). The image downloaded in the above command is the official Nginx one, which saves us from having to build our own image.
* "run" is the command to create a new container
* The "--name" flag is how we specify the name of the container (if left blank one is automatically assigned but this may make organisation of containers more difficult)
* "-p" specifies the port we are exposing in the format of -p local-machine-port:internal-container-port. In this case we are mapping Port 80 in the container to Port 80 on the server.
* "nginx" is the name of the image on dockerhub, Docker will download the image automatically if it is missing.
* The "-d" flag allows us to run this container in the background, which is what we want for a website.

Type `docker ps -a` to view any current containers, currently that should be limited to nginx.

Type `docker stop docker-nginx` to stop the container.

Type `docker start docker-nginx` to start the container.

Type `docker rm docker-nginx` to remove the container (WARNING THIS WILL DELETE IT).


## Application

### app.py
This starts flask and currently returns `hello, world`. Configuration is to be done via environment variables and CLI options wherever possible. 

### Makefile
A Makefile is a collection of rules. Each rule is a recipe to do a specific thing. 
* `build`: Builds a docker image using the Dockerfile in the pwd.
* `run`: Runs the docker image that was built and links the pre-specified ports.
* `dev`: Runs a local version of the flask app in development mode
* `install`: Installs requirements as specified in requirements.txt and requirements_dev.txt
* `lint`: Runs pylint and pydocstyle on python files within the project directory.

### requirements.txt
Contains a list of items to be installed using pip install. Run "pip install -r requirements.txt" to install the required packages or type `make install`.

### Dockerfile
Defines the image to be built and runs the flask app when the container is created via `docker run`. The Dockerfile is to be used for setting up the production version of the app. Presently it runs the flask app in production mode and specifies a host which allows for external network connections. 

## CI/CD

### .pylintrc
This is the configuration file for the [pylint](https://github.com/rdkr/project-code/wiki/Python). Currently everything has been left to the default settings with the exception of the constant naming style rule which has been changed to allow all title cases (rather than requiring it to be in capitals).

### .githooks
Acts as the central location for the project [git hooks](https://github.com/rdkr/project-code/wiki/Git). By default the git hooks directory is $GIT_DIR/hooks, however this has been changed to ".githooks" via the `core.hooksPath` command which is initiated by the `make install` command. 

#### pre-commit
A git hook which currently executes the `make-lint` command when `git commit` is run. 

### requirements_dev.txt
Similar to the requirements.txt file, however the items within this file relate to developer tools (pylint and pydocstyle) rather than for the app.py application. Run "pip install -r requirements_dev.txt" to install the required packages or type `make install`.