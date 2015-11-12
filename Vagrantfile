# -*- mode: ruby -*-
# # vi: set ft=ruby :

$num_instances = 3
$instance_name_prefix = "core"
$forwarded_ports = {}

Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end
  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.box = "coreos-beta"
  config.vm.box_url = "http://beta.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json"

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%02d" % [$instance_name_prefix, i] do |c|
      c.vm.hostname = vm_name

      if $expose_docker_tcp
        c.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i), auto_correct: true
      end

      $forwarded_ports.each do |guest, host|
        c.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
      end

      ip = "172.17.8.#{i+100}"
      c.vm.network :private_network, ip: ip

      c.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']
      c.vm.provision :file, :source => ".provision/cloud-config", :destination => "/tmp/vagrantfile-user-data"
      c.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

    end
  end

end
