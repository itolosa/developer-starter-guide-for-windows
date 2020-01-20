# Known "bugs"

# Umask 0000 / Permisos no se guardan
Cuando inicias windows subsystem for linux (wsl), te das cuenta que el umask es 000 o 0000. 
O bien lo otro que notas, es que al crear un archivo aunque cambies sus permisos, este se reinicia a los permisos 777.
Esto puede generarte grandes problemas cuando deseas instalar aplicaciones locales, crear carpetas, clonar repositorios, etc.
Antes de comenzar a usar WSL lo primero que deberías hacer es arreglar esto.

Lo primero que hay que entender es que hay 2 tipos de umask. Uno que lleva por defecto el sistema de archivos y otro por defecto
para los procesos ejecutados por un usuario.

Lo otro que hay que comprender es que el sistema de archivos de windows, funciona de manera distinta al de linux.
Hay mucha literatura en cuanto a no escribir archivos de linux usando windows. Aunque al reves no sea tan dramático.

## Permisos y Umask del sistema de archivos
La configuración inicial no permite guardar los permisos de los archivos creados en el sistema de archivos de windows.
Pero existe una forma de mantenerlos y es creando un archivo llamado wsl.conf y agregando un contenido similar a este:
```
[automount]
root = /mnt/
options = metadata, umask=022
```
La opcion metadata permite toda esta magia de guardar los permisos. Umask=022 configura el umask por defecto a 022.

## Umask de procesos (usuario)
Por defecto Ubuntu 18.04 usa 002 para umask de la sesión. Tambien es comun configurarlo a 022.

Sin embargo en WSL el umask es 000. Esto genera que todos los procesos al comienzo usen umask 000, lo que es grave.
Pero Por que ocurre esto?

Esto es debido a la forma de inicio de la sesión de shell que entrega WSL. 
En Linux se usa /bin/login para darte una sesión. Este programa setea el umask a 002 por defecto,
ademas de realizar otras inicializaciones. Sin embargo, en windows no se utiliza ese mecanismo,
principalmente, como lo indican sus desarrolladores, debido a que supuestamente genera inestabilidad.
En su lugar, han creado un sistema que emula la inicialización de /bin/login pero
al parecer aún no esta muy completo.

Una forma de poder usar /bin/login es cambiando el comando de inicio de la sesion que usas de WSL por:
```
wsl.exe -u root -- /bin/login -f username
```
Donde `username` es tu nombre de usuario

Ahora, si te preocupa la supuesta inestabilidad que genera `/bin/login` no esta mal agregar el comando `umask 002` en cualquiera de estos archivos:
```
/etc/profiles
~/.profile
~/.bashrc
~/.zshrc
```

# Dejo de funcionar la búsqueda en el menu inicio o en los directorios?

En el visor de eventos sección aplicaiciones es posible ver los logs de las aplicaciones que pueden estar causando esto.
Atención al proceso SearchUI.exe, ya que este proceso es típicamente el responsable de todo esto.
En mi caso el codigo de excepcion que retornaba SearchUI.exe era 0xc000027b asociado al paquete Windows.UI.Xaml.dll
En mi caso este error se generó luego de haber instalado un paquete de idioma.
La solución fue tan simple como desinstalar el paquete de idioma y volverlo a instalar.
Al parecer esto ocurre cuando se instalan paquetes de idioma y luego se instalan actualizaciones encima.

También hay otros foros que indican que el mismo problema se presenta por otros motivos como
- Iconos grandes en el panel de control (hay que setearlos en verlos pequeños... que locura).
- Antivirus
- Paquete de idiomas desinstalado incorrectamente (hay que usar un script y forzar el cambio de idioma).
- etc.

Aqui no voy a describir soluciones como usar sfc y otros, ya que nunca he confiado en los detectores de problemas automáticos de windows.
Y en mi opinión son bastante inútiles (espero equivocarme).

# Docker no monta un volumen

Cambia tu directorio de trabajo al sistema de archivos de windows. Es necesario copiar o mover las aplicaciones para allá.

# Docker no inicia luego de algunas limpiezas

Véase como regenerar el repositorio de WMI. Es necesario reiniciar.

