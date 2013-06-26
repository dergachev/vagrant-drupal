# Vagrant environment for Deploying Drupal

These are the steps to follow to to get Drupal up and running.

First, install berkshelf:

```bash
gem install berkshelf --no-rdoc --no-ri
vagrant plugin install vagrant-berkshelf
```

If you have an existing Drupal codebase, ensure that it's located in
`PROJECTROOT/site/`.  If it's present, that folder will be mounted inside the
vm as `/vagrant/site`, and will be copied over to
`/var/shared/sites/cooked.drupal/site` where it will be served by Apache.
Otherwise, the `deploy-drupal` cookbook will simply run `drush dl
drupal` to get you a clean D7 site.

If you have an existing SQL dump you'd like to pull in, ensure that it's
located in `PROJECTROOT/db/dump.sql.gz`, and it will be used to populate the
empty database. Otherwise `drush site-install` will be run to initialize a
the DB with a clean Drupal site.

To spin up a VM with a Drupal 7 site available at http://localhost:8080, simply
start the VM:

```
vagrant up
```

To back up the database, do the following:

```
vagrant ssh           # ssh in to the virtual machine
cd /var/shared/sites/cooked.drupal/site # this is your drupal site, inside the VM
drush sql-dump --gzip --result-file     # use this to run a backup; saved in under /home/vagrant/drush-backups/"
```

The newly created backup file will be automatically synced to the
`PROJECTROOT/db/` folder on your host machine.

## Git workflow

At this point you're expected to manually manage the git status of your
codebase. (We don't know how to do this yet).  If your source Drupal codebase
was already a git repo, just commit all the changes as usual. To import a newly
downloaded Drupal site into git, do the following:

```bash
echo -e "settings-priv.php sites/default/files" >> .gitignore
git init
git add .
git commit -m "Initial commit."
git remote add origin REPO_URL
git push -u origin master
```

### Warning

There are currently serious problems with `deploy-drupal` and git:

* settings-priv.php creation isn't implemented yet. 
  - Until this happens, committing settings.php into git will commit your
    (development) DB credentials too.
* currently the chef resource 'bash[copy-drupal-site]' in `drupal\_deploy`
  makes it impossible to keep the Drupal site as a subfolder of the git repo
  (eg PROJECTROOT/site).

## Hacking on the deploy\_drupal cookbook

To hack on the deploy\_drupal cookbook, do the following:

```
git clone https://github.com/amirkdv/vagrant_drupal.git # REPLACE WITH YOUR FORK
mv vagrant_drupal/site-cookbooks .
vim Berksfile # cookbook 'deploy_drupal', :path => 'site-cookbooks/deploy_drupal'
```
