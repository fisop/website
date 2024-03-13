# Kit de supervivencia para trabajos prácticos

Listamos aquí un material de referencia sobre un conjunto de conocimientos previos necesarios para la parte práctica de la materia.

Sin prejuicio a talleres puntuales que se planteen sobre la cursada, se recomienda el estudio individual de aquellas herramientas con las que los estudiantes estén menos familiarizados.

## Software a instalar
{: #tools}

El software indispensable para realizar los trabajos prácticos es:

  - Compiladores _GCC_ (con soporte para 32-bits) y [Clang]
  - _glibc_ (biblioteca estándar de C) y archivos _“include”_ de Linux
  - gdb, make, git, clang-format

En Debian y distribuciones derivadas, se puede instalar mediante:

```bash
$ sudo apt install make git gdb clang clang-format \
	libbsd-dev gcc-multilib libc6-dev linux-libc-dev
```

### **_QEMU_**

Es un _software_ que permite la virtualización de diferentes arquitecturas (similar a _VirtualBox_).
Además permite configurar la imagen de un _disco_ y se conecta fácilmente con _GDB_.

Es necesario únicamente para la realización del **TP2: sched**
{:.alert .alert-info}

Además de _QEMU_ es necesario instalar un simulador de BIOS (_seabios_).

```bash
$ sudo apt install seabios qemu-system-x86
```

La versión de _QEMU_ debe ser 2.5 o superior, y la versión de _seabios_ 1.10 o superior.

Existe un _known issue_ relacionado a la versión de _QEMU_ y la distribución del SO que se esté utilizando.
Una solución es descargar la versión correcta de _QEMU_ y hacer un **downgrade** de la versión actual.

Para saber si es necesario hacerlo, se recomienda ejecutar los siguientes comandos
y comparar su salida (en _especial_ tener en cuenta el _número_ final de _QEMU_,
en este caso **7.36**. La versión de su SO, puede variar):

```bash
$ qemu-system-i386 --version | head -1

QEMU emulator version 2.11.1(Debian 1:2.11+dfsg-1ubuntu7.36)

$ gcc --version | head -1

gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0

$ gdb --version | head -1

GNU gdb (Ubuntu 8.1.1-0ubuntu1) 8.1.1
```

Si su salida es similar con la que se muestra, entonces es necesario realizar el **downgrade** de la versión. Se puede hacer de la siguiente forma:

- Bajar el binario de QEMU

```bash
$ wget http://launchpadlibrarian.net/508305356/qemu-system-x86_2.11+dfsg-1ubuntu7.34_amd64.deb
```

- Realizar el _downgrade_

```bash
$ sudo dpkg -i qemu-system-x86_2.11+dfsg-1ubuntu7.34_amd64.deb
```

[Clang]: https://en.wikipedia.org/wiki/Clang

## Entorno virtual (recomendado)

Si bien la mejor alternativa es siempre correr en una distribución de Linux _nativa_, recomendamos a aquellos usuarios de Windows o MacOS que configuren un entorno Unix-like mediante el uso de una máquina virtual.

Proveemos un instructivo para configurar una máquina virtual de Ubuntu usando [Vagrant][vagrant] y [VirtualBox][virtualbox] en [esta página](vm.md).

[vagrant]: https://www.vagrantup.com/
[virtualbox]:https://www.virtualbox.org/

## Manejo básico de entornos Unix (WIP)

  - Referencias útiles para comenzar a investigar: [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/) y [Linux Journey](https://linuxjourney.com/). Los temas que se incluyen son variados, entre ellos:
    - _shell scripting & commands_
    - _Git_
    - _File System_
    - _Procesos del sistema_
    - _SSH_
    - _Permisos de usuario_
    - _Dispositivos externos_
    - _Networking_

  - [Google-fu](https://en.wiktionary.org/wiki/Google-fu)
  - Administrador de paquetes de la distribución instalada (apt, yum, etc) para instalar software.
  - Shell scripting, en _bash_ o shell de su preferencia.
  - Manejo de la línea de comandos:
    - man, vi, cd, ls, mkdir, chmod, chown, grep, sudo, head, tail, ps, cat, less, kill, strace
    - [Guía básica de la materia](https://docs.google.com/spreadsheets/d/1eKCDfeSbdcydg0VPmW0oaIONgv1DPnSSILOk2vQSv4Y/pubhtml)
  - Lenguaje C:
    - Manejo de punteros y memoria dinámica.
    - Uso de debugger _gdb_.
    - Compilación con [gcc](https://gcc.gnu.org)
  - Lenguaje Assembler intel X86
    - nasm [(Netwide Assembler)](http://www.nasm.us/)
    - Una [referencia rápida](https://www.cs.uaf.edu/2005/fall/cs301/support/x86/index.html)
    - Una [referencia un poco más completa](http://www.jegerlehner.ch/intel/IntelCodeTable.pdf)
  - Make: [uso de la herramienta y archivos Makefile](https://www.gnu.org/software/make/manual/html_node/Quick-Reference.html)
  - Recomendado instalar Linux en hardware real (no máquina virtual).
