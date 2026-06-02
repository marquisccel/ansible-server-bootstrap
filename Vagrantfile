Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "bootstrap-dev"

  # Private network — accessible from host machine
  config.vm.network "private_network", ip: "192.168.56.10"

  # Forward app port
  config.vm.network "forwarded_port", guest: 80,   host: 8080
  config.vm.network "forwarded_port", guest: 9100, host: 9100

  config.vm.provider "virtualbox" do |vb|
    vb.name   = "ansible-bootstrap-dev"
    vb.memory = "1024"
    vb.cpus   = 1
  end

  # Disable default /vagrant sync (not needed)
  config.vm.synced_folder ".", "/vagrant", disabled: true
end
