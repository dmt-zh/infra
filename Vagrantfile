Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"
  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/vagrant", disabled: true

  (1..5).each do |i|
    config.vm.define "server-#{i}" do |node|
      node.vm.hostname = "server-#{i}"
      node.vm.network "private_network", ip: "10.11.10.#{10 + i}", virtualbox__intnet: true
      node.vm.network "forwarded_port", id: "ssh", host: 2200 + i, guest: 22, host_ip: "127.0.0.1"

      node.vm.provider "virtualbox" do |vb|
        vb.name = "server-#{i}"
        vb.memory = 3072
        vb.cpus = 1
        vb.gui = false
      end

      # FIXME: при развертке в другом окружении требуется изменение пути
      wsl_pub_key_path = "//wsl.localhost/Ubuntu-22.04/home/dmt/.ssh/id_ed25519_virtualbox.pub"

      if File.exist?(wsl_pub_key_path)
        current_pub_key = File.read(wsl_pub_key_path).strip

        node.vm.provision "shell" do |s|
          s.inline = <<-SHELL
            mkdir -p /home/vagrant/.ssh
            if ! grep -q "#{current_pub_key}" /home/vagrant/.ssh/authorized_keys; then
              echo "#{current_pub_key}" >> /home/vagrant/.ssh/authorized_keys
            fi
            chmod 600 /home/vagrant/.ssh/authorized_keys
            chown -R vagrant:vagrant /home/vagrant/.ssh
          SHELL
        end
      end

    end
  end
end
