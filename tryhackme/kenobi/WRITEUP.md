**Introducción**
Se trata de una maquina de nivel fácil de la plataforma Tryhackme, iré haciendo un recorrido paso a paso por cada reto y resolución que se vaya planteando. Mi sistema para resolverlo es un entorno virtual con Kali Linux. La maquina de por si ofrece varias tareas a resolver, desplegar la maquina vulnerable, enumerar samba para tomar acciones, acceder inicialmente con ProFtpd y por último escalada de privilegios con manipulación de variación de ruta.
*Si durante el writeup la IP objetivo cambia, es porque este ejercicio se llevo a cabo en jornadas diferentes*

🔍**Reconocimiento**

Comenzamos accediendo a la plataforma en sí y resolviendo la primera task que es desplegar la maquina vulnerable. La plataforma es bastante amigable y te va guiando paso a paso, ya nos ha dado la IP para en este caso 10.128.174.237.
<img width="1386" height="287" alt="capt" src="https://github.com/user-attachments/assets/a55b193d-d590-4743-8f70-35704da25409" />


Vamos a abrir la máquina atacante kali linux, acceder a la terminal y hacer un primer ping para ver si conectamos con la máquina objetivo. Nos ha conectado perfectamente asi que vamos a comenzar con un escaneo básico de nmap simplemente el comando nmap 10.128.174.237. 

🔎**Enumeración**

El comando nmap nos muestra ya que tenemos 7 puertos abiertos, esto es una de las respuesta que nos requiere la interfaz de tryhackme para continuar el reto de la máquina.
![[capt2.png]]]

Introducimos la respuesta en la web y nos muestra que esta correcta y podemos continuar al siguiente task.
![[capt3.png]]

En el siguiente task empieza la plataforma explicando que es Samba. Nos explican que es el conjunto de programas de interoperaibilidad standard de Windows y Unix, y que permite a los usuarios acceder y utilizar archivos y otros recursos comunmente compartidos en una intranet.

Nos dicen que Nmap tiene la habilidad de enumerar los recursos SMB compartidos, y que existe un script para enumerar una amplia variedad de tareas y recursos compartidos. nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.128.174.237.![[Captura de pantalla 2026-04-21 230546.png]]Con el comando que me ha proporcionado la plataforma no me aparecía nada, dado que hay que rellenar una task que te pregunta cuantos servicios hay compartidos, finalmente he encontrado que era mejor ejecutar smbclient -L para listar y -N para intentar el acceso sin contraseña , ha fallado el acceso sin contraseña por protocolo SMB1 pero hemos encontrado que tiene 3 recursos compartidos.
Ahora vamos a ejecutar el comando smbclient //10.128.174.237/anonymous para intentar conectarnos como un cliente invitado anónimo, le damos a enter a la password y nos deja acceder sin esta.
![[Captura de pantalla 2026-04-21 231543.png]]Dentro de SMB ejecutamos el comando ls para que nos liste que hay, y encontramos que hay un fichero llamado log.txt que se nos solicita en una de las taks.
![[Captura de pantalla 2026-04-21 231148.png]]Introducimos el nombre y efectivamente es ese. La web de tryhackme me recomienda que ahora ejecute esto smbget -R smb://10.128.174.237/anonymous para descargar los archivos compartidos, sin embargo no me funciona. Asi que investigando por internet encuentro que acceda de nuevo a smbcliente, descargue desde ahí log.txt y luego lo abra y tachan! tenemos una información realmente valiosa!
![[Captura de pantalla 2026-04-21 232511.png]]Luego la plataforma nos indica que cuando hicimos el escaneo había abierto un puerto el 111 con un servicio llamado rpcbind, este servicio es un server que convierte el número de programa  remote procedure call(rpc) en direcciones universales. Cuando se inicia un servicio en RPC le dice a rpcbind la dirección que esta escuchando y el número de programa que esta preparado para servic, en nuestro caso es un acceso a un sistema NFS. La plataforma nos sugiere el siguiente comando para enumerar esto nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.128.174.237.![[Captura de pantalla 2026-04-21 232743.png|601]]

💥**Obtención de acceso inicial** 

En esta parte la plataforma comienza hablándonos de que es ProFTP, un servidor open-source FTP compatible con UNIX y windows que además ha sido vulnerable en versiones de software pasadas. La primera task que nos pide para ganar acceso la plataforma es que ejecutemos un netcat para conectar la máquina al puerto FTP. El puerto ftp es el 21 asi que ejecutaremos el comando nc 10.128.174.237 21![[Captura de pantalla 2026-04-21 234115.png]]Ahora que sabemos la versión que usa vamos a usar la herramienta searchsploit para ver si hay algún exploit para esa versión de software específico, para ello ejecutamos el comando searchsploit proftpd 1.3.5
![[Captura de pantalla 2026-04-21 234241.png]]Disponemos de 4 exploit reconocidos. Vamos a usar el exploit mod_copy, este exploit lo que permite es activar los comandos SITE CPFR Y SITE CPTO, estos comandos nos sirven para copiar y mover de un sitio a otro del servidor con cualquier cliente sin autenticar.
![[Captura de pantalla 2026-04-22 113206.png]]En el paso anterior cuando tuvimos acceso al log.txt pudimos ver que las credenciales de identificación se habían guardado en una ruta específica, asi que ahora usaremos el exploit anterior con esta ruta para moverlo a el disco que hemos visto montado antes llamado /var![[Captura de pantalla 2026-04-22 113241.png]]Con ambos comando tenemos listo nuestro archivo y movido, ahora vamos a montar ese disco en nuestra máquina. Usamos el comando mkdir /mnt/kenobiNFS, luego el comando mount 10.129.133.77:/var /mnt/kenobiNFS  y por ultimo listamos con ls -la /mnt/kenobiNFS. ![[Captura de pantalla 2026-04-22 113744.png]]Como podemos apreciar uno de los errores que he aprendido aquí es que estos comandos deben de ejecutarse con sudo previamente para tener permisos root.

Ahora ya tenemos montado la red nfs en nuestra máquina vamos a ir al directorio donde hemos movido las credenciales y vamos a abrirlas.

Tras copiar la clave privada desde el recurso NFS, el primer intento de autenticación SSH falló. El problema no estaba en la clave, sino en los permisos locales del archivo en la máquina atacante, ya que había sido copiado con sudo y pertenecía a root. Una vez corregida la propiedad del fichero, fue posible autenticarse como el usuario kenobi.![[Captura de pantalla 2026-04-22 115707.png]]Ahora que ya por fin tengo acceso la plataforma te pregunta que cual es el texto de user.txt dentro de kenobi, asi que usamos el comando cat para que nos de la respuesta.![[Captura de pantalla 2026-04-22 115856.png]]pegamos la respuesta en la plataforma y la tenemos correcta, hemos  completado el tercer task de la máquina!
![[capt17.png]]

🔐**Escalada de privilegios**

En este último task vamos a aprender la escala de privilegios con la manipulación de ruta variable. La plataforma nos enseña la diferente entre SUID, SGID y sticky notes. Son tres permisos que se ejecutan con un bit diferenciador cuando se escribe un comando, SUID te permite tener los mismos privilegios que el dueño del archivo, por lo tanto si el dueño es root ejecutaras el archivo como root, aunque tu no lo seas. SGID es algo parecido pero con los permisos del grupo propietario  y por último sticky es un bit que se añade que previene que el fichero sea borrado por otros usuarios.
La plataforma nos explica que los SUID pueden ser peligrosos debido a que algunos binarios  el cambio de contraseña con passwd necesitan ejecutarse con elevados privilegios, pero algunos archivos custom pueden tener ese SUID, asi que nos sugieren un comando para buscar este tipo de archivo find / -perm -u=s -type f 2>/dev/null

![[Captura de pantalla 2026-04-22 130736.png|586]]

Una vez ejecutamos la plataforma nos pregunta si vemos algún tipo de archivo raro o fuera de lo común, para esto hice un research en internet ya que al ser mi primera máquina no se que es lo común o lo que no, asi que encontré que era usr/bin/menu, podría ser algún tipo de binario custom, efectivamente al introducir esta respuesta en la plataforma me aparecía correcta.![[Captura de pantalla 2026-04-22 122445.png]]Procedemos a ejecutar la ruta y nos aparecen 3 opciones![[Captura de pantalla 2026-04-22 122627.png]]La plataforma nos enseña que con el comando strings antes de ejecutar otro comando muestra las líneas legibles de un binario.(buen aprendizaje)

![[Captura de pantalla 2026-04-22 131423.png]]
Con esto acabamos de aprender que el sistema por así decirlo cuando ejecutas la opción 1, dice algo como traedme a curl, pero claro no dice de donde, ahí es donde entra nuestro ataque, en modificar la ruta para que nos traiga lo que nosotros queremos un curl falso, en este caso una shell.

![[Captura de pantalla 2026-04-22 181813.png]]Accedemos al directorio /tmp y con el comando echo "/bin/sh" > curl
chmod +x curl(en este caso usamos 777 que es el equivalente númerico) creamos el falso curl y por último le decimos que ahora la ruta esta en tmp.

🏁**Flags**

De esta manera cuando ejecutamos de nuevo /ust/bin/menu y le damos a la opción 1 (curl) nos confirma que hemos accedido y si ejecutamos id o whoami nos dirá que somos root.

![[Captura de pantalla 2026-04-22 181950.png]]Por último la plataforma nos pide la flag que esta en el archivo root.txt en la ruta /root/root.txt asi que ejecutamos un cat y TACHAN! tenemos nuestra flag.![[Captura de pantalla 2026-04-22 181937.png]]Primera máquina vulnerada!!! Enhorabuena!!

💡**Conclusiones**
Se trata de una máquina de nivel fácil pero es imprescindible tener nociones básicas de algunas cosas especialmente de Linux. Si algún novato le gustaría intentarla le animo a ello y si no entiende algo que lo vaya buscando el porque es así y como ha llegado hasta ahí. La importancia de esta máquina y donde estaría centrado sobre todo el core de aprendizaje sería en dos cosas fundamentales. La primera que cuanto más puertos abiertos o expuestos significa una superficie de ataque más amplia para buscar vulnerabilidades y empezar con un acceso inicial. La segunda la configuración y el hardering de sistemas es esencial, en este caso el hecho de que un directorio custom exponga un permiso SUID es una falta de atención por parte del implementador y podría acabar en la fuga o perdida de datos de gran potencial en un caso real e incluso el compromiso del funcionamiento del sistema.

🧠**Lecciones aprendidas**

Teniendo en cuenta que esta ha sido mi primera máquina de verdad, hecha desde cero paso a paso y sobre todo entendiendo que comandos usar y porque se usan. Normalmente había competido en varias CTF's pero gracias a la ayuda de la IA muchos comandos se ejecutaban casi sin pensar el porque de eso, el hecho de centrarte en que estas usando y porque lo haces es un game changing total y una lección de aprendizaje enorme porque acabas la máquina con muchísimos conocimientos. Esta máquina aunque tiene un tiempo medio de 45 minutos yo le habré dedicado unas 3 horas precisamente por eso, porque me he obligado a entender el como y porqué de cada uno de los comandos que he usado.
