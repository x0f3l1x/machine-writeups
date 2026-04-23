**Introducción**

<img width="128" height="128" alt="OcA2KrK" src="https://github.com/user-attachments/assets/148af5e5-d86a-4ee6-943a-32dd149e25b7" />

Se trata de una maquina de nivel fácil de la plataforma Tryhackme, iré haciendo un recorrido paso a paso por cada reto y resolución que se vaya planteando. Mi sistema para resolverlo es un entorno virtual con Kali Linux. La maquina de por si ofrece varias tareas a resolver, desplegar la maquina vulnerable, enumerar samba para tomar acciones, acceder inicialmente con ProFtpd y por último escalada de privilegios con manipulación de variación de ruta.
*Si durante el writeup la IP objetivo cambia, es porque este ejercicio se llevo a cabo en jornadas diferentes*

🔍**Reconocimiento**

Comenzamos accediendo a la plataforma en sí y resolviendo la primera task que es desplegar la maquina vulnerable. La plataforma es bastante amigable y te va guiando paso a paso, ya nos ha dado la IP para en este caso 10.128.174.237.
<img width="1386" height="287" alt="capt" src="https://github.com/user-attachments/assets/a55b193d-d590-4743-8f70-35704da25409" />


Vamos a abrir la máquina atacante kali linux, acceder a la terminal y hacer un primer ping para ver si conectamos con la máquina objetivo. Nos ha conectado perfectamente asi que vamos a comenzar con un escaneo básico de nmap simplemente el comando nmap 10.128.174.237. 

🔎**Enumeración**

El comando nmap nos muestra ya que tenemos 7 puertos abiertos, esto es una de las respuesta que nos requiere la interfaz de tryhackme para continuar el reto de la máquina.
<img width="696" height="639" alt="capt2" src="https://github.com/user-attachments/assets/1e063a35-b232-4ea3-8203-ca9ca3d71fa2" />


Introducimos la respuesta en la web y nos muestra que esta correcta y podemos continuar al siguiente task.
<img width="1380" height="481" alt="capt3" src="https://github.com/user-attachments/assets/c7350c5e-6e34-44c1-acfa-cd4270cf01d7" />


En el siguiente task empieza la plataforma explicando que es Samba. Nos explican que es el conjunto de programas de interoperaibilidad standard de Windows y Unix, y que permite a los usuarios acceder y utilizar archivos y otros recursos comunmente compartidos en una intranet.

Nos dicen que Nmap tiene la habilidad de enumerar los recursos SMB compartidos, y que existe un script para enumerar una amplia variedad de tareas y recursos compartidos. nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.128.174.237.
<img width="1107" height="583" alt="Captura de pantalla 2026-04-21 230546" src="https://github.com/user-attachments/assets/015dc5b1-674c-4f8c-acc8-d0b289f80a89" />

Con el comando que me ha proporcionado la plataforma no me aparecía nada, dado que hay que rellenar una task que te pregunta cuantos servicios hay compartidos, finalmente he encontrado que era mejor ejecutar smbclient -L para listar y -N para intentar el acceso sin contraseña , ha fallado el acceso sin contraseña por protocolo SMB1 pero hemos encontrado que tiene 3 recursos compartidos.
Ahora vamos a ejecutar el comando smbclient //10.128.174.237/anonymous para intentar conectarnos como un cliente invitado anónimo, le damos a enter a la password y nos deja acceder sin esta.
<img width="871" height="273" alt="Captura de pantalla 2026-04-21 231543" src="https://github.com/user-attachments/assets/bc466dcf-8294-4fac-bd0c-b8eb8d650957" />

Dentro de SMB ejecutamos el comando ls para que nos liste que hay, y encontramos que hay un fichero llamado log.txt que se nos solicita en una de las taks.
<img width="1494" height="510" alt="Captura de pantalla 2026-04-21 231148" src="https://github.com/user-attachments/assets/67611f68-0af2-42e8-8160-84af3d168387" />

Introducimos el nombre y efectivamente es ese. La web de tryhackme me recomienda que ahora ejecute esto smbget -R smb://10.128.174.237/anonymous para descargar los archivos compartidos, sin embargo no me funciona. Asi que investigando por internet encuentro que acceda de nuevo a smbcliente, descargue desde ahí log.txt y luego lo abra y tachan! tenemos una información realmente valiosa!
<img width="761" height="527" alt="Captura de pantalla 2026-04-21 232511" src="https://github.com/user-attachments/assets/cdcdbfe0-b513-484a-af5a-1c9abe3fbd82" />

Luego la plataforma nos indica que cuando hicimos el escaneo había abierto un puerto el 111 con un servicio llamado rpcbind, este servicio es un server que convierte el número de programa  remote procedure call(rpc) en direcciones universales. Cuando se inicia un servicio en RPC le dice a rpcbind la dirección que esta escuchando y el número de programa que esta preparado para servic, en nuestro caso es un acceso a un sistema NFS. La plataforma nos sugiere el siguiente comando para enumerar esto nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.128.174.237.
<img width="777" height="545" alt="Captura de pantalla 2026-04-21 232743" src="https://github.com/user-attachments/assets/3b67b897-deee-4e7d-952e-ea06f93af409" />


💥**Obtención de acceso inicial** 

En esta parte la plataforma comienza hablándonos de que es ProFTP, un servidor open-source FTP compatible con UNIX y windows que además ha sido vulnerable en versiones de software pasadas. La primera task que nos pide para ganar acceso la plataforma es que ejecutemos un netcat para conectar la máquina al puerto FTP. El puerto ftp es el 21 asi que ejecutaremos el comando nc 10.128.174.237 21
<img width="711" height="240" alt="Captura de pantalla 2026-04-21 234115" src="https://github.com/user-attachments/assets/546cbaa1-22b2-4194-920e-bb255bff9c87" />

Ahora que sabemos la versión que usa vamos a usar la herramienta searchsploit para ver si hay algún exploit para esa versión de software específico, para ello ejecutamos el comando searchsploit proftpd 1.3.5
<img width="615" height="150" alt="Captura de pantalla 2026-04-22 113241" src="https://github.com/user-attachments/assets/1560a72c-874d-4242-b2b5-7f8890222d4c" />

Disponemos de 4 exploit reconocidos. Vamos a usar el exploit mod_copy, este exploit lo que permite es activar los comandos SITE CPFR Y SITE CPTO, estos comandos nos sirven para copiar y mover de un sitio a otro del servidor con cualquier cliente sin autenticar.
<img width="625" height="364" alt="Captura de pantalla 2026-04-22 113206" src="https://github.com/user-attachments/assets/4ef690b6-ea0f-4408-8057-21501f9dba02" />

En el paso anterior cuando tuvimos acceso al log.txt pudimos ver que las credenciales de identificación se habían guardado en una ruta específica, asi que ahora usaremos el exploit anterior con esta ruta para moverlo a el disco que hemos visto montado antes llamado /var
<img width="944" height="293" alt="Captura de pantalla 2026-04-21 234241" src="https://github.com/user-attachments/assets/84042a63-73c1-4f0b-b673-81cef1122344" />

Con ambos comando tenemos listo nuestro archivo y movido, ahora vamos a montar ese disco en nuestra máquina. Usamos el comando mkdir /mnt/kenobiNFS, luego el comando mount 10.129.133.77:/var /mnt/kenobiNFS  y por ultimo listamos con ls -la /mnt/kenobiNFS.
<img width="696" height="627" alt="Captura de pantalla 2026-04-22 113744" src="https://github.com/user-attachments/assets/ed6942bb-7dd8-4ff4-b435-31135656b674" />

Como podemos apreciar uno de los errores que he aprendido aquí es que estos comandos deben de ejecutarse con sudo previamente para tener permisos root.

Ahora ya tenemos montado la red nfs en nuestra máquina vamos a ir al directorio donde hemos movido las credenciales y vamos a abrirlas.

Tras copiar la clave privada desde el recurso NFS, el primer intento de autenticación SSH falló. El problema no estaba en la clave, sino en los permisos locales del archivo en la máquina atacante, ya que había sido copiado con sudo y pertenecía a root. Una vez corregida la propiedad del fichero, fue posible autenticarse como el usuario kenobi.
<img width="1119" height="630" alt="Captura de pantalla 2026-04-22 115707" src="https://github.com/user-attachments/assets/28ea22da-6870-4f6f-b479-72e235c07136" />

Ahora que ya por fin tengo acceso la plataforma te pregunta que cual es el texto de user.txt dentro de kenobi, asi que usamos el comando cat para que nos de la respuesta.
<img width="1032" height="223" alt="Captura de pantalla 2026-04-22 115856" src="https://github.com/user-attachments/assets/ac601fbe-169c-4693-844c-e80e2f05ff5a" />

pegamos la respuesta en la plataforma y la tenemos correcta, hemos  completado el tercer task de la máquina!
<img width="1377" height="385" alt="capt17" src="https://github.com/user-attachments/assets/d0250aa2-ee28-43b1-8030-00746dcd7ee8" />


🔐**Escalada de privilegios**

En este último task vamos a aprender la escala de privilegios con la manipulación de ruta variable. La plataforma nos enseña la diferente entre SUID, SGID y sticky notes. Son tres permisos que se ejecutan con un bit diferenciador cuando se escribe un comando, SUID te permite tener los mismos privilegios que el dueño del archivo, por lo tanto si el dueño es root ejecutaras el archivo como root, aunque tu no lo seas. SGID es algo parecido pero con los permisos del grupo propietario  y por último sticky es un bit que se añade que previene que el fichero sea borrado por otros usuarios.
La plataforma nos explica que los SUID pueden ser peligrosos debido a que algunos binarios  el cambio de contraseña con passwd necesitan ejecutarse con elevados privilegios, pero algunos archivos custom pueden tener ese SUID, asi que nos sugieren un comando para buscar este tipo de archivo find / -perm -u=s -type f 2>/dev/null

<img width="590" height="557" alt="Captura de pantalla 2026-04-22 130736" src="https://github.com/user-attachments/assets/28e4490d-b1e5-4ddd-b3ac-cab311085c77" />


Una vez ejecutamos la plataforma nos pregunta si vemos algún tipo de archivo raro o fuera de lo común, para esto hice un research en internet ya que al ser mi primera máquina no se que es lo común o lo que no, asi que encontré que era usr/bin/menu, podría ser algún tipo de binario custom, efectivamente al introducir esta respuesta en la plataforma me aparecía correcta.
<img width="1386" height="287" alt="capt" src="https://github.com/user-attachments/assets/be420dcc-3d28-45a3-a93e-dda1b88dc54d" />

Procedemos a ejecutar la ruta y nos aparecen 3 opciones
<img width="769" height="401" alt="Captura de pantalla 2026-04-22 122627" src="https://github.com/user-attachments/assets/29cf533f-c842-4184-a791-540b2e479f48" />

La plataforma nos enseña que con el comando strings antes de ejecutar otro comando muestra las líneas legibles de un binario.(buen aprendizaje)
<img width="573" height="534" alt="Captura de pantalla 2026-04-22 131423" src="https://github.com/user-attachments/assets/33a169e8-8568-4ab4-aefb-d9f5a45a6e03" />

Con esto acabamos de aprender que el sistema por así decirlo cuando ejecutas la opción 1, dice algo como traedme a curl, pero claro no dice de donde, ahí es donde entra nuestro ataque, en modificar la ruta para que nos traiga lo que nosotros queremos un curl falso, en este caso una shell.
<img width="1136" height="265" alt="Captura de pantalla 2026-04-22 181813" src="https://github.com/user-attachments/assets/8ce40a47-a4b8-4c98-81f8-3cd420cddfb4" />

Accedemos al directorio /tmp y con el comando echo "/bin/sh" > curl
chmod +x curl(en este caso usamos 777 que es el equivalente númerico) creamos el falso curl y por último le decimos que ahora la ruta esta en tmp.

🏁**Flags**

De esta manera cuando ejecutamos de nuevo /ust/bin/menu y le damos a la opción 1 (curl) nos confirma que hemos accedido y si ejecutamos id o whoami nos dirá que somos root.

<img width="1150" height="242" alt="Captura de pantalla 2026-04-22 181950" src="https://github.com/user-attachments/assets/6e02d3cd-738d-45c9-8b9a-0f98f98fdb5e" />

Por último la plataforma nos pide la flag que esta en el archivo root.txt en la ruta /root/root.txt asi que ejecutamos un cat y TACHAN! tenemos nuestra flag.
<img width="956" height="645" alt="Captura de pantalla 2026-04-22 181937" src="https://github.com/user-attachments/assets/67d69ee7-a359-430e-9ef0-01eef01a08e0" />

Primera máquina vulnerada!!! Enhorabuena!!

💡**Conclusiones**

Se trata de una máquina de nivel fácil pero es imprescindible tener nociones básicas de algunas cosas especialmente de Linux. Si algún novato le gustaría intentarla le animo a ello y si no entiende algo que lo vaya buscando el porque es así y como ha llegado hasta ahí. La importancia de esta máquina y donde estaría centrado sobre todo el core de aprendizaje sería en dos cosas fundamentales. La primera que cuanto más puertos abiertos o expuestos significa una superficie de ataque más amplia para buscar vulnerabilidades y empezar con un acceso inicial. La segunda la configuración y el hardering de sistemas es esencial, en este caso el hecho de que un directorio custom exponga un permiso SUID es una falta de atención por parte del implementador y podría acabar en la fuga o perdida de datos de gran potencial en un caso real e incluso el compromiso del funcionamiento del sistema.

🧠**Lecciones aprendidas**

Teniendo en cuenta que esta ha sido mi primera máquina de verdad, hecha desde cero paso a paso y sobre todo entendiendo que comandos usar y porque se usan. Normalmente había competido en varias CTF's pero gracias a la ayuda de la IA muchos comandos se ejecutaban casi sin pensar el porque de eso, el hecho de centrarte en que estas usando y porque lo haces es un game changing total y una lección de aprendizaje enorme porque acabas la máquina con muchísimos conocimientos. Esta máquina aunque tiene un tiempo medio de 45 minutos yo le habré dedicado unas 3 horas precisamente por eso, porque me he obligado a entender el como y porqué de cada uno de los comandos que he usado.
