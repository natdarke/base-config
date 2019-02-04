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

## Architecture

This application uses a Docker micro-services architecture. This approach is highly modular but it is relatively complex. It consists of a group of networked Docker containers that are called services. Each service is there to do a specific job. 

All services except for 2 are backend services which have their own web server and RESTful API used to communicate with other services on the Docker network. 

The other 2 services

1. Reverse Proxy
2. Front End

### Reverse Proxy

The reverse proxy sits at the 'front' of the application as a sort of gatekeeper. All HTTP requests that arrive on the host computer are sent to the reverse proxy. It then forwards the request, typically based on the URL, domain or port, and waits for a response before forwarding it to the sender. 

For example, the reverse proxy might be set up to forward all requests to www.mydomain.com on port 80 or 443 to the front end service.

It also enables and manages HTTP requests between services inside the application on the Docker network.

The reverse proxy service is a nginx server inside a docker container. It uses the Docker API to query the Docker network. By doing this it can understand what services exist and dynamically generate an appropriate nginx config file.

### Front End

A simple backend Node server which serves a front end React application for ALL requests to the www subdomain. The React app is a single minified HTML file with browser-safe, inline JavaScript and CSS.

The React app handles all URL-based routing, using React Router and application state with Redux.

Server Side rendering is not necessary as long as you follow some simple rules:

1. All URLs must render without asynchronous, client side server calls. Search engines can handle javascript rendering but it won't wait to see what _might_ render
2. You manage Search Engines' expectations using a sitemap and webmaster tools for each Secrach Engine, such as Google Search Console, to check indexing status and optimize visibility


### Other Services

#### Contentful Cache
An interface to a MySQL service that allows content data from a Contentful SaaS account to be cached. Data is stored in the MySQL container as it does NOT need to persist between application restarts (the true data is stored on Contentful's SaaS platform and can easily be re-cached).

#### MySQL
A MySQL container run from an image taken staight from DockerHub. 

Data can be optionally stored in a directory on the host computer i.e. NOT in the container. This method allows data to  persist between restarts and is achieved by using Docker `volume` when the constainer is run (the same method that allows code to be edited during development).

#### 


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

IN order to be useful, NPM packages must be published to a repository. They can be published to npmjs.com or they _can_ be published to your own repository using repository manager software such as Nexus. We currently only use npmjs.com

#### NPM JS account

NPM packages on npmjs.com can be public or private.
```
Auth Token Required?
==================================================================

Feature                             Public          Private
------------------------------------------------------------------
To Install                          No              Yes
To Publish                          Yes             Yes
```

NPM packages can be owned by a individual or an organisation. 
```
Ownnership
==================================================================
                                        Package owned by
Feature                             Individual      Organisation
------------------------------------------------------------------
Publishable by many individuals     No              Yes         
Packages can be private             No              Yes
```     
Our packages are owned by an organisation and are private. Therfore, you will have to do the following to install or publish them:

1. Have an npmjs.com account
2. Ask us to add your account to the organisation and give you permission to install and publish
3. Login to your account on the command line once. This will generate a token which is stored in an ~/.npmrc file. You won't be required to login again as you will be authenticated by the stored token for future installations or publications (unless you change your npmjs.com password).

#### NPM Package Development

If you need to work with an NPM package things get a bit more complicated as the code is not is the repo for the service. It is a module used by the service, not part of the service per se.

NPM packages are designed to be used by many services or applications. They should be useful _in general_ and are not supposed to be application specific

Therefore, if you need to work with an NPM package as part of a feature story it is probably worth splitting the working on the NPM package into a seperate task, to be completed before the work on the feature story begins

To complete the feature story you might not need to change a service the all, other than to change the version of the NPM package it uses.

#### NPM Package Testing. 

In order of importance, the following testing can/should be done for an NPM package

1. Feature testing and regression testing for each service/application that uses the new version of the package.
2. Unit tests for the package
3. Demo application for testing possible use cases for the package

#### Steps to develop a feature on an NPM package

Summary:

1. Work on the package
2. Use the new version of the package in the service
3. Use the new version of the service in the root app

---

1. Work on the package
    
    1. Clone package repo 
        ```
        mkdir ~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked

        cd ~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked

        git clone git@github.com:natdarke/[package-name].git
        ```

    2. Instruct git to ignore `node_modules_linked` directory
    
        Add this line

        ```
        node_modules_linked
        ```
        ...to `~/[your dev directory name]/[app-name]/service/[service-name]/.gitignore`

        > nodes_modules_linked is just a temporary directory designed to house packages during development. Therefore it should not be part of the repo


    3. In the package, branch from `develop` to `feature/[feature-name]`
        ```
        cd ~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked/[package-name]

        git fetch 
        
        git checkout develop

        git branch feature/[feature-name]

        git checkout feature/[feature-name]
        ```
    4. Link the package code to the service code using `npm link`
        ```
        cd ~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked/[package-name]

        npm link

        cd ~/[your dev directory name]/[app-name]/service/[service-name]

        npm link [package-name]
        ```
        > This allows you to work on the package code during devlopement and see the changes you make reflected in the service. This has similar utility to using a `volume` inside a docker container. 

        > In the case of both NPM Link And Docker Volumes, you are asking a parent 'thing' to temporarily replace some child code with a newer, development version of the same code on your local machine
        ```
        Method          Parent              Child
        -------------------------------------------------------
        Docker volume   Docker container    JS app code
        NPM Link        JS app Code         JS NPM dependency
        ```
        

    5. Work on `~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked/[package-name]` with git as per normal
    6. Bump the package version, according to [Semantic Versioning](#semantic-versioning) principles, and git commit the change

        e.g. change 
        ```
        {
            "name": "my-package-name",
            "version": "1.0.0",
        ```
        to
        ```
        {
            "name": "my-package-name",
            "version": "1.1.0",
        ```
        
        in `~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked/[package-name]/package.json`
    7. Push `feature/[feature-name]` branch
        ```
        git push origin feature/[feature-name]
        ```

    8. In GitHub request a code review

    9. Make any necessary changes from the code review and re-commit, push and request code review. Do NOT re-bump the version, as versions indicate a release

    10. Publish the package from the `feature/[feature-name]` branch with tag `feature-[feature-name]`
        ```
        npm login
        ```
        > You will be prompted for a username a password. You must have an npmjs.com account and that account must have been given permission to publish by its owner

        ```
        cd ~/[your dev directory name]/[app-name]/service/[service-name]/node_modules_linked/[package-name]

        npm publish --tag feature-[feature-name]
        ```
        > Tagging a a version of an npm module during development prevents it from being installed by default. The default tag is 'latest' which indicates that it is the latest production ready version. By using a different tag you are saying "This version is NOT production ready" and NPM will not install it unless explicitly asked to do so (in package.json or CLI)

2. Use the new version of the package in the service
    1. Create feature branch for the service.
    ```
    cd ~/[your dev directory name]/[app-name]/service/[service-name]
    git fetch 
    git checkout develop 
    git branch feature/[feature-name]
    git checkout feature/[feature-name]

    ```

    2. Bump the service version, according to Semantic Versioning principles, and git commit the change

    `service/[service-name]/package.json` bump dependency version to be same as version in `service/[service-name]/node_modules_linked/[package-name]/package.json` e.g. `^1.1.0`
    3. Push service feature branch and proceed as if this was a normal service feature development.
    4. When you are satisfied that this package has had enough testing, re-publish with the tag `latest`

3. Use the new version of the service in the root app

## Semantic Versioning

Services and NPM modules should follow Semantic Versioning

Format example 	- 1.1.1 (major release, minor release, patch release)
```
Major release	- Changes that break backward compatibility
Minor release	- Backward compatible new features
Patch release	- Backward compatible bug fixes
```

Specifying dependency versions in package.json 

There are many ways to specifiy which version of a package should be used in packege.json, but the most important ones are:
```
"package-name"  : "1.1.1"   - version 1.1.1 with any tag

"package-name"  : "~1.1.1"  - version 1.1.1 with any tag
                            - OR any higher patch release 
                                e.g. 1.1.2, 1.1.47
                            - IF they are tagged with 'latest'
                            - Higher versions are preferable
					- 
"package-name"	: "^1.1.1"  - version 1.1.1 with any tag
                            - OR any higher patch release
                                e.g. 1.1.2, 1.1.47, 
                            - OR any higher minor release
                                e.g. 1.2.5, 1.12.0
                            - IF they are tagged with 'latest'
                            - Higher versions are preferable
```

## Deploy to Production

## To Do
* Dynamic virtual host names. Has to work per domain
* Databases for Strapi must be seperated from the application and e.g mounted as volumes

