# ¿Qué es SSH?

***SSH*** o ***Secure Shell***, es un protocolo de administración remota que le permite a los usuarios controlar y modificar sus servidores remotos a través de Internet por medio de un mecanismo de autenticación.

Para utilizar SSH, debemos abrir la terminal en caso de Linux o Mac y ejecutar `ssh {user}@{host} `

`{user}` es el usuario del servidor al que se desea loguear 
`{host}` la dirección ip o nombre de dominio al q se desea ingresar.

Una vez entendido esto...

# ¿Cómo podemos mejorar la seguridad del protocolo SSH en nuestros servidores?

### Crear Usuarios en el servidor
Con el fin de evitar el ingreso al servidor usando el usuario root, lo ideal es crear la cantidad de usuarios necesarios.
Para eso, debemos conectarnos al servidor via SSH y crear los usuarios con el comando *useradd*

#### Un ejemplo de cómo hacerlo:

`useradd -m -s /bin/bash usuario`

`-m` nos sirve para crear un directorio raíz para el usuario

`-s` nos sirve para setear la shell

### Usar Login basado en claves públicas de SSH
SSH soporta varias formas de autenticar el login, pero se recomienda la autenticación basad en claves públicas, para mayor seguridad. Hay muchos métodos de generación de claves, pero el más robusto actualmente es *ed25519*. 
Por eso, a continuación generaremos una clave usando *ed25519*

#### Para generar la clave pública, abrimos la consola y ponemos:
`ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519 -C "user@ejemplo.com"`

`-o` nos sirve para salvar la clave privada usando el nuevo formato de OpenSSH.

`-a` nos sirve para aumentar la cantidad de rondas usadas por el algoritmo para generar la clave. A mayor cantidad elegida, disminuye la velocidad de la verificacion de la frase de la contraseña, pero aumenta la resistencia a los ataques de fuerza bruta.

`-f` nos sirve para especificar el nombre de la clave. En caso de querer que el SSH nos lo detecte automaticamente, debemos especificar el directorio .ssh

`-c` nos sirve para agregar un comentario. Es excluvisamente informativo y en general se suele poner quién genero la clave.

Una vez creada la clave pública, debemos copiarla en el servidor.

#### Para esto, usaremos el comando *ssh-copy-id.*

`ssh-copy-id {user}@{host}`

### TODAS LAS MODIFICACIONES QUE HAREMOS A CONTINUACIÓN, SERÁN SOBRE EL ARCHIVO *SSHD_CONFIG*
			
Para poder lograr esto, debemos conectarnos via SSH al servidor con nuestro usuario, una vez dentro debemos ejecutar el comando `sudo nano /etc/ssh/sshd_config` , esto nos abrirá el editor la configuración del protocolo SSH con el editor nano.

Dentro del archivo *sshd_config*, todas las líneas que empiecen con # son consideradas como comentarios, para que entren en vigencia, se debe sacar el # al inicio de la línea en cuestión.

### 1° Cambiar el puerto del SSH
Uno de los principales beneficios de cambiar el puerto de SSH, es evitar el típico escaneo de puertos. La gran mayoria de hackers buscan el puerto ssh el cual por defecto está seteado por defecto, en el puerto 22. Por eso, es recomendable el cambio de puerto

Para cambiar el puerto, debemos abrir el sshd_config, buscamos donde dice *Port 22* o *#Port 22* y lo cambiamos por otro puerto.
Lo ideal, sería ver qué puertos no estan en uso y evitar usar los típicos 222 o 2222... para mayor seguridad

#### Un ejemplo de cómo hacerlo:

`Port 5758`

A partir de ahora, para poder loguearse habra que poner `ssh {user}@{host}:{nuevo puerto}`


### 2° Deshabilitar el login del usuario root y reducir la cantidad máxima de intentos de login
Como todos sabemos, el usuario root es el usuario capaz de tener acceso ilimitado a todos los comandos dentro del servidor, por eso es recomendable, deshabilitar el ingreso con el usuario root, con el fin de reducir el daño posible que nos puedan realizar.
Para eso, debemos entrar al *sshd_config* y buscamos donde dice *#PermitRootLogin* y le agregamos *no*. También, buscamos donde dice *#MaxAuthTries 6* y *UsePAM yes* y los modificamos.

#### Un ejemplo de cómo hacerlo:

`PermitRootLogin no`

`MaxAuthTries 3`

`UsePAM no`


### 3° Limitar el Acceso Únicamente para los usuarios creados
Otra buena práctica, es limitar el ingreso por ssh únicamente a los usuarios que deseemos, con el fin de seguir aportando más seguridad, por eso es recomendable especificar cuáles usuarios estan permitidos loguearse.
Para eso, debemos entrar al *sshd_config*, buscamos donde dice *#AllowUsers*, en caso de que no aparezca, agregar *AllowUsers* y poner los usuarios creados anteriormente.

#### Un ejemplo de cómo hacerlo:

`AllowUsers user1 user2 user3`

			
### 4° Deshabilitar las contraseñas vacías y el Login Basado en Passwords
Al haber seteado el login con claves públicas de SSH, es recomendable deshabilitar el login basado en password, con el fin de seguir aumentando la seguridad en nuestro servidor.
Para esto, debemos entrar al sshd_config y agregamos *AuthenticationMethods* y *PubkeyAuthentication*. También podemos evitar el uso de passwords vacías, para ello, buscamos donde dice *#PermitEmptyPasswords* y le agregamos *no*

#### Un ejemplo de cómo hacerlo:

`PermitEmptyPasswords no`

`AuthenticationMethods publickey`

`PubkeyAuthentication yes`

### 5° Cambiar el Tiempo Maximo de Inactividad
Para evitar que haya sesiones inactivas conectadas, podemos reducir el tiempo máximo permitido.
Para esto, debemos entrar al sshd_config y buscamos donde dice *ClientAliveInterval 300* y reducir dicho número. El número equivale a cantidad de segundos.

#### Un ejemplo de cómo hacerlo:

`ClientAliveInterval 120`

			
### 6° Deshabilitar el archivo .rhosts			
SSH puede emular el comportamiento del ya obsoleto RSH, al permitir a los usuarios habilitar el acceso inseguro a traves del archivo .rhosts, por ello es importante deshabilitarlo.
Para esto, debemos entrar al sshd_config, buscamos donde dice *#IgnoreRhosts yes* y lo descomentamos.

#### Un ejemplo de cómo hacerlo:

`IgnoreRhosts yes`			
							
### 7° Deshabilitar Autenticación basada en el host
La Autenticación basada en el host, es una autenticacion no interactiva, en general es usada para tareas de automatización, pero insegura ya que no controla quien usa el host.
Para esto, debemos entrar al *sshd_config*, buscamos donde dice *#HostbasedAuthentication no* y lo descomentamos.

#### Un ejemplo de cómo hacerlo:

`HostbasedAuthentication no`
			
### 8° Cambiar el protocolo

SSH tiene 2 protocolos que puede usar, el *Protocolo 1* es el más viejo y menos seguro, mientras que el *Protocolo 2* es el que se debería usar para aumentar la seguridad.
Para esto, debemos entrar al *sshd_config*, buscamos donde dice *#Protocol 1* y lo modificamos. En caso de no aparecer, debemos agregarlo.

#### Un ejemplo de cómo hacerlo:

`Protocol 2`

#### Luego de realizar todas las modificaciones necesarias en el *sshd_config*, es necesario guardar los cambios y reiniciar el servicio de SSH para que se efectuen los cambios realizados.
Para esto, debemos ejecutar el comando `sudo service sshd restart`
