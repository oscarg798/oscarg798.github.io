# Publicar una app en Firebbase Distribution con 

Para publicar una app en Firebase Distribution existen diversas formas, se puede usar el command line tools de Firebase 
o se puede usar Fastlane, en este caso usaremos Fastlane por la facilidad


1. Instalar  fastlane https://docs.fastlane.tools/getting-started/android/setup/
   `brew install fastlane`
2. en la terminal dirigirnos a el proyecto `cd my_project`
3. `fastlane init`  e ingresar la informacion que la herramienta nos pide 
4. Instalar el action para publicar `sudo fastlane add_plugin firebase_app_distribution`
4. Instalar el CLI de Fastlane https://firebase.google.com/docs/cli 
   `curl -sL firebase.tools | bash`
5. Logearnos en firebase `firebase login`
6. Abrir el archivo Fastfile en un editor de texto, este archivo esta en `my_proyect`/fastlane/Fastfile
7. Copia el siguiente fragmento de codigo en el archivo 

<script src="https://gist.github.com/oscarg798/4fe7a6e4e72edc714b7acad4cea263b0.js"></script>

* Donde `lane :nighthold do` es el nombre de la tarea que ejecutaremos
* `build_android_app(task: "assembleDebug")` es la linea que ejecutara la tarea para la variante del apk que queremos crear
* `app: "1:112:android:123"` es el identificador de nuestro proyecto de android en firebase, este se puede obtener clickeando el icono de android en firebase y dentro de la configuracion de dicha applicación
* `groups: "group"` se usa para distribuir la aplicación a un grupo especifico de usuarios, estos se pueden crear en firebase
*  firebase_cli_path: "/usr/local/bin/firebase" ubicacion del cli instalado en el paso 4

8. `Fastline firebase` 

Nota: En la version 0.14 de la accion para fastlane estamos teniendo un error `firebase_app_distribution' was not properly loaded` si esto sucede usaremos el comando `sudo chmod -R a+r  /Library/Ruby/Gems/2.6.0/gems/fastlane-plugin-firebase_app_distribution-0.1.4`
