# Ansible en Google Cloud

## ¿Qué es Google Cloud? ☁
Google Cloud es el espacio virtual a través del cual se puede realizar una serie de tareas que antes requerían de hardware o software y que ahora utilizan la nube de Google como única forma de acceso, almacenamiento y gestión de datos.

## ¿Qué son las Instancias de VM? 
Son un servicio de Googe Cloud que permite a los usuarios el alquiler máquinas virtuales de tamaño modificable para ejecutar sus aplicaciones informáticas. Permite pagar únicamente por la capacidad utilizada, en lugar de comprar o alquilar una máquina para utilizarla durante varios meses o años, con Google se alquila la capacidad por horas.

## Creación de Instancias de VM 🔧
1. Una vez registrados accederemos al Inicio del Panel de Control de Google Cloud. 
![](images/2.png "Panel de Control")

2. En el panel de navegación situado a la izquierda seleccionamos **Compute Engine > Instancias de VM**.

3.	Pulsamos en el panel superior en **Crear Instancia**.
![](images/3.png "Creamos una Instancia")

4.	En Nombre le asignamos un nombre a nuestra instancia.
![](images/4.png "Asignamos un nombre")

5.	En Disco de arranque seleccionamos la imagen que queramos para nuestra máquina, en nuestro caso será Ubuntu 18.04 LTS.

6.	En Cortafuegos es donde abriremos los puertos para conectarnos a esa máquina. Aquí Google nos lo pone fácil y nos da la opción de marcar las casillas de http y https, también deberemos de añadir ssh.

7.	Finalmente pulsamos en **Crear** y esperamos que se inicie nuestra instancia.
![](images/5.png "Pulsamos en Crear")

## Gestión de claves SSH 🔒

1.	Ahora necesitamos configurar el nodo principal (master) para que pueda conectarse al resto de nodos por SSH. Para ello ejecutamos el comando **ssh-keygen** en la máquina master para crear una clave SSH pública y otra privada.

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
En nuestro caso la clave pública está almacenada en la máquina principal en el directorio: */home/ubuntu/.ssh/id_rsa.pub*. 
Accedemos al directorio.

```
cd ~/.ssh/
```
Mostramos la clave y la copiamos.

```
cat id_rsa.pub
```

3.	Nos vamos a nuestra instancia que queremos conectar (eslave1) y pulsamos en EDITAR.
![](images/6.png "Pulsamos en Editar")

4.	En la sección Clave SSH insertamos la clave anterior y pulsamos en Guardar.
 ![](images/7.png "Pegamos la clave")

5.	Realizamos el mismo procedimiento para el resto de nodos. 

 

##	¿Qué es Ansible? 💻
Ansible es una herramienta que nos permite configurar, administrar y realizar instalaciones en sistemas cloud con múltiples nodos sin tener que instalar agentes software en ellos. 
Sólo es necesario instalar Ansible en la máquina principal desde la que vamos a realizar operaciones sobre el resto de nodos y ésta se conectará a los nodos a través de SSH. Como requisito únicamente requiere Python en el servidor remoto en el que se vaya a ejecutar para poder utilizarlo.

💡💡***Dato curioso***💡💡  
El nombre de Ansible es usado en la literatura de ciencia ficción para describir un dispositivo de comunicación más rápido que la luz. Instantáneo.


##	Arquitectura de Ansible
Existen dos tipos de servidores:
- Controlador: es la máquina desde la que se llevan a cabo las órdenes.
- Nodo: es manejado por el controlador a través de SSH.

![Arquitectura de Ansible](images/1.png "Arquitectura de Ansible")

##	Instalación de Ansible 🛠️
La instalación de Ansible la vamos a realizar únicamente en el nodo principal y este nodo se conectará al resto de nodos por SSH para realizar las tareas de administración.
Una vez que nos hemos conectado al nodo principal podemos proceder a realizar la instalación de Ansible. El sistema operativo que de nuestra máquina es Ubuntu, por lo tanto, la instalación la realizaremos con apt.
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
##	Configuración de Ansible ⚙️
De momento la única configuración que vamos a realizar en Ansible será editar el archivo de inventario /etc/ansible/hosts para incluir la lista de hosts sobre los que vamos a realizar tareas con Ansible.

Así, nuestro archivo ***/etc/ansible/hosts*** contendrá lo siguiente:

```
[googlehosts]
10.128.0.3
10.128.0.4
```
> En este caso las direcciones IP son internas, asi podemos parar y levantar las instancias cuantas veces queramos, ya que las IPS públicas cambian cada vez que las instancias se detienen. 

## Módulos de Ansible 🧩
Ansible dispone de una gran variedad de módulos que pueden ser utilizados desde la línea de comandos o en las tareas de los playbooks.

Existen módulos para trabajar con clouds (Amazon, Azure, etc.), clustering (Kubernetes, Openshift, etc.), bases de datos (Influxdb, Mongodb, MySQL, PostgreSQL, etc.), monitorización, mensajería, etc.

En la documentación oficial puede encontrar un listado de todos los módulos disponibles en [Ansible](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html).

A continuación vamos a realizar una breve demostración de cómo utilizar los módulos:

- ping

- shell

- apt

### Módulo *ping*
El módulo ping nos permite realizar un ping sobre todos los nodos del inventario o sobre algún nodo específico.

```
ansible all -m ping
```
Usamos el modificador -m para indicar a Ansible el módulo que queremos utilizar, y el modificador all para indicarle que queremos hacerlo sobre todos los nodos.  

![](images/8.png "Ping a todos los nodos")

Si sólo queremos hacer ping sobre uno de los nodos en lugar de usar all, indicaremos el nodo sobre el que queremos realizar la operación.

```
ansible 10.128.0.3 -m ping
```
![](images/9.png "Ping a un solo nodo")

### Módulo *shell*
Si no indicamos nigún módulo, por defecto Ansible utilizará el módulo shell, el cual nos permite ejecutar comandos sobre cada uno de los nodos.

El siguiente ejemplo ejecuta el comando uname -a sobre todos los nodos del inventario.

```
ansible all -m shell -a "uname -a"
```
En este caso el modificador -m nos permite indicar el módulo que queremos utilizar y el modificador -a nos permite indicar el comando.  

![](images/10.png "Módulo Shell")

### Módulo *apt*
El módulo apt nos permite utilizar el sistema de gestión de paquetes APT para los sistemas operativos Debian y Ubuntu.

En la [documentación oficial](https://docs.ansible.com/ansible/latest/modules/apt_module.html#apt-module) tenemos disponible todos los parámetros que podemos usar es este módulo.

Por ejemplo, el parámetro update_cache nos permite realizar la operación apt-get update. Si quisiéramos realizar un *apt-get update* sobre cada uno de los nodos ejecutaríamos el siguiente comando.

```
ansible all -m apt -a "update_cache=yes" -b
```

![](images/11.png "apt-get update")

> El modificador -b es para indicar que queremos realizar un escalado de privilegios (become) para poder ejecutar comandos como root.

Para poder realizar la instalación de un paquete será necesario hacer uso de los parámetros *name* y *state*:

- El parámetro *name* nos permite indicar el nombre del paquete que queremos instalar.
- El parámetro *state* nos permite indicar en qué estado queremos que se encuentre el paquete. En este caso vamos a indicar present, para indicar que queremos que esté instalado.

Ejemplo de cómo instalar git en cada uno de los nodos.

```
ansible all -m apt -a "name=git state=present" -b
```

![](images/12.png "git")












