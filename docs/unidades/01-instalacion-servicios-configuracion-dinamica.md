# Instalación de servicios de configuración dinámica de sistemas

## Dirección IP, máscara de red, puerta de enlace
> Una dirección IP identifica de forma única a cada dispositivo en una red basada en TCP/IP.

- **Dirección IP (IPv4)**: consta de cuatro números (octetos) separados por puntos (por ejemplo, 192.168.1.100) que identifican tanto la red como el host.
- **Máscara de red**: determina qué parte de la dirección IP corresponde a la red y cuál al host; por ejemplo, **255.255.255.0** (equivalente a **/24**) indica que los primeros 3 octetos son la parte de red y el último identifica al host.
- **Puerta de enlace (router)**: dirección IP del encaminador de la red local, que actúa como punto de salida hacia otras redes (típicamente, hacia Internet). Por ejemplo, en una pequeña oficina con red **192.168.1.0/24**, la puerta de enlace suele ser **192.168.1.1**.

**Notas y buenas prácticas:**
- Todos los equipos de una misma **subred comparten la misma máscara**, lo que define el rango de direcciones locales.
- Los equipos envían el tráfico destinado **fuera de la LAN** a la puerta de enlace para que el router lo reenvíe.
- Un técnico de soporte debe saber **configurar estos tres parámetros** en un PC: si se asignan mal (por ejemplo, una máscara incorrecta o puerta de enlace errónea), el equipo podría no comunicarse con otros o no tener acceso a Internet.
- En entornos de **PYME y centros educativos** es habitual usar direcciones **IP privadas** (como 192.168.x.x o 10.x.x.x) con una puerta de enlace que conecta con la red del proveedor de Internet.
- Es importante **planificar el direccionamiento** evitando conflictos y **documentando** qué IP se asigna a cada servidor, impresora o dispositivo importante.
- Por lo general, los **servidores o dispositivos críticos** se configuran con **IP fija**, mientras que a los **ordenadores de usuario** se les puede asignar **IP dinámica** mediante un servidor DHCP para simplificar la administración.

## DHCP (Dynamic Host Configuration Protocol)
> DHCP es un protocolo de configuración dinámica de host que **automatiza la asignación de parámetros IP** a los clientes de una red.

- En lugar de configurar manualmente IP, máscara, puerta de enlace y DNS en cada equipo, un **servidor DHCP** los distribuye bajo demanda.
- Esto **reduce errores de configuración** y facilita mover equipos entre redes.
- **Ejemplo**: en una empresa pequeña, el servidor DHCP puede asignar automáticamente direcciones en el rango **192.168.1.100–192.168.1.200** con su máscara y gateway correspondientes a cada portátil que se conecte, evitando tener que configurarlos a mano.

### Rangos, exclusiones, concesiones y reservas
- Un servidor DHCP trabaja sobre uno o varios **rangos/ámbitos (pools)** de direcciones IP definidas.
  - **Ejemplo de ámbito**: desde **192.168.1.100** hasta **192.168.1.200** para los clientes.
- **Exclusiones**: direcciones dentro del ámbito que **no se entregarán** porque están asignadas de forma fija a servidores u otros equipos (por ejemplo, excluir la **.100** si la usa una impresora con IP fija).
- **Concesiones (leases)**: cuando un cliente obtiene una IP, el servidor **“alquila”** la IP por un tiempo determinado (por ejemplo, **24 horas**). La concesión incluye la dirección otorgada, la **MAC** del cliente y el **tiempo de expiración**.
  - Si el cliente sigue activo, **intentará renovar** la concesión antes de que expire para conservar la misma IP.
  - Si se apaga o desconecta, el servidor **podrá recuperar** esa IP una vez vencido el plazo para asignársela a otro.
- **Reservas**: asignaciones fijas dentro del DHCP para que a una determinada **MAC** o cliente **siempre se le entregue la misma IP** específica.
  - Útil para **NAS, impresoras de red** u otros dispositivos que requieren IP estable, gestionándolos vía DHCP.
  - Evitan conflictos y facilitan la **administración centralizada** de direcciones.
- **Práctica en PYME**: crear **reservas** para servidores y dispositivos importantes, y dejar el resto del rango para **concesiones dinámicas** de los PCs de oficina.

### Funcionamiento del protocolo DHCP (DORA)
> El proceso DHCP sigue un esquema cliente-servidor de cuatro fases conocido por el acrónimo **DORA**.

1. **Discover (DHCP Discover)**: el cliente envía un broadcast para **encontrar servidores DHCP** disponibles.
2. **Offer (DHCP Offer)**: uno o varios servidores responden proponiendo **una configuración** (IP disponible y otros datos como máscara, gateway, DNS, etc.).
3. **Request (DHCP Request)**: el cliente **elige una oferta** y la solicita (normalmente en broadcast, indicando qué oferta acepta).
4. **ACK (DHCP ACK)**: el servidor confirma la **concesión** enviando el reconocimiento con la configuración definitiva.

**Consideraciones operativas:**
- Este ciclo garantiza que **no haya duplicados de IP** y que la configuración llegue correctamente.
- **Mensajes adicionales**:
  - **DHCP NAK**: negativa si el servidor rechaza una solicitud (por ejemplo, por expiración).
  - **DHCP Release**: el cliente libera voluntariamente la IP (por ejemplo, al apagarse).
- En entornos profesionales:
  - Asegurar **solo un servidor DHCP activo por segmento** para evitar conflictos de ofertas.
  - Ajustar **tiempos de arrendamiento**: más cortos en redes con muchos cambios; más largos en redes estables.
  - DHCP puede distribuir **DNS, nombre de dominio**, opciones **WINS** (redes Windows antiguas) u **opciones de boot PXE** para despliegue.

### Tipos de mensajes DHCP
- **Discover, Offer, Request y ACK** conforman el flujo básico.
  - **Discover, Request** y algunos **ACK** se envían en **broadcast** (el cliente aún no conoce servidor ni su propia IP).
  - El **Offer** puede ser **unicast** (si el servidor identifica al cliente) o **broadcast**.
- Mensajes de **gestión de concesión**:
  - **DHCP Release**: liberación de IP (al apagar o desconectar).
  - **DHCP Inform**: solicitar **información adicional** de configuración sin cambiar la IP.
  - **DHCP NAK**: cuando el cliente solicita una IP inválida o expirada, indicando que debe **reiniciar el proceso**.
- **Campos internos** de los mensajes: **MAC del cliente**, **ID de transacción** para emparejar solicitudes y respuestas.
- **Diagnóstico** con analizador de protocolos (por ejemplo, **Wireshark**):
  - Si no hay IP, verificar **Discover** y **Offer**.
  - **Sin Offer**: servidor no responde (posible **fallo del servicio** o **bloqueo por cortafuegos**).
  - **Hay Offer pero no ACK**: posible **interferencia de otro servidor**.
  - Comprender la secuencia **DORA** ayuda a resolver incidencias como **“IP address conflict”** (por ejemplo, dos servidores DHCP activos).

## Clientes DHCP en sistemas operativos libres y propietarios

### Instalación
- En la mayoría de **sistemas operativos** (Linux, BSD, Windows, macOS) el **cliente DHCP viene integrado** de serie.
- **Windows**: el servicio **Cliente DHCP** se ejecuta automáticamente si la interfaz está en modo “Obtener una dirección IP automáticamente”.
- **Linux de escritorio**: servicios como **NetworkManager** utilizan internamente un cliente DHCP (por ejemplo, **dhclient** o **systemd-networkd**).
- **Linux minimalista o embebido**: puede requerir instalar **isc-dhcp-client** (Debian/Ubuntu) si no está presente.
- En entornos estándar de **PYME o educativos**, los PCs y dispositivos ya traen **cliente DHCP listo para usar**; solo hay que **habilitarlo**.

### Configuración de interfaces de red para que obtengan su configuración por DHCP
- **Windows**:
  - Propiedades de **TCP/IP** de la tarjeta de red → **Obtener una dirección IP automáticamente** y **Obtener la dirección del servidor DNS automáticamente**.
  - Similar para **IPv6** si se configura.
- **Linux** (según distribución/herramientas):
  - Archivo **/etc/network/interfaces** (Debian clásico): poner **iface eth0 inet dhcp**.
  - **Red Hat/CentOS**: en **ifcfg-eth0** establecer **BOOTPROTO=dhcp**.
  - **Netplan (Ubuntu actual)**: en YAML indicar **dhcp4: true**.
  - Entorno gráfico: editar la conexión y elegir **Automático (DHCP)**.
- **Verificación**:
  - **Windows**: comando **ipconfig /all** muestra si la dirección está asignada por DHCP y el **servidor DHCP**.
  - **Linux**: **ip addr** o **ifconfig** muestran la IP obtenida; registros en **/var/log/syslog** reflejan la concesión.
- **Error común (APIPA)**:
  - Dirección **169.254.x.x** en Windows indica que **no se consiguió DHCP**.
  - Posibles causas: **cable**, **alcance Wi-Fi**, **VLAN incorrecta** en el switch, **servidor DHCP caído**.
- **Buena práctica**: usar **DHCP** para equipos cliente por **flexibilidad y gestión centralizada**.
  - Configurar **IP manual** solo para **servidores**, dispositivos que requieren IP fija **fuera del rango DHCP**, o cuando **no hay servidor DHCP**.
  - Incluso para servidores, muchas empresas optan por **reservas DHCP** para mantener el **control centralizado**.

## Servidores DHCP en sistemas operativos libres y propietarios

### Instalación
- **Linux (software libre)**:
  - Servidor más común: **ISC DHCP (isc-dhcp-server)**.
  - Alternativas: **dnsmasq** (ligero, routers/laboratorio) o **KEA DHCP** (también de ISC, más moderno).
- **Windows Server (software propietario)**:
  - Rol **DHCP** disponible (por ejemplo, 2016, 2019…); consola de **administración DHCP** para configurar ámbitos.
- **Pequeñas oficinas sin Windows Server**:
  - El **router del ISP** suele actuar como servidor DHCP (firmware propietario tipo **MikroTik, Cisco/Linksys**, etc.) con DHCP básico.
- **Identificar dónde reside DHCP**:
  - En empresas con **Active Directory**, probablemente el **controlador de dominio Windows Server**.
  - En oficinas pequeñas, **router ADSL/fibra** del operador.
- **Servidor Linux**:
  - Asegurar que el servicio **arranca al inicio** y tiene permisos para **escuchar en la interfaz** correcta (por ejemplo, en Ubuntu editar **/etc/default/isc-dhcp-server** para indicar la interfaz).

### Arranque
- **Linux**:
  - Servicio **dhcpd** habilitado tras la instalación.
  - Gestión habitual: **systemctl start isc-dhcp-server** y **systemctl enable isc-dhcp-server** (en SysV: **service dhcpd start** y configuración de **runlevels**).
- **Windows Server**:
  - Tras agregar el rol DHCP, el servicio **se inicia automáticamente** y arranca con el sistema.
  - Verificar en **services.msc** que esté **“En ejecución”**.
  - **Autorizar el servidor DHCP en Active Directory** si es miembro del dominio (evita servidores no autorizados).
- **Cortafuegos**:
  - **Linux**: abrir **UDP 67** en el servidor (UFW, firewalld…).
  - **Windows**: el rol DHCP crea reglas en **Firewall de Windows**, conviene confirmarlo.

### Ficheros y parámetros de configuración básica
- **Linux (ISC DHCP)**:
  - Archivo principal: **/etc/dhcp/dhcpd.conf**.
  - Definir **subredes** que servirá, **rango de IPs**, **máscara**, **gateway**, **DNS**, **duración de leases**, etc.
  - **Ejemplo (comentado)**:
    > # dhcpd.conf (ejemplo)  
    > # subnet 192.168.1.0 netmask 255.255.255.0 {  
    > #     range 192.168.1.100 192.168.1.200;  
    > #     option routers 192.168.1.1;  
    > #     option domain-name-servers 192.168.1.1, 8.8.8.8;  
    > #     default-lease-time 86400;  
    > #     max-lease-time 172800;  
    > # }
  - También incluir **option domain-name "empresa.local";** si se necesita definir el **dominio de búsqueda DNS**.
- **Windows Server**:
  - Configuración mediante **consola gráfica**:
    - Crear **nuevo ámbito (scope)** indicando red (ej.: **192.168.1.0/24**), **rango inicial-final**, **puerta de enlace**, **DNS**, y **lease time**.
    - Agregar **exclusiones** (por ejemplo, excluir **.1 a .50** si son IP fijas de servidores).
  - **Sugerencias**:
    - Windows puede **detectar la subred** del servidor y sugerir datos.
    - En Linux hay que ser **explícito** en el archivo.
  - Tras modificar la configuración, **reiniciar el servicio** para aplicar cambios (Linux: **systemctl restart dhcpd**; Windows: reiniciar servicio o refrescar en consola).
- **Buenas prácticas**:
  - **Documentar** la configuración DHCP (especialmente **reservas** y **exclusiones**).
  - Mantener servidores **actualizados** (evitar vulnerabilidades que permitan entregar **gateway/DNS maliciosos**).

### Información sobre concesiones (lease)
- El servidor DHCP registra **concesiones activas y expiradas**.
- **Linux/ISC DHCP**:
  - Fichero de arrendamientos: **/var/lib/dhcp/dhcpd.leases**.
  - Revisar también **/var/log/syslog** para mensajes del demonio (**DHCPACK on 192.168.1.101 to 00:11:22:33:44:55 (hostname)**).
- **Windows Server**:
  - La consola muestra en tiempo real **Direcciones IP en arrendamiento** bajo cada ámbito:
    - **IP**, **nombre de host**, **MAC (ID de hardware)** y **tiempo de expiración**.
  - Posible **liberar concesiones** (quitar arrendamiento) o **crear reservas** desde la consola.
- **Reservas en Linux**:
  - Editar **dhcpd.conf** añadiendo **host** con **fixed-address** para una **MAC**.
- **Ajuste de expiración**:
  - **Leases cortos** (horas) en redes de aulas con alta rotación.
  - **Leases largos** (días/semanas) en oficinas con PCs fijos (menos tráfico DHCP y mayor estabilidad de IP).
- **Auditoría y ciberseguridad básica**:
  - Almacenar o exportar periódicamente la **información de concesiones** para rastrear dispositivos conectados (detectar **MAC desconocidas**).
  - Proteger el servidor DHCP:
    - En **Windows**, mediante **autorización en AD**.
    - En general, evitar **rogue DHCP** segmentando la red o usando **DHCP snooping** en switches gestionables (filtrar respuestas solo desde el puerto autorizado del servidor legítimo).
- **Resumen operativo**:
  - La **gestión de concesiones DHCP** es una tarea rutinaria del administrador de red junior: mantenimiento (**limpieza de leases antiguos**, **configuración de reservas**) y **resolución de incidencias**.

