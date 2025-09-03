# UNIDAD 1

## 1. Instalación de servicios de configuración dinámica de sistemas

Introducción a los parámetros fundamentales de red y al papel de DHCP en la asignación automática de configuraciones. Se incluyen ejemplos prácticos, buenas prácticas y particularidades en sistemas libres y propietarios.

### Parámetros básicos de red

**Dirección IP** — Identifica de forma única a cada dispositivo en una red basada en TCP/IP. En IPv4 consta de cuatro octetos separados por puntos (por ejemplo, 192.168.1.100) que identifican tanto la red como el host.

**Máscara de red** — Determina qué parte de la dirección IP corresponde a la red y cuál al host; por ejemplo, 255.255.255.0 (equivalente a /24) indica que los primeros tres octetos son la parte de red y el último identifica al host. Todos los equipos de una misma subred comparten la misma máscara, lo que define el rango de direcciones locales.

**Puerta de enlace** — Dirección IP del router de la red local, que actúa como punto de salida hacia otras redes (típicamente, Internet).

En una pequeña oficina con red 192.168.1.0/24, la puerta de enlace suele ser 192.168.1.1. Los equipos envían el tráfico destinado fuera de la LAN a esa puerta de enlace para que el router lo reenvíe. Un técnico de soporte debe saber configurar estos tres parámetros en un PC; si se asignan mal (máscara incorrecta o puerta de enlace errónea), el equipo puede quedar sin comunicación o sin acceso a Internet.

En entornos de PYME y centros educativos es habitual usar direcciones IP privadas (como 192.168.x.x o 10.x.x.x) con una puerta de enlace que conecta con la red del proveedor de Internet. Es importante planificar el direccionamiento evitando conflictos y documentando qué IP se asigna a cada servidor, impresora o dispositivo importante.

Por lo general, los servidores o dispositivos críticos se configuran con IP fija, mientras que a los ordenadores de usuario se les puede asignar IP dinámica mediante un servidor DHCP para simplificar la administración.

### DHCP (Dynamic Host Configuration Protocol)

**Definición** — Protocolo de configuración dinámica de host que automatiza la asignación de parámetros IP a los clientes de una red. En lugar de configurar manualmente IP, máscara, puerta de enlace y DNS en cada equipo, un servidor DHCP los distribuye bajo demanda.

**Ventajas** — Reduce errores de configuración y facilita mover equipos entre redes. En una empresa pequeña, el servidor DHCP puede asignar automáticamente direcciones en el rango 192.168.1.100–192.168.1.200 con su máscara y gateway a cada portátil que se conecte, evitando configuraciones manuales.

### Rangos, exclusiones, concesiones y reservas

**Ámbitos/pools** — Un servidor DHCP trabaja sobre uno o varios rangos de direcciones IP definidas. Por ejemplo, un ámbito 192.168.1.100–192.168.1.200 para los clientes.

**Exclusiones** — Direcciones dentro del ámbito que no se entregarán porque estén asignadas de forma fija a servidores u otros equipos (por ejemplo, excluir la .100 si la usa una impresora con IP fija).

**Concesiones (leases)** — Cuando un cliente obtiene una IP, el servidor “alquila” la dirección por un tiempo determinado (por ejemplo, 24 horas). La concesión incluye la IP otorgada, la MAC del cliente y la expiración. El cliente intentará renovar antes de expirar para conservar la misma IP; si se apaga, el servidor podrá recuperar esa IP al vencer el plazo.

**Reservas** — Asignaciones fijas dentro del DHCP: a una determinada MAC siempre se le entrega la misma IP específica. Útil para dispositivos que requieren IP estable (NAS, impresoras) pero gestionados vía DHCP. En la práctica, un administrador junior crea reservas para servidores y dispositivos importantes y deja el resto del rango para concesiones dinámicas.

### Funcionamiento del protocolo DHCP (DORA)

El proceso sigue un esquema cliente-servidor de cuatro fases que evita duplicados de IP y asegura la entrega correcta de la configuración.

1. **Discover** — El cliente envía un broadcast para localizar servidores DHCP disponibles.
2. **Offer** — Uno o varios servidores responden con una oferta (IP disponible y otros datos como máscara, gateway, DNS).
3. **Request** — El cliente elige una oferta y envía un broadcast indicando qué oferta acepta (incluye IP y servidor).
4. **ACK** — El servidor confirma la concesión y envía la configuración definitiva; el cliente configura su interfaz.

Además, existen mensajes como **NAK** (negativa si la solicitud es inválida o la concesión expiró) o **Release** (el cliente libera la IP al apagarse). En entornos profesionales debe haber solo un servidor DHCP activo por segmento para evitar conflictos, y ajustar el tiempo de arrendamiento a las necesidades (corto en redes con mucha rotación, largo en redes estables).

Una ventaja práctica de DHCP es que también distribuye otros parámetros: servidores DNS, nombre de dominio e incluso opciones específicas (servidor WINS para redes antiguas, opciones de boot para PXE). Esto simplifica la configuración de puestos de trabajo en oficinas y aulas, reduciendo errores y facilitando el soporte técnico.

### Tipos de mensajes DHCP

**Discover, Offer, Request, ACK** — Conforman el flujo básico. Discover, Request y algunos ACK se envían en broadcast porque el cliente no conoce su IP ni el servidor al inicio. Offer puede ir en unicast si el servidor identifica al cliente, o en broadcast en caso contrario.

**Release** — Lo envía el cliente al apagar o desconectar para avisar que ya no usará la IP.

**Inform** — El cliente solicita información adicional de configuración sin cambiar su IP.

**NAK** — El servidor rechaza una IP inválida o una concesión expirada, forzando a reiniciar el proceso.

Los mensajes incluyen campos como la MAC del cliente (clave de identificación) y un ID de transacción para emparejar solicitudes y respuestas. Entender estos mensajes permite diagnosticar: si un cliente no obtiene IP, conviene capturar con un analizador (Wireshark) para verificar Discover y Offer; si no hay Offer, el servidor no responde (servicio caído o cortafuegos); si hay Offer pero no ACK, puede haber interferencia de otro servidor.

### Clientes DHCP en sistemas operativos libres y propietarios

**Instalación** — En la mayoría de sistemas (Linux, BSD, Windows, macOS) el cliente DHCP viene integrado. En Windows, el servicio Cliente DHCP se ejecuta automáticamente si la interfaz está en “Obtener una dirección IP automáticamente”. En Linux de escritorio, NetworkManager usa internamente un cliente (dhclient o systemd-networkd). En sistemas minimalistas puede ser necesario instalar isc-dhcp-client. En entornos de PYME o educativos, los PCs ya traen cliente DHCP listo; basta con habilitarlo.

**Configuración de interfaces para DHCP** — En Windows, en las propiedades TCP/IP de la tarjeta se elige “Obtener una dirección IP automáticamente” y “Obtener la dirección del servidor DNS automáticamente”. En Linux, depende de la distribución y herramientas:
- En /etc/network/interfaces (Debian clásico): iface eth0 inet dhcp.
- En Red Hat/CentOS: BOOTPROTO=dhcp en ifcfg-eth0.
- Con Netplan (Ubuntu actual): dhcp4: true en YAML.
- En entorno gráfico, se elige “Automático (DHCP)”.

Al conectar cable o activar Wi-Fi, el cliente negocia la configuración. Para verificar:
- En Windows, usar ipconfig /all para ver si la dirección proviene de DHCP y cuál es el servidor.
- En Linux, ip addr o ifconfig muestran la IP, y /var/log/syslog registra la concesión.

Una dirección APIPA (169.254.x.x) indica que no se obtuvo respuesta DHCP (problemas de cable, cobertura Wi-Fi, VLAN incorrecta, servidor caído). La buena práctica es usar DHCP para equipos cliente y reservar IP manual solo cuando sea necesario (servidores, dispositivos fuera del rango, ausencia de servidor). Incluso para servidores, muchas empresas optan por reservas DHCP para centralizar la gestión.

### Servidores DHCP en sistemas operativos libres y propietarios

**Instalación** — En Linux, el servidor más común es ISC DHCP (isc-dhcp-server); alternativas: dnsmasq (ligero) o KEA DHCP (más moderno). En Windows Server (2016, 2019, etc.), DHCP se agrega como rol desde el Administrador de servidores y se administra con su consola. En pequeñas oficinas sin Windows Server, el router del ISP suele actuar como servidor DHCP (firmware propietario tipo MikroTik, Cisco/Linksys). Es clave identificar dónde reside el servidor en la red.

**Arranque** — En Linux, el daemon (dhcpd) suele habilitarse automáticamente; se controla con systemctl start/enable isc-dhcp-server (o service dhcpd start en sistemas antiguos). En Windows Server, el servicio arranca automáticamente; se verifica en services.msc. En dominios Active Directory, hay que **autorizar** el servidor DHCP en AD para evitar servidores no autorizados. Revisar también el cortafuegos: abrir UDP 67 en Linux; en Windows, el rol crea reglas, pero conviene confirmarlo.

**Ficheros y parámetros de configuración básica** — En ISC DHCP, el archivo principal es /etc/dhcp/dhcpd.conf, donde se definen subredes, rangos, máscara, gateway, DNS y tiempos de lease. Un ejemplo típico:

> **Ejemplo (ISC DHCP):**  
> subnet 192.168.1.0 netmask 255.255.255.0 {  
> range 192.168.1.100 192.168.1.200;  
> option routers 192.168.1.1;  
> option domain-name-servers 192.168.1.1, 8.8.8.8;  
> default-lease-time 86400;  
> max-lease-time 172800;  
> }  
> option domain-name "empresa.local";

En Windows Server, la consola gráfica crea el ámbito indicando red (p. ej., 192.168.1.0/24), rango inicial–final, puerta de enlace (router option), DNS y lease time, y permite agregar exclusiones (p. ej., .1 a .50 para IP fijas). Tras modificar la configuración, reiniciar el servicio (Linux: systemctl restart dhcpd; Windows: reinicio del servicio o refrescar en consola).

Es buena práctica documentar la configuración, especialmente reservas y exclusiones, y mantener el servidor actualizado para evitar vulnerabilidades (un DHCP comprometido podría distribuir gateway o DNS maliciosos).

### Información sobre concesiones (lease)

El servidor registra concesiones activas y expiradas. En ISC DHCP, el fichero de arrendamientos suele ser /var/lib/dhcp/dhcpd.leases; también pueden consultarse los logs en /var/log/syslog (entradas como “DHCPACK on 192.168.1.101 to 00:11:22:33:44:55 (hostname)”).

En Windows Server, la consola muestra en tiempo real “Direcciones IP en arrendamiento” por ámbito (IP, nombre de host, MAC/ID de hardware y expiración). Esto permite localizar la IP de un PC por nombre (y viceversa), diagnosticar por qué un dispositivo no obtiene IP o liberar concesiones atascadas (quitar arrendamiento) y crear reservas. En Linux, las reservas se agregan al dhcpd.conf con host fixed-address para una MAC dada.

Cuando un lease expira sin renovación, la IP vuelve al pool. Ajustar el tiempo de concesión a las necesidades: en aulas con alta rotación, leases cortos (horas) reciclan direcciones; en oficinas con PCs fijos, leases largos (días/semanas) reducen tráfico y conservan IPs habituales.

El registro de concesiones es herramienta de auditoría: ayuda a saber qué dispositivos se conectaron y cuándo. Por ciberseguridad básica, conviene almacenar o exportar periódicamente esta información para detectar equipos no autorizados (MACs desconocidas). También se debe proteger el servicio frente a **rogue DHCP** segmentando la red o activando funciones de seguridad en switches (por ejemplo, **DHCP snooping** para permitir respuestas solo desde el puerto autorizado del servidor legítimo).

En resumen, la gestión de concesiones DHCP es una tarea rutinaria del administrador de red junior, tanto para mantenimiento (limpieza de leases antiguos, configuración de reservas) como para resolución de incidencias.
