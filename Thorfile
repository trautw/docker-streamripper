$app_name = "streamripper"
$backup_dropoff_dir =  "#{ENV['HOME']}/4backup/"
user_name = ENV['LOGNAME']

volumesdir = '/volume'

$app_image = "#{user_name}/#{$app_name}"
$app_container_image = "#{user_name}/#{$app_name}"
$app_container_name = "#{user_name}-#{$app_name}"

$config_dir = "#{volumesdir}/config"
$config_container = "#{$app_name}config"
$config_image = "#{user_name}/#{$config_container}"
$config_container_name = "#{user_name}-#{$config_container}"

$data_dir = "#{volumesdir}/data"
$data_container = "#{$app_name}data"
$data_image = "#{user_name}/#{$data_container}"
$data_container_name = "#{user_name}-#{$data_container}"


$docker = "/home/trautw/bin/docker-1.12"

dirname = Dir.pwd
while true do
  ["docker.rc",".docker.rc"].each do |configfile|
    if File.file?("#{dirname}/#{configfile}")
      $docker_rc = "#{dirname}/#{configfile}"
      break
    end
  end
  break if $docker_rc || dirname == '/'
  dirname = File.dirname(dirname)
end

$docker = ". #{$docker_rc}; #{$docker}" if $docker_rc

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
    run "thor app_stop"
    run "thor app_start"
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
    run "mkdir -p #{$backup_dropoff_dir}", verbose: false
    # daily backup
    run "#{$docker} run --rm --volumes-from #{$config_container_name} --workdir #{$config_dir} busybox tar cf - . | gzip > #{$backup_dropoff_dir}/#{$config_container_name}_`date '+%w'`.tz", verbose: false
    # weekly backup
    run "#{$docker} run --rm --volumes-from #{$config_container_name} --workdir #{$config_dir} busybox tar cf - . | gzip > #{$backup_dropoff_dir}/#{$config_container_name}_`date '+%U'`.tz", verbose: false
  end

  desc 'config_container_restore', "Restore data in container from #{$config_container}.tz"
  def config_container_restore
    run "zcat #{$config_container}.tz | #{$docker} run -i --rm --workdir / --volumes-from #{$config_container_name} busybox tar xvf -", verbose: false
    run "#{$docker} run --rm --volumes-from #{$config_container_name} busybox chown -R default:default #{$config_volumes}", verbose: false
  end

  desc 'config_container_shell', 'Shell to config container'
  def config_container_shell
    run "#{$docker} run --interactive=true --tty=true --rm --workdir=#{$config_dir} --volumes-from #{$config_container_name} busybox /bin/sh"
  end

  # Data
  desc 'data_image_build', 'Create data image and container'
  def data_image_build
    run "#{$docker} build -t #{$data_image} - <<EOM
FROM busybox
MAINTAINER Christoph Trautwein \"<christoph.trautwein@sinnerschrader.com>\"

RUN mkdir -p /volume/data
RUN chmod 755 /volume/data

VOLUME /volume/data
EOM"
    run "#{$docker} run --name #{$data_container_name} #{$data_image} /bin/true"
  end

  desc 'data_container_destroy', 'Destructive destroy of data container'
  def data_container_destroy
    run "#{$docker} rm #{$data_container_name}"
  end

  desc 'data_container_backup', 'Create tar of data in container'
  def data_container_backup
    run "mkdir -p #{$backup_dropoff_dir}", verbose: false
    # daily backup
    run "#{$docker} run --rm --volumes-from #{$data_container_name} --workdir #{$data_dir} busybox tar cf - . | gzip > #{$backup_dropoff_dir}/#{$data_container_name}_`date '+%w'`.tz", verbose: false
    # weekly backup
    run "#{$docker} run --rm --volumes-from #{$data_container_name} --workdir #{$data_dir} busybox tar cf - . | gzip > #{$backup_dropoff_dir}/#{$data_container_name}_`date '+%U'`.tz", verbose: false
  end

  desc 'data_container_restore', "Restore data in container from #{$data_container}.tz"
  def data_container_restore
    run "zcat #{$data_container_name}.tz | #{$docker} run -i --rm --workdir / --volumes-from #{$data_container_name} busybox tar xvf -", verbose: false
    run "#{$docker} run --rm --volumes-from #{$data_container_name} busybox chown -R default:default #{$data_dir}", verbose: false
  end

  desc 'data_container_cleanup', 'Cleanup data container'
  def data_container_cleanup
    run "cd smb; #{$docker} build -t smb_image ."
    run "#{$docker} run --tty=true --rm --privileged  --volumes-from #{$config_container_name} --volumes-from #{$data_container_name} --name smb_image_conatiner smb_image"
  end

  desc 'data_container_shell', 'Shell to data container'
  def data_container_shell
    run "#{$docker} run --interactive=true --tty=true --rm --workdir=#{$data_dir} --volumes-from #{$data_container_name} busybox /bin/sh"
  end

  # Server
  desc 'app_image_build','Create the server image'
  def app_image_build
    run "cd app;#{$docker} build -t #{$app_image} ."
  end

  desc 'app_start', "run #{$app_name}"
  def app_start
    run "#{$docker} run -d --volumes-from #{$config_container_name} --volumes-from #{$data_container_name} --name #{$app_container_name} -P #{$app_image}"
  end

  desc 'app_shell', "Run a shell in a #{$app_name} container"
  def app_shell
    run "#{$docker} run --interactive=true --tty=true --rm --volumes-from #{$config_container_name} --volumes-from #{$data_container_name} --name #{$app_container_name} -P #{$app_image} /bin/bash"
  end

  desc 'app_stop', "Stop #{$app_name}"
  def app_stop
    run "#{$docker} stop #{$app_container_name}"
    sleep 10
    run "#{$docker} kill #{$app_container_name}"
    run "#{$docker} rm   #{$app_container_name}"
  end

  desc 'show_samplecall', "show how to use #{$app_name}"
  def show_samplecall
    puts "No samples yet."
  end

end

