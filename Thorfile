user_name = ENV['LOGNAME']
$app_name = "streamripper"

$app_image = "#{user_name}/#{$app_name}"
$app_container_image = "#{user_name}/#{$app_name}"

$config_container = "#{$app_name}config"
$config_image = "#{user_name}/#{$config_container}"
$config_container_name = "#{user_name}-#{$config_container}"

$data_container = "#{$app_name}data"
$data_image = "#{user_name}/#{$data_container}"
$data_container_name = "#{user_name}-#{$data_container}"

$docker = "docker"
$docker = ". $HOME/.docker.rc; docker" if File.file?("$HOME/.docker.rc")
$docker = ". #{Dir.pwd}/../docker.rc; docker" if File.file?("#{Dir.pwd}/../docker.rc")
$docker = ". #{Dir.pwd}/docker.rc; docker" if File.file?("#{Dir.pwd}/docker.rc")

$domain = "docker.szz.chtw.de"
$env = "dev"

# Name:	dhcp-box.sinner-schrader.de
# $dns = "192.168.6.145"
$dns = "172.16.248.67"

class Default < Thor
  include Thor::Actions

  desc 'restart','All in one'
  def restart
    run "thor app_image_build"
    run "thor app_kill"
    run "thor app_start"
    run "thor show_samplecall"
  end

  # Config
  desc 'config_volume_build', 'Create config volume container'
  def config_volume_build
    run "#{$docker} build -t #{$config_image} - <<EOM
FROM busybox
MAINTAINER Christoph Trautwein \"<christoph.trautwein@sinnerschrader.com>\"

RUN mkdir -p /volume/config
RUN chmod 755 /volume/config

VOLUME /volume/config
EOM"
    run "#{$docker} run --name #{$config_container_name} #{$config_image} /bin/true"
  end

  desc 'config_container_destroy', 'Destructive destroy of config container'
  def config_container_destroy
    run "#{$docker} rm #{$config_container_name}"
  end

  desc 'config_container_backup', 'Create tar of config'
  def config_container_backup
    # daily backup
    run "#{$docker} run --rm --volumes-from #{$config_container_name} busybox tar cf - #{config_volumes} | gzip > $HOME/4backup/#{$config_container}_`date '+%w'`.tz", verbose: false
    # weekly backup
    run "#{$docker} run --rm --volumes-from #{$config_container_name} busybox tar cf - #{$config_volumes} | gzip > $HOME/4backup/#{$config_container}_`date '+%U'`.tz", verbose: false
  end

  desc 'config_container_restore', "Restore data in container from #{$config_container}.tz"
  def config_container_restore
    run "zcat #{$config_container}.tz | #{$docker} run -i --rm --workdir / --volumes-from #{$config_container_name} busybox tar xvf -", verbose: false
    run "#{$docker} run --rm --volumes-from #{$config_container_name} busybox chown -R default:default #{$config_volumes}", verbose: false
  end

  desc 'config_image_shell', 'Shell to config container'
  def config_image_shell
    run "#{$docker} run --interactive=true --tty=true --rm --volumes-from #{$config_container_name} busybox /bin/sh"
  end

  # Data
  desc 'data_image_build', 'Create data image and container'
  def data_image_build
    run "#{$docker} build -t #{$data_image} - <<EOM
FROM busybox
MAINTAINER Christoph Trautwein \"<christoph.trautwein@sinnerschrader.com>\"

RUN mkdir -p /volume/config
RUN chmod 755 /volume/config

VOLUME /volume/config
EOM"
    run "#{$docker} run --name #{$data_container_name} #{$data_image} /bin/true"
  end

  desc 'data_container_destroy', 'Destructive destroy of data container'
  def data_container_destroy
    run "#{$docker} rm #{$data_container_name}"
  end

  desc 'data_container_backup', 'Create tar of data in container'
  def data_container_backup
    run "#{$docker} run --rm --volumes-from #{$data_container_name} busybox tar cf - #{data_volumes} | gzip > $HOME/4backup/#{$data_container}_`date '+%w'`.tz", verbose: false
    run "#{$docker} run --rm --volumes-from #{$data_container_name} busybox tar cf - #{$data_volumes} | gzip > $HOME/4backup/#{$data_container}_`date '+%U'`.tz", verbose: false
  end

  desc 'data_container_restore', "Restore data in container from #{$data_container}.tz"
  def data_container_restore
    run "zcat #{$data_container}.tz | #{$docker} run -i --rm --workdir / --volumes-from #{$data_container_name} busybox tar xvf -", verbose: false
    run "#{$docker} run --rm --volumes-from #{$data_container_name} busybox chown -R default:default #{$data_volumes}", verbose: false
  end

  desc 'data_container_shell', 'Shell to data container'
  def data_container_shell
    run "#{$docker} run --interactive=true --tty=true --rm --volumes-from #{$data_container_name} busybox /bin/sh"
  end

  # Server
  desc 'app_image_build','Create the server image'
  def app_image_build
    run "cd app;#{$docker} build -t #{$app_image} ."
  end

  desc 'app_start', "run #{$app_name}"
  def container_start
    run "#{$docker} run -d --net=host --volumes-from #{$config_container_name} --volumes-from #{$data_container_name} --name #{$app_container_name} -P #{$app_image}"
  end

  desc 'app_kill', "Kill #{$app_name}"
  def app_kill
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

