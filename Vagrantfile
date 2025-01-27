Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  
  # Configuración de red con IP fija
  config.vm.network "private_network", ip: "192.168.57.101"
  
  # Configuración de puerto redirigido
  config.vm.network "forwarded_port", guest: 8000, host: 8000
  
  # Configuración del proveedor VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
end