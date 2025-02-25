#Máquina : Lookup
#Dificultad : Easy

### Nmap scan

![Screenshot_2025-02-25_12_25_38](https://github.com/user-attachments/assets/c29aedc1-48e8-4467-94a7-cd921dffea91)



### Descubrimos puerto 22 y 80 abiertos. Modificamos /etc/hosts para tener acceso a la web y nos encontramos con una página de logueo.



![Screenshot_2025-02-25_12_26_04](https://github.com/user-attachments/assets/c6ae44e4-a6ca-47fd-b234-3b41542906d2)


### Al no conseguir ningún subdominio ni directorios interesanes mediante dirsearch y wfuzz probamos con credenciales predeterminadas y con burpsuite observamos la respuesta POST


![Screenshot_2025-02-25_12_27_03](https://github.com/user-attachments/assets/0c5f93e5-1d42-44e5-a813-6cd8d943c46a)


### Podemos observar que el login está mal configurado ya que cuando acertamos un usuario solo nos dice que la contraseña es incorrecta. En este caso nos ha servido con admin. Procedemos a enumerar usuarios con un script hecho en python teniendo en cuenta que al acertar un usuario nos dará error solo en la contraseña


![Screenshot_2025-02-25_12_53_24](https://github.com/user-attachments/assets/2eb516ac-a4a4-4b7a-bdf9-0f99d62ed078)


### Encontramos el usuario jose por lo que procederemos a realizar fuerza bruta a la contraseña obteniendo un resultado favorable.


![Screenshot_2025-02-25_13_04_49](https://github.com/user-attachments/assets/77fbf015-9b58-4f7b-b47b-d648f56fbaa4)



### De primeras no podemos acceder porque redirecciona a un subdominio con lo cual modificamos el archivo hosts y ya tendríamos acceso a él y encontramos lo que parece un sistema de gestión de archivos, buscando el entorno en el que me encuentro, veo que es elFinder v2.1.47,
vulnerable a CVE-2019-9194, encontré un script en github que me ayudó en el proceso. https://github.com/hadrian3689/elFinder_2.1.47_php_connector_rce


![Screenshot_2025-02-25_13_14_22](https://github.com/user-attachments/assets/a788a063-58c0-4ce0-9558-ec289e30fb55)



![Screenshot_2025-02-25_13_19_25](https://github.com/user-attachments/assets/76825ea3-2594-4c21-a9cb-19230485d792)



### Enumeramos usuarios, encontramos root y think. Usando ls -la vemos directorios interesantes pero no accesibles.


![Screenshot_2025-02-25_13_34_33](https://github.com/user-attachments/assets/86292338-fcb6-495f-b1c1-a2794eb354aa)


### Abriremos un servidor en python para descargar linpeas remotamente y al finalizar el escaneo me llama la atención un directorio que no reconoce linpeas con lo que podríamos deducir que es un directorio que no está por default en Linux /usr/sbin/pwm. Al iniciarlo ejecuta el comando 'id' para exrae el nombre de usuaio y el ID. Con lo que podríamos hacer PATH Hijacking donde podemos ejecutar 'id' sin especificar la ruta ejecutando la primera coincidencia que exista.

![Screenshot_2025-02-25_14_03_16](https://github.com/user-attachments/assets/cb28ac79-10b9-46d3-8a55-21c4ee6c7376)

### 1. Creamos un falso id en /tmp/id

echo -e '#!/bin/bash\necho "uid=1000(think) gid=1000(think) groups=1000(think)"' > /tmp/id

Le damos permisos de ejecución

chmod +x /tmp/id

### 2. Modificamos el PATH para que nuestro id se ejecute primero

export PATH="/tmp:$PATH"

### 3. Ahora el programa al ejecutar id, usará nuestro /tmp/id en lugar del verdadero /bin/id.


![Screenshot_2025-02-25_14_39_33](https://github.com/user-attachments/assets/56b505e0-e2ae-4301-8741-fff9b6c6185f)


### Obtenemos lo que parecen ser contraseñas, probaremos a usar hydra obteniendo resultado positivo y nos conectamos por ssh


![Screenshot_2025-02-25_14_42_12](https://github.com/user-attachments/assets/fc617c53-5e9a-40b3-a986-851b77f6127a)


![Screenshot_2025-02-25_14_43_39](https://github.com/user-attachments/assets/b15f6ce1-24ca-4213-b998-419f6aa65a6b)


### Conseguido el user.txt procedemos a escalar privilegios, observamos que con simplemente usar sudo -l, nos indica que podemos ejecutar comandos como sudo en /usr/bin/look


![Screenshot_2025-02-25_14_50_07](https://github.com/user-attachments/assets/97e39fb4-d837-4eca-9c22-927e52495fd8)


### Sabiendo esto podemos usar GTFOBins y ejecutar un comando que lea la rsa_key que es la manera más rápida de acceder al root user, obteniendo la rsa_key.

![imagen](https://github.com/user-attachments/assets/d8b4b3ca-7e97-4b5b-bcd7-983294c3a65d)


![Screenshot_2025-02-25_14_56_38](https://github.com/user-attachments/assets/9c55e0ba-9695-4d99-b10a-aa99b10519f3)

### La copiamos a un archivo y ejecutamos ssh -i

![Screenshot_2025-02-25_15_06_37](https://github.com/user-attachments/assets/a1adb472-5ad5-406b-b59c-925305ddaf4a)




































