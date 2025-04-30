## Reconocimiento

Escaneo con nmap, vemos que nos lista algunos directorios

![1](https://github.com/user-attachments/assets/c48a4f5c-131a-45c9-b7b8-6accf1274bd9)

![4](https://github.com/user-attachments/assets/accc8acf-a5c8-451e-a1ee-7a7a0a7c763f)


Sigo en busca de información con nikto y descubro que una de las entradas el robots nos da un código 200, una vez accedo encuentro lo que parece ser otro directorio más.

![2](https://github.com/user-attachments/assets/b6377be2-ff7e-4f99-931c-c6a6f98b7fe5)

![5](https://github.com/user-attachments/assets/715f0666-c2db-4a22-892c-caec33180548)

Accedemos a la web por si tuviéramos más información

![3](https://github.com/user-attachments/assets/31753734-c3ee-406a-953e-79ff605eea3a)

Al no ver nada interesante vamos al directorio que hemos encontrado antes donde parece que está la página web.

![6](https://github.com/user-attachments/assets/881ee9ca-d808-4de8-b286-4421a9ae5510)

## Explotación

La página tampoco te deja hacer mucho, he estado probado inyección SQL en los pocos sitios que me dejaba ya que la máquina en la descripción nos daba ese tipo de información. Después de un rato de búsqueda encuentro con que cuando nos dirigimos a nuestro carro de la compra,
hay un pequeño input en el que podemos probar inyección SLQ, dándonos un error de SQL syntax, con lo que podemos deducir que es vulnerable a SQLi Error-Based. Con lo que procedemos a enumerar.

![7](https://github.com/user-attachments/assets/7d3af035-6cbe-4bb8-8cf2-696ea74309c5)

Enumerar bases de datos, encontramos una base de datos llamada Wordpress

![8](https://github.com/user-attachments/assets/4f4021e9-b4ab-4a42-8280-f159ab4acbf1)

Este escaneo con sqlmap me permitión saber que la tabla se llama wp_users, pero al ir tan lento preferí hacerlo de manera manual.

![11](https://github.com/user-attachments/assets/d91286b4-eed4-4f6f-bfee-6c58f4d48684)

Sabiendo que la tabla de usuarios en Wordpress es wp_users, podemos acceder en busca de usuarios

![10](https://github.com/user-attachments/assets/f0977420-c92b-42dc-b100-30174d01223d)

![10 5](https://github.com/user-attachments/assets/dc931dfc-8abb-49a9-a29d-f4094b015f7d)

Obtenemos usuarios y contraseñas en formato hash-MD5, la intentaremos crackear con jhon obteniendo credenciales de la base de datos.

![14](https://github.com/user-attachments/assets/b3bf9dea-ae77-47d5-948f-6b4a2c8c67d6)

En este punto no super muy bien que hacer porque seguí enumerando con distintas herramientas pero no encontré nada interesante hasta que se me ocurrió enumerar subdominios, obteniendo resultado positivo.

![15](https://github.com/user-attachments/assets/eb2eebee-0075-4751-aa02-babcadd5c602)

![16](https://github.com/user-attachments/assets/016dd65e-a0c2-4770-9e2f-3697c1b91e22)

Modificado el /etc/hosts y accediendo a la web me encuentro con un mensaje

![17](https://github.com/user-attachments/assets/26a34498-7aed-44f0-b01d-2a087715d978)

Procedo a enumerar directorios de la web encontrando lo que parece ser la web que redirige al Wordpress

![18](https://github.com/user-attachments/assets/783ee030-0fcc-478c-8d2d-c3e449b31443)

![19](https://github.com/user-attachments/assets/e3b35bad-de2c-4cfd-ae63-84b949828136)

Sabiendo que el login de wordpress es wp-admin, no hace falta enumerar ese directorio, en caso de que no existiera, procedería a enumerar con alguna herramienta

![20](https://github.com/user-attachments/assets/90f79242-7309-4c5b-88e0-de5f5d5143e9)

Fui probando credenciales de wp-jeffrey y wp-eagle pero al acceder al dasboard apenas tenía permisos para modificar la CMS, sin embargo con wp-yura al iniciar sesión apareció un mensaje de confirmación del e-mail del administrador

![21](https://github.com/user-attachments/assets/51ce2c28-671f-4fc9-a50b-d04d014bc62d)

Una vez accedi al dashboard se podía ver que ahora si teníamos permisos de administrador o por lo menos de usuario privilegiado en la CMS.

![22](https://github.com/user-attachments/assets/013d25ad-9f7b-4015-8d1c-6e3e5ce29626)

Como podía modificar los temas, fui al theme editor y modifiqué el 404.php para poder insertar ejecución remota de comandos, quise asegurarme de la ruta del theme, con lo que realicé un escaneo rápido con wpscan

![23](https://github.com/user-attachments/assets/802097b8-e730-4d0a-97ca-7918d179934a)

![24](https://github.com/user-attachments/assets/af1a4f64-b281-4389-87f8-97fa260a00c7)

![25](https://github.com/user-attachments/assets/6ad9fc04-dd4b-4ec7-b505-9abe50669da2)

Una vez conseguí ejecutar comandos de manera remota, hice URL-encode a una pequeña shell y poniendome en escucha con netcat conseguí intrusión remota en el servidor. Saniticé la tty y comencé a buscar información sobre usuarios o información interesante. Hay un usuario
llamado Orka, pero no tenemos los permisos necesarios para ver más información.

![26](https://github.com/user-attachments/assets/2127b999-d372-48c2-9f4d-a22669e0cb90)

![27](https://github.com/user-attachments/assets/5ba3f817-a035-4c74-bbc3-397dbe1a43f5)

No encontré archivos interesantes, tampoco tenía muchos permisos para acceder a directorios así que decidí mirar si había algún puerto interno de la máquina interesante que fuera accesible, encontrando el puerto 11211, que con una búsqueda en internet pude ver
que el servicio que tiene es "memcached" que es un sistema de caché en memoria de alto rendimiento y distribuido. No es una base de datos tradicional, sino una herramienta para guardar temporalmente datos que se usan mucho, para acelerar el acceso a ellos. Nunca me había
encontrado con este puerto así que directamente busqué "Port 11211 pentesting" encontrándome con una web donde lo explicaba muy bien.

![28](https://github.com/user-attachments/assets/5febe255-414c-4db5-bbda-84857529a20f)

![29](https://github.com/user-attachments/assets/4f6bbcdb-0ebc-4004-bc8e-1615fe2f4b8b)

Viendo que el servicio deja conectarse sin autenticación por telnet comencé a hacer pruebas, poniendo códigos que tenía la página y funcionaban, así que usé get username y get password para obtener credenciales. Ahora simplemente cambiando de usuario e introduciendo
las credenciales correctas, hemos pasado a ser el usuario Orka

![30](https://github.com/user-attachments/assets/0a3e97ac-fb9d-445f-99eb-77fc090701d3)

![31](https://github.com/user-attachments/assets/81db1a82-71cb-4977-80cc-1abdd08eccd6)

![32](https://github.com/user-attachments/assets/7373f1fa-6908-4b6f-8a33-a82b3c138551)

## Escalada de privilegios

Encontramos que podemos ejecutar como root un archivo llamado "bitcoin", al probar, nos pide una contraseña, podemos llevar el binario "bitcoin" a nuestra máquina y probar si con ghidra obtenemos algo interesante. También encuentro algo extraño en el sistema de archivos,
porque en /usr/sbin/ tenemos permisos de escritura.

![34](https://github.com/user-attachments/assets/6375fb2c-a16e-4ef1-858c-23e4845c4ed1)

![35](https://github.com/user-attachments/assets/7d4927c0-7924-4891-8d7e-eec786f343e0)

En ghidra aparece la contraseña

![ghydra](https://github.com/user-attachments/assets/2b2ebe2d-02ef-4ef6-a5ef-84ebdb3214ed)

Podemos crear un binario fake llamado python porque el transfer.py no está usando la ruta absolutad si no solamente "python", y viendo que donde primeramente busca es en /usr/sbin/ (justo dodnde tenemos permisos de escritura podemos crear nuestro binario llamado
python y modificarlo para que nos de una shell como root.

![36](https://github.com/user-attachments/assets/4b35ac6d-3733-4f0d-883a-15df084cf1cf)

Finalmene conseguimos acceso root

![37](https://github.com/user-attachments/assets/0c43f49c-697e-4e4a-ba46-f225edeb00c5)


