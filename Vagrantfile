Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  # Servidor DHCP
  config.vm.define "srv" do |srv|
    srv.vm.network "private_network", ip: "192.168.57.10", virtualbox__intnet: "intnet"
    
    srv.vm.provision "shell", inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      apt-get update
      apt-get install -y isc-dhcp-server
      
      sysctl -w net.ipv4.ip_forward=1
      iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
      
      sed -i 's/INTERFACESv4=""/INTERFACESv4="eth1"/g' /etc/default/isc-dhcp-server
      
      cp /vagrant/dhcpd.conf /etc/dhcp/dhcpd.conf
      systemctl restart isc-dhcp-server
    SHELL
  end

  # Cliente 1
  config.vm.define "c1" do |c1|
    c1.vm.network "private_network", type: "dhcp", virtualbox__intnet: "intnet"
  end

  # Impresora con MAC fija
  config.vm.define "printer" do |printer|
    printer.vm.network "private_network", mac: "080027112233", type: "dhcp", virtualbox__intnet: "intnet"
  end
end
