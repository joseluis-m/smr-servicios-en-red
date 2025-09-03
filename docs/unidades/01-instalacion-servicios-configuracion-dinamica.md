# 1. Instalación de servicios de configuración dinámica de sistemas

## Dirección IP, máscara de red, puerta de enlace
**Resumen:** La **dirección IP**, la **máscara de red** y la **puerta de enlace** permiten identificar dispositivos, definir subredes y enrutar tráfico hacia otras redes.

Una **dirección IP** identifica de forma única a cada dispositivo en una red basada en TCP/IP. En _IPv4_ consta de cuatro números (octetos) separados por puntos (por ejemplo, 192.168.1.100) que identifican tanto la **red** como el **host**. La **máscara de red** determina qué parte de la dirección IP corresponde a la red y cuál al host; por ejemplo, una máscara 255.255.255.0 (equivalente a **/24**) indica que los primeros 3 octetos son la parte de red y el último identifica al host. Todos los equipos de una misma subred comparten la misma máscara, lo que define el rango de direcciones locales. La **puerta de enlace** es la dirección IP del encaminador (router) de la _red local_, que actúa como punto de salida hacia otras redes (típicamente, hacia Internet). Por ejemplo, en una pequeña oficina con red **192.168.1.0/24**, la puerta de enlace suele ser **192.168.1.1**. Los equipos envían el tráfico destinado fuera de la **LAN** a esa puerta de enlace para que el router lo reenvíe. Un técnico de soporte debe saber **configurar** estos tres parámetros en un PC: si se asignan mal (por ejemplo, una **máscara incorrecta** o **puerta de enlace errónea**), el equipo podría no comunicarse con otros o no tener acceso a Internet. En entornos de **PYME** y centros educativos es habitual usar **direcciones IP privadas** (como 192.168.x.x o 10.x.x.x) con una puerta de enlace que conecta con la red del proveedor de Internet. Es importante **planificar el direccionamiento** evitando conflictos y **documentando** qué IP se asigna a cada servidor, impresora o dispositivo importante. Por lo general, los **servidores** o dispositivos críticos se configuran con **IP fija**, mientras que a los ordenadores de usuario se les puede asignar **IP dinámica** mediante un servidor _DHCP_ para simplificar la administración.

> **Ejemplo:** En 192.168.1.0/24, fija 192.168.1.1 al router, reserva 192.168.1.2–.50 para servidores/impresoras y usa 192.168.1.100–.200 para clientes.

> **Advertencia:** Una **máscara** mal configurada o una **gateway** fuera de la subred impiden el acceso a otros equipos o a Internet.

**Checklist de planificación básica**

- [ ] Definir **subred** y **máscara**.
- [ ] Asignar **puerta de enlace**.
- [ ] Reservar IPs para **servidores** y **dispositivos críticos**.
- [ ] Documentar **rango dinámico** y **exclusiones**.
- [ ] Verificar **conectividad** entre segmentos.

**Parámetros base (ejemplos)**

Elemento | Descripción | Ejemplo
---|---|---
Dirección IP | Identificador único del host | 192.168.1.100
Máscara de red | Parte red/host | 255.255.255.0 (/**24**)
Puerta de enlace | Salida a otras redes | 192.168.1.1
IP privada | No enrutable en Internet | 10.0.0.5
Asignación | Fija vs. dinámica | Servidor fija; PC por _DHCP_

---

## DHCP (Dynamic Host Configuration Protocol)
**Resumen:** _DHCP_ automatiza la **asignación** de IP, **máscara**, **gateway** y **DNS** a clientes, reduciendo errores y facilitando cambios.

_DHCP_ es un protocolo de configuración dinámica de host que **automatiza** la asignación de parámetros IP a los **clientes** de una red. En lugar de configurar manualmente **IP**, **máscara**, **puerta de enlace** y **DNS** en cada equipo, un **servidor DHCP** los distribuye bajo demanda. Esto reduce **errores de configuración** y facilita **mover equipos** entre redes. Por ejemplo, en una empresa pequeña, el servidor DHCP puede asignar automáticamente direcciones en el rango **192.168.1.100–192.168.1.200** con su máscara y gateway correspondientes a cada portátil que se conecte, evitando tener que configurarlos a mano.

> **Nota:** Además de IP, DHCP puede entregar **DNS**, **dominio de búsqueda**, **WINS** (legado) u **opciones de arranque** para _PXE_.

**Qué puede entregar DHCP**

Parámetro | Descripción | Ejemplo
---|---|---
IP | Dirección del cliente | 192.168.1.123
Máscara | Segmentación | 255.255.255.0
Gateway | Ruta por defecto | 192.168.1.1
DNS | Resolución de nombres | 192.168.1.1 y 8.8.8.8
Dominio | Sufijo de búsqueda | empresa.local

---

## Rangos, exclusiones, concesiones y reservas
**Resumen:** Un **ámbito** define rangos; las **exclusiones** apartan IPs; las **concesiones** asignan temporalmente; las **reservas** fijan IPs por **MAC**.

Un servidor DHCP trabaja sobre uno o varios **rangos** o **ámbitos** de direcciones IP definidas (también llamados **pools**). Por ejemplo, se puede configurar un ámbito que abarque desde **192.168.1.100** hasta **192.168.1.200** para los clientes. Dentro de ese rango se pueden establecer **exclusiones**, que son direcciones dentro del ámbito que no se entregarán porque quizás estén asignadas de forma fija a servidores u otros equipos (por ejemplo, excluir la .100 si la usa una impresora con IP fija). Cuando un cliente obtiene una IP, el servidor crea una **concesión** (_lease_ en inglés): la IP se “alquila” al cliente por un tiempo determinado (por ejemplo, 24 horas). La concesión incluye la dirección otorgada, la **MAC** del cliente y el **tiempo de expiración**. Si el cliente sigue activo, intentará **renovar** la concesión antes de que expire para conservar la misma IP. Si se apaga o desconecta, el servidor podrá **recuperar** esa IP una vez vencido el plazo para asignársela a otro. Las **reservas** son asignaciones fijas dentro del DHCP: se configura que a una determinada MAC o cliente siempre se le entregue la misma IP específica. Esto es útil para dispositivos que requieren **IP estable** (como un servidor **NAS** o una impresora de red) pero se quiere gestionar vía DHCP en vez de configurarlos manualmente. Las reservas evitan **conflictos** y facilitan la **administración centralizada** de direcciones. En la práctica, un administrador de sistemas junior en una PYME creará reservas para servidores y dispositivos importantes, y dejará el resto del rango para concesiones dinámicas de los PCs de oficina.

> **Ejemplo:** Reserva 192.168.1.50 para la **MAC** 00:11:22:33:44:55 (impresora); excluye 192.168.1.2–.49 (servidores).

**Conceptos clave de ámbito**

Término | Qué es | Ejemplo | Beneficio
---|---|---|---
Ámbito (pool) | Rango servible | 192.168.1.100–.200 | Control de reparto
Exclusión | IPs no entregables | .2–.50 | Evitar choques con fijas
Concesión | “Alquiler” temporal | 24 h | Reciclado automático
Reserva | IP fija por MAC | MAC → .50 | Estabilidad y control

---

## Funcionamiento del protocolo DHCP
**Resumen:** El ciclo **DORA** coordina **descubrimiento**, **oferta**, **solicitud** y **confirmación** para asignar una IP sin duplicados.

El proceso DHCP sigue un esquema cliente-servidor de **cuatro fases** conocido por el acrónimo **DORA**. Cuando un equipo cliente se conecta a la red y necesita configuración, envía un **DHCP Discover** (descubrimiento DHCP) en **broadcast** para encontrar servidores DHCP disponibles. Uno o varios servidores DHCP responden con un **DHCP Offer** (oferta), que propone una **configuración** (una IP disponible dentro de su rango y otros datos, como máscara, gateway, DNS, etc.). El cliente recibe uno o más offers y elige uno (normalmente solo hay un servidor en redes pequeñas). Luego el cliente emite un **DHCP Request** (solicitud) también en broadcast, indicando qué oferta acepta (incluye la IP elegida y la identificación del servidor que la ofreció). Finalmente, el servidor correspondiente confirma la concesión enviando un **DHCP ACK** (_acknowledgement_, reconocimiento) al cliente con la configuración definitiva. A partir de entonces, el cliente configura su **interfaz** con esos parámetros y puede usar la red. El servidor registra la **concesión** con su tiempo de arrendamiento. Este ciclo garantiza que no haya **duplicados de IP** y que la configuración llegue correctamente al nuevo dispositivo. Además de estos mensajes principales, existen otros como **DHCP NAK** (negativa, si el servidor rechaza una solicitud, por ejemplo por expiración) o **DHCP Release** (cuando el cliente libera voluntariamente la IP, por ejemplo al apagarse). En entornos profesionales, el administrador debe asegurarse de que solo haya **un servidor DHCP** activo por segmento de red para evitar conflictos de **ofertas**, y configurar adecuadamente opciones como el **tiempo de arrendamiento** (por ejemplo, arrendamientos más cortos en redes con muchos cambios frecuentes, o más largos en redes estables). Una ventaja práctica de DHCP es que también distribuye otros parámetros: además de IP, máscara y puerta de enlace, suele proporcionar automáticamente la dirección de servidores **DNS**, el nombre de **dominio** de la empresa e incluso **opciones específicas** (como servidor **WINS** para redes Windows antiguas, o opciones de **boot** para clientes _PXE_ en despliegue de sistemas). Esto simplifica enormemente la **configuración** de puestos de trabajo en oficinas y aulas, reduciendo **errores típicos** (como teclear mal la IP o DNS) y facilitando el **soporte técnico**.

1. **Discover**  
   El cliente anuncia en broadcast que necesita configuración.
2. **Offer**  
   El servidor propone IP y parámetros.
3. **Request**  
   El cliente solicita formalmente la oferta elegida.
4. **ACK**  
   El servidor confirma y activa la concesión.

> **Advertencia:** Dos **servidores DHCP** en el mismo segmento pueden causar **IPs duplicadas** o parámetros incoherentes.

**Mensajes DORA (síntesis)**

Mensaje | Emisor | Transporte | Finalidad
---|---|---|---
Discover | Cliente | Broadcast | Búsqueda de servidor
Offer | Servidor | Unicast/Broadcast | Propuesta de configuración
Request | Cliente | Broadcast | Aceptación de oferta
ACK | Servidor | Unicast/Broadcast | Confirmación de concesión

---

## Tipos de mensajes DHCP
**Resumen:** Además de **DORA**, existen **NAK**, **Release** e **Inform**, útiles para **errores**, **liberación** y **petición** de datos adicionales.

Como se ha descrito, los mensajes fundamentales son **Discover**, **Offer**, **Request** y **ACK**, que conforman el flujo básico de asignación. **Discover**, **Request** y algunos **ACK** se envían en **broadcast** porque al principio el cliente no conoce ningún servidor ni su propia IP. El **Offer** normalmente lo envía el servidor en **unicast** al cliente (si lo puede identificar) o a través de broadcast si no conoce aún su dirección. Además de estos, DHCP incluye mensajes para gestionar la concesión: **DHCP Release** (liberación de IP, que envía el cliente al apagar o desconectar para avisar que ya no usará esa IP) y **DHCP Inform** (usado a veces cuando el cliente solo quiere solicitar información adicional de configuración sin cambiar su IP). También existe **DHCP NAK**, que un servidor puede enviar si un cliente solicita una IP **inválida** o cuya concesión ya **expiró**, indicando que debe reiniciar el proceso. Internamente, los mensajes DHCP contienen campos como la dirección **MAC** del cliente (clave para identificarlo) y un **ID de transacción** para emparejar solicitudes con respuestas. Un técnico de redes debe entender estos tipos de mensajes para **diagnosticar problemas**: por ejemplo, si un cliente no obtiene IP, conviene ver en un **analizador de protocolos** (como **Wireshark**) si está haciendo **Discover** y recibiendo **Offer**; si no hay Offer, es que el servidor no responde (posible **fallo del servicio DHCP** o **bloqueo por cortafuegos**), y si hay Offer pero no ACK, quizá **otro servidor** interfirió. Comprender la secuencia **DORA** y los mensajes ayuda a resolver incidencias comunes, como **“IP address conflict”** (cuando un equipo tiene IP duplicada por haber dos servidores DHCP activos, por ejemplo).

> **Nota:** Verifica filtros del **cortafuegos** y la **VLAN** correcta al analizar capturas.

**Mensajes y usos**

Mensaje | Uso principal | Cuándo aparece
---|---|---
NAK | Rechazo/invalidación | Solicitud no válida o expirada
Release | Liberar IP | Apagado o desconexión del cliente
Inform | Pedir info extra | Cliente con IP manual necesita DNS/Dominio

---

## Clientes DHCP en sistemas operativos libres y propietarios

### Instalación
**Resumen:** Los **clientes DHCP** vienen **integrados** en _Windows_, _Linux_ y _macOS_; en sistemas mínimos puede requerir **paquete** específico.

En la mayoría de sistemas operativos tanto **libres** (_Linux_, _BSD_, etc.) como **propietarios** (_Windows_, _macOS_), el **cliente DHCP** viene integrado de serie, ya que es esencial para la **conectividad** en redes modernas. En **Windows**, por ejemplo, el servicio **Cliente DHCP** se ejecuta automáticamente para obtener la configuración de la red si la interfaz está en modo “Obtener una dirección IP automáticamente”. En distribuciones **Linux** de escritorio, servicios como **NetworkManager** utilizan internamente un cliente DHCP (como **dhclient** o **systemd-networkd**) para configurar la red. En un sistema Linux **minimalista** o **embebido**, es posible que se necesite instalar manualmente un paquete de cliente DHCP (por ejemplo, **isc-dhcp-client** en **Debian/Ubuntu**) si no estuviera presente. No obstante, en entornos estándar de **PYME** o **educativos**, prácticamente todos los PCs y dispositivos de usuario ya traen un cliente DHCP listo para usar, por lo que no suele requerir **instalación manual**, solo asegurarse de que está **habilitado**.

> **Nota:** En Linux mínimos, confirma la presencia de **dhclient** o **systemd-networkd** antes de diagnosticar.

**Checklist de comprobación**

- [ ] Servicio de **cliente DHCP** activo.
- [ ] Interfaz en modo **automático** (IPv4/IPv6).
- [ ] **Logs** accesibles para diagnóstico.
- [ ] **Cortafuegos** local sin bloquear DHCP.

**Tabla de referencia**

Sistema | Cliente habitual | Gestión
---|---|---
Windows | Cliente DHCP (servicio) | Panel de red
Linux | dhclient / systemd-networkd | NetworkManager/CLI
macOS | _configd_ (gestión de red) | Preferencias de red

### Configuración de interfaces de red para que obtengan su configuración por DHCP
**Resumen:** En **Windows** activa “automático”; en **Linux** define **DHCP** en el archivo o herramienta correspondiente y **verifica** con **comandos**.

Configurar una **interfaz** para usar DHCP es normalmente una tarea sencilla. En **Windows**, se accede a las **propiedades de TCP/IP** de la tarjeta de red y se selecciona “Obtener una dirección IP automáticamente” y “Obtener la dirección del servidor DNS automáticamente”. Esto hace que Windows use DHCP para la **IPv4** (y similar para **IPv6** si se configura). En un entorno **Windows Server** o cliente, esta es la **configuración predeterminada** a menos que se especifique una IP fija. En sistemas **Linux**, la configuración depende de la **distribución** y las herramientas: en muchas distribuciones modernas se usa un archivo de configuración (por ejemplo, en **/etc/network/interfaces** en Debian clásico, se pondría iface eth0 inet dhcp para que la interfaz eth0 use DHCP; en **Red Hat/CentOS**, en ifcfg-eth0 se pondría BOOTPROTO=dhcp; con **Netplan** en Ubuntu actual se indicaría dhcp4: true en la config **YAML**, etc.). Si se utiliza **entorno gráfico**, basta con editar la conexión de red y elegir “Automático (DHCP)”. Una vez habilitada la opción, al conectar el cable o activar la **Wi-Fi**, el cliente DHCP del sistema negociará con el servidor para obtener los parámetros. Desde la perspectiva de un **técnico de soporte**, es importante verificar que el equipo efectivamente recibe una **IP correcta**: en Windows, el **Comando:** ipconfig /all mostrará si la dirección está asignada por DHCP y cuál es el servidor DHCP; en Linux, **Comando:** ip addr o **Comando:** ifconfig mostrarán la IP obtenida, y suele haber **registros** en **/var/log/syslog** sobre la concesión recibida (cliente DHCP indicando la IP y otros datos). Un error común es que un equipo obtenga una **dirección APIPA** (por ejemplo **169.254.x.x**) que Windows asigna automáticamente si no consiguió un DHCP; esto indica que no llegó **respuesta** del servidor, posiblemente por un problema de **cable**, de **alcance Wi-Fi**, de **configuración del switch** (**VLAN** incorrecta) o porque el **servidor DHCP** está caído. La buena práctica es siempre que sea posible **usar DHCP** para los equipos cliente, ya que facilita **cambiar de red** sin reconfigurar y permite al administrador **centralizar** la gestión IP. Solo en casos necesarios (**servidores**, dispositivos que requieren **IP fija** fuera del rango DHCP, o cuando no hay servidor DHCP) se **configuran IP manualmente**. Incluso para servidores, muchas empresas optan por **reservas DHCP** como se comentó antes, para mantener todo **controlado** desde el servidor central.

> **Ejemplo:** Si ves **169.254.0.0/16** en ipconfig, sospecha de **falta de respuesta** DHCP.

**Verificaciones rápidas**

Paso | Acción | Objetivo
---|---|---
1 | **Comando:** ipconfig /all | Confirmar asignación por DHCP
2 | **Comando:** ip addr | Ver IP en Linux
3 | Revisar **/var/log/syslog** | Ver DORA y concesión
4 | Comprobar **VLAN** y **Wi-Fi** | Alcance correcto

---

## Servidores DHCP en sistemas operativos libres y propietarios

### Instalación
**Resumen:** En **Linux** destaca **ISC DHCP**, **dnsmasq** o **KEA**; en **Windows Server**, el **rol DHCP** se añade desde el **Administrador de servidores**.

Un **servidor DHCP** puede instalarse tanto en plataformas **Linux** (software libre) como en **Windows Server** (software propietario). En Linux, el servidor DHCP más común es **ISC DHCP** (_Internet Systems Consortium DHCP Server_), que se instala típicamente con el paquete **isc-dhcp-server**. Otras alternativas actuales incluyen **dnsmasq** (ligero, útil en pequeños routers o para laboratorio) o **KEA DHCP** (también de ISC, más moderno). En **Windows**, el servicio DHCP está disponible en las ediciones de **Windows Server** (por ejemplo, 2016, 2019…) como un **rol** que se agrega mediante el **administrador de servidores**. Al instalarlo, Windows proporciona una **consola de administración DHCP** para configurar los **ámbitos**. En pequeñas oficinas sin un Windows Server dedicado, a veces el propio **router del ISP** actúa como servidor DHCP; esos routers suelen tener firmware propietario (tipo **MikroTik**, **Cisco/Linksys**, etc.) que incluyen DHCP básico preconfigurado. Un administrador de sistemas junior debe saber **identificar** dónde reside el servidor DHCP en la red: en una empresa con **dominio Active Directory** seguramente será el **controlador de dominio Windows Server**; en una oficina más pequeña sin servidor, probablemente el **router ADSL/fibra** del operador desempeña esa función. Si se opta por un servidor Linux, la instalación implica asegurarse de que el servicio **arranca al inicio** y tiene permisos para **escuchar** en la interfaz correcta (por ejemplo, en Ubuntu **editar** **/etc/default/isc-dhcp-server** para indicar la **interfaz**).

> **Nota:** En equipos SOHO, confirma si el **router** ya entrega DHCP para evitar **servidores duplicados**.

**Plataforma y paquete**

Plataforma | Paquete/Rol | Dónde se gestiona
---|---|---
Linux | isc-dhcp-server, dnsmasq, KEA | Archivos y systemd
Windows Server | Rol DHCP | Consola DHCP
Router ISP | DHCP integrado | Web del equipo

### Arranque
**Resumen:** Asegura **servicio activo** al **inicio**, **autorización** en _AD_ en Windows y **puerto UDP 67** abierto en **cortafuegos**.

Una vez instalado, el servicio DHCP debe **iniciarse** y mantenerse en ejecución. En **Linux**, tras la instalación suele habilitarse automáticamente el servicio **dhcpd** (daemon DHCP) para iniciar al arrancar el sistema. Se puede controlar con comandos estándar: **Comando:** systemctl start isc-dhcp-server para arrancarlo, **Comando:** systemctl enable isc-dhcp-server para que inicie en cada arranque (en sistemas SysV antiguas, era con service dhcpd start y configurando runlevels). En **Windows Server**, después de agregar el rol DHCP, el servicio se inicia automáticamente y también arranca con el sistema; se puede verificar en la lista de servicios (**services.msc**) que el **Servicio de servidor DHCP** esté en estado "En ejecución". En Windows es importante, tras instalar, **autorizar** el servidor DHCP en **Active Directory** si es un servidor miembro del dominio – esto previene que **servidores DHCP no autorizados** operen en la red (una medida de seguridad en entornos **AD**). Un servidor no autorizado no entregará direcciones a menos que se autorice en AD. El técnico debe comprobar este paso, ya que es un **olvido común** al instalar DHCP en Windows Server por primera vez. Además, se debe tener en cuenta el **firewall**: en Linux, **abrir el puerto UDP 67** en el servidor (puerto de servicio DHCP) si está activado **UFW**, **firewalld** o similar; en Windows, la función de DHCP suele crear automáticamente **reglas** en el **Firewall de Windows**, pero es bueno confirmarlo.

> **Advertencia:** Sin **autorización en AD**, un servidor DHCP Windows **no servirá** direcciones en un dominio.

**Checklist de puesta en marcha**

- [ ] **Servicio** DHCP en ejecución.
- [ ] **Habilitado** al arranque (enable).
- [ ] **Autorizado** en **AD** (Windows).
- [ ] **UDP 67** permitido en firewall.
- [ ] Interfaz **correcta** seleccionada.

### Ficheros y parámetros de configuración básica
**Resumen:** Define **subred**, **rango**, **gateway**, **DNS** y **tiempos**; en Linux edita **/etc/dhcp/dhcpd.conf**, en Windows usa la **consola**.

En **Linux**, la configuración del servidor DHCP se realiza **editando** su fichero principal (en **ISC DHCP**, el archivo **/etc/dhcp/dhcpd.conf**). En este archivo se definen los **parámetros básicos**: las redes o **subredes** que servirá, con su **rango** de IPs disponibles, la **máscara**, **gateway**, **DNS**, **duración** de los _leases_, etc. Por ejemplo, una sección típica podría ser:

```apacheconf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.1, 8.8.8.8;
    default-lease-time 86400;
    max-lease-time 172800;
}
```

Esto define un **ámbito** para la red **192.168.1.0/24** con IPs .100–.200 y especifica que la **puerta de enlace** y **DNS** primario es **192.168.1.1** (quizá el propio router o servidor) y DNS secundario **8.8.8.8**, con _lease_ por defecto de **1 día**. También se pueden incluir más **option** según necesidades (por ejemplo, **option domain-name "empresa.local";** para definir el **dominio** de búsqueda DNS). En **Windows Server**, la configuración se realiza mediante la **consola gráfica**: se crea una nueva **ámbito (scope)** indicando la red (ej: **192.168.1.0/24**), el rango inicial-final, la **puerta de enlace** (router option), los **DNS**, y el **lease time**. La consola permite agregar **exclusiones** fácilmente (por ejemplo excluir **.1 a .50** si esas son IP fijas de servidores). Los parámetros básicos a configurar son esencialmente los **mismos** en ambos sistemas. Un detalle: en Windows, el servidor DHCP puede **detectar automáticamente** si la subred del servidor es, por ejemplo, **192.168.1.0/24** y sugerir esos datos. En cambio en Linux, hay que ser **explícito** en el archivo. En ambos casos, tras **modificar** la configuración, se debe **reiniciar** el servicio para aplicar cambios (en Linux **Comando:** systemctl restart isc-dhcp-server, en Windows **reiniciar servicio** o usar la opción de **refrescar** en la consola). Es buena práctica **documentar** la configuración DHCP, especialmente cualquier **reserva** o **exclusión**, para que el equipo de soporte conozca qué direcciones deben **asignarse manualmente** si es el caso. También conviene **mantener** los servidores DHCP **actualizados**, ya que ha habido **vulnerabilidades** en el pasado (un servidor DHCP comprometido podría entregar configuración maliciosa, como **gateway erróneo** o **DNS falseados**).

> **Nota:** Tras cambios en **dhcpd.conf**, recuerda **reiniciar** el servicio para aplicar la nueva configuración.

**Parámetros esenciales**

Parámetro | Propósito | Ejemplo
---|---|---
Subred | Ámbito de servicio | 192.168.1.0/24
Rango | IPs dinámicas | 192.168.1.100–.200
Router | Puerta de enlace | 192.168.1.1
DNS | Resolución | 192.168.1.1, 8.8.8.8
Leases | Duración | 86400 s (24 h)

### Información sobre concesiones (lease)
**Resumen:** El servidor mantiene **registros** de **concesiones**; en Linux en **/var/lib/dhcp/dhcpd.leases**, en Windows en la **consola**; ajusta **tiempos** a tu entorno.

El **servidor DHCP** lleva un registro de las **concesiones** activas y expiradas. En **Linux/ISC DHCP**, existe un fichero de arrendamientos (**leases**) normalmente en **/var/lib/dhcp/dhcpd.leases**, donde el demonio va anotando cada concesión otorgada. Ahí se puede **consultar** qué IP tiene cada **MAC**, desde cuándo hasta cuándo. Sin embargo, es más común usar **comandos** o **revisar logs** para ver esta info de forma legible. Por ejemplo, en ISC DHCP se pueden habilitar **logs detallados** en **/var/log/syslog** que indiquen “DHCPACK on 192.168.1.101 to 00:11:22:33:44:55 (hostname)” cada vez que asigne una IP. En **Windows Server**, la **consola DHCP** muestra en tiempo real la lista de **Direcciones IP en arrendamiento** bajo cada ámbito: allí se ve la **IP**, el **nombre de host** del cliente si lo informó, su **MAC** (llamada **ID de hardware**) y el **tiempo de expiración**. Un administrador junior debe saber **verificar** esta lista para, por ejemplo, **localizar** la IP de un PC en particular a partir de su nombre o viceversa, o para **diagnosticar** por qué un dispositivo no obtiene IP (quizá ya tiene pero está en **conflicto**). También desde la consola se pueden **liberar concesiones** (en Windows con clic derecho → **quitar arrendamiento**, útil si un cliente se quedó con IP "atascada" y queremos reciclarla manualmente) o **crear reservas** (asignar una IP fija a una MAC determinada desde el servidor). En **Linux**, para agregar una reserva se **edita** el **dhcpd.conf** añadiendo un host **fixed-address** para esa MAC. En cuanto a la **expiración**, cuando un lease expira sin renovación, la IP vuelve al **pool** de disponibles. Es importante **ajustar** el **tiempo de concesión** a las necesidades: en redes de **aulas** donde cada día se conectan equipos distintos (p.ej. portátiles de estudiantes), un lease **corto** (unas horas) permite reciclar direcciones rápidamente. En una **oficina** con PCs fijos, leases **largos** (días o semanas) reducen el **tráfico DHCP** y garantizan que normalmente cada PC conserve su IP usual (lo que facilita usarlo para **identificación** en algunos sistemas). En cualquier caso, el archivo o registro de concesiones es una herramienta de **auditoría**: permite saber qué dispositivos estuvieron conectados y cuándo. Por motivos de **ciberseguridad** básica, es recomendable **almacenar** o **exportar** periódicamente esta información, ya que puede ayudar a **rastrear** si un equipo no autorizado se conectó a la red (veríamos una MAC desconocida obteniendo IP a cierta hora). También conviene **proteger** el servidor DHCP: en Windows mediante **autenticación en AD** como se mencionó, y en general asegurando que atacantes no puedan introducir un **rogue DHCP** (uno falso) en la red, lo cual un técnico de redes vigilará **segmentando** la red o usando funciones de seguridad en switches (por ejemplo, **DHCP snooping** en switches gestionables, que permite filtrar respuestas DHCP solo desde el puerto **autorizado** donde está el servidor legítimo). En resumen, la **gestión de concesiones DHCP** es una tarea **rutinaria** del administrador de red junior, tanto para **mantenimiento** (limpieza de leases antiguos, configuración de reservas) como para **resolución de incidencias**.

> **Advertencia:** Habilita **DHCP snooping** en switches gestionables para bloquear **servidores DHCP falsos**.

**Operaciones habituales (resumen)**

Acción | Dónde | Resultado
---|---|---
Ver leases | Consola DHCP / fichero leases | IP ↔ MAC y expiración
Liberar lease | Consola Windows (quitar) | Reciclar IP “atascada”
Crear reserva | dhcpd.conf / consola | IP fija por MAC
Ajustar tiempos | Parámetros de lease | Tráfico vs. estabilidad
Auditar | Exportes/Logs | Trazabilidad de accesos
