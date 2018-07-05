# Auto deploy service using git hook

## Create ssh public key from PC (or CI server) if ~/.ssh/id_rsa.pub doesn't exist

    cd ~/.ssh
    ssh-keygen

## Create user account (eg www, the service will run as the user) and copy client's public key

    user add www

    cd /home/www
    mkdir .ssh 
    cd .ssh
    touch authorized_keys
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    (copy client ~/.ssh/id_rsa.pub string to authorized_keys)   

    (test connection from client: ssh www@deploy-server-ip)

## Config git repository in deploy server

    mkdir ~/git
    cd ~/git
    git init --bare ./project.git
    git clone ./project.git

    vi ~/git/project.git/post-receive
    (add hook script...)
    chmod +x ~/git/project.git/post-receive

### post-receive examples ( suppose to use supervisord to manage the service process )

* golang

        #!/bin/sh

        unset GIT_DIR
        srcpath="/home/www/git/project"
        dstpath="/usr/local/bin/project"

        cd $srcpath && git pull
        cp -f $srcpath/myapp $dstpath
        cp -f $srcpath/supervisord.conf $dstpath
        chmod +x $srcpath/myapp

        supervisorctl -c /etc/supervisord.conf restart myapp

* java

        #!/bin/sh

        unset GIT_DIR
        srcpath="/home/www/git/project"
        dstpath="/usr/local/bin/project"

        cd $srcpath && git pull
        cp -f $srcpath/*.jar $dstpath

        supervisorctl -c /etc/supervisord.conf restart myapp

* node js

        #!/bin/sh

        unset GIT_DIR
        srcpath="/home/www/git/project"
        dstpath="/usr/local/bin/project"

        cd $srcpath && git pull

        cp -f $srcpath/*.js $dstpath
        cp -f $srcpath/*.json $dstpath

        supervisorctl -c /etc/supervisord.conf restart myapp

* web

        #!/bin/sh

        unset GIT_DIR
        srcpath="/home/www/git/project"
        dstpath="/usr/share/web"

        cd $srcpath && git pull
        cp -rf $srcpath/build $dstpath


## Commit and push executables from PC (or CI server)
    mkdir ~/git
    cd ~/git
    git clone www@deploy-server-ip:/home/www/git/project.git
    cd ./project
    (cp files to ./project)
    git add .
    git commit -m "commit message"
    git push origin master

### Use makefile to automate deploy work

* make file content

        .PHONY: deploy

        BUILD_DIR = $(ROOT_DIR)/build
        DEPLOY_GIT_WORK_DIR = $(HOME)/git/project
        CURR_TIME = $(shell date +"%Y-%m-%d %H:%M:%S")
        HOST_NAME = $(shell hostname)

        deploy:
            cd $(DEPLOY_GIT_WORK_DIR) && git pull
            cp -rf $(BUILD_DIR)/myapp $(DEPLOY_GIT_WORK_DIR)
            cd $(DEPLOY_GIT_WORK_DIR) && git add . && git commit -m "From $(HOST_NAME) auto deploy at: $(CURR_TIME)" && git push origin master
            
* make file usage

        make deploy
        
