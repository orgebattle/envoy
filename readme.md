# Envoy Use Case Demo

> TESTED ON UBUNTU14.04
> Assuming deploying a Laravel Project
> You may customize to any platform, framework

## 1. What is this? What for?

[Envoy is a SSH task runner](https://laravel.com/docs/envoy) written in PHP language by the Laravel team. SSH task runner enables you to automate a predefined SERVER JOB in your LOCAL COMPUTER, WITHOUT having to manually ssh in to the server.

**It was born in Laravel world, but can be applicable to any project (framework or language agnostic).** 

This repository is an example of an Envoy script (`envoy.blade.php`), that demos the usefulness and the powerfulness of it. 

## 2. Demo scenario

Note that Envoy is not a deployment tool, such as capistrano, fabric, or deployer, but can be used as such. Git neither is a deployment tool. 

In this example Envoy script, written for code deployment scenario in conjunction with Git, poses a strategy that lowers the headache of code deployment to the remote server by adopting the following strategy, you can:

1.  achieve zero down time of the service.
2.  achieve zero conflict in remote server.
3.  maintain release histories (by not overwriting the previous release).

To do that, the script takes `git clone` strategy rather than `git pull` or `git fetch`. Once the clone is done, the script symlinks the current clone to the document root of the web server.

This script may not fit your needs 100% though, I bet you get the idea what to do next with Envoy.

## 3. Try yourself!

### 3.1. Install `laravel/envoy` executable 

This is an one time process. We need the `envoy` executable.

```bash
# at your local computer

~ $ composer global require laravel/envoy
~ $ envoy --version # Laravel Envoy version x.x.x
```

> **`Note`** 
> 
> Make sure to place the `~/.composer/vendor/bin` directory in your `PATH` OS variable, so that that the `envoy` executable can be accessible when you run the `envoy` command in your terminal.

### 3.2. Download the recipe scripts

`envoy.blade.php` is a task definition file, and must be placed in every project.

```bash
# at your local computer

~ $ cd project
~/project $ wget https://raw.githubusercontent.com/appkr/envoy/master/envoy.blade.php
```

### 3.3. Installing SSH keys on each server

This process has nothing to do with Envoy, but required for this demo script to be working. 

Considering the deployment process, the servers must be able to talk to each other. 

On the local machine, you push the code to the Github repository, then you would publish `$ envoy run deploy` task (task will propagated to the remote server through the ssh). Then the remote server clones the code from the Github repository. In this scenario,

1.  Your local computer MUST have:
    1.  ssh PRIVATE key to connect to the remote server
    2.  ssh PRIVATE key to connect to the Github
2.  The remote server MUST have:
    1.  ssh PUBLIC key to authenticate the local machine 
    2.  ssh PRIVATE to connect to the Github
3.  The Github MUST have:
    1.  ssh PUBLIC key to verify your local machine
    2.  ssh PUBLIC key to verify the remote server
    
We assume that 1 and 2.i are ready. For 2.ii, see [this page](https://help.github.com/articles/generating-an-ssh-key). Note the fact that, from the perspective of Github, our remote server is just a Github client, like your local computer. So, the remote server has to have a private key to connect to Github. 

### 3.4. Edit the configuration & Run the first envoy task

Edit your variables at `@server` and `@setup` section of `envoy.blade.php`.

```php
// envoy.blade.php

// ip address, domain, or alias. 
// Whatever name you use to connect to the server via ssh.
@servers(['vm' => 'deployer@envoy.vm'])

@setup
  $username = 'deployer';                       // username at the server
  // ...
@endsetup
```

Let's run the very first ssh task. In behalf of you the envoy ssh into the remote server and run the 'hello' command and print the result on your computer. 

```bash
# at your local computer

~/project $ envoy run hello
# [envoy.vm]: Hello Envoy! Responding from envoy.vm
```

### 3.5. Customize your envoy script

Following envoy tasks are predefined out of the box. Why not add or modify for yours?

-   `hello`
    : Print "Hello Enovy!" to check ssh connection
-   `deploy`
    : Publish new release
-   `list`
    : Show list of releases
-   `checkout`
    : Checkout(rollback) to the given release (must provide `--release=/path/to/release`)
-   `prune`
    : Purge old releases (must provide `--keep=n`, where n is a number of release to keep)

For example,

```bash
# at your local computer
# after pushing new code to Github ...

~/project $ envoy run deploy
```

This produces the following on the remote server.

```bash
web
├── releases
│   └── release_YmdHis
│       ├── # other dirs or files
│       └── .env -> /home/deployer/web/shared/.env
├── shared
└── envoy.appkr.kr -> /home/deployer/web/releases/release_YmdHis
```

Where `web/releases` is housing of all releases, distinguished by directory name of `release_YmdHis`. `web/shared` is a shared resources that every release has to link to, like cache storage, user uploaded files, etc. `web/envoy.appkr.kr` is the active release which is symbolic linked to the document root of the web server.

![](public/envoy-deployment.png)

## 4. Workflow recap

With `scripts/provision.sh`, `scripts/serve.sh` you can quickly provision a server and Nginx sites. Find a detailed usage in each file.

### 4.1. Preparing a PHP application server

With one line of command you will get a fully-functioning PHP application server. PHP runtime, FPM, composer, MySql, Redis, ... You may freely customize the provision script.

```sh
# at your REMOTE computer
# AFTER DOWNLOADING THE SCRIPT, 
# READ THE FILE CAREFULLY AND CUSTOMIZE IT AS YOU WISH.

user@envoy.vm:~$ sudo -s
root@envoy.vm:~# wget https://raw.githubusercontent.com/appkr/envoy/master/scripts/provision.sh

# THE SCRIPT WILL PROVISION A PHP APPLICATION SERVER 
root@envoy.vm:~# bash provision.sh deployer
```

### 4.2. Deploy

Now you can push the code to Github, and let it be published on your remote server.

```sh
# at your LOCAL computer
 
~/project $ envoy run deploy
```

### 4.3. Make It Work

Now that the code is available on your remote server, get back to the remote to finish the whole process.

```sh
# at your REMOTE computer

user@envoy.vm:~$ sudo -s
root@envoy.vm:~# wget https://raw.githubusercontent.com/appkr/envoy/master/scripts/serve.sh

# THE SCRIPT WILL ADD AN NGINX SITE. 
root@envoy.vm:~# bash serve.sh envoy.vm /home/deployer/www/envoy.appkr.kr/public
```

## 5. Side Note

The following is the short explanation of this repository.

```bash
.
├── envoy.blade.php           # Envoy script
└── scripts
    ├── provision.sh          # Script for server provision
    ├── officer.php           # PHP script that will do the clerical job of keeping the release history ledger 
    │                         #  in the server, and will be installed on the server while running any task.
    │                         #  (if one doesn't exist)
    └── serve.sh              # Script for setting up a Nginx sites
# Not listed dir/files were laid there for the purpose of demo.
```

## 6. License

[MIT](https://github.com/appkr/envoy/blob/master/LICENSE) 




