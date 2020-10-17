¿Qué es IPTABLES?

# ¿Qué es IPTABLES?
IPTABLES es una herramienta avanzada de filtrado de paquetes en Linux. 
Se encarga de inspeccionar, modificar, reenviar, redirigir y/o eliminar paquetes de red de IP. El código para filtrar paquetes de IP viene integrado en el kernel y está organizado en una colección de 5 tablas, cadenas y reglas, las cuales están asociadas a una eventual acción, que si se cumple, se ejecuta la regla.

### Tablas:

La tabla ***raw*** se encarga de filtrar los paquetes antes que cualquier otra tabla. Se utiliza principalmente para configurar exenciones de seguimiento de conexiones en combinación con el objetivo *NOTRACK*

La tabla ***filter*** es la tabla por defecto.

La tabla ***nat*** se utiliza para la traducción de direcciones de red (por ejemplo redirección de puertos). Debido a las limitaciones en iptables, el filtrado no se debe hacer aquí..

La tabla ***mangle***  se utiliza para la alteración de los paquetes de red especializados (por ejemplo para balanceo de carga)

La tabla ***security*** se utiliza para reglas de conexión de red ***M***andatory ***A***ccess ***C***ontrol* (Por ejemplo en SELinux)

En general, solo se utilizan *filter*, *nat* y *mangle*

### Cadenas:
Las tablas contienen cadenas, que son listas de reglas que se siguen consecutivamente. 
La tabla *filter* contiene tres cadenas integradas: *INPUT*, *OUTPUT* y *FORWARD*
La tabla *nat* incluye las cadenas *PREROUTING*, *POSTROUTING*, y *OUTPUT*.
Las cadenas, por defecto tienen establecido *ACCEPT*, pero se pueden establecer como *DROP*, en caso de querer descartar la conexión. La política predeterminada siempre se aplica al final de una cadena solamente.

`INPUT`: Es el tráfico entrante, luego de haber sido ruteado y destinado al sistema local.
`OUTPUT`: Es el tráfico saliente originado en el sistema local, inmediatamente luego de haber ingresado a la pila de red del kernel.
`FORWARD`: es el tráfico entrante, luego de haber sido ruteado y destinado hacia otro host (reenviado).
`PREROUTING`: Es el tráfico entrante, justo antes de ingresar a la pila de red del kernel. Las reglas en esta cadena son procesadas antes de tomar cualquier decisión de ruteo respecto hacia dónde enviar el paquete.
`POSTROUTING`: Es el tráfico saliente originado en el sistema local o reenviado, luego de haber sido ruteado y justo antes de ser puesto en el cable.
`OUTPUT`: Es el tráfico saliente originado en el sistema local o reenviado, luego de haber sido ruteado y justo antes de ser puesto en el cable.

### Reglas:
El filtrado de los paquetes se basa en reglas, que se especifican por condiciones que el paquete de red debe satisfacer para que se pueda aplicar la regla y una acción a tomar, cuando el paquete coincide con la condicion anterior.
Las acciones se especifican con *-j* o *--jump*. Las reglas pueden ser *ACCEPT*, *DROP*, *QUEUE* , *RETURN* , *REJECT* o *LOG* .

Ahora, sabiendo todo esto

## ¿Cómo podemos mejorar la seguridad en nuestros servidores con IPTABLES?

Una buena práctica, es borrar las reglas anteriores y empezar desde 0.
Para esto, debemos abrir la terminal y escribir:

`iptables -F`
`iptables -P FORWARD ACCEPT`
`iptables -P OUTPUT	ACCEPT`
`iptables -P INPUT ACCEPT`

`-F` - flush --> borra todas las reglas de una cadena
`-P` - policy → explica al kernel qué hacer con los paquetes que no coincidan con ninguna regla.

Luego, seteamos como default, que rechace todo y vamos abriendo puertos y servicios segun necesitemos

`iptables -P INPUT DROP`
`iptables -P FORWARD DROP`
`iptables -P OUTPUT DROP`

Con esto nos quedaremos sin internet, por lo que a continuación debemos empezar a crear reglas permisivas.

### Abrir puerto SSH
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

**RECOMENDACION** 
Conviene cambiar el puerto SSH a otro para evitar los tipicos escaneos de script kiddies y hacer un honeypot o directamente filtrar el puerto. También se puede limitar el ingreso por SSH a IPs fijas.

### Abrir puertos HTTP/HTTPS (Apache)
`iptables -A INPUT -m state --state NEW -p tcp --dport 80 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT`

### Abrir puerto IMAP, SMTP y POP3 (En caso de tener servidor de email)
`iptables -A INPUT -m state --state NEW -p tcp --dport 143 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 25 -j ACCEPT`
`iptables -A INPUT -m state --state NEW -p tcp --dport 110 -j ACCEPT`

### Abrir servidor DNS
`iptables -A INPUT -m state --state NEW -p udp --dport 53 -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 53 -j ACCEPT`

