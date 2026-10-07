Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64" # Puedes usar debian o ubuntu
  
  # Servidor DHCP
  config.vm.define "srv" do |srv|
    srv.vm.network "public_network"
    srv.vm.network "private_network", ip: "192.168.57.10", virtualbox__intnet: "intnet"
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