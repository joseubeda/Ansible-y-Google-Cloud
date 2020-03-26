# Ansible en Google Cloud

## ¿Qué es Google Cloud?
Google Cloud es el espacio virtual a través del cual se puede realizar una serie de tareas que antes requerían de hardware o software y que ahora utilizan la nube de Google como única forma de acceso, almacenamiento y gestión de datos.

## ¿Qué son las Instancias de VM?
Son un servicio de Googe Cloud que permite a los usuarios el alquiler máquinas virtuales de tamaño modificable para ejecutar sus aplicaciones informáticas. Permite pagar únicamente por la capacidad utilizada, en lugar de comprar o alquilar una máquina para utilizarla durante varios meses o años, con Google se alquila la capacidad por horas.

## Creación de Instancias de VM
1. Una vez registrados accederemos al Inicio del Panel de Control de Google Cloud.
2. En el panel de navegación situado a la izquierda seleccionamos Compute Engine > Instancias de VM.
3.	Pulsamos en el panel superior en Crear Instancia.
4.	En Nombre le asignamos un nombre a nuestra instancia.
5.	En Disco de arranque seleccionamos la imagen que queramos para nuestra máquina, en nuestro caso será Ubuntu 18.04 LTS.
6.	En Cortafuegos es donde abriremos los puertos para conectarnos a esa máquina. Aquí Google nos lo pone fácil y nos da la opción de marcar las casillas de http y https, también deberemos de añadir ssh.
7.	Y listo, pulsamos en Crear y esperamos que se inicie nuestra instancia.

## Gestión de claves SSH

1.	Ahora necesitamos configurar el nodo principal (master) para que pueda conectarse al resto de nodos por SSH. Para ello ejecutamos el comando ssh-keygen en la máquina master para crear una clave SSH pública y otra privada.

```
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:j2ASXKcdWnUFUeerAktsj6E2Xs4uWmA3GcmCpBrp4oI ubuntu@ip-172-31-81-167
The key's randomart image is:
+---[RSA 2048]----+
|   .  . +.. +=o .|
| .o....*.. .   o |
|o. .o.o+.       .|
|o.   .. +       .|
|o.  .oo+S*     . |
|+   .oo.=o*   .  |
|E.     =.+.o .   |
|.     +.=   .    |
|     ...o+       |
+----[SHA256]-----+
```

2.	Una vez creada las claves SSH tenemos que copiar la clave pública en cada uno de los nodos que vamos a gestionar con Ansible.
En nuestro caso la clave pública está almacenada en la máquina principal en el directorio: /home/ubuntu/.ssh/id_rsa.pub. 
Accedemos al directorio.

```
cd ~/.ssh/
```
Mostramos la clave y la copiamos.

```
cat id_rsa.pub
```

3.	Nos vamos a nuestra instancia que queremos conectar (eslave1) y pulsamos en EDITAR.
 
4.	En la sección Clave SSH insertamos la clave anterior y pulsamos en Guardar.
 
5.	Realizamos el mismo procedimiento para el resto de nodos. 

 

##	¿Qué es Ansible?
Ansible es una herramienta que nos permite configurar, administrar y realizar instalaciones en sistemas cloud con múltiples nodos sin tener que instalar agentes software en ellos. 
Sólo es necesario instalar Ansible en la máquina principal desde la que vamos a realizar operaciones sobre el resto de nodos y ésta se conectará a los nodos a través de SSH. Como requisito únicamente requiere Python en el servidor remoto en el que se vaya a ejecutar para poder utilizarlo.

***Dato curioso:***
El nombre de Ansible es usado en la literatura de ciencia ficción para describir un dispositivo de comunicación más rápido que la luz. Instantáneo.


##	Arquitectura de Ansible
Existen dos tipos de servidores:
- Controlador: es la máquina desde la que se llevan a cabo las órdenes.
- Nodo: es manejado por el controlador a través de SSH.

![Arquitectura de Ansible](images/1.png "Arquitectura de Ansible")

##	Instalación de Ansible
La instalación de Ansible la vamos a realizar únicamente en el nodo principal y este nodo se conectará al resto de nodos por SSH para realizar las tareas de administración.
Una vez que nos hemos conectado al nodo principal podemos proceder a realizar la instalación de Ansible. El sistema operativo que de nuestra máquina es Ubuntu, por lo tanto, la instalación la realizaremos con apt.
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
##	Configuración de Ansible
De momento la única configuración que vamos a realizar en Ansible será editar el archivo de inventario /etc/ansible/hosts para incluir la lista de hosts sobre los que vamos a realizar tareas con Ansible.

















