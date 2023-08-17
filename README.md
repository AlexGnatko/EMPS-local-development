# Docker environment for local EMPS development

## Prerequisites
1. Docker
2. git
3. nodejs (npm)
4. bower (global)
5. git-cloned EMPS6
6. app (website) folders with your projects

This is not a completely stand-alone container to run the EMPS development environment. 
The docker container itself will only run nginx, PHP and MySQL. The code to execute will be
mapped from your local folder. The EMPS6 framework with all its dependencies
will also be mapped (from another folder).

Your local system (outside of docker) should be able to run PHP CLI (preferably PHP 8.1
or higher), nodejs (npm), composer, and bower (`npm install -g bower`).

## Installing EMPS6
Find a folder on your PC that used to be your PHP include_path in the old days before
composer. On my PC, it is `D:\W\_includes`, on my servers it's usually `/srv/www/lib`.
If you are not planning to edit several different EMPS6 websites, you might find it
easy to actually create a `lib` folder right here in this folder (it's already added
to `.gitignore`).

So, go to `lib` folder and execute:

```
git clone https://github.com/AlexGnatko/EMPS6.git
```

After that, go to the `lib/EMPS6` folder and run `composer update`.
Then go to `lib/EMPS6/6.X` and run `npm install`, `bower install`. This
will install all the JS and PHP dependencies. This is probably not a great way of
installing dependecies, as they will be targeted for your real system, but will
eventually run in a docker container - a virtual system. But in practice this approach
does seem to be working fine.

## Revise the docker-compose.yaml file

The `docker-compose-sample.yaml` file contains some paths that you might want to revise.
First, copy this file to `docker-compose.yaml` (this is file is excluded from the git repo).
Find all `volumes:` settings and check them.
Some of the paths are relative to this folder, but some point to a full path on my PC, `//d/w`. 
On your PC it could be a different folder,
or it could be the relative `./lib` folder. Set this value correctly. Also,
you might want to revise the port numbers. I've used 84 for the main app and
5001 for phpMyAdmin (to create the database and later view the data).

## Include path or where EMPS6 is installed

Pay attention to the `local.ini` file, this file is the local PHP configuration file.
It should contain at least the `include_path` setting. As you can see, in my setup,
the `include_path` points to `/srv/www/_includes`, which is actually `d:/w/_includes`,
as the `/srv/www` folder in the container is mapped to the real `//d/w` folder.

## Revise the Dockerfile

The `Dockerfile-sample` should be copied to `Dockerfile` in order to be able to install
`php` with all the necessary dependencies. You can also edit your local version of `Dockerfile`
to add other modules and apps you need for your projects. This file is excluded from the repo
so that you don't accidentaly rewrite your local version with updates.

## Configure your local development websites

In the `./nginx/conf.d` you will find a folder which is mapped to the nginx config folder.
`oprosnik.conf` is a sample file that you can edit to set it up for another local website.
Create another `.conf` file and edit the root folder, host names and any other settings
you might need to edit. Keep in mind that the port nubmers are internal, the external port
84 is mapped to internal port 80.

## Run docker-compose
Now, all you need to bring up the container is to run this command in this folder:
```
docker-compose up -d
```
After this, the main app will be available at `http://localhost:84/` (choose a
different host name to access a different website), and
phpMyAdmin will be at `http://localhost:5001/`.

* `docker-compose ps` - list the running containers
* `docker-compose down` - bring down all running containers
* `docker-compose build` - rebuild the containers

## Create the database
Please note that the YAML file created a persistent folder for MySQL data, which is
smart because when you stop/restart/rebuild the docker image the data will not be
lost.

Go to `http://localhost:5001/` and create new databases for your projects.

After this, go to:

* `http://{project_host}:84/sqlsync/` - to create the DB tables.
* `http://{project_host}:84/ensure_root/qwerty/` - to create the root user with password
  `qwerty`.
* `http://{project_host}:84/init_settings/` - to create the default settings.

Also, you can go to `http://localhost:5001/` and import an existing database dump into
a local database.

