#!/bin/bash
set -x
 
# Actualizar la lista de paquetes disponibles
sudo apt update

# Actualizar todos los paquetes instalados
sudo apt upgrade -y

# Instalar el servicio isc-dhcp-server
sudo apt install isc-dhcp-server -y

# Deshabilitar NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager

# Cambiar la configuración en /etc/default/isc-dhcp-server
sudo bash -c 'cat << EOF > /etc/default/isc-dhcp-server
INTERFACESv4="enX0"
INTERFACESv6=""
EOF'

# Cambiar la configuración en /etc/dhcp/dhcpd.conf
sudo bash -c 'cat << EOF > /etc/dhcp/dhcpd.conf
option domain-name "mi-red.local";
option domain-name-servers ns1.mi-red.local, ns2.mi-red.local;

default-lease-time 600;
max-lease-time 7200;

subnet 172.5.10.0 netmask 255.255.255.0 {
  option routers 172.5.10.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  pool {
    max-lease-time 3600;
    range 172.5.10.10 172.5.10.20;
  }
  # Configuración adicional para DHCP
  authoritative;
  failover peer "FAILOVER" {
    primary;
    address 172.5.10.100;
    port 647;
    peer address 172.5.10.101;
    peer port 647;
    max-unacked-updates 10;
    max-response-delay 30;
    load balance max seconds 3;
    mclt 1800;
    split 128;
  }
}
EOF'

# Cambiar la configuración en /etc/network/interfaces
sudo bash -c 'cat << EOF > /etc/network/interfaces
auto enX0
iface enX0 inet static
        address 172.5.10.1
        netmask 255.255.255.0
        network 172.5.10.0
        broadcast 172.5.10.255
EOF'

# Reiniciar el servicio networking
sudo systemctl restart networking

# Reiniciar el servicio dhcp
sudo systemctl restart isc-dhcp-server

# Mostrar un mensaje de finalización
echo "Actualización de paquetes y configuración completada, networking reiniciado."
