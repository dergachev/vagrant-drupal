site :opscode

group :deploy do
  cookbook 'drush', :git => 'git://github.com/homemade/chef-drush.git'
  cookbook 'xhprof', :git => 'git://github.com/msonnabaum/chef-xhprof.git'
  cookbook 'deploy-drupal', :git => 'git://github.com/amirkdv/chef-deploy-drupal.git'
  #cookbook 'deploy-drupal', :path=> 'site-cookbooks/deploy-drupal' 
end

group :test do
  cookbook 'minitest-handler'
end
