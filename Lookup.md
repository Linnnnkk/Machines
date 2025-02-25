#Máquina : Lookup
#Dificultad : Easy

### Nmap scan

![Screenshot_2025-02-25_12_25_38](https://github.com/user-attachments/assets/c29aedc1-48e8-4467-94a7-cd921dffea91)



Descubrimos puerto 22 y 80 abiertos. Modificamos /etc/hosts para tener acceso a la web y nos encontramos con una página de logueo.



![Screenshot_2025-02-25_12_26_04](https://github.com/user-attachments/assets/c6ae44e4-a6ca-47fd-b234-3b41542906d2)


Al no conseguir ningún subdominio ni directorios interesanes mediante dirsearch y wfuzz probamos con credenciales predeterminadas y con burpsuite observamos la respuesta POST


![Screenshot_2025-02-25_12_27_03](https://github.com/user-attachments/assets/0c5f93e5-1d42-44e5-a813-6cd8d943c46a)


Podemos observar que el login está mal configurado ya que cuando acertamos un usuario solo nos dice que la contraseña es incorrecta. En ese caso nos ha servido con admin. Procedemos a enumerar usuarios con un script hecho en python teniendo en cuenta que en el momento que
acertemos con un usuario nos dará error solo en la contraseña


![Screenshot_2025-02-25_12_53_24](https://github.com/user-attachments/assets/2eb516ac-a4a4-4b7a-bdf9-0f99d62ed078)


Encontramos el usuario jose por lo que procederemos a realizar fuerza bruta a la contraseña obteniendo un resultado favorable.


![Screenshot_2025-02-25_13_04_49](https://github.com/user-attachments/assets/77fbf015-9b58-4f7b-b47b-d648f56fbaa4)



De primeras no podemos acceder porque redirecciona a un subdominio con lo cual modificamos el archivo hosts y ya tendríamos acceso a él y encontramos lo que parece un sistema de gestión de archivos, buscando el entorno en el que me encuentro, veo que es elFinder v2.1.47,
vulnerable a CVE-2019-9194, encontré un script en github que me ayudó en el proceso. https://github.com/hadrian3689/elFinder_2.1.47_php_connector_rce


![Screenshot_2025-02-25_13_19_25](https://github.com/user-attachments/assets/76825ea3-2594-4c21-a9cb-19230485d792)



Enumeramos usuarios, encontramos root y think. Usando ls -la vemos directorios interesantes pero no accesibles.


![Screenshot_2025-02-25_13_34_33](https://github.com/user-attachments/assets/86292338-fcb6-495f-b1c1-a2794eb354aa)


Abriremos un servidor en python para descargar linpeas remotamente y al finalizar el escaneo me llama la atención un directorio que no reconoce linpeas con lo que podríamos deducir que es un directorio que no está por default en Linux /usr/sbin/pwm


![Screenshot_2025-02-25_14_03_16](https://github.com/user-attachments/assets/cb28ac79-10b9-46d3-8a55-21c4ee6c7376)























