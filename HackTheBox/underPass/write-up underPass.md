# Write-up: underPass

## Explotando vulnerabilidades en SNMP y Mosh

**Herramientas utilizadas:**
● nmap para escaneo de puertos
● nikto para análisis de Apache
● snmpwalk para enumeración de SNMP
● ffuf para fuerza bruta de rutas
● mosh para escalación de privilegios
**¿Qué aprendí?**
Importancia de revisar credenciales por defecto
Abuso de privilegios en sudo
Métodos de escalación de privilegios en sistemas reales
_Hack The Box - Pwned!_

## Flag user

### Escaneo inicial

Primero realice un escaneo rápido con nmap utilizando half-opening con el parametro -sS
para hacer envios de paquetes Syn, lo cual me reveló la siguiente información:
Los puertos 22 y 80 están abiertos, por seguridad, realicé un escaneo con -sT, que usa el


método three-way handshake que nos ayuda a confirmar que los puertos realmente estan
abiertos.Se logra apreciar que ssh tiene la versión 8.9p1 la cual es teóricamente reciente
por lo cual no encontré mucha información sobre CVSS relevantes para poder obtener
acceso mediante ssh.

### Análisis del servicio HTTP

Debido que el puerto 80 el de http estaba abierto y corriendo en apache, decidí hacer un
escaneo con nikto para poder obtener más información relevante, la cual no obtuve nada
relevante.

#### Escaneo de puertos UDP

Hice un escaneo para puertos de udp, el cual reveló que el puerto 161 estaba abierto, este
puerto es SNMP (Simple Network Management Protocol). Con ayuda de snmpwalk puede
obtener más información sobre el servicio, en el cual dice
**"UnDerPass.htb is the only daloradius server in the basin!"**
Esto nos da más pistas por donde podemos realizar un ataque.

### Acceso a daloRADIUS

Tras un análisis exhaustivo con ffuf, encontré la ruta de inicio de sesión:
**[http://10.10.11.48/daloradius/app/operators/login.php](http://10.10.11.48/daloradius/app/operators/login.php)**


Intentamos con las credenciales default del servicio las cuales son:
**usuario: administrator
contraseña: radius**
Sorprendentemente, el acceso fue exitoso, lo que indica una configuración insegura en
daloRADIUS.

#### Obtención de credenciales del usuario

Al acceder al panel podremos observar que hay un usuario registrado
**Usuario: svcMosh
Contraseña: 412DD4759978ACFCC81DEAB01B382403 (Hash MD5)**
Usando el servicio **CrackStation** (https://crackstation.net), logré descifrar la contraseña en
texto plano:
**underwaterfriends**
Con esta información y verificando que ssh permite conexiones con usuario y contraseña


Procedí a realizar la conexión, la cual fue exitosa
y así es como obtuve la primera flag de user.

## Escalación de Privilegios

### Análisis del entorno


El nombre del usuario nos da un pequeño adelanto de que se está usando Mosh ( Mobile
Shell), lo cual nos indica que podemos hacer un privilage scalation con mosh.
Comprobamos con **sudo -l** para ver si tenemos permisos de super usuario
el cual nos indica que el único comando que podemos ejecutar con sudo en localhost es
/usr/bin/mosh-server, el cual nos da la siguiente información
leyendo la documentación de mosh, podemos entender que el primer párrafo es un key para
poder ejecutar mosh y escalar privilegios.

#### Exploit de Mosh

https://mosh.org/
mosh nos indica que podemos correr este comando
$ MOSH_KEY= **key** mosh-client **remote-IP remote-PORT**
Adaptando el comando a nuestra key, ip remoto y el puerto el cual es indicado al generar la


key, logramos hacer un escalado de privilegios exitoso y obteniendo la flag.

## Conclusión

Este write-up documenta el proceso de explotación de la máquina _underPass_ ,
aprovechando vulnerabilidades en configuraciones débiles (credenciales por defecto en
daloRADIUS) y abuso de privilegios en Mosh. Un administrador debería mitigar estos
problemas con buenas prácticas de seguridad, como:
● Deshabilitar credenciales por defecto.
● Limitar los privilegios de sudo.
● Usar autenticación más segura en SSH y SNMP.
Este ejercicio refuerza la importancia del pentesting para identificar y corregir fallas antes de
que sean explotadas por atacantes reales.


