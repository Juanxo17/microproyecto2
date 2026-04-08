Vagrant.configure("2") do |config|

  config.vm.define "control-node" do |ctrl|
    ctrl.vm.box = "ubuntu/jammy64"
    ctrl.vm.hostname = "control-node"
    ctrl.vm.network "private_network", ip: "192.168.56.10"
    ctrl.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
    ctrl.vm.provision "shell", inline: <<-SHELL
      apt-get update -y
      apt-get install -y software-properties-common curl unzip

      # Instalar Terraform
      curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
      echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com jammy main" > /etc/apt/sources.list.d/hashicorp.list
      apt-get update -y && apt-get install -y terraform

      # Instalar Ansible
      add-apt-repository --yes --update ppa:ansible/ansible
      apt-get install -y ansible

      echo "control-node listo"
    SHELL
  end

  config.vm.define "vm-haproxy" do |hap|
    hap.vm.box = "ubuntu/jammy64"
    hap.vm.hostname = "vm-haproxy"
    hap.vm.network "private_network", ip: "192.168.56.11"
    hap.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
  end

  config.vm.define "vm-microservices" do |ms|
    ms.vm.box = "ubuntu/jammy64"
    ms.vm.hostname = "vm-microservices"
    ms.vm.network "private_network", ip: "192.168.56.12"
    ms.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
  end

end