# Práctica Servidor DHCP con Vagrant - RBC

Este repositorio contiene la arquitectura de red y la configuración del servicio DHCP (`isc-dhcp-server`) desplegado sobre máquinas virtuales utilizando **Vagrant** y **VirtualBox**.

---

## 📐 Arquitectura de Red

La topología está compuesta por tres máquinas virtuales sobre la red privada/interna `intnet` (`192.168.57.0/24`):

1. **`srv` (Servidor DHCP):** 
   - Conectado a la red pública y a la red privada `intnet`.
   - **IP Estática:** `192.168.57.10`.
2. **`c1` (Cliente Dinámico):** 
   - Cliente configurado para obtener dirección IP de forma dinámica en la red `intnet`.
3. **`printer` (Cliente con IP Reservada):** 
   - Dispositivo con dirección MAC fija (`08:00:27:11:22:33`) que recibe siempre la IP fija `192.168.57.111`.

---

## 🛠️ Explicación de los Archivos de Configuración

### 1. `Vagrantfile`
Define y despliega automáticamente el entorno virtualizado:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  # Servidor DHCP
  config.vm.define "srv" do |srv|
    srv.vm.network "public_network"
    srv.vm.network "private_network", ip: "192.168.57.10", virtualbox__intnet: "intnet"
  end

  # Cliente Dinámico (c1)
  config.vm.define "c1" do |c1|
    c1.vm.network "private_network", type: "dhcp", virtualbox__intnet: "intnet"
  end

  # Cliente con IP Fija por MAC (printer)
  config.vm.define "printer" do |printer|
    printer.vm.network "private_network", mac: "080027112233", type: "dhcp", virtualbox__intnet: "intnet"
  end
end

# --- Parámetros Globales ---
default-lease-time 86400;             # Concesión por defecto: 1 día (86400 seg)
max-lease-time 691200;                # Concesión máxima: 8 días (691200 seg)
option domain-name "rbc.test";        # Dominio personalizado
option domain-name-servers 10.0.0.2, 4.4.4.4; # Servidores DNS globales

# --- Declaración de la Subred (Rango Dinámico) ---
subnet 192.168.57.0 netmask 255.255.255.0 {
  range 192.168.57.20 192.168.57.50;  # Rango de asignación dinámica para clientes como c1
  option routers 192.168.57.10;       # Puerta de enlace por defecto
}

# --- Reserva Fija por MAC (Printer) ---
host printer {
  hardware ethernet 08:00:27:11:22:33; # Dirección MAC del dispositivo
  fixed-address 192.168.57.111;       # IP fija reservada
  default-lease-time 7200;            # Tiempo de concesión específico: 2 horas
  max-lease-time 7200;
}

# Validar sintaxis del archivo dhcpd.conf
sudo dhcpd -t

# Reiniciar el servicio DHCP
sudo systemctl restart isc-dhcp-server.service

# Verificar que el servicio está activo y escuchando en el puerto UDP 67
sudo systemctl status isc-dhcp-server.service
sudo ss -lun
