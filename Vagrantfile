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

  # Installs the previously exported site code and SQL dump via deploy-drupal
  config.vm.provision :chef_solo do |chef|
    chef.json.merge!({
      "deploy-drupal" => {
        "get_project_from" => { "path" => "/vagrant" }, # path to copy existing project from
      # "get_project_from" => { "git"  => "[url-to-repo]"}, # url to clone existing repo from
        
        "sql_load_file" => "/vagrant/db/dump.sql.gz", # path to database dump file, default is ""
        "post_install_script" => "scripts/post.sh", # bash script to run after loading database dump, default is ""
        
        "deploy_dir" => "/var/shared/sites", # projects are created in this directory, "var/shared/sites" is default
        "project_name" => "cooked.drupal", # vhost name and project root directory name, "cooked.drupal" is default
        "drupal_root_dir" => "site", # name of Drupal site directory (relative to project directory), "site" is default
        "drupal_files_dir" => "site/default/files", # "site/default/files" is default
        
        "drupal_dl_version" => "drupal-7", # Drupal version to download if no existing project found, "drupal-7" is default
        "dev_group_name" => "vagrant", # use Vagrant default user group as dev group, default is "root"
      },  
      "mysql" => {
        "server_root_password" => "root",
        "server_debian_password" => "root",
        "server_repl_password" => "root"
      },
      "memcached" => {
        "listen" => "127.0.0.1", # by default world accessible
        "memory" => "256", # by default 64M
      }, 
      "run_list" => [ "deploy-drupal" ]
    })
  end
  
  # Test that site and SQL dump were installed correctly and are being served by Apache
  config.vm.provision :shell, :inline => "curl --silent localhost:80 | grep '<title>' | grep 'cooked.drupal'"
  
end
