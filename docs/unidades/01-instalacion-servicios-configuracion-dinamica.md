# 1. Instalación de servicios de configuración dinámica de sistemas

## Dirección IP, máscara de red, puerta de enlace.
Una dirección **IP** identifica de forma unívoca a cada equipo en una red basada en TCP/IP. En IPv4 se expresa en notación decimal con puntos (por ejemplo, 192.168.10.25) y se combina con una **máscara de red** o **prefijo CIDR** (255.255.255.0 ≡ /24) que separa la parte de red y la de host. La **puerta de enlace** (gateway) es la IP del router al que se envía el tráfico destinado fuera de la subred local.
Una configuración coherente exige que IP y gateway pertenezcan a la misma subred y que la máscara sea idéntica para todos los equipos de esa subred. Además, suele definirse al menos un **servidor DNS** y, opcionalmente, un **dominio de búsqueda** para resolver nombres internos.
Ejemplo aplicado (aula o pyme): subred 192.168.30.0/24, gateway 192.168.30.1, clientes 192.168.30.50–192.168.30.200, DNS interno 192.168.30.10 y secundario público 1.1.1.1. Esta planificación evita conflictos, facilita el soporte y prepara el entorno para la asignación dinámica mediante DHCP.
Errores típicos que provocan pérdida de conectividad: máscara distinta entre equipos, gateway fuera de subred, IPs duplicadas por configuraciones manuales y uso de DNS públicos en equipos que necesitan resolver servicios internos.

---

## DHCP (Dynamic Host Configuration Protocol):
El **DHCP** automatiza la entrega de parámetros de red a los clientes (IP, máscara, gateway, DNS, NTP, dominio, rutas, opciones PXE, etc.). Reduce errores humanos, acelera el alta de equipos y permite políticas homogéneas en aulas y oficinas.

### Rangos, exclusiones, concesiones y reservas.
Un **rango** (scope) define el conjunto de direcciones entregables en una subred (por ejemplo, 192.168.30.50–192.168.30.200). Las **exclusiones** marcan direcciones dentro de ese rango que no deben asignarse automáticamente (por ejemplo, reservar 192.168.30.50–192.168.30.59 para dispositivos especiales). Una **concesión** (lease) es el “alquiler” temporal de una IP a un equipo identificado, por lo general, por su MAC; incluye tiempos de renovación y expiración. Las **reservas** fijan una IP estable para un dispositivo concreto (mapeo MAC→IP) manteniendo la gestión centralizada en el servidor.
En entornos con varias subredes o VLAN, conviene segmentar rangos por función (aulas, administración, invitados) y documentar reservas en el inventario para agilizar el soporte y la trazabilidad.

### Funcionamiento de protocolo DHCP.
El ciclo básico en IPv4 se recuerda como **DORA**:
1) **Discover**: el cliente anuncia por broadcast que necesita configuración.
2) **Offer**: el servidor propone una IP y opciones.
3) **Request**: el cliente solicita explícitamente la oferta elegida.
4) **ACK**: el servidor confirma y entrega la configuración (o **NAK** si la deniega).
Los puertos de trabajo son UDP 67 (servidor) y 68 (cliente). A mitad del arrendamiento (T1) el cliente intenta renovar; si falla, lo reintenta en T2. En redes con **VLAN**, los broadcasts no cruzan routers, por lo que se usa **DHCP Relay** (ip helper) para reenviar solicitudes al servidor central. Entre las **opciones** habituales destacan router/gateway (003), DNS (006), dominio (015), NTP (042), rutas (121/249) y parámetros PXE para imaging masivo.
La alta disponibilidad se obtiene con **failover** (equilibrio y toma de control) o con servidores redundantes y base de concesiones compartida, lo que resulta clave en redes de aulas y sedes.

### Tipos de mensajes DHCP.
En **IPv4** se utilizan, entre otros: DISCOVER, OFFER, REQUEST, DECLINE, ACK, NAK, RELEASE, INFORM.
En **IPv6 (DHCPv6)**: SOLICIT, ADVERTISE, REQUEST, CONFIRM, RENEW, REBIND, REPLY, RELEASE, DECLINE, RECONFIGURE, INFORMATION-REQUEST. En IPv6 conviven **SLAAC** y combinaciones SLAAC+DHCPv6 para distribuir opciones como DNS.

---

## Clientes DHCP en sistemas operativos libres y propietarios:
### Instalación.
En **GNU/Linux** el cliente DHCP forma parte del sistema (por systemd-networkd, NetworkManager o dhclient/dhcpcd según la distribución). En **Windows** y **macOS** el cliente viene habilitado por defecto en los adaptadores de red. En máquinas virtuales, el modo **NAT** utiliza el DHCP del hipervisor; en **bridge**, el cliente negocia con el servidor de la LAN.

### Configuración de interfaces de red para que obtengan su configuración por DHCP.
En **Linux** con Netplan (Ubuntu Server) se activa dhcp4 en la interfaz y se aplican cambios con netplan apply.
En **Linux** con NetworkManager (estaciones de aula) se establece ipv4.method auto y se levanta la conexión con nmcli.
En **Windows** (PowerShell) se habilita DHCP en la interfaz con Set-NetIPInterface y se renueva con ipconfig /release e ipconfig /renew.
Comprobaciones útiles: ipconfig /all (Windows), nmcli dev show o journalctl -u NetworkManager (Linux) y revisión de resolv.conf para DNS. Un síntoma típico de fallo es obtener una dirección **APIPA** (169.254.x.x), que indica ausencia de respuesta del servidor.

---

## Servidores DHCP en sistemas operativos libres y propietarios:
### Instalación.
En **GNU/Linux** son habituales **ISC DHCP Server** (isc-dhcp-server), **Kea DHCP** (alto rendimiento, backend SQL) y **dnsmasq** (ligero, DHCP+DNS caché idóneo para laboratorios). En **Windows Server** se agrega el rol **DHCP Server** y, en dominios, se **autoriza** el servidor en Active Directory para evitar servidores no autorizados.

### Arranque.
En **Linux** los servicios se gestionan con systemd (enable/now/status del servicio correspondiente). En **Windows Server** se inicia el servicio **DHCP Server** y se valida su estado desde la consola de administración.
La supervisión continua de logs y alertas (agotamiento de pool, NAK repetidos, conflictos ARP) permite actuar antes de que los usuarios perciban cortes.

### Ficheros y parámetros de configuración básica.
En **ISC DHCP** la configuración reside en /etc/dhcp/dhcpd.conf. Esquema típico para un aula:
default-lease-time 3600
max-lease-time 7200
authoritative
subnet 192.168.30.0 netmask 255.255.255.0 {
range 192.168.30.50 192.168.30.200;
option routers 192.168.30.1;
option domain-name "centro.local";
option domain-name-servers 192.168.30.10, 1.1.1.1;
option ntp-servers 192.168.30.10;
host prn-aula-1 { hardware ethernet 00:11:22:33:44:55; fixed-address 192.168.30.60; }
}
En **dnsmasq** (laboratorios pequeños) se combinan DHCP y DNS caché en un único servicio con dhcp-range y dhcp-option para gateway y DNS.
En **Kea DHCP** la configuración es JSON y permite almacenar concesiones en base de datos, exponer API y definir reservas granulares.
En **Windows Server** se crean **Ámbitos** con rango, máscara y exclusiones, se definen **Opciones** (router, DNS, dominio) y, si se realiza imaging, se habilitan opciones PXE. Los asistentes guían el proceso y la consola ofrece vistas de estado y herramientas de exportación.

### Información sobre concesiones (lease).
En **ISC DHCP** las concesiones activas y expiradas se registran en el fichero dhcpd.leases (habitualmente en /var/lib/dhcp/) con IP, MAC, tiempos y nombre de host. En **Kea** pueden consultarse en ficheros o base de datos y por API; en **dnsmasq** se listan en dnsmasq.leases. En **Windows Server**, el panel Address Leases muestra en tiempo real IP, MAC, nombre y expiración, y permite liberar concesiones problemáticas o convertirlas en reservas.
La observación de estos registros es esencial para diagnosticar “no obtiene IP”, localizar conflictos, identificar dispositivos desconocidos y ajustar el **lease time** según rotación (más corto en aulas, más largo en oficinas).

---

**Consejos orientados a empleo (técnico microinformático, soporte, admin junior):** automatizar altas con reservas por MAC documentadas; usar plantillas por sede/VLAN y control de cambios; integrar DHCP, DNS y NTP coherentes; evitar DNS públicos en equipos que consumen recursos internos; aplicar seguridad básica (DHCP Snooping en switches, bloqueo de puertos no autorizados en aulas).
