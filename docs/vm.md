# Entornos de desarrollo

Se recomienda utilizar para la cursada un entorno basado en Linux (e.g. una distribución como Ubuntu). Es importante por razones de compatibilidad: algunas de las tareas (sobre todo los desafíos) están pensados para Linux, y no se traducen directamente a otros sistemas operativos.

Para los TPs se utilizará **qemu**, que si bien es posible correrlo en otros sistemas, no lo hemos probado exhaustivamente y se nos dificultará darles soporte.

Por esta razón, recomendamos utilizar una distribución basada en Linux; o bien emularla haciendo uso de una máquina virtual.

## Máquina virtual para FISOP

A continuación se describe cómo levantar una máquina virtual que podrá utilizarse como entorno para los labs y TPs de la materia. Se hace uso de _Vagrant_ y _VirtualBox_.

### Dependencias:

Para poder levantar la _VM_ se necesitan:

- Vagrant: se puede descargar de [https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)
- VirtualBox: se puede descargar de [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
- Vagrant guest plugins: se puede instalar corriendo `vagrant plugin install
  vagrant-vbguest`

### Uso

Vagrant permite configurar máquinas virtuales a través de una especificación de las mismas en un archivo llamado Vagrantfile.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/bionic64'

  config.vm.network 'forwarded_port', guest: 3000, host: 3000
  config.vm.hostname = 'sisop'

  config.vm.provider 'virtualbox' do |vb|
    vb.memory = '2048'
    vb.name = config.vm.hostname
    # Descomentar si se quiere que la VM levante con interfaz gráfica
    # vb.gui = true
  end

  config.vm.provision 'shell', privileged: false, inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y git make gdb qemu-system-x86 python3-dev
    sudo apt-get install -y seabios libbsd-dev gcc-multilib libc6-dev linux-libc-dev clang vim
    # Descomentar si se quiere que la VM levante con interfaz gráfica
    # sudo apt-get install -y lubuntu-desktop virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
  SHELL
end
```

Dejar este archivo (Vagrantfile) en un directorio (e.g. fisop-vm/) y estando en
ese directorio:
- **vagrant up**: levanta la máquina virtual (la primera vez va a tardar unos minutos en descargar la imagen base)
- **vagrant halt**: apaga la máquina virtual
- **vagrant destroy**: elimina la máquina virtual y **todos sus contenidos**

### Detalles de la máquina virtual

Esta máquina virtual está basada en Ubuntu Bionic (18.04), y viene con los paquetes indicados en el [kit de supervivencia](kit.md) ya instalados.

**IMPORTANTE**: el directorio en la máquina host donde se encuentre el archivo
Vagrantfile (y desde donde se ejecutan los comandos vagrant) se comparte con
la máquina virtual. Dentro de la VM es accesible en /vagrant.
{: .alert .alert-primary}

**CUIDADO**: los datos fuera del directorio `/vagrant` de la VM **NO están persistidos** en el sistema huésped, si se ejecuta `vagrant destroy` o se elimina la VM de otra forma, se perderán los datos.
{: .alert .alert-warning}

La máquina virtual se instala con la interfaz gráfica de lubuntu. El usuario
`vagrant` tiene como pass `vagrant`. 

La primera vez que se corre se instalan todos los paquetes (puede demorar unos
minutos y requiere conexión a internet) y luego es necesario rebootear la VM
(esto puede lograrse corriendo en el host `vagrant halt` seguido de `vagrant
up`).

### Conectarse a la máquina virtual

Ejecutando el comando `vagrant ssh` (nuevamente, desde el directorio donde se encuentre el Vagrantfile) se puede obtener una **línea de comandos**
dentro de la máquina virtual.

Alternativamente, se puede levantar con interfaz gráfica, en tal caso hay que descomentar las líneas indicadas en el Vagrantfile **antes de levantar la VM por primera vez**. Cada vez que se haga **vagrant up** saldrá un pop-up con la interfaz gráfica.

### Contraindicaciones para JOS

Para los TPs grupales de JOS (específicamente el TP4), se hacen uso de archivos compartidos. Dependiendo del sistema de archivos del OS huésped, esto podría fallar si se compila JOS en la carpeta compartida `/vagrant` (dado que, por supuesto, este directorio pertenece al OS huésped, y por lo tanto, usa su sistema de archivos).

Si se tienen problemas (e.g. errores con `mmap`) se recomienda **hacer una copia** del código de JOS a algún otro directorio **dentro de la VM** (por ejemplo `/home/vagrant`). De este modo se evita el problema, pero hay que tener en cuenta que los archivos _no estarán persistidos fuera de la VM_: si se hiciera `vagrant destroy` esos datos se perderían.

#### Ejemplo

Pueden desarrollar JOS en un subdirectorio en su entorno local, por ejemplo, `desarrollo-jos`. Además de incluir el repositorio de `jos`, en ese directorio es donde copiarían el archivo `Vagrantfile`.

```
$ tree desarrollo-jos
desarrollo-jos
├── Vagrantfile
└── jos
```

En este ejemplo, los comandos como `vagrant up` y `vagrant ssh` se correrían desde `desarrollo-jos`. Desde dentro de la VM, los contenidos de `desarrollo-jos` se verán en `/vagrant`.

Antes de compilar, tienen que copiar el código de JOS a un directorio nativo de la VM, Esto lo pueden lograr, por ejemplo, con el siguiente script:

```
mkdir -p $HOME/jos
cp -r /vagrant/jos/* $HOME/jos
```

Luego pueden copiar JOS moviéndose a `$HOME/jos`

```
cd $HOME/jos
make grade
```

### Debbuging con gdb

Si no se utiliza interfaz gráfica, es posible obtener varias líneas de comando dentro de la VM (con `vagrant ssh` desde distintas terminales en el huésped).
