## Reconocimiento

Escaneamos con nmap y nos muestra dos puertos abiertos. 22 para SSH y 80 para HTTP con lo que voy a comprobar la página web.

![1](https://github.com/user-attachments/assets/337fad63-3f11-45cf-9892-b1cb4a556c42)

![2](https://github.com/user-attachments/assets/50b36a12-1874-435d-89bc-91c0282abdfa)

Al no encontrar directorios interesantos con gobuster, revisando la página y las dev tools, me dispongo a enumerar subdominios dando en este caso un resultado positivo.

![3](https://github.com/user-attachments/assets/76aa1035-184e-497d-b1ed-949f71a832a8)

Modificados /etc/hosts para tener acceso a la web y nos encontramos con un panel de login en el que utilizaremos las credenciales dadas por la descripción de la máquina : Olivia Cortez y olivi8

![4](https://github.com/user-attachments/assets/7da58840-08bb-46e2-a32d-9a064fe9e29e)


![5](https://github.com/user-attachments/assets/e622aac8-2117-4fe7-b755-7c8f79a4ca81)

Revisando por todas las pestañas no vemos nada interesante, puedo deducir incluso que este usuarios no tiene muchos permisos en la web. Sin embargo hay una pestaña mensajes que llama mi atención, sobre todo porque simplemente modificando el id del mensaje podemos ver
conversaciones entre los empleados.

![6](https://github.com/user-attachments/assets/39f81f3a-5d4b-4b31-8180-2fae53219131)

Parece que nuestro DEV TEAM no es muy consecuente con la seguridad de la empresa, ha pedido las credenciales a un empleado por medio de un chat alojado en la misma página de la empresa. Vamos a probar con esas credenciales. Ahora sí parece que este usuario tiene al menos,
más permisos porque somos capaces de ver los números de teléfono que antes estaban ocultos.

![7](https://github.com/user-attachments/assets/48361d29-33bc-417a-855e-27041d0583cf)

![8](https://github.com/user-attachments/assets/00062f12-f061-4cca-a4ca-650d4edd366c)

## Explotación

Después de un rato de búsqueda por el dashboard no encontré nada interesante, la única pestaña que más me llama la atención es settings así que me dirijo a ella.

![9](https://github.com/user-attachments/assets/4326dca6-2a4d-4c54-b722-2b077dec306f)

Podemos capturar la petición en Burp Suite por si puediéramos modificar parámetros y observamos que si borramos el parámetro password obtenemos un 500 internal error en el que nos dirige a un archivo "settings.ejs" usado en páginas con Node.js o Express.js.

![11](https://github.com/user-attachments/assets/694bc8bc-8bd9-4e8b-bf27-2e5d6bf4a185)

Lo que me llevó a buscar información sobre Pentesting de Node.js y archivos .ejs siéndome el único enlace útil este https://eslam.io/posts/ejs-server-side-template-injection-rce/. Testeando el payload con curl vemos que recibimos señal del servidor.

![12](https://github.com/user-attachments/assets/cce2dc9c-c5ee-4111-a764-26dd16b7fe08)

![13](https://github.com/user-attachments/assets/d0e0ba1a-a15d-474d-a45a-e45670e09f51)

Así que usé netcat para recibir una conexión remota hacia mi máquina. Ahora somos el usuario web y tenemos acceso a la flag user.txt

![16](https://github.com/user-attachments/assets/315de5e2-6997-499f-ab82-66b9ffe90817)

![17](https://github.com/user-attachments/assets/264b5390-2372-4856-9d97-71ba242f8df6)

![18](https://github.com/user-attachments/assets/0a0350a7-f5e8-4417-bdfc-7f828e9d2180)

## Escalada de privilegios

Al hacer sudo -l vemos que tenemos permisos de ejecución como root en sudoedit, voy a exportar el editor a vim para que nos deje leer el archivo /etc/passwd

![19](https://github.com/user-attachments/assets/c4b8ad25-091a-44dd-ab9e-d6ec3d103b69)

Ahora en teoría ejecutando el sudoedit en la ruta donde nos deja ejecutarlo, nos debería de abrir el archivo /etc/passwd

![20](https://github.com/user-attachments/assets/d9efc213-8552-42d2-8042-94fb5587e170)

Obtenemos el hash de root, así que ya podríamos entrar como root, en mi caso al ser una CTF y sé que la flag va a estar en /root/root.txt voy a modificar el comando para que haga lo mismo solo que en vez de leer /etc/passwd va a leer /root/root.txt

![21](https://github.com/user-attachments/assets/1a81fa5c-bcef-4f2a-a378-adf942a849e1)

![22](https://github.com/user-attachments/assets/74c071b8-af36-4af0-8bf1-87f4c76e9b5b)
