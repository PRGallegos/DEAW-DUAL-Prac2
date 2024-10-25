# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Configuración de Vagrant para los servidores DNS tierra y venus.
Vagrant.configure("2") do |config|

    # Configuración común para instalar Apache y Bind9 en ambos servidores si es necesario
    config.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y bind9 bind9utils bind9-doc
        systemctl restart bind9
        SHELL

    # Configuración del servidor tierra (tierra.sistema.test)
    config.vm.define "tierra" do |tierra|
      tierra.vm.box = "debian/bookworm64"
      tierra.vm.network "private_network", ip: "192.168.57.103"
      tierra.vm.hostname = "tierra.sistema.test"

      tierra.vm.provision "shell", inline: <<-SHELL
        cp -v  /vagrant/named.conf.options /etc/bind/named.conf.options
        cp -v /vagrant/named.conf /etc/default/named
        cp -v /vagrant/named.conf.local /etc/bind/named.conf.local
        cp -v /vagrant/tierra.sistema.test.zone /etc/bind/tierra.sistema.test.zone
        cp -v /vagrant/tierra.sistema.test.rev /etc/bind/tierra.sistema.test.rev
      SHELL
    end

    # Configuración del servidor venus (venus.sistema.test).
    config.vm.define "venus" do |venus|
        venus.vm.box = "debian/bookworm64"
        venus.vm.network "private_network", ip: "192.168.57.102"
        venus.vm.hostname = "venus.sistema.test"
        venus.vm.provision "shell", inline: <<-SHELL
          cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
          cp -v /vagrant/named.conf /etc/default/named
          cp -v /vagrant/venus.named.conf.local /etc/bind/named.conf.local 
        SHELL
    end
end


