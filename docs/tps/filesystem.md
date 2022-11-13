# TP3: Filesystem FUSE

## Índice
{:.no_toc}

* TOC
{:toc .sidetoc}

## Introducción

En este trabajo implementaremos nuestro propio sistema de archivos (o _filesystem_) para Linux. El sistema de archivos utilizará el mecanismo de [FUSE][fuse-wiki] (_Filesystem in USErspace_) provisto por el [kernel][fuse-linux], que nos permitirá definir en _modo usuario_ la implementación de un _filesystem_. Gracias a ello, el mismo tendrá la interfaz VFS y podrá ser accedido con las syscalls y programas habituales (`read`, `open`, `ls`, etc).

La implementación del filesystem será enteramente en memoria: tanto archivos como directorios serán representados mediante estructuras que vivirán en memoria dinámica. Por esta razón, buscamos un sistema de archivos que apunte a la velocidad de acceso, y no al volumen de datos o a la persistencia (algo similar a [`tmpfs`][tmpfs]). Aún así, los datos de nuestro filesystem estarán representados _en disco_ por un archivo.

[fuse-wiki]: https://en.wikipedia.org/wiki/Filesystem_in_Userspace
[fuse-linux]: https://www.kernel.org/doc/html/latest/filesystems/fuse.html
[tmpfs]: https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html

### Software necesario

FUSE está compuesto de varios componentes: un módulo del kernel (que se encarga de hacer la delegación) y una librería de usuario que se utiliza como framework. Para ello se necesitará un sistema operativo que cuente con el kernel Linux, y que disponga de las [librerías de FUSE][libfuse] versión 2.

Puede instalarse todas las dependencias con lo siguientes comandos:
```
apt install pkg-config libfuse2 libfuse-dev
```

**AVISO**: El esqueleto del TP3 se encuentra disponible en [fisop/fisopfs](https://github.com/fisop/fisopfs){:.alert-link}.
{:.alert .alert-warning}

Si todas las dependencias están correctamente instaladas, deberían poder *compilar y ejecutar* el código del esqueleto. Secuencia de ejemplo:
```
// Compilación
$ ls
Makefile  README.md  fisopfs.c
$ make
gcc -ggdb3 -O2 -Wall -std=c11 -Wno-unused-function -Wvla    fisopfs.c  -D_FILE_OFFSET_BITS=64 -I/usr/include/fuse -lfuse -pthread -o fisopfs


// Creamos un directorio vacío
$ mkdir prueba
// Ejecutamos nuestro binario y le decimos donde queremos que monte nuestro filesystem
$ ./fisopfs prueba/
// Verificamos que se haya montado correctamente
$ mount | grep fisopfs
/vagrant/labs/fisopfs/fisopfs on /vagrant/labs/fisopfs/prueba type fuse.fisopfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

// Pruebas con el directorio montado
$ ls -al prueba/
total 0
drwxr-xr-x 2    1717 root       0 Dec 31  1969 .
drwxr-xr-x 1 vagrant vagrant  352 Nov 13 12:41 ..
-rw-r--r-- 1    1818 root    2048 Dec 31  1969 fisop

$ cat prueba/fisop
hola fisopfs!

// Desmontamos
$ sudo umount prueba
```

### Filesystem tipo FUSE

La compilación y ejecución del proceso manejará nuestro filesystem es algo distinta. El esqueleto ya está perparado (en el `Makefile`) para compilar incluyendo las dependencias necesarias, pero se recomienda leer [este artículo][cs135-fuse] antes de arrancar/cuando se modifique el Makefile.

[cs135-fuse]: https://www.cs.hmc.edu/~geoff/classes/hmc.cs135.201109/homework/fuse/fuse_doc.html#compiling

En ese artículo también está explicado cómo utilizar la librería de FUSE e implementar nuestras propias funciones. En el caso del esqueleto, únicamente están implementadas 3 primitivas del sistema de archivos: `getattr`, `readdir` y `read`.

```
static struct fuse_operations operations = {
        .getattr = fisopfs_getattr,
        .readdir = fisopfs_readdir,
        .read = fisopfs_read,
};
```

La documentación oficial de FUSE es muy útil para revisar las firmas de las funciones y los campos de cada struct de la librería. Pueden encontrar la [misma aquí][libfuse-docu-oficial]. Sin embargo, hay que tener en cuenta que esa documentación es **para la versión 3**, y por tanto podría tener algunas diferencias en la API/firma de algunas funciones.

La documentación para la versión 2 (la última, 2.9.9) más precisa es el código del include (`fuse.h`) en sí. Pueden encontrarlo [aquí][libfuse-2.9.9], y notarán que ahí se encuentran todos los structs, funciones auxiliares y firmas.

[libfuse-docu-oficial]: http://libfuse.github.io/doxygen/index.html
[libfuse-2.9.9]: https://github.com/libfuse/libfuse/blob/fuse-2.9.9/include/fuse.h

**AVISO**: Si tienen errores al momento de compilar, o ven alguna discrepancia con la documentación, es posible que estén usando FUSE versión 3{:.alert-link}.
{:.alert .alert-warning}

## Implementación

Implementaremos `fisopfs`, un filesystem de tipo FUSE definido por el usuario. El mismo deberá implementar un subconjunto de las [operaciones][fuse-operations] que soporta FUSE.

[fuse-operations]: http://libfuse.github.io/doxygen/structfuse__operations.html

### Representación del sistema de archivos

La estructura en memoria que se utilice para lograr tal funcionalidad es enteramente a diseño del grupo, y la misma deberá explicarse claramente, con diagramas, en el informe del trabajo (e.g. el archivo `TP3.md`). La primera parte de la entrega, consistirá en el diseño de las estructuras que almacenarán toda la información, y cómo las mismas se accederán en cada una de las operaciones.

<div class="alert alert-primary" markdown="1">
**Implementación: operaciones requeridas**
- Es *requisito* que el sistema de archivos soporte las siguientes funcionalidades:
  - Creación de archivos
  - Creación de directorios
  - Lectura de directorios, incluyendo los pseudo-directorios `.` y `..`
  - Lectura de archivos
  - Escritura de archivos
- La creación de directorios debe soportar al menos un nivel de recursión, es decir, directorio raíz y sub-directorio.
- Se deben diseñar las estructuras en memoria que almacenarán los archivos, directorios y cualquier otro tipo de estructuras auxiliares que vean necesarias.
</div>

### Persistencia en disco

Por otro lado, el filesystem _entero_ se representará como un único archivo en disco, con la extensión `.fisopfs`; y en el mismo se serializará toda la estructura del _filesystem_. Al montar el _filesystem_, se espera que toda esa información se lea de disco en memoria, y la operación continúe exclusivamente en memoria. Cuando el filesystem se monte (o si ocurre una llamada exclusiva a `fflush`), la data debe persistirse nuevamente en disco. De esta forma, a través de múltiples ejecuciones, los datos persistirán.

<div class="alert alert-primary" markdown="1">
**Tarea: persistencia en disco**
- Es *requisito* que el sistema de archivos se persista en disco:
  - En un único archivo, de extensión `.fisopfs`
  - Al lanzar el filesystem, se debe especificar un nombre de archivo, si no se hace, se elige por defecto.
  - Del archivo especificado se lee todo el _filesystem_, y se inicializan las estructuras acordemente (esto ocurre en la función `init`)
  - Si ocurre un `flush` o cuando el sistema de archivos se desmonta (esto ocurre en la función `exit`), la data debe persistirse en el archivo nuevamente
- El formato de la seralización del filesystem queda abierto al diseño. Se debe incluir en el informe las decisiones que se hayan tomado al respecto.
</div>

## Etapas de entrega

La entrega consistirá en dos etapas:
* Acá hay que explicar que la primer semana son reuniones de diseño y luego la implementación final

## Bibliografía útil

A continuación se presentan algunos enlaces y bibliografía útiles como referencia.
  - OSTEP, capítulo 39: [_Interlude: Files and Directories_][ostep-cap39] (PDF)
  - OSTEP, capítulo 40: [_File System Implementation_][ostep-cap40] (PDF)
  - The Linux Programming Interface, capítulo 14: _File systems_
  - Manuales de Intel: [_Intel® 64 and IA-32 Architectures Software Developer's Manual Volume 3A: System Programming Guide, Part 1_][intel] (PDF)

[ostep-cap39]: https://pages.cs.wisc.edu/~remzi/OSTEP/file-intro.pdf
[ostep-cap40]: https://pages.cs.wisc.edu/~remzi/OSTEP/file-implementation.pdf

{% include anchors.html %}
{% include footnotes.html %}
