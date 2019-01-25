# Documentation for Robot

* [Set Up App on Local Host](#set-up-app-on-local-host)
* [Develop a Feature or Fix a Bug](#develop-a-feature-or-fix-a-bug)
* [Testing](#testing)
* [Develop with NPM packages](#develop-with-npm-packages)
* [Develop with Multiple services](#develop-with-multiple-services)

---

> This application and documentation assumes the following directory structure.

```
~/[your dev directory name]/[app name]/root
~/[your dev directory name]/[app name]/service
~/[your dev directory name]/[app name]/service/[service-1-name]
~/[your dev directory name]/[app name]/service/[service-2-name]
```
---

## Set Up App on Local Host

> **This is necessary for development AND testing**


1. Install and Configure Git
    ```
	sudo apt-get install git

    sudo git config --global user.name "Your Name"

    sudo git config --global user.email "name@domain.com"
    ```

2. Install Docker following these instructions:
    * Linux Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/
	* Windows: https://docs.docker.com/docker-for-windows/install/
	* Mac: https://docs.docker.com/docker-for-mac/install/

3. Install Docker Compose following these instructions:
	https://docs.docker.com/compose/install/
 
4. Map The Virtual Host to Local Host for Each Service.

    * In /etc/hosts, add the following lines
	
        ```
        127.0.0.1    frontend.robot
        127.0.0.1    contentful.robot
        127.0.0.1    static.robot
        ```
  
5. Create a GitHub account and ask us to add it to the Robot group

	> You will now be able to view the Robot repositories

6. Set up SSH keys on your GitHub account 
	
    * https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/ 
    * Use tabs at the top of the page to select Mac/Windows/Linux and follow the instructions

    > You will now be able to clone the Robot repositories using the SSH protocol without giving a password


7. Clone 'Root' from GitHub.
    
    > 'Root' is just the Docker Compose config files and environment variable files that allow you to start the app.

    > `[app-name]` is 'robot', in this case
	
    ```
    mkdir -p ~/[your dev directory name]/[app-name]/root
    
    cd ~/[your dev directory name]/[app-name]/root

    git clone git@github.com:natdarke/robot-root.git
    ```

8. Use Docker Compose to start the app
	
    * #### For Development
        ```
        cd ~/[your dev directory name]/[app-name]/root

        sudo docker-compose -f docker-compose.dev.yml up -d
        ```
    * #### For Regression Testing
        ```
        cd ~/[your dev directory name]/[app-name]/root

        sudo docker-compose -f docker-compose.test.yml up -d
        ```
    * #### For feature Testing
        > This file needs to be created and deleted by the developer, per feature
        ```
        cd ~/[your dev directory name]/[app-name]/root

        sudo docker-compose -f docker-compose.[feature-name].yml up -d
        ```
	
    > This might take a few minutes, as some (or all) of the Docker images may have to be downloaded from Docker Hub, before the container can be created from them.

	> You will see how the progress of each container. When it is finished you will see the message TO DO ""
	
9. View the working app 

    * In your browser go to `frontend.robot`

	> You should see the application working

10. Now you are ready to build a feature, fix a bug or test
    

---

## Develop a Feature or Fix a Bug

### Assumptions
* The application is working on your local machine, following the instructions in [Set Up App on Local Host](#set-up-app-on-local-host)
* The feature only requires work on 1 service. If you your feature requires that you work on more than one service, see [Develop with Multiple services](#develop-with-multiple-services)
* You don't need to work on any NPM packages. If you do, see [Develop with NPM packages](#develop-with-npm-packages)

### Background reading
*  GitFlow methodology is an essential part of the the workflow we use. Understanding it would be very useful.

### General Rules For Git
* You should view your local master and develop branches as 'read only'
* Do not change them with local commits or merges
* The only way they should change is by updating them from the remote repo using pull
* New branches should always be made from an up-to-date develop
* When you are finished, push the branch to the remote repo, where it can be merged into develop after code-review and testing

### Steps

1. Get the code for the service you need to work on

    * IF you HAVE NOT cloned this service before

        1. Clone the service you need to work on
            
            ```
            mkdir -p ~/[your dev directory name]/robot/service
            
            cd ~/[your dev directory name]/robot/service

            git clone git@github.com:natdarke/[service-name].git

            ```
            > `[service-name]` will be the same name used in `docker-compose.yml` or `docker-compose.dev.yml`
        2. Fetch remote branches and checkout develop
            ```
            cd [service-name]
            git fetch
            git checkout develop
            ```

    * IF you HAVE cloned this service before
        
        1. Make sure the develop branch exists locally and is up to date
            ```
            cd ~/[your dev directory name]/[app-name]/service/[service-name]
            git fetch
            git checkout develop
            git pull develop
            ```
            If you have followed the [General Rules For Git](#general-rules-for-git) (above) there will NOT be a merge conflict.

2. Create a feature branch from the develop branch and check it out
	
    ```
    git checkout develop
    
    git branch feature/[feature name]

    git checkout feature/[feature name]
    ```
    > base the feature name on the story or task name/id

3. Disable volumes

    In `docker-compose.dev.yml` enable the `volume` for the service you need to work on, by removing the `#` comment on 2 lines

    **_Do not commit these changes in Git_**

    For example, if the service you need to work on is `static` then you need to change
    ```
    static:
        image: natdarke/robot-service-static:develop
        # volumes:
        # - "../services/static/src:/var/www"
        ports: 
        - "5003:80"
        environment:
        - environment/dev/robot-service-static.env
    ```
    to
       
    ```
    static:
        image: natdarke/robot-service-static:develop
        volumes:
        - "../services/static/src:/var/www"
        ports: 
        - "5003:80"
        environment:
        - environment/dev/robot-service-static.env
    ```
    > The path before the colon (e.g. `../services/static/src`) is a relative path to the source code of the service you checked out from GitHub, so make sure your directory structure is reflects this.

4. Start / Re-Start the application

    ```
    cd ~/[your dev directory name]/[app-name]/root

    sudo docker-compose -f docker-compose.dev.yml down

    sudo docker-compose -f docker-compose.dev.yml up -d
    ```

5. See your changes

    To see any changes you must have cloned the service and mounted it as a `volume` in the `docker-compose.dev.yml` file (see stages 1-3)

    If you make a change to the front end service you will be able to see you change immediately

    > This because React uses a special development server that listens to, and serves, the uncompiled source code

    For other services (i.e. back end) you will have to refresh the page to see the results.

    > You may realise that the code you need to change is, in fact, part of an NPM package. If this is the case, follow the instructions in [Develop with NPM packages](#develop-with-npm-packages)

6. Make code changes on this branch and commit each change until the task is complete
	
    ```
    git commit -am "Description of change"
    ```

7. Push the feature branch
    
    ```
    git push origin feature/[feature name]
    ```

8. Create a feature tag
    * The purpose of tagging 2 fold:
        1. To indicate that a commit represents to completion of a feature, subject to testing
		2. To trigger an image build in Docker Hub, tagged with the same name
	* Tag your latest commit:	
        
        ```
        git tag -a feature-[feature name]-1 -m "Feature [feature name] is ready for testing"
        ```

        > Incrament the tag number by one each time this feature needs to be tested 
	* Or tag a commit by id: (only do this if, for some reason, your latest commit is not the commit that completed this feature)
        ```
        git tag -a feature-[feature name]-1 [commit-id] -m "Feature [feature name] is ready for testing"
        ```
	    > Be careful not to give your tag the same name as the branch. It will cause errors.

9. Push the feature tag
    ```
    git push origin feature-[feature name]-1
    ```
    > This triggers an automatic feature image build on DockerHub, tagged with the same name i.e. feature-[feature name]-1		

10. In `service/root` branch `develop` to `feature/[feature-name]`.
11. In `docker-compose.test.yml` change the tag of the service image to be `feature-[feature-name]` e.g. change
    ```
    front-end:
        image: natdarke/robot-service-front-end:develop
    ```
    to
    ```
    front-end:
        image: natdarke/robot-service-front-end:feature-[feature-name]
    ```
12. Commit and push
13. Instruct tester to test the feature using `docker-compose.test.yml` on branch `feature/[feature-name]`

---
### Working On Multiple Services
---
### Working with NPM packages

#### Development

If you need to work with an NPM package things get a bit more complicated as the code is not is the repo for the service. It is a module used by the service, not part of the service per se.

NPM packages are designed to be used by many services or applications. They should be useful _in general_ and are not supposed to be application specific

Therefore, if you need to work with an NPM package as part of a feature story it is probably worth splitting the working on the NPM package into a seperate task, to be completed before the work on the feature story begins

To complete the feature story you might not need to change a service the all, other than to change the version of the NPM package it uses.

#### Testing. 

In order of importance, the following testing can/should be done for an NPM package

1. Feature testing and regression testing for each service/application that uses the new version of the package

2. Unit tests for the package

3. Demo application for testing possible use cases for the package

#### Steps

1. Clone package repo to `service/[service-name]/node_modules_linked/[package-name]`
2. Add `service/[service-name]/node_modules_linked` to .gitignore
3. Branch from `develop` to `feature/[feature-name]`
4. NPM Link from `service/[service-name]/node_modules_linked/[package-name]` to `service/` 
5. Work on `service/[service-name]/node_modules_linked/[package-name]` with git as per normal
6. When you are finished, bump version in `service/[service-name]/node_modules_linked/[package-name]/package.json` and commit
7. Push `feature/[feature-name]` branch
8. Pull request. Code review. Change. Commit. push `feature/[feature-name]` branch.
9. Publish `[feature-name]` branch with tag `feature-[feature-name]`
10. Create feature branch for the service.
11. In the service feature branch `service/[service-name]/package.json` bump dependency version to be same as version in `service/[service-name]/node_modules_linked/[package-name]/package.json` e.g. `^1.1.0`
12. Push service feature branch and proceed as if this was a normal service feature development.
13. When you are satisfied that this package has had enough testing, re-publish with the tag `latest`

## Deploy to Production

## To Do
* Dynamic virtual host names. Has to work per domain
* Databases for Strapi must be seperated from the application and e.g mounted as volumes

