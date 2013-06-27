# Vagrant Drupal Template

This repo exists to suggest a template for organizing your new or existing
Drupal project repo, to be compatible with the
[deploy-drupal](https://github.com/amirkdv/chef-deploy-drupal) cookbook. By
following our suggestions, you will have a complete workflow for quickly getting 
a Vagrant VM that's ready to serve your as your complete dev environment, including:

* deploying a Drupal development site, either from scratch, or from an existing codebase and DB dump,
* ability to edit the deployed codebase, and commit changes to upstream git repo
* ability to backup database and uploaded files to host machine, in case VM gets destroyed

## Setup

This guide will cover the workflow around 3 main cases:

* deploying a fresh install of Drupal
* comitting the freshly-installed Drupal site into a git repo
* deploying existing Drupal codebase and SQL dump 

## Deploying a fresh Drupal install

For this case, there is nothing to do. Simply run `vagrant up` and the
`deploy-drupal` cookbook will download the latest stable version of D7,
and run `drush site-install` for to populate a clean Drupal database.

If you don't customize the settings in `Vagrantfile`, then:

* In the host, the fresh Drupal install will be accessible http://localhost:8080
* In the VM, the fresh Drupal install will be accessible at http://localhost:80
* In the host, 'PROJECTROOT' will contain only the initial checkout of the project repo, cotaining this README.md.
* In the VM, `/vagrant` will be a synced mount of PROJECTROOT, containing this README.md.
* In the VM, `/vagrant` will be copied to `/var/shared/sites/cooked.drupal/`
* In the VM, apache will serve Drupal from `/var/shared/sites/cooked.drupal/site/`
* In the VM, calls to `drush sql-dump --result-file` will be stored in `/home/vagrant/drush-backups`
* In the host, `PROJECTROOT/db` is a synced folder linked to `/home/varant/drush-backups`
* The VM will mount your PROJECTROOT (this folder) in /vagrant

## Creating project repo

After the initial deployment, it's recommended to commit the newly created site
to a newly-created git repo, as follows:

```bash
vagrant ssh
cd /var/shared/sites/cooked.drupal

# make a backup of the site
drush --root='./site' sql-dump --gzip --result-file='./db/dump.sql' # NB: this creates dump.sql.gz

# clean up: remove README.md, etc.

echo "settings-priv.php sites/default/files" | tr ' ' '\n' >> .gitignore
git init
git add .
git commit -m "Fresh Drupal site created by deploy-drupal cookbook"

# create a new repo, say at github

git remote add origin https://github.com/USERNAME/NEWREPO.git
git push -u origin master
```

At this point, we recommend to blow away PROJECTROOT on the host, and recreate
it from the NEWREPO.git:

```bash
# exit vagrant ssh
vagrant destroy # be sure you committed and pushed all your code & db!
cd $PROJECTROOT/..
mv $PROJECTROOT $PROJECTROOT-backup
git clone https://github.com/USERNAME/NEWREPO.git $PROJECTROOT
cd $PROJECTROOT
```

Now simply redeploy the Drupal dev environment from the new Drupal repository.

## Deploying from existing project repo

At this point we assume you have cloned a Drupal project repository, contianing a
Drupal codebase, SQL dump, and suitable `Vagrantfile` and `Berksfile` configurations.
Note that the default `Vagrantfile` contains the following configuration for the
`deploy-drupal` cookbook: 

```ruby
'drupal-deploy' => { 
  'sql_load_file' => '/vagrant/db/dump.sql.gz', # if non-existant, DB will be initialized via 'drush si'
  'codebase_source_path' =>  '/vagrant/site', # if folder is empty, will download D7 instead
},
```

Adjust these as needed, then simply run `vagrant up` to do the following:

* Mount $PROJECTROOT as a synced folder, available on the VM at `/vagrant`
* Copy `/vagrant` to `/var/shared/sites/cooked.drupal`, but only once.
* Set up Apache to serve `/var/shared/sites/cooked.drupal/site`
* Load the specified SQL dump into the Drupal database, but only once.

At this point,

* In the VM, your drupal site is accessible at http://localhost:80
* In the host, your drupal site is accessible at http://localhost:8080
* In the VM, make modifications, commits and pushes from the git repo at `/var/shared/sites/cooked.drupal/site`
* In the host, ocassionally run `git pull` to update the codebase in $PROJECTROOT.
* In the host, `$PROJECTROOT/db` will contain all the database dumps produced via `drush sql-dump --gzip --result-file`. Commit these as appropriate.
* In the host, make any desired changes to `Vagrantfile` configuration by editing `$PROJECTROOT/Vagrantfile`. Commit these as appropriate.

## Installation

These are a few pre-requisites to install:

* install Virtualbox 4.2+ from https://www.virtualbox.org/wiki/Downloads
* install Vagrant 1.2+ from http://downloads.vagrantup.com/
* install the berkshelf plugin, via `vagrant plugin install vagrant-berkshelf`

### Warning

There are currently serious problems with `deploy-drupal` and git:

* settings-priv.php creation isn't implemented yet. 
  - Until this happens, committing settings.php into git will commit your
    (development) DB credentials too.
* currently the chef resource 'bash[copy-drupal-site]' in `deploy-drupal`
  makes it impossible to keep the Drupal site as a subfolder of the git repo
  (eg PROJECTROOT/site).

## Hacking on the deploy-drupal cookbook

To hack on the `deploy-drupal` cookbook, fork https://github.com/amirkdv/chef-deploy-drupal, then do the following:

```
mkdir site-cookbooks
git clone https://github.com/USERNAME/chef-deploy-drupal.git site-cookbooks/deploy-drupal
vim Berksfile # uncomment the following line:   cookbook 'deploy-drupal', :path => 'site-cookbooks/deploy-drupal'
```
