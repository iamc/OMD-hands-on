*******************************************
Monitorización de sistemas con Nagios y OMD
*******************************************

:Autor: Iñigo Aldazabal
:email: inigo_aldazabal@ehu.es
:fecha: 21/Junio/2013


Guía de configuración básica de monitorización de sistemas utilizando `Nagios`_
y la colección de extensiones de nagios `Check_MK`_. Nos serviremos del sistema ya empaquetado `Open
Monitoring Distribution (OMD)`_ que integra tango Nagios como Check_MK, además
de otras muchos extensiones de Nagios y otras utilizades, y que facilita enormemente la
puesta en marcha y gestión del sistema.

Utilizaremos dos máquinas virtuales sobre VirtualBox para poder seguir esta
guía paso a paso.

.. _`Nagios`: http://www.Nagios.org/
.. _`check_mk`: http://mathias-kettner.com/check_mk.html
.. _`Open Monitoring Distribution (OMD)`: http://omdistro.org/

.. .. header:: ###Section###
.. footer:: ###Page###
.. contents::
.. section-numbering::

.. y esto es un comentario. Orden #=-~


Software necesario
==================

 * VirtualBox:
    software de virtualización https://www.virtualbox.org/

 * Máquina vitual para servidor OMD: CentOS 6.3 con entorno gráfico
    http://sourceforge.net/projects/virtualboximage/files/CentOS/6.3/CentOS-6.3-x86.7z

 * Máquina virtual a monitorizar: CentOS 5.7 básica
    http://sourceforge.net/projects/virtualboximage/files/CentOS/5.7/CentOS-5.7-i386.7z

 * Open Monitoring Distribution - OMD 
    http://omdistro.org/

 * Check_MK 
    Conjunto de extensiones de nagios, integrado en OMD
    http://mathias-kettner.com/check_mk.html. Agente de monitorización a instalar en los equipos
    a controlar.



Configuración de VirtualBox
===========================

Descomprimimos las imágenes de las máquinas virtuales, y abrimos el ``.vbox``
con VirtualBox. Si nos da un error sobre que el UUID del disco ya está en uso
por ejemplo porque ya hemos abierto una copia de esta misma imagen basta con
cambiar la UUID de la imagen .vdi con el comando::

    VBoxManage internalcommands sethduuid CentOS-5.7.vdi

Hay que sustituir la UUID vieja por la que nos indica este comando en el
fichero de configuración .vbox.

Antes de arrancar las máquinas virtuales queremos configurar
una red local interna a nuestro ordenador para poder conectar las máquinas
virtuales entre ellas. Para ello, seleccionamos la máquina virtual
correspondiente y le añadimos un nuevo adaptador de red del tipo ``host-only``
(CentOS 6.3 x86 -> settings | network | Adapter 2 | Enable + attached to "Host-only Adapter").

Anotamos la dirección MAC de la tarjeta para configurar las IP de la red
interna de forma estática como veremos mas adelante. En este caso los datos que
indicaremos serán:

**CentOS 6.3 - Servidor OMD**

:MAC: 08:00:27:C1:99:2D
:IP:  192.168.56.100


**CentOS 5.7 - a monitorizar**

:MAC: 08:00:27:42:79:DF
:IP:  192.168.56.10


**PC VirtualBox Host**

:IP: 192.168.56.1


Configuración del servidor e instalación de  Nagios / OMD
=========================================================

Configuración de red
--------------------

Arrancamos la máquina virtual y configuramos la IP estática. Para ello cremos el fichero ``/etc/sysconfig/network-scripts/ifcfg-eth1``::

    #/etc/sysconfig/network-scripts/ifcfg-eth1
    DEVICE=eth1
    BOOTPROTO=none
    IPADDR=192.168.56.100
    NETMASK=255.255.255.0
    ONBOOT=yes
    HWADDR=08:00:27:C1:99:2D
    DEFROUTE=yes
    NAME="eth1"

Y reiniciamos la red::

    service network restart


Configuración de envío de correos 
---------------------------------

Para comprobar si el sistema puede enviar correos electrónicos mediante postfix hacemos::

    echo "Test mail from postfix" | mail -s "Test Postfix" inigo_aldazabal@ehu.es

y comprobamos el log de postfix (``/var/log/maillog``) si el mensaje no nos
llega. En nuestro caso funciona sin mas configuración, pero puede ser necesario
indicar un smtp "relay host" en ``/etc/postfix/main.cf``. Se puede utilizar
para probar por ejemplo en SMTP de google. Ver las indicaciones en http://freelinuxtutorials.com/quick-tips-and-tricks/configure-postfix-to-use-gmail-in-rhelcentos/


Instalación de OMD
------------------

Seguimos directamente las instrucciones de la web de OMD para CentOS http://omdistro.org/doc/quickstart_redhat adaptándoslos a nuestra versión de CentOS, en este caso CentOS 6 con arquitectura i386.


Instalación de paquetes
~~~~~~~~~~~~~~~~~~~~~~~

Instalamos el repositorio ``epel`` ::

    rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm

y descargamos e instalamos el paquete de OMD (también se podría instalar el
repositorio de OMD como explican en
http://labs.consol.de/nagios/omd-repository/) ::

    wget http://files.omdistro.org/releases/centos_rhel/omd-1.00-rh61-30.i386.rpm
    yum install omd-1.00-rh61-30.i386.rpm

Esto nos instala en nuestro caso 35 paquetes y actualiza 3.


Creamos un nuevo "sitio" de OMD y lo arrancamos::

    omd create test
    omd start test


Configuración acceso al servidor web
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Probamos a acceder a http://localhost/test y nos da un error de "OMD: Site not
started". En las FAQ indica que esto puede pasar en CentOS y para solucionarlo
basta con hacer::

    /usr/sbin/setsebool httpd_can_network_connect 1

Si queremos hacer este cambio permanente hay que añadir la opción ``-P`` al
comando. En este caso el comando tarda un cierto tiempo, incluso minutos, en ejecutarse. Paciencia.

Y ahora ya podemos acceder al interface sin problemas en http://localhost/test o
http://192.168.56.100/test con usuario/clave por defecto omdadmin/omd.


Si queremos acceder al interface web desde otros equipos tenemos que abrir el
puerto correspondiente en el firewall de CentOS, que en este csao viene
activado por defecto, mediante el GUI o en consola mediante el comando::

    /usr/bin/system-config-firewall-tui

en el apartado *Customize*, el último de la lista, servicio *WWW (HTTP)* (se
activa/desactiva con espacio).


Acceso a nuestro "sitio" en OMD
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Al crear un sitio OMD crea a su vez un usuario en el sistema que servirá para gestionar este
sitio de forma independiente. De esta forma podemos tener varios "sitios" diferentes
para pruebas, producción, etc.

Para acceder a la gestión del sitio que nos interese basta con hacer ``su``  al nuevo sitio/usuario::

    su - test

Ver explicación del funcionamiento en http://omdistro.org/doc/configuration_basics y todas las opciones de configuración en  http://mathias-kettner.com/check_mk.html.


Nosotros en general utilizaremos el sistema *WATO - Check_MK's Web Administrator
Tool*.


Configuración del equipo a monitorizar
======================================

Configuración de red
--------------------

Como antes arrancamos la máquina virtual a monitorizar (CentOS-5.7) y configuramos la IP estática. Para ello cremos el fichero ``/etc/sysconfig/network-scripts/ifcfg-eth1``::

    #/etc/sysconfig/network-scripts/ifcfg-eth1
    DEVICE=eth1
    BOOTPROTO=none
    IPADDR=192.168.56.10
    NETMASK=255.255.255.0
    ONBOOT=yes
    HWADDR=08:00:27:42:79:DF
    DEFROUTE=yes
    NAME="eth1"

Y reiniciamos la red::

    service network restart


Instalación del agente check_mk
-------------------------------

Descargamos e instalamos el agente sin mas complicación::

    wget http://mathias-kettner.com/download/check_mk-agent-1.2.2p2-1.noarch.rpm
    wget http://mathias-kettner.com/download/check_mk-agent-logwatch-1.2.2p2-1.noarch.rpm
    yum install --nogpgcheck check_mk-agent-1.2.2p2-1.noarch.rpm \
        check_mk-agent-logwatch-1.2.2p2-1.noarch.rpm

Si queremos, para mayor seguridad podemos restringir el acceso a la ejecución de check_mk solamente desde el servidor
OMD que acabamos de configurar. Para ello basta con añadir a ``/etc/xinetd.d/check_mk``::

    $> vim /etc/xinetc.d/check_mk
    ...
    only_from = 192.168.56.100
    ...

y recargamos la configuración de ``xinetd``::

    $>/etc/init.d/xinetd reload


Monitorización de discos duros con S.M.A.R.T.
---------------------------------------------

Si monitorizamos un host "real" (i.e. **no** una máquina virtual) nos
interesará monitorizar el estado de sus discos duros. Check_mk no busca el
check de S.M.A.R.T. al hacer el inventario y tenemos que explícitamente
instalar el plugin que el propio check_mk nos deja en el servidor de
monitorización.

El plugin se denomina ``smart`` y se encuantra en el servidor de monitorización
en ``~/share/check_mk/agents/plugins/smart``. Hay que copiarlo desde el propio servidor al sistema a monitorizar
al directorio de plugins de check_mk, ``/usr/lib/check_mk_agent/plugins/``. 

Si estamos en el servidor como el usuario regular ``test`` en este caso basta
con::

    # su - test
    # scp ~/share/check_mk/agents/plugins/smart  \
          user@remote-host:/usr/lib/check_mk_agent/plugins/smart

Si todavía no hemos realizado el inventario inicial de este host (ver siguiente
apartado) e instalamos el plugin antes de hacerlo, los chequeos correspondientes 
aparecerán directamente al realizarlo. Veremos dos por cada disco: uno para la temperatura y otro para 
el estado de S.M.A.R.T. Si el inventario estaba ya realizado previamente  basta con rehacerlo 
veremos como aparecen los nuevo chequeos.

.. note::

    Al rehacer el inventario de un equipo los chequeos que ya estaban
    inventariados previamente conservan todo el historial, gráficas, etc.



Configuración básica de OMD
===========================

En general realizaremos la configuración a través del interface gráfico *"Multisite"* que forma 
parte del paquete Check_MK. Concretamente utilizaremos el *"WATO - Check_MK's Web Administration Tool"*.

En primer lugar configuraremos un usuario para que reciba las alertas, y tras
ello añadiremos los equipos a monitorizar.


Creación de usuario en OMD
--------------------------

Vamos a **WATO-Configuration | Users & Contacts | New User** asegurándonos de
añadirlo a un *contact group* en este caso solo hay uno, *everybody*, y de
marcar *enable notifications* para poder recibir notificaciones.

Guardamos los cambios (*save* en la parte inferior) y vemos que en la ventana
principal de *Users & Contacts* aparece una indicación de que hay un cambio
respecto a la configuración guardada (parte superior izquierda, *1 Changes*).
Pinchamos donde pone *1 Changes* y luego en *Activate Changes* para propagar
los cambios.


Integración de nuevo *host* a monitorizar
------------------------------------------

Antes de añadir un nuevo equipo, si se trata de un ordenador en el cual tenemos
que instalar el agente de check_mk, éste lo tenemos que instalar *antes* de
realizar el inventario en check_mk, tal y como ya lo hemos indicado
previemente.

Para añadir el nuevo host vamos a **WATO-Configuration | Hosts & Folders |
Create new host**. Ahí solo añadimos el *Hostname* (indicamos la IP),
*Permissions* (grupo *Everybody*) y *Alias* (CentOS5.7-VM). Pinchamos en *Save
& go to Services* y alli seleccionamos / desseleccionamos los checks que nos 
interesa monitorizar. Le damos a *Save manual check configuration* y de nuevo 
activamos los cambios que se muestran pendientes como hicimos al crear un
usuario.

Si ahora vamos a la página principal del interface de check_mk (**Views |
Dashboards |  Main Overview**) vemos que ya tenemos un host monitorizado y en
este caso 19 servicios.

.. note::

    Resulta conveniente utilizar el propio servidor OMD para que se monitorice a si mismo. Para
    ello basta con instalar el agente de Check_MK en el servidor y añadir el host *localhost* en WATO.



Prueba de notificación
----------------------

Seleccionamos cualquier servicio, por ejemplo *CPU utilization* y le damos al
icono del martillo para ejecutar comandos sobre el servicio. Se nos despliegan
varios menús y vamos a **Various Commands | Fake check results** y le damos a
*Critical*. Confirmamos y vemos en el *Main Overview* y en otras páginas que
efectivamente el servicio aparece como crítico durante un rato (hasta el
siguiente check).

Efectivamente nos llega un correo con el aviso del fallo, y otro con la
recuperacíon del fallo.


Referencias
===========

**Máquinas virtuales**

 * Oracle VirtualBox, sistema de virtualización multiplataforma: https://www.virtualbox.org/

 * Máquinas virtuales preparadas con instalaciones de CentOS para VirtualBox: http://virtualboxes.org/images/centos/  

**Nagios**

 * Web: http://www.nagios.org/
 
 * Documentación oficial: http://nagios.sourceforge.net/docs/nagioscore/3/en/toc.html

 * Nagios Exchange: repositorio de chequeos y extensiones http://exchange.nagios.org/

 * *"Building a Monitoring Infrastructure With Nagios"*, David Josephsen, Prentice Hall 2007


**Check_MK**

 * Web: http://mathias-kettner.com/check_mk.html

 * Documentación oficial: http://mathias-kettner.com/checkmk.html


**OMD**

 * Web: http://omdistro.org/

