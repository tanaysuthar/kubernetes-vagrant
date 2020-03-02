Vagrant.configure("2") do |config|
  config.vm.define "k8s-master" do |controller|
    controller.vm.box = "bento/ubuntu-16.04"
    controller.vm.hostname = "controller"
    controller.vm.network :private_network, ip: "192.168.50.10"
    controller.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
    end
  end

  (1..2).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "bento/ubuntu-16.04"
      node.vm.hostname = "node#{i}"
      node.vm.network :private_network, ip: "192.168.50.1#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end
    end
  end
end
