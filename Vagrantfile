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

      sed -i 's/INTERFACESv4=""/INTERFACESv4="eth1"/g' /etc/default/isc-dhcp-server

      cat << 'EOF' > /etc/dhcp/dhcpd.conf
default-lease-time 86400;
max-lease-time 691200;
option domain-name "rbc.test";
option domain-name-servers 10.0.0.2, 4.4.4.4;

subnet 192.168.57.0 netmask 255.255.255.0 {
    range 192.168.57.20 192.168.57.50;
    option routers 192.168.57.10;
}

host printer {
    hardware ethernet 08:00:27:11:22:33;
    fixed-address 192.168.57.111;
    default-lease-time 7200;
    max-lease-time 7200;
}
EOF


      sysctl -w net.ipv4.ip_forward=1
      iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

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
