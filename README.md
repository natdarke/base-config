# Documentation for Robot

* [Set Up Dev Environment](#set-up-dev-environment)
* [Develop a Feature](#develop-a-feature)
* [Test a Feature](#test-a-feature)
* [Deploy to Production](#deploy-to-production)

## Set Up Dev Environment

### Stage 1 : Install Development Dependencies

> Stages 1-3 will only have to completed once

1. Install Git

	* `sudo apt-get install git`
    * `sudo git config --global user.name "Your Name"`
    * `sudo git config --global user.email "name@domain.com"`
    

2. Install Node and NPM

	* **_To do: instructions including using NVM_**

3. Install Docker following these instructions:
    * Linux Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/
	* Windows: https://docs.docker.com/docker-for-windows/install/
	* Mac: https://docs.docker.com/docker-for-mac/install/

4. Install Docker Compose following these instructions:
	https://docs.docker.com/compose/install/

### Stage 2 : Map The Virtual Host to Local Host for Each Service  

5. In /etc/hosts, add the following lines
	
    ```
    127.0.0.1    frontend.robot
	127.0.0.1    contentful.robot
	127.0.0.1    static.robot
    ```

### Stage 3 : GitHub Access     
6. Create a GitHub account and ask us to add it to the Robot group

	> You will now be able to view the Robot repositories

7. Set up SSH keys on your GitHub account 
	
    * https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/ 
    * Use tabs at the top of the page to select Mac/Windows/Linux and follow the instructions

    > You will now be able to clone the Robot repositories using the SSH protocol without giving a password


### Stage 4 : Clone Repositories from GitHub

8. Clone the Root Application from GitHub. 
    > This just contains the Docker Compose config files and environment variable files i.e. deployment code
	
    * `mkdir -p ~/[your dev directory name]/robot/root && cd ~/[your dev directory name]/robot/root`
    * `git clone git@bitbucket.org:NatDarke/robot.git`

9. Clone each Service
	
    * `mkdir -p ~/[your dev directory name]/robot/service && cd ~/[your dev directory name]/robot/service`
    * `git clone git@bitbucket.org:NatDarke/robot-site.git`
	* `git clone git@bitbucket.org:NatDarke/robot-contentful.git`
	* ...etc
	
    > You don't HAVE to clone all services. You CAN just clone the services you need to work on, but you would need to edit `docker-compose.dev.yml` accordingly i.e. remove or comment-out `volumes` for the service. This causes the code already existing inside the container to be used, instead the of local code

10. Checkout the develop branch for each service
	
    * `cd ~/my-dev-work/robot/service/robot-site`
	* `git fetch`
	* `git checkout develop`
	* `cd ~/my-dev-work/robot/service/robot-contentful`
	* `git fetch`
	* `git checkout develop`

### Stage 5 : Start the App in Development Mode

11. Use Docker Compose to start
	
    * `cd ~/my-dev-work/robot/compose`
	* `sudo docker-compose -f docker-compose.dev.yml up -d`
	
    > This might take a few minutes, as some (or all) of the Docker images may have to be downloaded from Docker Hub, before the container can be created from them.

	> You will see how the progress of each container
	When it is finished you will see the message TO DO ""
	
12. View the working app 

    * In your browser go to `frontend.robot`

	> You should see the application working

13. See your changes
    * If you make a change to the front end service you will be able to see you change immediately
    * If you make a change to any other service you will have to refresh the page to see the results

    > This only applies if you have cloned the service and the volume is mounted in the `docker-compose.dev.yml` file (the default)

14. Now you are ready to build a feature or fix a bug
    * See Feature Development

## Develop a Feature

## Test a feature

## Deploy to Production
### To Do
* Dynamic virtual host names. Has to work per domain
* Databases for Strapi must be seperated from the application and e.g mounted as volumes

