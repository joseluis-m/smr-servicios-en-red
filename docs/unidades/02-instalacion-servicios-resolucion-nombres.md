# 2. Instalación de servicios de resolución de nombres

## Sistemas de nombres planos y jerárquicos
**Resumen:** La **resolución de nombres** traduce nombres legibles a **direcciones IP**. Existen **sistemas planos** y **jerárquicos** con implicaciones de **escalabilidad** y **administración**.

En las redes informáticas, la resolución de nombres permite traducir nombres fáciles de recordar (por ejemplo, servidorventas) a direcciones IP numéricas. Hay dos enfoques básicos: sistemas de nombres planos y jerárquicos. Un sistema de nombres plano es aquel en el que los nombres no tienen estructura de niveles; todos los nombres están en un único espacio donde deben ser únicos y normalmente se resuelven mediante un **archivo** o una **tabla central**. Un ejemplo clásico es el **archivo hosts** (presente en todos los sistemas operativos), donde se puede mapear manualmente nombres a IPs de forma fija. Por ejemplo, una escuela pequeña podría mantener en cada PC una entrada en **C:\Windows\System32\drivers\etc\hosts** o **/etc/hosts** con “192.168.10.5 servidorarchivos” para que ese nombre apunte a la IP indicada. Esto es un esquema plano: cada equipo tiene su lista, y los nombres no tienen relación jerárquica entre sí. Este método funciona en entornos **muy reducidos**, pero no **escala bien** (hay que actualizar el archivo en cada equipo cuando cambia algo). Otros ejemplos de nombres planos serían **NetBIOS/WINS** en redes Windows antiguas: cada PC tenía un nombre NetBIOS único en la red local, resuelto por **difusión** o un **servidor WINS** central, sin dominios jerárquicos.

En cambio, un sistema de nombres **jerárquico** organiza los nombres en varios niveles, formando una **estructura de árbol**. El ejemplo universal es el **DNS (Domain Name System)** público de Internet, que es jerárquico: los nombres de dominio están estructurados por niveles separados por puntos (por ejemplo, www.empresa.com tiene “www” como subnombre, dentro del dominio “empresa.com”, que a su vez está bajo “.com”). En sistemas jerárquicos, se pueden **delegar** distintas porciones del **espacio de nombres** a diferentes servidores, lo que permite **escalar** a redes muy grandes. En un entorno corporativo, también se suelen usar nombres jerárquicos: por ejemplo **archivo.sede1.midominio.local**, **archivo.sede2.midominio.local**, etc., donde cada sede tiene subdominios. La ventaja es que evita **colisiones** y facilita **búsquedas ordenadas**. Resumiendo, los nombres planos son **simples** pero **limitados** a redes pequeñas con pocos nombres, mientras que los jerárquicos permiten **gestionar** miles o millones de nombres distribuidos en diferentes niveles. Un técnico en redes debe comprender esto, porque al diseñar la **nomenclatura** de hosts de una empresa conviene definir una **estructura** (por ejemplo, usar un **subdominio** por delegación o separar por departamento: **ventas.empresa.lan**, **taller.empresa.lan**, etc.) en lugar de tener una lista plana enorme de nombres.

> **Ejemplo:** Para un aula, entradas en **/etc/hosts** sirven; para un campus con varias sedes, conviene **DNS jerárquico**.

**Comparativa breve**

Elemento | Plano | Jerárquico
---|---|---
Gestión | Local en **hosts** | **DNS** distribuido
Escalabilidad | Baja | Alta
Cambios | Manuales por equipo | **Delegación** y **zonas**
Riesgo | **Inconsistencias** | **Autoridad** clara

---

## Espacio de nombres de dominio
**Resumen:** El **espacio de nombres** en **DNS** es **global**, **único** y **jerárquico** desde la **raíz** hasta **subdominios**, aplicable a Internet y a entornos **privados**.

El espacio de nombres de dominio se refiere al conjunto estructurado de todos los nombres posibles en un sistema jerárquico como DNS. En DNS, el espacio de nombres es **global**: abarca desde la **raíz** (un nivel conceptual vacío representado por el **punto final**) hacia abajo por múltiples niveles (**TLDs**, dominios, subdominios...). Por ejemplo, consideremos el dominio completo **www.instituto.edu.es.** – este pertenece a un espacio de nombres donde el nivel raíz “.” contiene el subespacio “**es**” (código de España), dentro de “es” está “**edu**”, y dentro de “edu.es” está “**instituto**”, y dentro de ese dominio hay un host llamado “**www**”. Cada **punto** define una **división** en la jerarquía. El espacio de nombres es **único a nivel mundial** para DNS público: no pueden existir dos dominios iguales con la misma ascendencia. Esto lo garantiza la forma en que se **delega** y **registra** los nombres de dominio. En entornos **privados** (como un dominio Windows AD “**empresa.local**”), el espacio de nombres de esa red es **separado** del público, pero aun así mantiene la jerarquía. Entender el concepto de espacio de nombres ayuda al administrador a **diseñar nombres coherentes** y a saber, por ejemplo, por qué dos equipos no pueden tener el mismo nombre DNS en la misma **zona**, o cómo un mismo nombre puede existir en espacios distintos sin conflicto (ej: **archivo.sede1** en el espacio **sede1.empresa.local** no interfiere con **archivo.sede2** en **sede2.empresa.local**). También implica considerar **convenciones**: a nivel de Internet, el espacio se divide en dominios de primer nivel (**.com**, **.org**, **.es**, etc.), luego dominios de segundo nivel (p. ej. **empresa.com**), etc. El administrador de sistemas debe decidir si su red usará un **subdominio** del dominio público de la empresa o un **dominio privado**. Por ejemplo, una PYME con dominio Internet **miempresa.com** podría usar internamente **miempresa.local** o **intranet.miempresa.com**. Usar un espacio de nombres de dominio bien planificado **evita confusiones** y **colisiones** de nombres.

> **Nota:** Evita reutilizar un dominio público que no posees para el **entorno interno**.

**Niveles típicos**

Elemento | Descripción | Ejemplo
---|---|---
Raíz | Origen del árbol | .
TLD | Dominio de primer nivel | **.es**, **.com**
Dominio | Segundo nivel | **empresa.com**
Subdominio | Delegación interna | **ventas.empresa.com**

---

## Dominios genéricos
**Resumen:** Los **gTLD** como **.com**, **.org** o **.net** no están ligados a países; su **uso** y **propósito histórico** orientan **interpretación** y **configuración**.

En el contexto DNS, dominios genéricos suele referirse a los **TLD genéricos (gTLD)** tradicionales, como **.com**, **.org**, **.net**, **.edu**, **.gov**, etc., que no están asociados a un país específico (a diferencia de los **ccTLD** como **.es**, **.fr**, **.mx**). Históricamente, cada gTLD tenía un **propósito**: .com (comercial), .org (organizaciones sin lucro), .net (redes, ISP), .edu (educación, principalmente EE.UU.), .gov (gobierno EE.UU.), .mil (militar EE.UU.), aunque hoy .com y .net se usan de forma global. Además, desde mediados de la década de 2010 se **ampliaron** los gTLD con muchos nuevos (por ejemplo, **.tech**, **.shop**, **.madrid**, etc.). Un técnico de grado medio debe **reconocer** estas categorías: por ejemplo, saber que un dominio .com suele indicar una **empresa**, que un .org tal vez sea una **ONG**, y que .edu y **.gob.es** pueden indicar instituciones **educativas** o **gubernamentales**. En redes locales, el término “dominio genérico” podría no usarse, pero es importante conocerlo para entender la **estructura de Internet**. También implica entender que al crear **nombres internos** no debemos usar dominios genéricos sin **registro**. Lo apropiado es usar un **dominio que se posee**, o uno **reservado**. En resumen, los dominios genéricos definen **categorías globales** en el espacio DNS y conviene conocerlos para administración de servicios.

> **Advertencia:** No configures internamente **empresa.com** si no posees el dominio; evita **conflictos** con el DNS público.

**Referencias rápidas**

gTLD | Uso histórico | Observación
---|---|---
.com | Comercial | Uso global
.org | Sin lucro | Proyectos/ONG
.net | Redes/ISP | Uso generalizado
.edu | Educación (EE. UU.) | Restricciones país
.gov | Gobierno (EE. UU.) | Restricciones país

---

## Delegación
**Resumen:** La **delegación DNS** transfiere **autoridad** de una **zona hija** mediante **registros NS** (y **glue**), habilitando un **sistema distribuido** y **escalable**.

La delegación DNS es el mecanismo por el cual se asigna la **responsabilidad** de una porción del **espacio de nombres** a otro **servidor de nombres**. Gracias a la delegación, DNS es **distribuido** y **escalable**. Por ejemplo, el **dominio raíz** delega la gestión de **.com**; a su vez, .com delega **empresa.com** al servidor de la entidad que registró ese dominio. En una delegación, en la **zona padre** se crean **registros NS** apuntando a los servidores de la zona hija. A nivel interno, un administrador también utiliza la delegación: imaginemos una universidad con un dominio principal **univ.edu.es**, y cada facultad gestionada de forma autónoma con su **subdominio** (ej: **fisica.univ.edu.es**, **artes.univ.edu.es**). El administrador de la zona padre **crea delegaciones** para cada subdominio, apuntando a los **DNS** servidores que llevan esas facultades. Así cada unidad puede administrar sus nombres de máquinas sin sobrecargar al DNS central. Para implementar delegación se configuran registros **NS** en la zona padre, y asegurar que la zona hija **existe** en los servidores designados (además se suele añadir un **glue A** si el servidor delegado está dentro de la misma zona). Delegar no es **redirigir**, sino **transferir autoridad**. En entornos **Windows con Active Directory**, se usa delegación cuando hay **dominios hijos** o para separar carga. También es común delegar un **subdominio público** a **DNS internos** corporativos (ej: **corp.empresa.com**). Errores de delegación pueden causar que los **nombres no se resuelvan**. Se usan **nslookup** o **dig** para verificar delegaciones.

> **Nota:** Verifica **coherencia** entre lo que dice la **zona padre** y lo que sirve la **zona hija**.

**Checklist de delegación**

- [ ] Crear **NS** en la **zona padre**.
- [ ] Publicar **glue** si procede.
- [ ] Servir **zona hija** en DNS designados.
- [ ] Probar con **Comando:** dig sub.zona NS.
- [ ] Revisar **serial** y **autoridad**.

---

## Funcionamiento DNS (Domain Name Service)
**Resumen:** El **cliente** consulta a un **resolver**; si no hay **caché**, se recorre la **cadena de delegación** hasta un **autoritativo**, con **TTL** y **caché**.

El DNS funciona mediante consultas de resolución de nombres que pueden ser **iterativas** o **recursivas**, y maneja también la **resolución inversa**. Cuando un cliente necesita resolver un nombre (ej. **www.ejemplo.com**), típicamente usa un **resolver** configurado con la dirección de uno o más **servidores DNS** al cual le hace la **petición recursiva**. Si ese servidor no la sabe en **caché**, hará una serie de consultas **iterativas**: primero preguntará a un **servidor raíz**; este remitirá a un **TLD** (.com), que remitirá al **autoritativo** del dominio (**ejemplo.com**), que responderá con el **registro** solicitado (p. ej., **A 93.184.216.34**). El **resolver** cachea las respuestas según el **TTL** para acelerar futuras consultas.

> **Ejemplo:** Un resolver corporativo sin caché pregunta a **raíz → .com → autoritativo de ejemplo.com** y guarda la respuesta por el **TTL**.

**Flujo básico**

Elemento | Función | Observación
---|---|---
Cliente | Solicita nombre | Pregunta al **resolver**
Resolver | Hace recursión | **Caché** por TTL
Raíz/TLD | Iterativo | Devuelve **referencias**
Autoritativo | Respuesta final | Registros **A/AAAA/MX**…

---

## Consultas DNS
**Resumen:** Una **consulta** incluye **nombre**, **tipo** y **clase**; usa **UDP/TCP 53** y devuelve **códigos** como **NOERROR**, **NXDOMAIN** o **SERVFAIL**.

Son las peticiones que pueden solicitar diferentes **tipos de registros** (**A**, **AAAA**, **MX**, **TXT**, etc.). Una consulta tiene **nombre**, **tipo** y **clase** (**IN**). Se envían vía **UDP 53** (habitual) o **TCP 53** si la respuesta es grande o para **transferencias de zona**. Un administrador puede efectuar consultas manuales usando herramientas: en **Windows**, **nslookup**; en **Linux**, **dig** o **host**. Por ejemplo, **Comando:** nslookup www.google.com 8.8.8.8. La respuesta DNS contiene un **código de estado** (**NOERROR**, **NXDOMAIN**, **SERVFAIL**) y puede incluir **alias (CNAME)**. Un técnico de soporte debe probar a distintos **servidores** para aislar fallos (DNS local vs externos).

> **Advertencia:** Si una zona responde **NXDOMAIN**, comprueba **ortografía** y **autoridad** del servidor consultado.

**Tipos de consulta**

Parámetro | Descripción | Ejemplo
---|---|---
Tipo | Registro buscado | **A**, **MX**, **AAAA**
Clase | Normalmente **IN** | Internet
Transporte | **UDP/TCP 53** | TCP para **AXFR/IXFR**

---

## Consultas iterativas y recursivas
**Resumen:** La **recursiva** delega el trabajo en el **resolver**; la **iterativa** devuelve **mejor referencia**. Los resolvers combinan ambos modos.

En **recursiva**, el cliente espera que el servidor haga todo el trabajo hasta obtener la **respuesta final** o un **error**; este modo se usa cuando un dispositivo final consulta a su **servidor DNS** configurado. Ese servidor actúa como **resolutor recursivo** y hará a su vez consultas **iterativas** siguiendo la cadena de delegación. En una consulta **iterativa**, un servidor DNS responde con la **mejor información** que tiene sin preguntar a otros por nosotros. Los resolvers aceptan **recursión** de clientes locales y realizan **iterativas** hacia fuera. Es recomendable permitir **recursión** solo a la **red local** por **seguridad**.

> **Nota:** Un **open resolver** expuesto públicamente puede ser abusado en **amplificación DDoS**.

**Resumen operativo**

Modo | Quién hace el trabajo | Uso típico
---|---|---
Recursiva | **Resolver** | Cliente → DNS interno
Iterativa | **Servidor consultado** | Resolver → **Raíz/TLD**

---

## Resolución inversa
**Resumen:** La **inversa** convierte **IP → nombre** con zonas **in-addr.arpa/ip6.arpa** y **PTR**; es clave para **logs**, **correo** y **diagnóstico**.

Es el proceso **DNS inverso**: dado una IP, encontrar el **nombre** asociado. DNS tiene zonas especiales: **in-addr.arpa** (IPv4) y **ip6.arpa** (IPv6). Por ejemplo, la IP **192.168.1.50** se invierte como **50.1.168.192.in-addr.arpa**; si existe zona **PTR**, responderá con el nombre (ej: “**servidorventas.empresa.local**”). Es útil en **administración**, **seguridad** y ciertos **protocolos** (correo comprueba coherencia **PTR**). En **Windows AD** con DNS integrados, los clientes pueden **registrar** su PTR automáticamente. Un error común es **olvidar** crear la **zona inversa**, lo cual puede causar **retrasos** en conexiones. La delegación de **inversas públicas** suele estar en manos del **ISP**; en IPs privadas, se crean **zonas inversas** internas.

> **Ejemplo:** Para **192.168.1.0/24**, crea **1.168.192.in-addr.arpa** y añade **PTR** para servidores clave.

**Checklist inversa**

- [ ] Crear **zona** inversa adecuada.
- [ ] Añadir **PTR** de servidores principales.
- [ ] Verificar con **Comando:** dig -x IP.
- [ ] Coordinar **ISP** para IPs públicas.
- [ ] Integrar con **DHCP/DNS** si procede.

---

## Resolvers
**Resumen:** El **resolver local (stub)** atiende a **aplicaciones** y consulta al **DNS configurado**; el **resolver recursivo** cachea, usa **forwarders** o **root hints** y aplica **políticas**.

Un **resolver DNS** es el componente que resuelve consultas en nombre de las **aplicaciones**. El **resolver local** recibe peticiones y formula la consulta al **servidor** configurado (p. ej., router o DNS público). Puede considerar **hosts**, **WINS/LLMNR** en ciertos entornos. El **servidor recursivo** actúa como resolver global, obteniendo la **respuesta final** y **cacheando** resultados. Configuración del resolver: en cada cliente, hay que configurar la dirección IP del **servidor DNS** a utilizar. En sistemas libres, suele estar en **/etc/resolv.conf** (o gestionado por **NetworkManager/systemd-resolved**). En **Windows**, se configura en **Propiedades TCP/IP** (automático por **DHCP**). Se pueden poner varios servidores, pero el cliente suele probar **secuencialmente**. Buenas prácticas: el primario debe estar **disponible** localmente y el secundario **redundante**. Los resolvers recursivos pueden tener **reenviadores (forwarders)** hacia DNS externos o resolver con **root hints**. Pueden aplicar **políticas** (filtrado de dominios maliciosos). En clientes, **ipconfig /displaydns** muestra la caché en Windows; **nslookup** y **dig** diagnostican problemas.

> **Advertencia:** En redes **AD**, no uses **DNS externos** en clientes; usa **DNS internos** y configura **forwarders** en el servidor.

**Puntos clave**

Elemento | Cliente (stub) | Resolver recursivo
---|---|---
Configuración | **/etc/resolv.conf**, GUI, DHCP | **Forwarders**, **root hints**
Caché | Local mínima | **TTL** gestionado
Seguridad | Básica | **Políticas**, acceso **ACL**

---

## Servidores de nombres: Características. Tipos (primario, secundario, caché, reenviador)
**Resumen:** Un **servidor DNS** puede ser **primario**, **secundario**, **caching-only** o **reenviador**; cada rol aporta **autoridad**, **redundancia** y **rendimiento**.

Un servidor de nombres (**DNS**) almacena información de **zonas** y responde a **consultas**. Sus características incluyen **autoridad** para dominios y **recursión** opcional.

**Servidor DNS primario (master):** Contiene la **copia principal** de la **zona**. Es donde se **editan** cambios. Define el **SOA** y es la autoridad para **replicar** a **secundarios**.

**Servidor DNS secundario (slave):** Copia **solo lectura** que obtiene datos del primario mediante **transferencias de zona** (**AXFR/IXFR**). Es autoritativo y aporta **redundancia** y **distribución de carga**. Requiere **TCP 53** abierto y control de **allow-transfer**.

**Servidor caché (caching-only):** No es autoritativo; hace **recursión** y **cachea** respuestas para mejorar **rendimiento**.

**Servidor reenviador (forwarder):** Reenvía consultas a otro **DNS upstream**. Útil para **aprovechar caché**, **filtrar** o cumplir políticas de **ISP**. Puede tener **fallback** a resolución directa.

En una PYME con **Active Directory**, los DNS internos suelen ser **autoritativos** para la zona local, **secundarios** entre sí, y **recursivos** con **reenviadores** configurados. Es importante no exponer **DNS internos** a Internet si contienen datos **sensibles**.

> **Nota:** Mantén al menos **dos** servidores **autoritativos** por zona para **resiliencia**.

**Roles y objetivos**

Tipo | Autoridad | Recursión | Uso principal
---|---|---|---
Primario | Sí | Opcional | **Edición** y origen
Secundario | Sí | Opcional | **Redundancia**
Caching-only | No | Sí | **Rendimiento**
Reenviador | No | Sí (proxy) | **Políticas**/caché central

---

## Zonas primarias y secundarias. Transferencias de zona
**Resumen:** La **zona primaria** es **editable**; la **secundaria** se sincroniza por **AXFR/IXFR** controladas por **serial (SOA)** y **TCP 53**.

Una **zona DNS** es un conjunto de **registros** autoritativos. La zona **primaria** es la **copia principal** editable; la zona **secundaria** es la **copia** replicada. La **transferencia de zona** sincroniza secundarias con el primario: **AXFR** (completa) e **IXFR** (incremental). Requiere **TCP 53** y es buena práctica **restringir** transferencias (**allow-transfer**) para seguridad. Un error común es **olvidar** actualizar el **serial** del **SOA** al editar archivos de zona en **BIND**, impidiendo que secundarios detecten cambios. En **AD-integrated**, la replicación es **multi-master** (no usa AXFR).

> **Ejemplo:** Verifica serial con **Comando:** dig @dns_secundario midominio.com SOA y compara con el **primario**.

**Checklist transferencia**

- [ ] **TCP 53** abierto entre **primario** y **secundario**.
- [ ] **allow-transfer** restringido.
- [ ] **Serial SOA** incrementado.
- [ ] Forzar **refresh** si hay desfase.
- [ ] Zonas en **subredes** o sitios distintos.

---

## Base de datos DNS: Estructura. Tipos de registros
**Resumen:** Una zona contiene **SOA**, **NS** y registros como **A/AAAA**, **CNAME**, **MX**, **PTR**, **TXT**, **SRV**, **CAA**; cada uno cumple un **propósito**.

La base de datos DNS de una **zona** se compone de múltiples **registros**:

- **SOA (Start of Authority):** Servidor primario, **contacto**, **serial**, **refresh/retry/expire**, **minimum TTL**.
- **NS:** Servidores de **nombres** autoritativos para la zona.
- **A/AAAA:** Nombre → **IPv4/IPv6**.
- **CNAME:** **Alias** que apunta a nombre **canónico** (evitar coexistencia con **MX** u otros en el mismo nombre).
- **MX:** Servidores de **correo** con **prioridad**.
- **PTR:** **Inversa** IP → nombre.
- **TXT:** Texto para **SPF**, **verificaciones** y otros usos.
- **SRV:** **Ubicación de servicios** (puerto/host), p. ej., **_ldap._tcp** en **AD**.
- **CAA:** Autoridades de **certificación** permitidas.

La estructura incluye **Nombre**, **TTL**, **Clase (IN)**, **Tipo** y **Datos**. En **BIND** es un **archivo de texto**; en **Windows**, en **AD** o **.dns**; en la nube, **interfaces web**. Un administrador debe **dominar** los tipos para configurar **web**, **correo**, **VoIP**, **O365**, etc., y limpiar **registros obsoletos** (p. ej., **scavenging** en Windows DNS).

> **Advertencia:** No mezcles **CNAME** con otros registros del **mismo nombre**.

**Mapa de registros**

Tipo | Finalidad | Ejemplo
---|---|---
SOA | Autoridad/serial | hostmaster, **TTL**
NS | DNS autoritativos | **ns1**, **ns2**
A/AAAA | Dirección | www → **192.0.2.10**
MX | Correo | **10 mail.midominio.com**
CNAME | Alias | **www → servidor**
TXT | SPF/validaciones | **v=spf1 …**
SRV | Servicios | **_ldap._tcp** puerto
CAA | CA permitidas | **letsencrypt.org**

---

## DNS Dinámico
**Resumen:** La **actualización dinámica (RFC 2136)** integra **DHCP-DNS** y **AD**; el **DynDNS público** mantiene nombres con **IP cambiantes**.

El término DNS dinámico puede referirse a dos conceptos:

**Actualización dinámica de DNS:** Permite a **clientes/servidores** actualizar registros en el **DNS** de forma **automatizada**. En **AD**, los clientes registran/actualizan su **A** y **PTR** mediante actualizaciones **seguras**. Un **DHCP** puede insertar **A/PTR** al otorgar IP. En **BIND**, se permite con **TSIG** y herramientas como **nsupdate**. Ventaja: coherencia **DHCP-DNS** sin editar manualmente. Requiere **permisos** y control de **seguridad**.

**Servicios de DNS dinámico público (DynDNS):** Asociar un **nombre** a una **IP pública** cambiante mediante **No-IP**, **Dyn**, **DuckDNS**, etc. Un **cliente** en el **router** o en un **PC** detecta cambios y actualiza el **registro A**. Útil para **acceso remoto** sin **IP fija**.

> **Nota:** Controla **quién** puede registrar **qué** para evitar **suplantaciones** en DNS dinámico.

**Aplicaciones típicas**

Caso | Interno | Externo
---|---|---
AD/DHCP | Sí (seguro) | No
IoT/Clientes | TSIG/ACL | Cliente del **router**
Acceso remoto | — | **ddclient**/**API**

---

## Clientes DNS (resolvers) en sistemas operativos libres y propietarios: Configuración
**Resumen:** En **Windows**, configura **DNS** en **TCP/IP** o recibe por **DHCP**; en **Linux/macOS**, usa **/etc/resolv.conf** o gestores; evita mezclar **internos** con **públicos**.

En sistemas **Windows**, la configuración del **cliente DNS** se hace en la **interfaz de red** (preferido/alternativo). Con **DHCP**, el servidor provee el/los **DNS**. **Comando:** ipconfig /all muestra los servidores asignados. En entornos de **dominio**, es crítico que los clientes usen **DNS internos**; usar **DNS externos** impide resolver **recursos internos**. En sistemas **libres** (Linux, macOS), se configura en **/etc/resolv.conf** o mediante **NetworkManager**/**systemd-resolved**; con **DHCP** también se autoconfigura. Es posible especificar **múltiples DNS**, pero conviene no mezclar **ámbitos** distintos (interno+externo) para evitar **fugas** o **fallos** de nombres internos.

Herramientas de **diagnóstico**: **nslookup**, **dig**, **host**; en Windows, **PowerShell** con **Resolve-DnsName**. **Comando:** dig midominio.com any, **Comando:** dig @192.168.1.10 ejemplo.local A, **Comando:** host 8.8.8.8, **Comando:** Resolve-DnsName -Name ejemplo.com -Type MX.

> **Ejemplo:** Si un PC “no entra al dominio”, revisa que apunte al **DNS interno**, no a **8.8.8.8**.

**Checklist cliente**

- [ ] DNS **preferido** y **alternativo** correctos.
- [ ] Recepción por **DHCP** validada.
- [ ] Evitar mezcla **interno/público**.
- [ ] Probar con **nslookup/dig**.
- [ ] Revisar **caché** local (**ipconfig /displaydns**).

---

## Servidores DNS en sistemas operativos libres y propietarios

### Instalación
**Resumen:** En **Linux**, instala **BIND9/Unbound/PowerDNS/Knot/dnsmasq**; en **Windows Server**, añade el **rol DNS** (frecuente con **AD**).

La instalación de un servidor DNS en sistemas libres suele implicar instalar software como **BIND9** (clásico), **Unbound** (resolutor), **PowerDNS**, **Knot DNS** o **dnsmasq** (simple). Se instala vía gestor de paquetes (ej.: **Comando:** apt install bind9). En **Windows Server**, el servidor DNS viene como **rol** (se añade en “Agregar roles y características”) y suele instalarse junto con el **controlador de dominio** por requisitos de **AD**. También existen **appliances** (Infoblox), aunque excede el nivel básico.

> **Nota:** Tras instalar, asegúrate de que el servicio **arranca** y **escucha** en la **IP** esperada.

**Resumen instalación**

Plataforma | Método | Comentario
---|---|---
Linux | Paquetes (**bind9**, **unbound**) | Servicio **systemd**
Windows | Rol **DNS Server** | GUI y **PowerShell**
Appliance | Imagen dedicada | Gestión web

### Arranque y parada
**Resumen:** Control con **systemctl** en **Linux** y **services.msc** en **Windows**; reinicia tras **cambios** manuales.

En **Linux**: **Comando:** systemctl start/stop/restart bind9, **Comando:** systemctl enable bind9. En **Windows**: **services.msc** → “Servidor DNS” o desde la **consola DNS**. Generalmente corre en **segundo plano**; reinicia si editas **named.conf** a mano.

> **Advertencia:** No pauses zonas salvo necesidad; usa **restart/reload** para aplicar cambios.

**Operativa básica**

Acción | Linux | Windows
---|---|---
Iniciar | systemctl start bind9 | services.msc → Iniciar
Habilitar | systemctl enable bind9 | Inicio automático
Reiniciar | systemctl restart bind9 | Reiniciar servicio

### Ficheros y parámetros de configuración básica
**Resumen:** En **BIND**, usa **named.conf** (+ **options/local**); define **recursion**, **forwarders**, **allow-query/transfer** y **zonas**.

En BIND (Linux), el archivo principal es **/etc/bind/named.conf** (incluye **named.conf.options**, **named.conf.local**, etc.). Ahí se configuran opciones globales (**recursion**, **allow-query**, **forwarders**, **listen-on**, tamaño de **caché**) y se definen **zonas**.

# options {
#     directory "/var/cache/bind";
#     recursion yes;
#     allow-query { any; };
#     forwarders { 8.8.8.8; 8.8.4.4; };
# };
# zone "midominio.local" {
#     type master;
#     file "/etc/bind/db.midominio";
#     allow-update { none; };
# };
# zone "1.168.192.in-addr.arpa" {
#     type master;
#     file "/etc/bind/db.192.168.1";
# };
# zone "." {
#     type hint;
#     file "/etc/bind/db.root";
# };

En **Windows Server DNS**, los parámetros se ajustan en la **GUI** (reenviadores, **root hints**, zonas **primarias/secundarias** o **AD-integradas**) y los archivos se guardan en **C:\Windows\System32\dns\** si no están integradas.

> **Nota:** Restringe **allow-query**/**allow-transfer** en servidores **expuestos**.

**Parámetros clave**

Parámetro | Propósito | Ejemplo
---|---|---
recursion | Resolver para clientes | yes/no
forwarders | Upstream DNS | 1.1.1.1; 8.8.8.8
allow-transfer | Seguridad | Solo **secundarios**
listen-on | Interfaces | 192.168.1.10

### Archivos de zona
**Resumen:** Una zona **master** tiene un **archivo** con **SOA**, **NS** y registros; cuida **serial**, **TTL** y **puntos finales** en **FQDN**.

Cada zona master tiene un **archivo de texto** con sus registros. Formato típico:

# $TTL 1h
# @   IN SOA   ns1.midominio.local. hostmaster.midominio.local. (
#        2025090101 ; Serial (yyyymmddnn)
#        1h         ; Refresh
#        15m        ; Retry
#        1w         ; Expire
#        1h         ; Minimum
#      )
#     IN NS    ns1.midominio.local.
#     IN NS    ns2.midominio.local.
# ns1 IN A     192.168.1.10
# ns2 IN A     192.168.1.11
# equipo1 IN A 192.168.1.50
# equipo2 IN A 192.168.1.51
# mail   IN A  192.168.1.60
#     IN MX 10 mail.midominio.local.
# www    IN CNAME equipo1.midominio.local.

La **@** se refiere al **origin** de la zona. Cuidado con el **punto final** en **FQDN** (si se omite, se añade el **sufijo** de zona). En Windows se maneja por **consola**. Las zonas **inversas** contienen **PTR**.

> **Advertencia:** Planifica **TTL** (bájalo antes de una **migración** para propagar cambios rápido).

**Buenas prácticas**

- [ ] Incrementar **serial** en cambios manuales.
- [ ] Verificar **puntos** en FQDN.
- [ ] TTL acorde a **criticidad**.
- [ ] Limpieza de **registros obsoletos**.

---

## Configurar servidores primarios, secundarios y cachés
**Resumen:** Define **master**, habilita **allow-transfer**, crea **slave** con **masters { … }**, registra **NS** y configura **recursión/forwarders**.

Configurar un **primario**: definir la **zona** como **master** (BIND: **type master + file**; Windows: **zona primaria**), introducir **registros**, definir **SOA** y **serial**. Configurar un **secundario**: en BIND, **type slave** con **masters { IP_del_master; }**; en Windows, **zona secundaria** indicando la IP del **primario**. En el **primario**, permitir la **transferencia** al secundario. Probar **sincronización** y registrar **NS** en la zona (y en la **padre** si aplica). Respecto a **cachés** (resolutores), en BIND se habilita **recursion yes** y **allow-query** para la **LAN**; en Windows, cualquier servidor sin zonas puede actuar como **recursivo**; ajusta **reenviadores** o **root hints** según política.

En entornos mixtos (**Linux/Windows**), integra como **secundarios** de uno u otro según convenga. Monitoriza **logs** (**syslog**, **Visor de sucesos**) y usa **Nagios** o similar para comprobar **disponibilidad**.

> **Nota:** Ubica **secundarios** en **sitios** o **subredes** diferentes para **resiliencia**.

**Tabla de pasos**

Paso | Acción | Resultado
---|---|---
1 | Crear **zona master** | Autoridad de edición
2 | Configurar **allow-transfer** | Seguridad de replicación
3 | Crear **zona slave** | Redundancia
4 | Registrar **NS** | Descubrimiento correcto
5 | Ajustar **forwarders** | Rendimiento/filtrado

---

## Herramientas para consultar a un servidor DNS
**Resumen:** Usa **nslookup**, **dig**, **host** y **PowerShell** para **validar** registros, **propagación** y **diagnosticar** errores.

**nslookup:** Disponible en **Windows** y Unix. Consulta simple, modo **interactivo** (**set type=MX**). **Comando:** nslookup nombre servidorDNS.

**dig:** Más **potente** y **flexible** (Linux/macOS, disponible en Windows con BIND utils). **Comando:** dig midominio.com ANY, **Comando:** dig @8.8.8.8 ejemplo.com A, **Comando:** dig -x 192.168.1.50.

**host:** Comando **sencillo** en Linux. **Comando:** host www.redlocal, **Comando:** host 8.8.8.8.

**PowerShell (Windows):** **Comando:** Resolve-DnsName -Name ejemplo.com -Type MX.

Existen utilidades **gráficas/web** (por ejemplo, verificadores de **propagación**). Estas herramientas permiten **verificar** funcionamiento tras configurar un **DNS interno**, revisar **MX**, o aislar si “no hay Internet” es un problema de **DNS**.

En **Windows AD**, **Comando:** dcdiag /test:DNS verifica la **salud** del DNS del dominio. En **Linux**, revisa **/var/log/syslog** o **Comando:** journalctl -u bind9 para **errores** de carga de zona.

> **Ejemplo:** Si **ping 8.8.8.8** responde pero **nslookup www.google.com** falla, el problema probablemente es **DNS**.

**Tabla de referencia**

Herramienta | Plataforma | Uso típico
---|---|---
nslookup | Windows/Unix | Pruebas básicas
dig | Linux/macOS/Win | Diagnóstico avanzado
host | Linux | Consultas rápidas
Resolve-DnsName | Windows | Scripts/PowerShell
