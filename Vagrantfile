
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
 if Vagrant.has_plugin? "vagrant-vbguest"
 config.vbguest.no_install = true
 config.vbguest.auto_update = false
 config.vbguest.no_remote = true
 end
  # Configuración de la máquina balancer
  config.vm.define "balancer" do |balancer|
    balancer.vm.box =  "centos/stream8"
    balancer.vm.network "private_network", ip: "192.168.50.30"
    balancer.vm.network :forwarded_port, guest: 80, host:4577
    balancer.vm.hostname = "balancer"
  end
  # Configuración de la máquina serv1
  config.vm.define "serv1" do |serv1|
    serv1.vm.box =  "centos/stream8"
    serv1.vm.network "private_network", ip: "192.168.50.10"
    serv1.vm.network :forwarded_port, guest: 80, host:4578
    serv1.vm.hostname = "serv1"
  end
  # Configuración de la máquina serv2
  config.vm.define "serv2" do |serv2|
    serv2.vm.box =  "centos/stream8"
    serv2.vm.network "private_network", ip: "192.168.50.20"
    serv2.vm.network :forwarded_port, guest: 80, host:4579
    serv2.vm.hostname = "serv2"
  end
config.vm.define :servidor3 do |servidor3|
 servidor3.vm.box = "centos/stream8"
 servidor3.vm.network :private_network, ip: "192.168.50.40"
 servidor3.vm.hostname = "servidor3"
 end
config.vm.define :cliente do |cliente|
 cliente.vm.box = "centos/stream8"
 cliente.vm.network :private_network, ip: "192.168.50.7"
 cliente.vm.hostname = "cliente"
end
end
