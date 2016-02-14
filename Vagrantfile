
Vagrant.require_version ">= 1.7.0"

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network "forwarded_port", guest: 50042, host: 50042, protocol: "udp"

  config.vm.provision "ansible" do |ansible|
    ansible.groups = {
        "vagrant_openvpn" => ["default"]
    }
    ansible.verbose = "v"
    ansible.playbook = "playbook.yml"
  end

end
