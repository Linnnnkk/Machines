
# LinkVortex.htb
Máquina: LinkVortex        OS : Linux       Dificultad : Easy

Obtenemos la ip 10.10.11.47, la analicé con Whatweb por si hubiera alguna redirección a un servidor web, y efectivamente nos llevaba a la página http://linkvortex.htb/.

![whatweb](https://github.com/user-attachments/assets/0f64f3ec-0638-4b8b-9041-72c20da13ed8)

Con lo que edité el archivo /etc/hosts para poder acceder a la página web.

![edthosts](https://github.com/user-attachments/assets/2f37e39d-6049-4b61-a932-7d12a4db9771)

Una vez editado ya tenía acceso a la web http://linkvortex.htb/. 


Al realizar un escaneo visual, pude encontrar un nombre de usuario llamado 'admin', sabiendo que estoy en una GhostCMS, podríamos deducir que existen directorios en el enlace 'linkvortex.htb/ghost/' y así era. 



De manera predeterminada solo con añadir el directorio /ghost/ me redirigió a un panel de login, probé con credenciales predeterminadas pero no funcionó.


![login](https://github.com/user-attachments/assets/6c10b61f-ac88-406e-9768-edfa6b01620b)



Procedí a usar nmap para ver que puertos tenía abiertos. Encontramos dos, puerto 22 para SSH y puerto 80 para Apache httpd web.

![nmap](https://github.com/user-attachments/assets/183b2339-79ed-4d44-a400-befc5ccc6c94)


Seguidamente continué enumerando directorios con dirsearch, primero utilicé http://linkvortex.htb/ para el escaneo y encontré directorios como /rss/, /robots.txt, /sitemap.xml, que eran accesibles y mostraban información sobre que la página efectivamente hace uso del directorio /ghost/.


![dirseach](https://github.com/user-attachments/assets/82a15e61-54d2-47da-88fb-2abf8c79f4c5)


Enumeré los posibles directorios que pudiera haber, pero solo me enumeró uno y fue una clave púlica RSA http://linkvortex.htb/ghost/.well-known/jwks.json con la que no obtuve mucha información significativa, así que me seguí con la enumeración de subdominios.


Enumeración subdominios con wfuzz

![subdomain](https://github.com/user-attachments/assets/046f2fce-7e5a-4f92-9254-f4de52b8a8cf)


Accedí al subdominio dev.linkvortex.htb modificando el archivo hosts para permitirme el acceso y encontré lo que parecía ser una web en construcción.



![construccionn](https://github.com/user-attachments/assets/ad3cdde3-bac0-4557-b799-c8e6f45a630a)





Procedí a una enumeración de directorios con dirsearch de nuevo y encontré un directorio .git abierto que descargué con `wget --mirror -I .git http://dev.linkvortex.htb/.git/`


![git](https://github.com/user-attachments/assets/9c559044-fd09-44e4-bbe2-8b1ba7dd1b30)




Con `git restore .` restauro todos los archivos anteriores y modificaciones que haya habido siempre que estés en el work tree directory.

![restore](https://github.com/user-attachments/assets/c62b27bf-3514-45c5-aa9f-5d3243976204)

Hice cat al archivo 'Dockerfile.ghost' para ver si tenía algo interesante, nombra un archivo 'config.production.json' pero no pude obtener nada más con lo que me puse a buscar archivos con ese nombre o que tuviera palabras clave, 'users', 'password', 'login', 'credentials', 'auhentication', 'db', etc, encontrando un archivo interesante en /dev.linkvortex.htb/ghost/core/test/regression/api/admin/authentication.test.js, en el que podemos ver lo que parece ser una contraseña expuesta. 

![docker](https://github.com/user-attachments/assets/dd1c963d-c23c-4c7c-9032-acc023fcf717)


![usua](https://github.com/user-attachments/assets/96297677-7930-4f35-9260-14316d69f780)


Probamos en http://linkvortex.htb/ghost/#/signin con el usuario que hemos visto antes 'admin', pero como debemos introducir un e-mail, probaremos a usar el e-mail admin@linkvortex.htb

![acceso](https://github.com/user-attachments/assets/1b5fbd7e-f631-4272-8812-13490035c95d)


Al ser la versión 5.58 de GhostCMS es vulnerable al CVE-2024-40028, que permite a usuarios logeados subir archivos symlink con los que se puede ejecutar un Arbitrary File Read que saque información sensible archivos del sistema operativo del servidor.


En github encontramos un script que nos sirve. 

![etc](https://github.com/user-attachments/assets/4fa73557-9983-4205-bdc7-6a23deeab235)

Ahora podemos usar la información del path que nos dió Dockerfile.ghost para ver si podemos ver algo interesante '/var/lib/ghost/config.production.json' y encontramos en SMTP un usuario y contraseña que usaremos en
SSH `ssh bob@10.10.11.47`

![cuenta](https://github.com/user-attachments/assets/cf4cf122-bcf1-4c1c-8b5b-76d24a65c48c)

![whoami](https://github.com/user-attachments/assets/96a8a98c-7b4a-4550-98e8-0606964cfe99)

![user](https://github.com/user-attachments/assets/d4c991b1-e3a1-4f84-b9c9-147575d0fef5)

Usando `python3 -m http.server:8001` creé un servidor para poder descargarme LinEnum.sh de manera remota.

![wget](https://github.com/user-attachments/assets/6c245e50-df8c-4d57-b30f-acbe14fc0d79)

LinEnum nos da unos directorios que pueden ser manipulados.

![sudo](https://github.com/user-attachments/assets/8212ab6e-ab5b-4bec-992c-12735aefbf81)

![escalada](https://github.com/user-attachments/assets/eb284522-13e9-42cb-92a2-3c5bef5269ba)

Podría escalar privilegios ya que me permite ejecutar como sudo comandos en los dos directorios que he visto antes siempre que use la extensión .png y el parámetro CHECK_CONTENT=true para que me permita hacer cat
al archivo donde no tenga permisos.



`bob@linkvortex:~$ ln -s /root/root.txt yuju.txt`


`bob@linkvortex:~$ ls`


`LinEnum.sh  user.txt  yuju.txt`


`bob@linkvortex:~$ ln -s /home/bob/yuju.txt yuju.png`


`bob@linkvortex:~$ ls`


`LinEnum.sh  user.txt  yuju.png  yuju.txt`


`bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/yuju.png`


`Link found [ /home/bob/yuju.png ] , moving it to quarantine`


`Content:`



























 


