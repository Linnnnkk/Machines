## Reconocimiento

El escaneo de nmap solo nos detecta el puerto 5003 abierto, que parece ser que aloja una web.

![1](https://github.com/user-attachments/assets/f554ef78-08c0-4139-8749-3c17fe25fd3d)

![2](https://github.com/user-attachments/assets/d3623c29-ef3d-4c42-b52b-2552302791e9)

Al investigar un poco por la página, llama mi atención el nombre Pickle, que es un módulo de Python que permite serializar y deserializar objetos Python, y en algunos casos puede ser usado para serializar datos como cookies del lado del servidor. Curiosamente en la solicitud
POST cuando buscamos en search, aparece el parámetro search_cookies con datos encodeados en base64. Podría ser una pickel string que contiene la última búsqueda que se realizó.

![3](https://github.com/user-attachments/assets/7013c8d3-dd62-4b05-8138-0c22777c3198)

![4](https://github.com/user-attachments/assets/ba861a9f-2df6-444c-9d8d-d4ced0a7b7da)

Gracias a esta guía https://davidhamann.de/2020/04/05/exploiting-python-pickle/ modifiqué un payload para hacer ping a mi máquina para realizar un testeo, y luego lo modifiqué para crear una reverse shell.

![5](https://github.com/user-attachments/assets/619f4739-054e-4924-9257-e4c8deb94235)

El script de la imagen anterior nos dejará este código en base64, que nos servirá como payload para sustituirlo en el parámetro search_cookies

![6](https://github.com/user-attachments/assets/9a54bd5b-4ddf-4369-8534-7980f7505286)

![7](https://github.com/user-attachments/assets/c0c1ee39-d00b-4db7-b886-9dc53be951a9)

![8](https://github.com/user-attachments/assets/25387145-4921-4866-87f3-92b38f1c3e0d)

![9](https://github.com/user-attachments/assets/6ddfa43a-4fcf-401e-8866-7012b57f9708)

Entramos en la máquina directamente como root, lo primero que me llama la atención en un archivo sqlite3, abro una conexión en netcat y transfiero los archivos

![12](https://github.com/user-attachments/assets/4c9c20fd-f050-45cd-a0c2-fe4c97e9aa5b)

Una vez con el archivo en mi máquina local, lo abro y voy en búsqueda de usuarios, obtengo 6 usuarios y 6 contraseñas hasheadas, intenté crackearlas durante un buen rato con hashcat pero no pude obtener ninguna, así que volví a la máquina vícitma en busca de más
información

![14](https://github.com/user-attachments/assets/5fcc92ee-d9d0-4945-82ea-f39f0dd91873)

![15](https://github.com/user-attachments/assets/5522c705-4c4c-4de0-840b-0695a42b9ffd)

Después de un rato de búsqueda, recordé que en el historial podría haber algo interesante y efectivamente parece que alguien ha intentado conectarse a una máquina que está en una red interna mediante ssh con el usuario ramsey.

![10](https://github.com/user-attachments/assets/896112be-0565-42f8-970d-81cfe12509e2)

Me puse a buscar con distintos comandos y noté la falta de varios de ellos en esta máquina. Lo logré con `nc -zv 172.17.0.2 1-65535` y efectivamente esa máquina tenía el puerto 22 abierto.

![11](https://github.com/user-attachments/assets/f9da982f-9399-41ed-b50c-f402728f0f15)

Intenté conectarme por la máquina atacante pero anteriormente ya me había dado cuenta de que no tenía acceso a internet, intenté hacer pivoting con chisel pero al ser un laboratorio antigüo cuando transfería el chisel a la máquina víctima tenía una versión demasiado nueva,
pdría haber descargado una versión de chisel anterior, pero estaba tan cansado de intentar incluso compilar la librería faltante en la máquina victima y decidí cofigurar el gestor de paquetes para que todo el tráfico HTTP de `apt` pase por un proxy ubicado en la IP y puerto
que yo quiera (en este caso he usado Burp Suite). Esto puede ser útil cuando:

  - Si la máquina remota no tiene acceso directo a internet, pero tú sí (y estás en la misma red o tienes un túnel), puedes levantar un proxy (como Squid o mitmproxy) en tu máquina local.

![16](https://github.com/user-attachments/assets/772eb7d8-21c5-4688-9840-3d6b653a5777)

Actualicé los repositorios e instalé también el cliente de SSH y al no tener ningún tipo de contraseña pero si el usuario decido hacer fuerza bruta con hydra al usuario Ramsey.

![17](https://github.com/user-attachments/assets/37446dc6-b048-4025-91dc-5c656670dd49)

Conseguimos las credenciales de ramsey y al entrar obtenemos el user.txt

![18](https://github.com/user-attachments/assets/13592a46-dcc8-4aeb-bf62-28d504777ba0)

## Escalada de privilegios

Al usar `sudo -l` vemos que tenemos acceso como 'oliver' a un archivo localizado en /home/ramsey

![19](https://github.com/user-attachments/assets/fd5e4f87-7fdd-445d-b08d-dd29f8deb124)

Al observarlo vemos que el código tal y como está escrito es vulnerable. En la parte del (input) podrías intentar ver si somos capaces de ejecutar comandos.

![20](https://github.com/user-attachments/assets/ee37bc5b-0f1e-48a8-a4aa-ac255fb13b82)

Creamos una imagen 'shell.png' en la que pondremos `os.system('/bin/bash')` y la descargaremos a través de wget al directorio /home/ramsey, una vez dentro cambiaremos el nombre a payload.png y ejecutaremos el script usando `sudo -u oliver /usr/bin/python /home/ramsey/vuln.py`

![21](https://github.com/user-attachments/assets/4d823ccf-fe0d-4af8-8735-2ad35fcde673)

Habremos cambiado al usuario oliver, ahora veremos los permisos que tiene este usuario y si nos permite elevarnos a root.

![22](https://github.com/user-attachments/assets/37423e3e-59d0-4344-9e9e-dc1ce05c1757)

Tenemos un archivo que podemos ejecutar como root llamado dockerScript.py, al abrirlo vemos este contenido

```python
import docker

# oliver, make sure to restart docker if it crashes or anything happened.
# i havent setup swap memory for it
# it is still in development, please dont let it live yet!!!
client = docker.from_env()
client.containers.run("python-django:latest", "sleep infinity", detach=True)
```

Modificamos el archivo y damos permisos a /bin/bash para acceder como root y el root.txt

![23](https://github.com/user-attachments/assets/a6d0ae95-e34e-4431-8774-d1e0b980fa09)

![24](https://github.com/user-attachments/assets/4d21f18d-e80f-40fc-81c9-99466cc7ef0c)




