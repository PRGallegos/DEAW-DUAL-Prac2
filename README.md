# 📘 DEAW-DUAL-Prac2

    Práctica: Configuración de DNS Maestro-Esclavo con Bind9
    Este proyecto configura un sistema DNS maestro-esclavo utilizando Bind9 en dos servidores:
    tierra.sistema.test (maestro) y venus.sistema.test (esclavo), dentro de máquinas virtuales gestionadas por Vagrant.

## Descripción General

    Este proyecto configura un entorno DNS maestro-esclavo utilizando Vagrant para la gestión de máquinas virtuales y Bind9 como el software de servidor DNS en sistemas Debian. Se desarrolló para proporcionar un sistema DNS autoritativo capaz de manejar la resolución de nombres y de gestionar un servidor secundario o esclavo que sincronice automáticamente las zonas configuradas en el servidor maestro.

    Este README describe paso a paso el funcionamiento de la configuración, desde la instalación hasta la comprobación de la transferencia de zona, y el progreso de los cambios realizados.

---

## Objetivos del Proyecto

    1. Configurar un servidor DNS maestro en una máquina virtual llamada `tierra`.
    2. Configurar un servidor DNS esclavo en otra máquina virtual llamada `venus`.
    3. Implementar zonas directa e inversa en el servidor maestro que se sincronicen con el servidor esclavo.
    4. Verificar el correcto funcionamiento mediante herramientas de consulta DNS como `dig`.
    5. Documentar y facilitar el despliegue mediante Vagrant y archivos de configuración compartidos.

---

# 📝 Requisitos

    Vagrant instalado.
    VirtualBox o cualquier otro proveedor compatible con Vagrant.
    Conexión a internet para descargar las cajas base y dependencias.

# 📂 Estructura del Proyecto

    Vagrantfile: Archivo de configuración que define las máquinas virtuales.
    .gitignore: Excluye ciertos directorios y archivos (como .vagrant y backups) del control de versiones.
    README.md: Documentación del proyecto.
    LICENSE: Archivo de licencia (GPLv3 o Creative Commons).

    ```plaintext

    PRACTICA2/
    ├── .gitignore # Archivos y directorios excluidos del control de versiones
    ├── LICENSE # Licencia del proyecto
    ├── README.md # Documentación del proyecto
    ├── Vagrantfile # Configuración de Vagrant para las máquinas virtuales
    ├── tierra/
    │ ├── named.conf # Configuración DNS para el servidor maestro
    │ ├── sistema.test.zone # Archivo de zona directa
    │ └── sistema.test.rev # Archivo de zona inversa
    └── venus/
    └── named.conf # Configuración DNS para el servidor esclavo

# Instalación de Dependencias

    Bind9: Instalado automáticamente en ambas máquinas mediante el Vagrantfile.

# Uso de Vagrant

    El Vagrantfile define y configura dos máquinas virtuales:
    tierra (maestro): IP 192.168.57.103
    venus (esclavo): IP 192.168.57.102x
    Para iniciar las máquinas, navega al directorio del proyecto y ejecuta: vagrant up
    Esto configurará e instalará automáticamente Bind9 en ambas máquinas, además de realizar las configuraciones iniciales necesarias.

---

# Configuración de Bind9

# Configuración del Servidor Maestro (tierra)

    En la máquina tierra, el archivo de configuración named.conf define:

    Zonas DNS para sistema.test (directa) y 57.168.192.in-addr.arpa (inversa).
    Opciones para habilitar la validación DNSSEC, el reenvío de consultas, y restricciones de acceso.

    options {
        directory "/var/cache/bind";
        listen-on { any; };
        allow-query { localhost; 192.168.57.0/24; };
        dnssec-validation yes;
        recursion yes;
        allow-recursion { 127.0.0.0/8; 192.168.57.0/24; };
        forwarders { 208.67.222.222; };

    };

    zone "sistema.test" {
        type master;
        file "/etc/bind/sistema.test.zone";
        allow-transfer { 192.168.57.102; };
    };

    zone "57.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/sistema.test.rev";
        allow-transfer { 192.168.57.102; };
    };

    Archivos de Zona (sistema.test.zone y sistema.test.rev)
    Los archivos de zona proporcionan la configuración detallada para las resoluciones directa e inversa en el dominio sistema.test.

# Configuración del Servidor Esclavo (venus)

    En la máquina venus, el archivo venus/named.conf especifica:

    Las zonas sistema.test y 57.168.192.in-addr.arpa como esclavas.
    La IP del maestro 192.168.57.103 para sincronizar las zonas.
    Archivo venus/named.conf:

    ```
    options {
        directory "/var/cache/bind";
        listen-on { any; };
        allow-query { localhost; 192.168.57.0/24; };
        dnssec-validation yes;
        recursion yes;
        allow-recursion { 127.0.0.0/8; 192.168.57.0/24; };
    };

    zone "sistema.test" {
        type venus;
        file "/var/cache/bind/sistema.test.zone";
        tierra { 192.168.57.103; };
    };

    zone "57.168.192.in-addr.arpa" {
        type venus;
        file "/var/cache/bind/sistema.test.rev";
        tierra { 192.168.57.103; };
    };
    ```

---

# 📝 Comprobaciones y Pruebas

## Verificación de Transferencia de Zona

    Para comprobar que la zona se transfiere correctamente del maestro al esclavo, conecta al esclavo y ejecuta:
    dig @localhost sistema.test AXFR

## Consultas de Prueba

    Ejecuta estas consultas desde la máquina anfitrión o desde las máquinas virtuales:

    Consulta de registros A:
    ```
        dig @192.168.57.103 tierra.sistema.test A
    ```

    Consulta inversa:
    ```
       dig @192.168.57.103 -x 192.168.57.103
    ```

    Consulta de alias (CNAME):
    ```
        dig @192.168.57.103 ns1.sistema.test
    ```

    Consulta de servidores NS:
    ```
        dig @192.168.57.103 sistema.test NS
    ```

## Verificación de Logs

    Revisa los archivos de log en /var/log/syslog en ambas máquinas para verificar que las transferencias de zona y otras consultas funcionan correctamente.

    Progreso y Actualizaciones
    Inicialización: Configuración básica de Vagrant con una red privada para dos servidores virtuales.
    Instalación de Bind9: Automatizada mediante un script en Vagrantfile.
    Configuración del Maestro: Añadidas opciones de seguridad y configuraciones de zona en el archivo named.conf del maestro.
    Configuración del Esclavo: Especificada la IP del maestro para la transferencia de zona.
    Pruebas y Verificación: Pruebas con dig y revisiones de logs para asegurar el correcto funcionamiento.
    Documentación: Actualización de este README para detallar la estructura, configuración y pruebas.

# Solución de Problemas

    Error de conexión SSH en Windows: Utiliza Git Bash o instala OpenSSH desde la configuración de Windows.
    Permisos en Archivos de Configuración: Verifica que los archivos tengan permisos adecuados para que Bind9 los lea correctamente (chmod 644 archivo).

# 👤 Autor

    Pedro Rodríguez Gallegos
    Módulo: Despliegue de Aplicaciones Web
