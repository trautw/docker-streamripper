user_name = ENV['LOGNAME']
$app_name = "streamripper"
$server_image = "#{user_name}/#{$app_name}"
$docker = "docker"
$docker = ". #{Dir.pwd}/../env.source; docker" if File.file?("#{Dir.pwd}/../env.source") 
$docker = ". #{Dir.pwd}/env.source; docker" if File.file?("#{Dir.pwd}/env.source") 

$domain = "docker.szz.chtw.de"
$env = "dev"

# Name:	dhcp-box.sinner-schrader.de
# $dns = "192.168.6.145"
$dns = "172.16.248.67"

class Default < Thor
  include Thor::Actions

  desc 'restart','All in one'
  def restart
    run "thor image_build"
    run "thor container_kill"
    run "thor container_start"
    run "thor show_samplecall"
  end

  # Server
  desc 'image_build','Create the server image'
  def image_build
    run "cd app;#{$docker} build -t #{$server_image} ."
  end

  desc 'container_start', "run #{$app_name}"
  def container_start
    run "#{$docker} run -d --net=host --name #{$app_name} -P #{$server_image}"
  end

  desc 'container_kill', 'Kill server'
  def container_kill
    run "#{$docker} stop #{$app_name}"
    sleep 10
    run "#{$docker} kill #{$app_name}"
    run "#{$docker} rm   #{$app_name}"
  end

  desc 'show_samplecall', "show how to use #{$app_name}"
  def show_samplecall
    ssh_host = "#{$app_name}.#{$env}.#{$domain}"
    ssh_hostip = `dig +short @#{$dns} #{ssh_host}`.chomp
    ssh_port = `#{$docker} port sshd 22`.split(':')[1].chomp
    docker_host = `. ../env.source;echo $DOCKER_HOST`.split('/')[2].split(':')[0]
    puts "ssh -p #{ssh_port} root@#{ssh_host}"
    puts "ssh -p #{ssh_port} root@#{ssh_hostip}"
    puts "ssh -p #{ssh_port} root@#{docker_host}"
  end

end

