**Vulnversity**
**Introducción**  
La máquina propuesta para esta semana se trata de una máquina de nuevo de la plataforma THM llamada vulnversity, nivel fácil, vamos a entrenar el reconocimiento, la localización de directorios usando Gobuster, comprometer el webserver y una escalada de privilegios. 

*Atención, recuerda que si durante el writeup ves diferentes IP's objetivo se debe a que el ejercicio se ha llevado a cabo en varias jornadas*

🔍**Reconocimiento con nmap**

Una vez que como de costumbre hemos levantado la máquina y nos ha dado nuestra ip *10.129.170.188*, vamos a hacer un ping para ver si tenemos respuesta y luego un reconocimiento inicial con nmap.
<img width="848" height="301" alt="Pasted image 20260508021359" src="https://github.com/user-attachments/assets/78863f31-b21b-4ee9-93fb-95ca8ab74021" />
<img width="447" height="480" alt="Pasted image 20260508020840" src="https://github.com/user-attachments/assets/a0452b9a-92ea-44cf-ad0b-825bea4f4afb" />
<img width="637" height="717" alt="Pasted image 20260508020824" src="https://github.com/user-attachments/assets/8b703622-cfd4-4678-92b3-2bdf5a8479fc" />

Lanzamos nmap con los siguientes comandos útiles para enfocarnos en reconocer lo máximo posible de nuestro objetivo.
-A: es un reconocimiento agresivo y puede hacer mucho ruido, es decir tenemos que tener cuidado de en que situaciones usarlo, pero nos dice la mayoría de información posible algo parecido a todo en uno.

-p-: con esta opción nmap nos escanea todos los puertos es decir los 65535 disponibles.

-sV: Una vez que tenemos los puertos abiertos al descubierto le lanzamos un -sV que nos va a decir que versión tienen los puertos encontrados esto nos va a ser muy útil para encontrar vulnerabilidades a la hora de querer entrar.

-v: Por último el verbose mode, es un modo bastante detallado que tarda algo más pero te ofrece toda la información completa mientras vas viendo el progreso en tiempo real.
<img width="593" height="485" alt="Pasted image 20260508021752" src="https://github.com/user-attachments/assets/db489d66-3d4c-43ac-a7e2-4b5ebd71d3ad" />
Con este reconocimiento ya podemos ver que en la máquina objetivo hemos descubierto cosas bastantes interesantes, tiene 6 puertos abiertos, el sistema operativo más usado es Ubuntu, en el puerto 3333/tcp tiene un servidor web con apache, y la versión del http proxy en el 3128/tcp es squid 4.10 . 

Con todo este reconocimiento tenemos una superficie de ataque bastante amplia con varios vectores que probar.

🔎**Enumeración con gobuster**
Gobuster es una herramienta de fuerza bruta contra rutas de ficheros o url por ejemplo. Esta herramienta es buena para enumeración por el hecho de que al igual que nmap tiene varias opciones que según nuestra situación podemos usar. La instalamos con el comando *sudo apt-get install gobuster *
<img width="679" height="237" alt="Pasted image 20260508022538" src="https://github.com/user-attachments/assets/ffc72d2c-6cb5-4008-aece-948cb548535a" />Al igual que vimos en otras máquinas con John the ripper, gobuster también necesita una lista de comparación para usar la fuerza bruta, en nuestro caso que usamos kali linux, por defecto tenemos varias listas incluidas en el directorio /usr/share/wordlists/dirbuster/
<img width="1036" height="395" alt="Pasted image 20260508023410" src="https://github.com/user-attachments/assets/36861d99-8350-4835-b1b7-68565229934e" />
Le indicamos a la herramienta *gobuster dir -u (ip objetivo) -w(para adjuntar la wordlist) y la ruta de la wordlist que usaremos*.

como vemos mientras esta haciendo la busqueda nos aparece ya unas urls donde parece que hay imagenes, css, fuentes e internal que es el que me llama la atención, veamos que encontramos si ponemos esta url en el navegador.
<img width="648" height="240" alt="Pasted image 20260508032353" src="https://github.com/user-attachments/assets/23f42bee-2f08-425c-9efe-a80bce88e225" />
<img width="789" height="685" alt="Pasted image 20260508023839" src="https://github.com/user-attachments/assets/ceef77ec-bda9-481d-ac41-328cd0717dc6" />
<img width="899" height="591" alt="Pasted image 20260508023810" src="https://github.com/user-attachments/assets/951e61e0-a4f1-408b-a998-a641d929dbfe" />
<img width="876" height="530" alt="Pasted image 20260508023754" src="https://github.com/user-attachments/assets/9f6ee9d6-1261-425a-80f6-8f7d600125c0" />
<img width="1187" height="740" alt="Pasted image 20260508023935" src="https://github.com/user-attachments/assets/b3908dd3-de2e-40e6-bada-ff623c22ced9" />


Gobuster y dirbuster son herramientas que funcionalmente hacen lo mismo pero se estructuran de manera diferente. Gobuster como su nombre indica está escrito en go mientras que dirbuster no es una herramienta como tal si no un script escrito en python, por lo que consume más memoria ram y es un poco más lento que gobuster, sin embargo dirbuster tiene un filtro avanzado mejor.

💥**Compromiso de webserver con BurpSuite**

Uno de los vectores que mas nos convence para atacar esta máquina es el formulario de subida que tenemos en la web, ya que si somos capaces de elevar el ataque a subir un archivo que se ejecute nos habilitaría el control del server. Para ello vamos a hacer uso de la herramienta Burpsuite, es la primera vez que la uso así que voy a hacer un poco de researching, pero basicamente lo que hace es interceptar toda las peticiones entre nosotros y cualquier app web, pudiendo ver, modificar y atacar desde ahí.
![Uploading Pasted image 20260508032353.png…]()
Vamos a configurar la herramienta de la manera adecuada para interceptar el tráfico, subir un archivo y activar el modo intruder de la herramienta, este modo va a automatizar un ataque enviando varias peticiones a la vez para que podamos cargar un payload.

Lo primero que vamos a hacer es crear una wordlist, ejecutamos *nano extensions.txt* y añadimos las siguientes palabras
<img width="336" height="246" alt="Pasted image 20260508032855" src="https://github.com/user-attachments/assets/cec774fa-811c-4de7-8d3b-335b00e646df" />
Ahora vamos a la herramienta y desde el navegador con el proxy interceptando nos vamos a la url que nos permite subir un archivo.
<img width="1262" height="600" alt="Pasted image 20260508033602" src="https://github.com/user-attachments/assets/3123147e-8bb6-4b9a-adb2-2a76b036a8c5" />



Vale hemos interceptado la petición y la hemos mandado al módulo intruder, ese modulo es el encargado de lanzar y automatizar el ataque. ![[Pasted image 20260508034003.png]]
Aquí le hemos indicado que ahora cuando suba el archivo pruebe esa extensión a cambiarla por la lista que hemos realizado antes.
<img width="704" height="595" alt="Pasted image 20260508034003" src="https://github.com/user-attachments/assets/9c4ae755-53e1-4f67-b1fe-051bb94c3831" />
El ataque que va a realizar es de tipo snipper, es decir va a ir probando una extensión cada vez que es el ataque más adecuado para hacer fuzzing de extensiones que es lo que buscamos.

Vale aquí ha habido un aprendizaje bastante grande, es decir un error de más de media hora buscando soluciones. Cuando vamos a hacer fuzzing a una extensión, debemos de asegurarnos que a la hora de hacer wordlist no incluya el . de la extensión, porque la herramienta no lo va a procesar como un . y me daba fallo una y otra vez, tanto a la hora de seleccionar el token en position como la wordlist JAMAS se incluye el .
<img width="1332" height="471" alt="Captura de pantalla 2026-05-08 040721" src="https://github.com/user-attachments/assets/10ef282c-273d-4dff-a138-f8e91ed43504" />
Aquí podemos ver como efectivamente la extensión *.phtml *la admitido, así que vamos a explotar esta vulnerabilidad y vamos a subirle una reverse shell para podernos conectar al server.
<img width="1431" height="416" alt="Pasted image 20260508041924" src="https://github.com/user-attachments/assets/fe8d7e82-8984-4d7e-be82-095ab2f8b4eb" />

<img width="693" height="156" alt="Pasted image 20260508042003" src="https://github.com/user-attachments/assets/833b91f4-0976-457e-bcec-f5e295d7da39" />
Vamos a abrir nuestro listener con el comando *nc -lvnp 1234* y nos vamos a poner en escucha.
<img width="886" height="179" alt="Pasted image 20260508042734" src="https://github.com/user-attachments/assets/384a8de3-0534-446c-8589-becbf95713b1" />
En cuanto visitamos la url del archivo se ejecuta y nuestro terminal se conecta.
<img width="1391" height="386" alt="Pasted image 20260508042803" src="https://github.com/user-attachments/assets/239e92cb-69d6-4970-aee2-a93351f1a853" />
Ahora que estamos dentro vamos a ejecutar*whoami, id, y ls** para ver que nos encontramos.
<img width="1070" height="692" alt="Pasted image 20260508043120" src="https://github.com/user-attachments/assets/ef73194f-758c-4acb-a953-1f9acca1b82f" />
Aquí toca de nuevo reconocer, enumerar y buscar para escalar privilegios. Me he entrado en la carpeta home para buscar a ver quien es el usuario y que tiene por ahí, hemos encontrado que el usuario es bill y que tenía un archivo de texto llamado user con una flag
<img width="434" height="488" alt="Pasted image 20260508043922" src="https://github.com/user-attachments/assets/3868998e-aa33-4ed7-8edf-4a19b9aa9359" />


🔐**Escalada de privilegios con SUID**
Ahora que ya hemos comprometido el sistema y estamos dentro del server vamos a hacer una búsqueda de lo que aprendimos en la primera máquina, los archivos con permiso SUID. Vamos a ejecutar el comando *find / -perm -4000 2>/dev/null*  con esto buscamos en todo linux los directorios y archivos con permisos suid, y con la última parte del comando eliminamos errores que nos puedan aparecer.

<img width="702" height="573" alt="Pasted image 20260508050107" src="https://github.com/user-attachments/assets/8fe846d8-e74f-4dc8-b393-204114a79fea" />
Ya de un vistazo vemos una cosa que nos llama la atención, que es el directorio */bin/systemtcl* que systemctl tenga permiso SUID es un peligro porque es el encargado de controlar servicios del sistema y ejecutar procesos. 

Lo que vamos a hacer va a ser crear un servicio malicioso para que lo ejecute systemtcl y el solo nos de permiso root. 
<img width="655" height="281" alt="Pasted image 20260508051155" src="https://github.com/user-attachments/assets/460251d8-559a-4a9c-9f2f-2db10e7a73f2" />
<img width="1040" height="414" alt="Pasted image 20260508091020" src="https://github.com/user-attachments/assets/1410e6c3-5ca9-4709-a77f-e5293c168dbe" />

Aquí paso por paso, una vez que hemos metido nuestro servicio malicioso en *cd /tmp/* ejecutamos el comando */bin/systemctl link /tmp/root.service* esto le dice al systemctl de la máquina que haga link con los ficheros del directorio /tmp/ que son ficheros de servicio de sistema válidos.

Luego comando */bin/systemctl enable --now root.service* con esto le indicamos que lo active inmediatamente.
Por último comprobamos los permisos *ls -l /bin/bash y ejecutamos */bin/bash -p* (para preservar los privilegios ahora que lo lanzamos como SUID)y efectivamente con *whoami* podemos ver que ya somos root y tenemos los máximos privilegios de la máquina.


🏁**Post-explotación /root**

Una vez que hemos ganado privilegio en root, vamos a ver si podemos encontrar algo, ya que tenemos acceso vamos a ir al directorio de root y ver que hay . *cd/root/*
Haciendo un *ls* vemos que hay un archivo llamado *root.txt* y que al abrirlo nos muestra un código flag.

<img width="866" height="362" alt="Pasted image 20260508101123" src="https://github.com/user-attachments/assets/8f3ecdda-b015-420d-bc12-2d596cc4c5a7" />
<img width="1035" height="510" alt="Pasted image 20260508101112" src="https://github.com/user-attachments/assets/b4256564-3af5-44e2-af58-e476e226396c" />

💡**Conclusiones**  

Esta máquina me ha gustado bastante tiene una dinámica diferente y totalmente paralela de herramientas pero que luego se acaban uniendo con un propósito. Aunque es mi tercera máquina ya voy teniendo un dominio más ligero en especial en cuanto a la etapa de reconocimiento y enumeración porque ya aprendes a observar donde puede estar el vector de ataque de la máquina o lo que podemos hacer. El hecho de usar herramientas diferentes en cada máquina también te enseña a hacer comparativa entre ellas y hacer un buen researching de para que sirve cada una y en que casos debes usarlas y cuales no.

🧠**Lecciones aprendidas**

Muchas veces al igual que en otras máquinas entras en bucle por un fallo tonto, ahí es donde ya vas oliendo que vas a aprender porque una cosa simple que debería de llevarte poco más de 1 minuto de repente se queda estancada y te puede coger casi 30, es aquí donde realmente se aprende, tanto técnicamente como personalmente en cuanto a la paciencia, la resiliencia y la capacidad de observación más allá de lo puramente lógico. Una cosa que me encanta es que aunque no sale en el writeup, muchas veces yo mismo tomo un camino paralelo y voy ejecutando herramientas anteriores para ver como funcionan en esta máquina o si pueden aportar algo más.
