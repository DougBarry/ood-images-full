# -*- mode: ruby -*-
# vi: set ft=ruby :

URL_ONDEMANDRPM = "https://yum.osc.edu/ondemand/1.8/ondemand-release-web-1.8-1.noarch.rpm"

unless Vagrant.has_plugin?("vagrant-vbguest")
  puts 'Installing vagrant-vbguest Plugin...'
  system('vagrant plugin install vagrant-vbguest')
end

unless Vagrant.has_plugin?("vagrant-hosts")
  puts 'Installing vagrant-hosts Plugin...'
  system('vagrant plugin install vagrant-hosts')
end

unless Vagrant.has_plugin?("vagrant-cachier")
  puts 'Installing vagrant-cachier Plugin...'
  system('vagrant plugin install vagrant-cachier')
end

# This workaround is needed during the brief period when a new CentOS point
# release becomes available but the CentOS base box hasn't been updated yet.
# It's also required if we choose to stick with some specific point release.
begin
  class FixGuestAdditions < VagrantVbguest::Installers::CentOS
    def dependencies
    packages = super

    # If there's no "kernel-devel" package matching the running kernel in the
    # default repositories, then the base box we're using doesn't match the
    # latest CentOS release anymore and we have to look for it in the archives...
    if communicate.test('test -f /etc/centos-release && ! yum -q info kernel-devel-`uname -r` &>/dev/null')
      env.ui.warn("[#{vm.name}] Looking for the CentOS 'kernel-devel' package in the release archives...")
      packages.sub!('kernel-devel-`uname -r`', 'http://mirror.centos.org/centos' \
                                                '/`grep -Po \'\b\d+\.[\d.]+\b\' /etc/centos-release`' \
                                                '/{os,updates}/`arch`/Packages/kernel-devel-`uname -r`.rpm')
    end

    packages
  end
end

  # Anything with priority over 5 overrides the default installer...
  VagrantVbguest::Installer.register(FixGuestAdditions, 100)
rescue NameError
  # The "VagrantVbguest" class won't be available during the first pass,
  # when vagrant is checking for required plugins and stuff like that...
end

Vagrant.configure(2) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.auto_detect = true
  end
  
  config.vm.box = "bento/centos-7"

  config.vm.define "ood", primary: true, autostart: true do |ood|
    ood.vm.network "forwarded_port", guest: 80, host: 8080
    ood.vm.network "private_network", ip: "10.0.0.100"
    ood.vm.provision "shell", inline: <<-SHELL
      yum update -y -q yum
      yum update -y
      yum upgrade -y
      yum install -y kernel-devel
      /sbin/rcvboxadd setup
      yum install -y epel-release centos-release-scl lsof sudo
      yum group install -y "Development Tools"
      yum install -y #{URL_ONDEMANDRPM}
      yum install -y ondemand
    SHELL
    ood.vm.provision "shell", path: "ood-setup.sh"
    ood.vm.provision "shell", inline: "systemctl enable httpd24-httpd"
    ood.vm.provision "shell", inline: "systemctl start httpd24-httpd"
    ood.vm.provision "shell", inline: "hostnamectl set-hostname ood"
    ood.vm.provision "shell", inline: "cp -f /vagrant/hosts /etc/hosts"
    ood.vm.provision "shell", inline: "cp -f /vagrant/example.yml /etc/ood/config/clusters.d/example.yml"
    ood.vm.provision "shell", path: "slurm-setup.sh"
    # Install guest additions automatically (if not already builtin)...
    ood.vbguest.auto_update = !config.vm.box.match(/^(?:carlosefr|bento)/)
    ood.vbguest.allow_downgrade = false
  end

  config.vm.define "head", primary: false, autostart: true do |head|
    head.vm.network "private_network", ip: "10.0.0.101"
    head.vm.provision "shell", inline: <<-SHELL
      yum update -y -q yum
      yum update -y
      yum upgrade -y
      yum install -y kernel-devel
      /sbin/rcvboxadd setup
      yum install -y epel-release centos-release-scl lsof sudo
      yum group install -y "Development Tools"
    SHELL
    head.vm.provision "shell", path: "head-setup.sh"
    head.vm.provision "shell", inline: "hostnamectl set-hostname head"
    head.vm.provision "shell", inline: "cp -f /vagrant/hosts /etc/hosts"
    head.vm.provision "shell", path: "slurm-setup.sh"
    head.vm.provision "shell", inline: "systemctl enable slurmd"
    head.vm.provision "shell", inline: "systemctl start slurmd"
    head.vm.provision "shell", inline: "systemctl enable slurmctld"
    head.vm.provision "shell", inline: "systemctl start slurmctld"
    # Install guest additions automatically (if not already builtin)...
    head.vbguest.auto_update = !config.vm.box.match(/^(?:carlosefr|bento)/)
    head.vbguest.allow_downgrade = false
  end

#  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", disable: true
  config.vm.synced_folder "./ood-home", "/home/ood", type: "virtualbox", mount_options: ["uid=1001","gid=1001"]

end

