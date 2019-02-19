# Documentation for Robot

* [Architecture](#architecture)
    * [Root](#root)
    * [Services](#services)
        * [Reverse Proxy](#reverse-proxy)
        * [Front End](#front-end)
        * [Back End HTTP Services](#backend-http-services)
            * [Contenful Cache](#contentful-cache)
        * [Backend Database Services](#backend-database-services)
            * [MySql](#mysql)
    * [NPM Packages](#npm-packages)
    * [Volumes](#volumes)
* [Set Up App on Local Host](#set-up-app-on-local-host)
* [Develop a Feature or Fix a Bug](#develop-a-feature-or-fix-a-bug)
* [Testing](#testing)
* [Develop with NPM packages](#develop-with-npm-packages)
* [Develop with Multiple services](#develop-with-multiple-services)


---

## Architecture

This application uses a Docker micro-services architecture. This approach is highly modular but it is relatively complex. It consists of a group of networked Docker containers that are called services. Each service is there to do a specific job.

### Root
The root is just some config files used to start up the application. It consists of

1. Docker Compose yml files
    * Used to bootstrap the app, including downloading and caching the Docker image for each service, running one or many containers from each image and networking the containers on a Docker network
    * There are different yml files for development, testing and production
2. Environment variables for each service

### Services

1. One Reverse Proxy
2. One Front End
3. Many Backend Services


#### Reverse Proxy

The reverse proxy sits at the 'front' of the application as a sort of gatekeeper. When an HTTP(S) request arrives on the host computer, it is forwarded to the reverse proxy service in the application. The reverse Proxy then forwards the request to one of the other services, typically based on the URL, domain and/or port.

For example, the reverse proxy might be set up to forward all requests for www.mydomain.com on port 80/443 to the front end service, and all requests for admin.mydomain.com on post 80/443 to the CMS service.

It also enables and manages HTTP requests between services inside the application on the Docker network.

The reverse proxy service is a nginx server inside a docker container. It uses the Docker API to query the Docker network. By doing this it can understand what services exist and dynamically generate an appropriate nginx config file.

It can also provide load balancing, caching, compression and other useful utilities.


#### Front End

A simple backend Node server which serves a front end React application for ALL requests to the www subdomain. The React app is a single minified HTML file with browser-safe, inline JavaScript and CSS.

The React app handles all URL-based routing, using React Router and application state with Redux.

Server Side rendering is not necessary as long as you follow some simple rules:

1. All URLs must render without asynchronous, client side server calls. Search engines can handle javascript rendering but it won't wait to see what _might_ render
2. You manage Search Engines' expectations using a sitemap and webmaster tools for each Secrach Engine, such as Google Search Console, to check indexing status and optimize visibility

Even a front end service needs a simple backend web server and it would be quite easy to extend the routing on this web server include API endpoints on /api, whilst serving the front end app on other urls. Mixing the front end app and backend API like this is quite common, but using a Docker micro-services architecture allows us to keep things completely modular, so we prefer to keep the font end as simple as possible and delegate API responsibilities to other services.


#### Backend HTTP Services

Backend HTTP services have:

1. Web Server
2. RESTful API

The API is exposed to other backend http services on the local Docker network via the Reverse Proxy.

It might also need to be exposed to the internet. Common reasons for this would be:
1. It is used by the front end service after the front end app has been served i.e. by the web browser
2. It needs to receive POST requests from a 3rd party SaaS app (web hooks)

##### Contentful Cache

Contentful is a SaaS headless CMS. This service caches content from a Contentful account and makes it available to the Front End service. 

The purpose is to provide an API for the Front End to use that is much faster than making API calls directly to Contentful.

It has 3 main features

1. Content In
    - An API to allow Contentful to POST data
    - This happens using web hooks set up in Contentful. When content is added or changed Contentful POSTs the changes to   
2. Content Out
    - An API to allow the Front End service to GET data
3. Admin
    - An API to reset/rebuild the cache and other utilities

The cached content is stored in a mySQL database in a mySQL service. Therefore, this service is an API between a mySQL service and other services, both internal (Front End) and external (Contentful)

Performance is significantly better than making API call directly to Contentful because:

1. The API and the MySQL tables are customised for the needs of this application. This means that:
    1. There is no need for multiple API calls, complex database queries or unnecessary processing
    2. JSON responses are formatted to be convenient for the front end
2. We are in control of hardware performance. It is not affected by performance issues or outages on Contentful's CDN. We can dedicate as many resources as needed to make this service performant. 

An additional benefit of the cache is that API calls directly to Contentful are rarely needed. Contentful seem to change their price plans regularly but they are sometimes based on the number API calls, in which case one of the cheaper plans can be used.

Data is stored in the MySQL container, not [Volumes](#volumes), as it does NOT need to persist between application restarts (the real data is stored on Contentful's SaaS platform and can easily be re-cached).

#### Backend Database Services

##### MySQL
A MySQL container run from an image taken staight from DockerHub. 

Data can be optionally stored in a directory on the host computer i.e. NOT in the container. This method allows data to  persist between restarts and is achieved by using Docker [Volumes](#volumes) when the constainer is run (the same method that allows code to be edited during development).

### Volumes
A volume is simply a directory on the host machine that can be accessed from inside a container.

In terms of applilcation architecture, it is used to store data that needs to persist between application restarts i.e. is stored on a hard drive, not in RAM.

Typically this means that a DB Service (container) will use a volume on the host machine to store its data.

This can also be accomplished using a service from your cloud infrastructure provider e.g. AWS "EBS Volume", or Digital Ocean "Block Storage Volume" - you can mount a volume completely outside your host machine to its /mnt directory and from there, mount it to the container. This is a very flexible and safe way to keep valuable data available to a Docker-based micro-services application.

Volumes are also essential when developing using containers. [Using Volumes in Development](#using-volumes-in-development)


## Development

### Local Directory Structure
> This application and documentation assumes the following directory structure during development

```
~/[your dev directory name]/[app name]/root
~/[your dev directory name]/[app name]/service
~/[your dev directory name]/[app name]/service/[service-1-name]
~/[your dev directory name]/[app name]/service/[service-2-name]
```
### Using Volumes in Development

* It is very inconvenient to work on code inside a container during development, not least because it is a purely CLI environment
* A developer can mount a volume from their local machine inside a container, thereby allowing them to work on local code but have that code used within an application container/service
* Docker Compose provides a convenient way to do this in a yml file


## Set Up App on Local Host (Linux)

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
    * #### For Regression Testing and UAT
        ```
        cd ~/[your dev directory name]/[app-name]/root

        sudo docker-compose -f docker-compose.test.yml up -d
        ```
    * #### For feature Testing
        > This file needs to be created and deleted by the developer, per feature
        ```
        cd ~/[your dev directory name]/[app-name]/root

        git fetch

        git checkout feature/[feature-name]

        git pull feature/[feature-name]

        sudo docker-compose -f docker-compose.test.yml up -d
        ```
	
    > This might take a few minutes, as some (or all) of the Docker images may have to be downloaded from Docker Hub, before the container can be created from them.

	> You will see how the progress of each container. When it is finished you will see a confirmation message
	
9. View the working app 

    * In your browser go to `frontend.robot`

	> You should see the application working

10. Now you are ready to build a feature, fix a bug or test
    

---

## Develop a Feature or Fix a Bug

Developing a feature involves complex processes for collaboration, building and deployment. Git, GitHub, NPM, NPM JS, Docker, Docker Compose and Docker Hub all play their part 

### Assumptions
* The application is working on your local machine, following the instructions in [Set Up App on Local Host](#set-up-app-on-local-host)
* The feature only requires work on 1 service. If you your feature requires that you work on more than one service, see [Develop with Multiple services](#develop-with-multiple-services)
* You don't need to work on any NPM packages. If you do, see [Develop with NPM packages](#develop-with-npm-packages)


### Branches

We use GitFlow as our branching strategy for services and modules. https://nvie.com/posts/a-successful-git-branching-model/

- Permanent
    - master
    - develop

- Temporary
    - feature/[story-id]
    - bug/[task-id]
    - release/[release-num]
    - hotfix/[task-id]

#### Master Branch

Master is a permanent branch that reflects the current code in production

Rules

1. Merging to master should only ever happen in 2 situations:
    1. Standard Release to Production
        * After all testing has been complete on the release branch, release in merged into master and released to production
    2. Emergency Bug Fix
        * When a bug in production is found, a hotfix branch is made from master, the bug is fixed and then merged into master and develop
2. Only merge on the remote repo
3. Never use your local master branch
4. Never branch from master (always from develop)
5. All merges to master should be conflict free. If this is not true, you are not following the rules.

#### Release

A release branch is a temporary branch created when some work has been completed e.g. at the end of a sprint after several features have been developed and bugs fixed

* Used for regression and UAT testing
* Should be named `release/[version]`
* Bugs found during testing are fixed on this branch and merged into develop.
* When testing is complete, it is merged into master and deleted

#### Develop Branch

`develop` is a permanent branch used for development. Like the master branch it is never deleted and should be viewed as a read only branch, except for merges done in the remote repo

If code is on the develop branch it means, very specifically, that :

1. All features on it have been tested
2. All code has been code reviewed

Rules
1. Always start new work by pulling `develop` and branching from that. If there is a conflict then something is very wrong and you should probably get a fresh copy by deleting your local develop branch and pulling from the remote again
2. Never merge into local develop branch
3. Never push to remote develop branch
4. Never delete remote develop branch
5. Always merge to develop in the remote repo (GitHub) but only after a PR, code review and feature testing

#### Feature Branch

Feature branches are temporarily created for an individual developer to complete a story

* One feature branch per story or task.
* Should be named `feature/[feature-name][story-id]`. This could include the story id from you issue tracking softward e.g. JIRA
* Used to make feature builds i.e. Docker images
* Should be deleted after use
    * After a feature build has been tested, the branch is merged into develop in GitHub and the branch can be deleted, both locally and on GitHub

#### Bug

Same as a feature branch but used to fix a bug, rather than build a new feature
* Should be named `bug/[bug-name][bug-id]`


### Steps

1. Get the code for the service you need to work on

    * If you HAVE NOT cloned this service before

        1. Clone the service you need to work on
            
            ```
            mkdir -p ~/[your dev directory name]/[app-name]/service
            
            cd ~/[your dev directory name]/[app-name]/service

            git clone git@github.com:natdarke/[service-name].git

            ```
            > `[service-name]` will be the same as the name used in `docker-compose.yml` or `docker-compose.dev.yml`
        2. Fetch remote branches and checkout develop
            ```
            cd [service-name]
            git fetch
            git checkout develop
            ```

    * If you HAVE cloned this service before
        
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

3. Enable volumes

    In `docker-compose.dev.yml` enable the `volume` for the service you need to work on, by removing the `#` comment on 2 lines

    **_Do not commit these changes in Git. Discard all changes after development. **

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

    If you are working on the front end service you will be able to see you change immediately

    > This because React uses a special development server that listens to, and serves, the uncompiled source code

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

        > Tag names are currently very similar to branch names. This should change to avoid errors
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

Working on multiple services for a single story is no different from working on a single service but for each service you must ensure that you :

1. During development enabled the `volume` in docker-compose.dev.yml
2. For testing, make sure you change the tag of the appropriate image name in docker-compose.test.yml 
---
### Working with NPM packages

NPM packages are published modules that can be added as a dependency to an application (or another NPM package) by listing it in "dependencies" in package.json. 

Therefore, an NPM package will be stored in 2 locations on the internet:

1. A Git repository
    - Purpose
        - Development
    - Contains
        - Source code
        - package.json including devDependencies
        - LICENCE and README.md
        - Webpack config
2. An NPM repository
    - Purpose
        - Distribution
    - Contains
        - Source code
        - package.json NOT including devDependencies
        - LICENCE and README.md
        - Distribution code (created at build time)
        - SourceMap file for debugging (created at build time)

#### Source Code

- Uncompiled
- Usually not browser safe
- Not necessarily Node safe
- Can use latest ES features that might not work in the run time environment
- Needs to be compiled to guarantee it will work

#### Distribution Code

- Compiled 
- Browser safe
- Node safe
- Needs to be built before publishing
- Can be debugged if a SourceMap file was generated during the build. 

#### Security

- Malicious code can be hidden distribution code because it is compiled and not human readable
- 3rd parties can and should view distribution code with suspicion and may prefer to use the source code instead and do their own compilation

#### Versioning

- The version of an NPM package is found in package.json
- It should follow the Semantic Versioning format

#### Publishing to NPM JS

In order to be useful, NPM packages must be published to a repository. They can be published to npmjs.com or they _can_ be published to your own repository using repository manager software such as Nexus. We currently only use npmjs.com

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
Ownership
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



 
#### NPM Tagging

Tagging is a useful way to indicate the intended use of a package version.

NPM packages need to be published during development. How do you stop a development versions of published packages being used accidentally? Tagging is also a solution to this problem.

- Tags indicate the purpose of a specific version 
	- e.g. develop, test, latest
- Tags cannot be used on more than one version. It is supposed to indicate the the most recent version of something 
	- e.g. if 0.0.10 is tagged as 'develop', it indicates that it is the most recently published development version of the package
- However, a version can have many tags
	- e.g. if 0.0.10 is tagged as 'develop' and 'test', it indicates that it is the most recently published development and test version of the package.
- 'latest' tag
	- 'latest' is a special tag. 
		- It is the only tag that is used by NPM
		- It means 'most recent production version'
	- By default, NPM tags everything with 'latest' unless you tag it with something else. 
	- The package version tagged with 'latest' will be installed automatically unless the user specifies a version or a tag
- Other tags
	- The main purpose for using other tags is to publish a new version that is NOT tagged as 'latest' and will, therefore, NOT be installed by automatically, either when using ^ or ~ when specifying dependency versions in package.json or when installing using the command line
	- This is useful for pre-release versions of a package that need to be published for development or testing.

#### NPM Package Development

If you need to work with an NPM package the workflow gets a bit more complicated as the code is not in the repo for the service. It is a module used by the service, not part of the service _per se_.

NPM packages are designed to be used by many services or applications. They should be useful _in general_ and are not supposed to be application specific

Therefore, if you need to work with an NPM package as part of a feature story it is probably worth splitting the working on the NPM package into a seperate task, to be completed before the work on the feature story begins

To complete the feature story you might not need to change a service at all, other than to change the version of the NPM package it uses.


#### NPM Package Testing. 

NPM modules need to be tested as individual entities as they are intended for general use in many applications. However, ideally they would also have regression testing in all apps that use it

The following testing can be done for an NPM package

1. Feature testing during development
2. Regression testing for each service/application that uses the new version of the package.
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

    2. Bump the service version, according to Semantic Versioning principles, and git commit the change `service/[service-name]/package.json` bump dependency version to be same as version in `service/[service-name]/node_modules_linked/[package-name]/package.json` e.g. `^1.1.0`
    3. Push service feature branch and proceed as if this was a normal service feature development.
    4. When you are satisfied that this package has had enough testing, re-publish with the tag `latest`

3. Use the new version of the service in the root app
    1. Create a new fetaure branch from master
        - Root is, essentially, a config and doesn't need a full branching strategy like services and NPM packages
    2. In `docker-compose.test.yml` change the version of the 



#### Versioning and Tagging
- Tagging is a concept used in Git, NPM, Docker and other systems to add semantic value to versioning.
- Version Number vs Tag
    - Version Number
        - i.e. a version ID
        - Mandatory. i.e. all 'versions' have some kind of id or number
        - Numeric or alpha-numeric
        - Sometimes randomly generated
        - Sometimes manually created, in which case [Semantic Versioning](#semantic-versioning) should be used
            - e.g. 1.2.1
    - Tag
        - i.e. key word
        - Optional, in addition to a version number
        - Usually a word or words
        - Should be descriptive metadata
        - e.g "develop" might indicate that this version is the latest version in use during development
- Version Number and Tags in use
    - NPM
        - A publication (i.e. a release) is equivalent to a 'version'
        - The version number is created manually in package.json
            - It is recommended that you use [Semantic Versioning](#semantic-versioning) for this
        - Published NPM packages can be tagged
    - Git
        - A code commit is equivalent to a 'version'
        - The version number is the commit ID
            - Auto-generated
            - Non-semantic
        - Any commit can be tagged
        - Pushing a tagged commit to GitHub can trigger builds in DockerHub. 
            - Needs to be set up in DockerHub
            - Will work on any branch
            - Useful for automating feature builds
    - Docker
        - An image build can is equivalent to a version
        - The version number is the build id


## Semantic Versioning

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



## Testing

### Unit Testing

### Feature Testing

### Smoke Testing

### 


## Deploy to Production

## To Do
* Dynamic virtual host names. Has to work per domain
* Databases for Strapi must be seperated from the application and e.g mounted as volumes

