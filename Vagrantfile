# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
  config.vm.network :forwarded_port, guest: 80, host: 8080

  # ensures that 'drush sql-dump --gzip --result-file' will place backup in a persistent folder
  config.vm.synced_folder "./db", "/home/vagrant/drush-backups/"

  # precise64.box doesn't have chef 11, which we require
  config.vm.provision :shell, :inline => <<-HEREDOC
    apt-get install -y build-essential
    gem install chef --version 11.0.0 --no-rdoc --no-ri --conservative
  HEREDOC

  # install some convenience tools
  config.vm.provision :shell, :inline => <<-HEREDOC
    apt-get update
    apt-get install -y curl vim git
    # gem install sass compass
  HEREDOC

  # Installs the previously exported site code and SQL dump via deploy-drupal
  config.vm.provision :chef_solo do |chef|
    reset = ENV["reset"].nil? ? "" : ENV["reset"]
    chef.json.merge!({
      "deploy-drupal" => { 
        "dev_group_name" => "vagrant", # use Vagrant default user group as dev group
        "reset" => reset, # if set to "true", provisioning starts with obliterating existing project
        "sql_load_file" => "/vagrant/db/dump.sql.gz", # if non-existant, DB will be initialized via 'drush si', can also be relative (db/dump.sql.gz)
        "copy_project_from" =>  "/vagrant", # if folder is empty, will download D7 instead
        "site_path" => "site" # site is default, this can be removed
      },  
      "mysql" => {
        "server_root_password" => "root",
        "server_debian_password" => "root",
        "server_repl_password" => "root"
      },
      "run_list" => [ "deploy-drupal" ]
    })
  end
  
  # Test that site and SQL dump were installed correctly and are being served by Apache
  config.vm.provision :shell, :inline => "curl --silent localhost:80 | grep '<title>' | grep 'cooked.drupal'"
  
end
