ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_version = "20220718.0.1"
  config.vm.box_check_update = false
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  config.vm.define "worker" do |worker|
    worker.vm.hostname = "worker"
    worker.vm.network "private_network", ip: "192.168.100.10"
    worker.vm.provider "virtualbox" do |vb|
      vb.name = "worker"
      vb.memory = 6144
      vb.cpus = 2
    end
  end

  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.100.11"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "master"
      vb.cpus = 2
      vb.memory = "2048"
    end
  end

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "Kontainerd.yml"
    ansible.become   = true
  end
end
