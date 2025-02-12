# Máquina Instant HTB

### Escaneo de puertos abiertos y servicios


### Modificamos el archivo /etc/hosts para logar acceso a la web.


### Nos permite descargar una app en la que usaremos apktool en busca información sensible expuesta.

`apktool d ~/Downloads/instant.apk -o apk -s`


### Con la herramienta `grep -r -i ""` buscaremos archivos que contengan palabras clave en busca de algún tipo de información útil.


### Encontramos subdominios en lo que parece ser un archivo de configuración de red. Modificamos /etc/hosts para lograr acceso. Parecen dos páginas web similares, vamos a realizar enumeración de directorios.

`wfuzz -c -u 'http://instant.htb/FUZZ/' -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hc 404,301`
