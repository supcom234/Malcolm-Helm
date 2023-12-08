# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.require_version ">= 2.3.7"
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/jammy64"
  config.disksize.size = '500GB'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

  # Extra NIC for listening off of
  # config.vm.network "private_network", type: "dhcp", auto_config: false, virtualbox__intnet: "promiscuous"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessable to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    # Customize the amount of memory on the VM:
    vb.name = "Malcolm-Helm"
    vb.memory = "16192"
    vb.cpus = 8
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y

    # Turn off password authentication to make it easier to login
    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    sudo systemctl restart sshd

    # Setup RKE2
    curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_VERSION=v1.26.6+rke2r1 sh -
    mkdir -p /etc/rancher/rke2
    echo "cni: calico" > /etc/rancher/rke2/config.yaml
    
    systemctl start rke2-server.service
    systemctl enable rke2-server.service

    mkdir /root/.kube
    mkdir /home/vagrant/.kube    

    cp /etc/rancher/rke2/rke2.yaml /home/vagrant/.kube/config
    cp /etc/rancher/rke2/rke2.yaml /root/.kube/config
    chmod 0600 /home/vagrant/.kube/config
    chmod 0600 /root/.kube/config
    chown -R vagrant:vagrant /home/vagrant/.kube

    ln -s /var/lib/rancher/rke2/bin/kubectl /usr/local/bin/kubectl
    snap install helm --classic
    node_name=$(kubectl get nodes -o jsonpath="{.items[0].metadata.name}")
    kubectl label nodes $node_name cnaps.io/node-type=Tier-1

    kubectl apply -f /vagrant/vagrant_dependencies/sc.yaml

    grep -qxF 'alias k="kubectl"' /home/vagrant/.bashrc || echo 'alias k="kubectl"' >> /home/vagrant/.bashrc

    # Load specific settings 
    grep -qxF 'fs.inotify.max_user_instances=8192' /etc/sysctl.conf || echo 'fs.inotify.max_user_instances=8192' >> /etc/sysctl.conf
    grep -qxF 'fs.file-max=1000000' /etc/sysctl.conf || echo 'fs.file-max=1000000' >> /etc/sysctl.conf
    grep -qxF 'vm.max_map_count=1524288' /etc/sysctl.conf || echo 'vm.max_map_count=1524288' >> /etc/sysctl.conf
    sysctl -p
    swapoff -a    
    reboot
  SHELL

  config.vm.provision "reload"

  config.vm.provision "shell", inline: <<-SHELL
    touch /home/ubuntu/test123.txt
    helm install malcolm /vagrant/chart -n malcolm --create-namespace
    echo "You may now ssh using ssh vagrant@<hostIp>"
    hostname -I
  SHELL

end
