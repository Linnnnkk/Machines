## Reconocimiento

Escaneo con nmap

![1](https://github.com/user-attachments/assets/368034f7-3d99-4359-8fc0-561b7133a567)



Vemos que tiene el puerto 80 abierto con lo que accedemos a la web en busca de información

![3](https://github.com/user-attachments/assets/68870778-bfe0-42c5-a1db-e1de1b4d2ec6)


Al hacer fuzzing web, encontramos con lo que parece ser un panel de login de administración, que, además, nos deja registrarnos.

![2](https://github.com/user-attachments/assets/450072ee-ad4b-4b2b-a144-7ba1d8e35dfb)





![4](https://github.com/user-attachments/assets/29baa32d-92df-4828-9889-243aa00b3742)


![5](https://github.com/user-attachments/assets/c63bf677-cda2-41f4-95ad-9024a2158481)


![5 1](https://github.com/user-attachments/assets/7c4e2982-16c2-429a-856e-8bedff577682)



Una vez registrados podemos tener acceso al dashboard, donde lo único interesante es que podemos subir un archivo de imagen de perfil, pero solo puede el administrador admin@sky.thm, también podemos interactuar con el ResetPassword.

![5 2](https://github.com/user-attachments/assets/ecd9ac3d-be5d-49c0-9a7b-91c2c3fac94c)



## Explotación



![5 3](https://github.com/user-attachments/assets/04d7aa22-a18d-4da3-9ad3-4e8572b95b94)



Vamos a interceptar la petición POST en burp suite del reseteo de contraseña ya que es de lo poco interesante que nos deja hacer la web.

![5 4](https://github.com/user-attachments/assets/7aa123fd-b1ef-4bc4-9a34-d567b25f0822)


![5 5](https://github.com/user-attachments/assets/22f305d5-dd33-45da-a536-d081ffd17848)

Podemos hacer bypass al ResetPassword en Burp Suite conociéndo el e-mail del usuario que queramos, en este caso probamos con el e-mail del admin que hemos visto antes que tenía permisos para subir archivos al perfil.

![5 6](https://github.com/user-attachments/assets/1dca98d7-38b6-4fa1-aae4-da02b8e49349)


![5 7](https://github.com/user-attachments/assets/7c8dc456-db18-4370-865c-7e635b22fb5d)



Obtenemos acceso de Admin, ahora nos debería dejar subir un archivo, probaremos a subir un archivo .php que nos permita ejecución remota de comandos.

![shell](https://github.com/user-attachments/assets/c53e2582-dc12-414e-8ef1-694c0733f103)


![5 8](https://github.com/user-attachments/assets/142fd00b-7890-408c-910b-7f555a243b26)


![5 9](https://github.com/user-attachments/assets/b43b1a1c-5f50-4771-9e0d-d11f4477e8b5)



Al no tener ningún problema en subir el archivo, solo nos faltaría encontrar el directorio donde está el archivo shell.php, podemos usar burp suite en busca de información, ya que queremos buscar la imagen de perfil, podemos buscar en el response del repeater la palabra 'profile' en busca de información sobre la imagen subida.

![5 1 1](https://github.com/user-attachments/assets/2b0c47c4-de0d-4f78-a4f1-0924fe667098)



Obteniendo un resultado positivo, procedemos a abrir el archivo y probar si funciona.

![5 1 2](https://github.com/user-attachments/assets/ad891c30-fd2b-4d39-a374-1773ad0debdf)


Como efectivamente hemos logrado realizar ejecución remota de comandos, usaremos el comando bash -c "reverse_shell" URL-encodeado y obtendremos intrusión remota al servidor.


![5 1 4](https://github.com/user-attachments/assets/0be93aac-c0bb-4d5e-8f3e-6b691376c2c7)


![6](https://github.com/user-attachments/assets/4f7692c6-be51-42e5-b5cb-43d77854771e)


Abrimos un servidor en python y descargamos linpeas para ver las vulnerabilidades que pueda tener


![6 1](https://github.com/user-attachments/assets/18f483d0-a156-48cb-b1d8-fc9a84763383)



Lo más interesante que aparece es que al parecer hay un archivo de configuración de mongodb, por lo que voy a comprobar si hubiera una base de datos en la red local.


![7](https://github.com/user-attachments/assets/e76acc8a-9fa4-4b81-a104-5737072ddec9)



Oservando los puertos abiertos, vemos que el puerto 27017 está abierto y buscando en internet coincide con el puerto de MongoDB.

![8](https://github.com/user-attachments/assets/afc32c99-6df8-4305-ac5f-91473846b9a5)



Termino de confirmarlo viendo el archivo /etc/passwd en el que hay un usuario llamado mongodb

![10 1](https://github.com/user-attachments/assets/70b3e21f-ed0a-4228-b57d-63dd8921366f)



De hecho, nos deja ejecutar el comando 'mongo' que es el que se utiliza para conectarse a base de datos mongo, y para nuestra sorpresa no nos pide autenticación con lo que procedemos a inspeccionar archivos interesantes que podamos ver.

![9](https://github.com/user-attachments/assets/997140d4-4ccc-4708-9de4-274afccaaa41)

Al inspeccionar la base de datos 'backup' vemos que podemos enumerar usuarios y contraseñas

![10](https://github.com/user-attachments/assets/0b57baf0-9a46-48ed-91a3-16b66c8a3604)


Usamos la contraseña vista antes para el usuario webdeveloper y conseguimos cambiar de usuario a unos con mayores privilegios


![11](https://github.com/user-attachments/assets/9ec100f0-d259-4c8d-b60c-2bf7cc686c0e)


![12](https://github.com/user-attachments/assets/78769a5a-27af-4ffd-ab99-adeab41e108e)


![13](https://github.com/user-attachments/assets/41501eb1-efd2-4b32-94d4-148bd9132366)

## Escalada de privilegios

El usuario webdeveloper tiene permisos para ejecutar el comando /usr/bin/sky_backup_utility con sudo sin necesidad de contraseña. Además, la variable de entorno LD_PRELOAD estaba permitida, lo que facilita la carga de bibliotecas compartidas personalizadas.

Creamos un archivo C que define una función constructor que cambia el UID y GID al usuario root y ejecuta una shell de root.

![escalada1](https://github.com/user-attachments/assets/132f8d75-7c6c-4197-8621-1b4674151a8f)

Compilamos el archivo en una biblioteca compartida utilizando gcc.

![escalada2](https://github.com/user-attachments/assets/486e01e3-d3d8-462e-88f3-e68a73e15f37)

Establecemos la variable de entorno LD_PRELOAD para cargar la biblioteca compartida y ejecutamos el comando sky_backup_utility con sudo.

![escalada3](https://github.com/user-attachments/assets/523bc922-0fbe-4808-bbd2-ade5278afe67)

Una vez cargado el programa, confirmamos que tenemos shell de root.

![escalada4](https://github.com/user-attachments/assets/71b46e97-84f2-4da1-94af-04ef3ba324de)


