# TP2: _Scheduling_, interrupciones y _threads_


## Índice
{:.no_toc}

* TOC
{:toc .sidetoc}


## Introducción

**AVISO**: antes de comenzar, verificar que se tiene instalado [el software necesario](../kit.md#tools){:.alert-link}.
{:.alert .alert-warning}

En este trabajo se extenderá un sistema operativo preexistente para explorar tres grandes áreas del diseño de un sistema operativo moderno: el mecanismo de cambio de contexto, el _scheduling_ (i.e. planificación) dirigido por hardware, y el soporte para múltiples hilos de ejecución (_threads_). El kernel a utilizar será una modificación de JOS, un exokernel educativo con licencia libre del grupo de [Sistemas Operativos Distribuidos][pdos] del MIT.

[pdos]: https://pdos.csail.mit.edu/

JOS está diseñado para correr en la arquitectura Intel x86, y para poder ejecutarlo utilizaremos QEMU que emula dicha arquitectura.

**AVISO**: las diapositivas están en [sched](https://docs.google.com/presentation/d/1XvHYx1Fo7fMPvcLOqMCRNIw3hCB05jLkU36oU0QFFQ0?usp=sharing)
{:.alert .alert-info}


## Esqueleto
{: #skel}

**AVISO**: El esqueleto se encuentra disponible en [fisop/sched](https://github.com/fisop/sched){:.alert-link}.
{:.alert .alert-warning}

**IMPORTANTE**: leer el archivo `sched/README.md` que se encuentra en la raíz del proyecto. Contiene información sobre cómo realizar la compilación de los archivos, y cómo ejecutar el formateo de código.
{:.alert .alert-warning}

### Integración
{: #integration}

Suponiendo que ya se **clonó** el repositorio privado en algún directorio:

```bash
git clone git@github.com:fiubatps/sisop_<año_cuatrimestre>_g1 tps
cd tps
```

Para integrar el esqueleto de la cátedra, hacer:

- **asegurarse** de estar en la rama **main**
```bash
git checkout main
```

- **agregar remoto** de la cátedra
```bash
git remote add sched git@github.com:fisop/sched.git
```

- **creación** de la rama **base**
```bash
git checkout -b base_sched
git push -u origin base_sched
```

- **merge** del esqueleto
```bash
git fetch --all
git merge sched/main --allow-unrelated-histories
git push origin base_sched
```

- **creación** de la rama **entrega**
```bash
git checkout -b entrega_sched
git push -u origin entrega_sched
```

**IMPORTANTE**: asegurarse de siempre commitear en la rama **entrega_sched**.
{:.alert .alert-danger}

### Compilación
{: #compile}

La compilación se realiza mediante `make`. En el directorio `obj/kern` se puede encontrar:

  - _kernel_ — el binario ELF con el kernel
  - _kernel.asm_ — assembler asociado al binario

La política de _scheduling_ se elige al compilar:

  - `USE_RR=1` — scheduler _round robin_ (Partes 1 y 2)
  - `USE_PR=1` — scheduler con prioridades (Parte 3)

Por ejemplo:

```bash
make qemu USE_RR=1
make qemu USE_PR=1
```

### Ejecución
{: #run}

Para correr JOS, se puede usar `make qemu` o `make qemu-nox`.

Para ejecutar _un proceso de usuario_ en particular dentro del kernel, se puede usar `make run-<proceso>` o `make run-<proceso>-nox`. Como ejemplo, `make run-hello-nox` correrá el proceso de usuario `user/hello.c`.

Los programas de usuario que se ejecutan al arrancar el kernel se eligen con `ENV_CREATE(user_<nombre>, ENV_TYPE_USER)` en `kern/init.c`. Cualquier programa nuevo en `user/` tiene que agregarse además a `KERN_BINFILES` en `kern/Makefrag` para que se compile y quede embebido en el kernel. Los tests `user/sleeptest.c`, `user/priotest.c` y `user/threadtest.c` ya están declarados ahí.

Las pruebas automáticas se corren con:

```bash
make grade USE_RR=1
make grade USE_PR=1
```

El entorno de desarrollo se puede levantar con Docker:

```bash
./dock build   # construir la imagen
./dock run     # entrar al container, con el repo montado en /sched
```

### Depurado
{: #debug}

El _Makefile_ de JOS incluye dos reglas para correr QEMU junto con GDB.

En una terminal ejecutar:

```
$ make qemu-gdb USE_RR=1
***
*** Now run 'make gdb'.
***
qemu-system-i386 ...
```

y en otra distinta:

```
$ make gdb
gdb -q -ex 'target remote ...' -n -x .gdbinit
Reading symbols from obj/kern/kernel...done.
Remote debugging using 127.0.0.1:...
0x0000fff0 in ?? ()
(gdb)
```

#### Depurado de una triple fault
{: #debug-triple-fault}

En la arquitectura x86, el sistema se reinicia automáticamente cuando ocurre una “triple falla” _(triple fault)_. QEMU, por omisión, obedece esta especificación.

Sin embargo, durante el desarrollo de sistemas operativos en modo protegido de x86, las _triple fault_ ocurren casi exclusivamente por un bug en el kernel. Por esto, es más deseable que QEMU detenga la ejecución en lugar de reiniciarse constantemente.

QEMU no ofrece soporte _directo_ para detectar fallas triples y detener la ejecución, pero existen un set de opciones que se acercan bastante a ese propósito.

Por tanto, si en una determinada versión del desarrollo ocurre que QEMU se reinicia constantemente, se recomienda probar lo siguiente:

  - correr QEMU con las opciones: `-no-reboot -no-shutdown -d cpu_reset` (estas
    opciones pueden añadirse en la variable `QEMUOPTS` en el archivo _GNUmakefile_)

  - si el error realmente fue un _Triple fault_, se mostrará ese error en la
    última línea del archivo _qemu.log_, y se podrá consultar el estado de
    los registros mediante el monitor de QEMU (`Ctrl-A C → info registers`)


## Uso de _IA_

**IMPORTANTE**: es requisito mencionar cómo utilizaron herramientas de _IA_ en la implementación. Dicha mención no debe estar realizada con _IA_.
{:.alert .alert-danger}


## Implementación

La implementación del TP se dividirá en cuatro partes, de dificultad creciente.

1. Implementación del cambio de contexto [^ctx-switch]
2. Implementación de un scheduler _round robin_ preemptive con timer.
3. Implementación de un scheduler con prioridades y _aging_.
4. Implementación de _threads_ en espacio de usuario.

[^ctx-switch]: Tanto de modo kernel a modo usuario como **viceversa**.

**IMPORTANTE**: las partes 1 y 2 deben completarse en ese orden antes de seguir con 3 y 4.
{:.alert .alert-warning}

### Parte 1: Cambio de contexto

JOS mantiene un arreglo en memoria como PCB (_Process Control Block_), aunque llama _environment_ a los procesos. De aquí en más se usarán las palabras _proceso_ y _environment_ como sinónimos siempre que hablemos en el contexto de JOS.

Las funciones que se encargan de alocar espacio para un proceso nuevo, crear su espacio de direcciones virtuales y cargar el código en memoria ya se encuentran implementadas, como se puede ver en el archivo `kern/env.c`.

Entre tales funciones se encuentran:
- `env_alloc`: que reserva el espacio en el PCB para un proceso nuevo, y le inicializa algunos parámetros
- `env_setup_vm`: que inicializa el espacio de direcciones virtuales (i.e. el _page directory_) del proceso
- `load_icode`: que carga el código del proceso a partir del binario compilado
- `env_destroy` y `env_free`: para eliminar a un proceso una vez que termina

No las modificaremos en esta parte, pero es importante entender dónde y cómo son llamadas para comprender el flujo de vida de un proceso en JOS. En la Parte 3 se pedirán cambios puntuales en `env_alloc` y en la Parte 4 en `env_free`.

La definición de un _environment_ puede encontrarse en `inc/env.h` y contiene, entre otras cosas, los campos necesarios para realizar el _cambio de contexto_. A continuación algunos de los campos del mismo struct.

```
struct Env {
	struct Trapframe env_tf;	// Saved registers
	struct Env *env_link;		// Next free Env
	envid_t env_id;			// Unique environment identifier
	envid_t env_parent_id;		// env_id of this env's parent
	enum EnvType env_type;		// Indicates special system environments
	unsigned env_status;		// Status of the environment
	uint32_t env_runs;		// Number of times environment has run
	int env_cpunum;			// The CPU that the env is running on
	pde_t *env_pgdir;		// Kernel virtual address of page dir
  [...]
}
```

Los más importantes del _struct_ son: `env_id`, que identifica al environment; `env_pgdir`, que contiene su _page directory_ (i.e. su espacio de direcciones virtuales a través de la tabla de paginación inicial) y `env_tf`, que mantiene el _estado de todos los registros_ para ese environment.

#### De modo _kernel_ a modo _usuario_
{: #kernel-user}

A partir de esa información, el kernel podrá ejecutar cualquier proceso. La función que se encarga de tomar un proceso y ejecutarlo es `env_run`, en `kern/env.c`. Como parámetro, esta función acepta un `struct Env *` y deberá realizar lo siguiente:

1. Actualizar la variable global `curenv` del kernel, con el nuevo proceso a ser ejecutado
2. Modificar el _estado_ del environment `env_status` para indicar que está siendo ejecutado. La lista de estados puede verse en un `enum` dentro de `inc/env.h`.
3. Realizar el _cambio de contexto_.
	1. Cargar la tabla de paginación del environment con `env_load_pgdir` (función ya implementada)
	2. Llamar a la función `context_switch` para restaurar el estado de CPU

Será la función `context_switch` (en `kern/switch.S`) la que restaure completamente el estado del environment a correr, y que realice el cambio de contexto a _modo usuario_. Es decir, esta función **no hace return jamás**, y como resultado de la misma la CPU pasará a ejecutar código de usuario en `ring 3`.

Para ello, se utilizará la ayuda del hardware, mediante la instrucción `iret` ("_interrupt return_"). Dicha instrucción permite modificar conjuntamente los registros `cs`, `eip` y `esp` de forma atómica, tomando valores desde el stack. El formato que requiere del stack para ser invocada es específico a la arquitectura x86.

Cabe notar que el resto de los registros definidos en `struct Trapframe` deben ser restaurados previamente, dado que `iret` no los modifica.

#### De modo _usuario_ a modo _kernel_
{: #user-kernel}

El _cambio de contexto_ descrito anteriormente nos permite realizar el pasaje de modo kernel a modo usuario (es decir, de `ring 0` a `ring 3`). Sin embargo, dicho mecanismo no puede utilizarse para volver a modo kernel, dado que requiere de la instrucción privilegiada `iret`.

Para volver al modo kernel, se utilizan _interrupciones_. Las interrupciones son eventos generados por _hardware_ que interrumpen al CPU en su ciclo de instrucciones y trasladan la ejecución de una forma controlada a otro contexto, permitiendo cambiar registros importantes (`eip`, `cs`, `esp`, etc.) a valores fijos definidos previamente.

El kernel configura las interrupciones en `kern/trap.c`, mediante la función `trap_init`. Ahí se genera la tabla de interrupciones (la IDT) con referencias a los _handlers_ de cada tipo de interrupción.

Un tipo de interrupción común es la `syscall`. Todas las syscalls pasarán por esta única interrupción, y desembocarán en la función `syscall` del lado del kernel que se encargará de determinar qué _syscall_ se necesita ejecutar y llamar a la función `sys_*` adecuada. En `kern/syscall.c`.

Así, un _handler_ para la interrupción de las _syscalls_ se define de la siguiente forma:

```
SETGATE(idt[T_SYSCALL], 0, GD_KT, &trap48, 3);
```

Los detalles de la macro `SETGATE` no son importantes, pero mediante los parámetros se está indicando al CPU que siempre que se genere la _interrupción número 48_ (la que corresponde a las syscalls, dado que `T_SYSCALL=48`), esperamos que se llame a la función `trap48`, que se corresponde al _handler_ de la interrupción de ese número.

Mediante el resto de los parámetros, específicamente `GD_KT` (_Global Descriptor, Kernel Text_) se está indicando al CPU que siempre que se llame a ese _handler_, se deberá hacerlo en el `ring 0`. La función `trap48` está definida, de forma auto-generada vía macros, en `kern/trapentry.S`.

Como es el kernel quien define la tabla de interrupciones, y se coloca a si mismo como punto de entrada luego de cualquier interrupción, dicha entrada al kernel está controlada y el paso de `ring 3` a `ring 0` es seguro.

Observando `kern/trapentry.S` todos los _handlers_ están generados usando las macros `TRAPHANDLER` y `TRAPHANDLER_NOEC`, y todos desembocan en la función `_alltraps`, que está incompleta y deberán implementar.

<div class="alert alert-primary" markdown="1">
**Tarea**
  - Implementar la función `context_switch` en `kern/switch.S`.
    - La función está en assembler, para la arquitectura x86
  - Completar la función `env_run`, en `kern/env.c`
  - Implementar la función `_alltraps` en `kern/trapentry.S`.
    - La función está en assembler, para la arquitectura x86
    - Al momento de invocarse la función, el stack está _en el mismo estado_ en el que lo dejamos al llamar a `iret`, con la excepción de los valores pusheados por las macros `TRAPHANDLER_NOEC` y `TRAPHANDLER`.
    - La función debe dejar un `struct Trapframe` en el stack, completando los registros faltantes, y terminar con una llamada a la función `trap`.
  - Modificar `kern/init.c` de forma _temporal_, para ejecutar un único proceso `user_hello`
  - **Informe**: Utilizar GDB para visualizar el cambio de contexto. Mostrar:
    - el contenido del _stack_ al inicio de la llamada `context_switch`
    - el contenido del _stack_ instrucción a instrucción
    - cómo se modifican los registros instrucción a instrucción
    - el camino inverso: desde una syscall en _user space_ hasta `trap` en el kernel
</div>

Con ambas tareas implementadas, la ejecución de cualquier proceso debería poder llegar a su fin. Sin embargo, solo podemos ejecutar un proceso a la vez dado que no hay _scheduler_ implementado.

### Parte 2: Scheduler _round robin_ preemptivo con timer

**Ejecutar**: `make qemu USE_RR=1` para compilar y correr, y `make grade USE_RR=1` para correr las pruebas
{:.alert .alert-danger}

Los tests automáticos validan el scheduler base: que los procesos alternen, no se pisen y terminen. El resto de esta parte (`sys_sleep` y las estadísticas) se valida a mano con `user/sleeptest.c` y con la salida del kernel.

#### El timer como mecanismo de _preemption_
{: #timer}

En la Parte 1, el kernel solo recupera el control cuando el proceso hace una syscall voluntariamente. Un scheduler real también debe poder **interrumpir** un proceso que no cede la CPU.

JOS ya tiene configurado el _LAPIC_ timer para generar interrupciones periódicas (`IRQ_TIMER`). Al producirse, `trap_dispatch` en `kern/trap.c` llama a `lapic_eoi()` para confirmar la interrupción y luego a `sched_yield()`. Con ésto, el scheduler puede reemplazar al proceso actual aunque éste no haya llamado a `sys_yield`.

Existe un contador global `ticks`, definido en `kern/trap.c` y declarado en `kern/trap.h`, que debe incrementarse en cada interrupción de timer. Este contador permite medir el tiempo transcurrido desde el arranque del kernel.

#### _Round robin_
{: #round-robin}

El esqueleto tiene preparado ya todo lo necesario para el _scheduler_, en `kern/sched.c`. La función `sched_yield` es la que se invoca cada vez que se necesita elegir el próximo proceso a ejecutar, y es aquí donde la política de scheduling deberá ser implementada.

Notar que `sched_yield` tiene dos posibles salidas: se elige y ejecuta un proceso llamando a `env_run`, o bien _no hay más procesos que ejecutar_ y se desemboca en `sched_halt`, donde efectivamente el kernel queda en estado _idle_.

Con _round robin_:
- Se recorre la lista de _environments_ de forma completa, empezando justo después del último proceso ejecutado.
- Se elige el primero que esté listo para ejecutarse.
- Si no hay ninguno y el proceso que se estaba ejecutando sigue en ejecución, se lo puede volver a elegir.
- Si no hay nada que ejecutar, el kernel queda en estado _idle_.

#### Syscall `sys_sleep`
{: #sys-sleep}

Implementar la syscall `SYS_sleep` (stub en `kern/syscall.c`). El wrapper de usuario en `lib/syscall.c` y su prototipo en `inc/lib.h` ya están provistos.

```c
int sys_sleep(uint32_t n);
```

La syscall:
- marca al proceso para que no sea ejecutable
- guarda el momento a partir del cual tiene que despertar
- debe retornar un valor

Considerar que en cada interrupción del _timer_ se debe actualizar el contador global y revisar si hay procesos que deban ser despertados.

**NOTA**: tener en cuenta que la syscall en sí no retorna, pero igualmente debe devolver un valor de retorno.
{:.alert .alert-info}

**IMPORTANTE**: atención al efecto que tiene _sleep_ sobre el final de la ejecución. Un proceso dormido no es ejecutable: ¿qué ocurre si todos los procesos duermen al mismo tiempo? ¿Cómo se distingue "no hay nada que ejecutar por ahora, pero el _timer_ va a despertar a alguien" de "ya no queda ningún proceso vivo"?
{:.alert .alert-warning}

#### Estadísticas
{: #rr-stats}

Al finalizar todos los procesos (en `sched_halt`), imprimir al menos:
- Número total de llamadas al scheduler.
- Número de veces que cada proceso fue ejecutado.
- Tiempo total de ejecución en ticks.

<div class="alert alert-primary" markdown="1">
**Tarea**
  - Agregar a JOS la política basada en _round robin_.
  - La función es: `sched_yield` en `kern/sched.c` dentro de `#ifdef SCHED_ROUND_ROBIN`.
  - Implementar `sched_wakeup_sleeping` en `kern/sched.c`.
  - Implementar `sys_sleep` en `kern/syscall.c`.
  - Inicializar `env_sleep_until` en `env_alloc` (`kern/env.c`): las entradas del arreglo `envs` se reciclan, así que un environment nuevo puede arrastrar el valor del anterior si no se resetea.
  - Completar las estadísticas en `sched_halt`.
  - Ejecutar `user/sleeptest.c` y verificar que los procesos duermen y despiertan correctamente. Para correrlo: `make run-sleeptest-nox USE_RR=1` (o bien reemplazar los `ENV_CREATE` de `kern/init.c` por `ENV_CREATE(user_sleeptest, ENV_TYPE_USER)`).
  - **Informe**:
    - Describir cómo el timer genera _preemption_: desde la interrupción hardware hasta el cambio de proceso.
    - Mostrar con GDB un momento en que `sched_yield` es llamado desde el handler de timer (no desde una syscall).
    - Comparar el comportamiento con y sin _preemption_ ejecutando `user/spin.c` (un solo `ENV_CREATE`, el propio programa hace `fork()`). El hijo hace `while(1) /* nada */` sin ceder la CPU nunca: si el padre logra volver a correr después (`"Killing the child..."`), fue exclusivamente por _preemption_ del timer, no por cooperación del hijo. Para desactivar la _preemption_ y comparar, comentar momentáneamente la llamada a `sched_yield()` en el caso `IRQ_TIMER` de `trap_dispatch` — el kernel debería quedar trabado en el hijo para siempre.
</div>

### Parte 3: Scheduler con prioridades y _aging_

**Ejecutar**: `make qemu USE_PR=1` para compilar y correr, y `make grade USE_PR=1` para correr las pruebas
{:.alert .alert-danger}

Los tests automáticos validan que el scheduler siga siendo correcto con la política de prioridades activa; el comportamiento propio de las prioridades se valida con `user/priotest.c` y con los tests propios que se pidan más abajo.

#### Motivación
{: #pr-motivation}

La política de scheduling _round robin_ es la más sencilla y simple de implementar; y aunque es justa (le da a todos los procesos la misma proporción del CPU) puede no ser suficiente para situaciones más reales. Usualmente los procesos son distintos entre sí en cuanto a importancia y carga para el sistema. Un scheduler con prioridades permite reflejar eso.

Sin embargo, un scheduler puramente basado en prioridades puede causar **starvation**: los procesos de baja prioridad nunca obtienen CPU si siempre hay procesos de mayor prioridad disponibles. El mecanismo de **aging** mitiga esto aumentando gradualmente la prioridad efectiva de los procesos que llevan mucho tiempo esperando.

#### Prioridades en el `struct Env`
{: #pr-env}

El `struct Env` ya tiene los campos:

```c
uint32_t env_priority;   // prioridad del environment
uint32_t env_wait_ticks; // ticks acumulados esperando (para aging)
```

#### Política de scheduling
{: #pr-policy}

En lugar de recorrer los procesos en orden circular, el scheduler debe elegir el proceso listo con mayor **prioridad efectiva**, entendida como la prioridad base del proceso más un bonus que crece con el tiempo que lleva esperando su turno:

```
prioridad_efectiva = env_priority + aging_bonus(espera acumulada)
```

#### Syscalls de prioridad
{: #pr-syscalls}

Implementar:

```c
int sys_getpriority(envid_t envid);
int sys_setpriority(envid_t envid, uint32_t priority);
```

Reglas de seguridad:
- Un proceso **no puede aumentar su propia prioridad**.
- Un proceso puede **reducir su propia prioridad**.
- Un proceso puede modificar la prioridad de sus hijos directos (con cualquier valor).
- `sys_setpriority` con `envid == 0` refiere al proceso actual.

Como `sys_getpriority` usa su valor de retorno tanto para la prioridad como para los códigos de error (negativos), el rango de prioridades válidas debe ser acotado y no negativo. Definir ese rango y documentarlo en el informe; también hace falta para la estadística de distribución de CPU por prioridad.

#### Prioridad inicial y herencia en `fork`
{: #pr-fork}

- Todo proceso nuevo debe recibir una prioridad por defecto al crearse (modificar `env_alloc` o `env_create`).
- Cuando un proceso hace `fork`, el hijo hereda la prioridad del padre (o una fracción de ella, a criterio del equipo — justificar en el informe).

#### Estadísticas
{: #pr-stats}

Además de las de la Parte 2, agregar:
- Historial de los últimos N procesos ejecutados (proceso + tick de inicio).
- Distribución de CPU por prioridad (cuántos ticks acumuló cada nivel de prioridad).

**PISTA**: `env_priority` es la prioridad **base** y el _aging_ no la modifica: el bonus vive en la prioridad efectiva, que se recalcula en cada decisión. Unas estadísticas que sólo muestren `env_priority` se ven idénticas en una corrida donde el _aging_ fue decisivo y en una donde nunca actuó.
{:.alert .alert-info}

<div class="alert alert-primary" markdown="1">
**Tarea**
  - Agregar a JOS la política basada en _prioridades_ con _aging_.
  - La función es: `sched_yield` en `kern/sched.c` dentro de `#ifdef SCHED_PRIORITIES`.
  - Implementar `sys_getpriority` y `sys_setpriority` en `kern/syscall.c`.
  - Modificar `env_alloc` en `kern/env.c` para asignar prioridad por defecto.
  - Modificar el manejo de `fork` para heredar prioridad.
  - Ejecutar `user/priotest.c` (vía `ENV_CREATE(user_priotest, ENV_TYPE_USER)` en `kern/init.c`).
  - **Informe**:
    - Describir la función de _aging_ elegida, la constante que usa y por qué se eligió ese valor.
    - Indicar el peor caso de espera de esa función, en cantidad de decisiones de scheduling, y explicar por qué eso alcanza para descartar _starvation_.
    - Mostrar con resultados del scheduler que un proceso de baja prioridad eventualmente obtiene CPU, y con el contraste sin _aging_ que sin ese mecanismo no lo obtenía.
    - Explicar la política de herencia de prioridad en `fork` y su razonamiento.
    - Comparar las estadísticas entre `USE_RR=1` y `USE_PR=1` para el mismo conjunto de procesos.
</div>

Al terminar esta parte, con `USE_PR=1`:

- El scheduler elige por prioridad efectiva, existen `sys_getpriority` y `sys_setpriority` con las reglas de seguridad de arriba, todo proceso arranca con una prioridad por defecto y uno creado usando `fork` hereda la del padre.
- Las estadísticas incluyen el historial de decisiones y la distribución de CPU por prioridad.
- `user/priotest.c` corre y muestra que la política favorece a los procesos de alta prioridad (dejar un único `ENV_CREATE(user_priotest, ENV_TYPE_USER)`; el binario ya está declarado). Cuidado con la expectativa: con prioridades el proceso de mayor prioridad acapara la CPU hasta terminar, así que lo que corresponde ver es a los hijos terminando en orden descendente de prioridad, y no cuatro series de líneas intercaladas de forma pareja. Correr el mismo test con `USE_RR=1` da el contraste.
- Un test propio demuestra que el _aging_ evita _starvation_. El escenario son dos procesos CPU-bound con prioridades distintas que nunca cedan la CPU por su cuenta (así lo único que puede sacársela es la _interrupción_ del timer). Hay que mostrar que el proceso postergado obtiene CPU _antes_ de que el otro termine. Los programas de usuario nuevos van en `user/` y hay que agregarlos a `KERN_BINFILES` en `kern/Makefrag`.
- El mismo test, anulando el bonus de _aging_, muestra la _starvation_ contra la que se compara.

### Parte 4: _Threads_ en espacio de usuario

**Ejecutar**: `make qemu USE_RR=1` o `make qemu USE_PR=1`: los _threads_ son independientes de la política de scheduling, y deben funcionar con ambas.
{:.alert .alert-danger}

#### Procesos vs _threads_
{: #threads-vs-procs}

Hasta ahora, cada `struct Env` tiene su propio _page directory_ (`env_pgdir`): su espacio de direcciones es completamente separado del de otros procesos. Para comunicarse, los procesos deben usar IPC.

Un **thread** es una unidad de ejecución que comparte el espacio de direcciones de su proceso padre. Múltiples _threads_ dentro del mismo proceso ven la misma memoria, lo que permite comunicación directa a través de variables compartidas. Ésto requiere coordinación (sincronización) para evitar condiciones de carrera.

#### Modelo de implementación
{: #threads-model}

En JOS, un _thread_ se representa como un `struct Env` con `env_type = ENV_TYPE_THREAD` cuyo `env_pgdir` apunta al **mismo** _page directory_ que su proceso padre. El scheduler lo trata exactamente igual que a un proceso normal: tiene su propio `env_tf` (registros), su propia pila, y puede estar en cualquier estado (`ENV_RUNNABLE`, `ENV_NOT_RUNNABLE`, etc.).

La diferencia clave entre las _syscalls_ que crean procesos y _threads_ es:
- `sys_exofork` crea un environment con un _page directory_ **propio**.
- `sys_thread_create` **comparte** el _page directory_ del proceso creador, sin crear uno nuevo.

#### Syscall `sys_thread_create`
{: #sys-thread-create}

```c
envid_t sys_thread_create(void *entry, void *ustack_top);
```

- Crea un nuevo `struct Env` con `env_type = ENV_TYPE_THREAD`.
- Copia `env_pgdir` del proceso actual al nuevo env (sin duplicar las páginas).
- Inicializa `env_tf` del _thread_ para que comience a ejecutar en `entry` con el stack en `ustack_top`.
- El _thread_ hereda el `env_parent_id` del proceso creador.
- Retorna el `envid` del nuevo _thread_.

Cuando el proceso padre es destruido (`env_free`), todos sus _threads_ deben ser destruidos también. Modificar `env_free` para contemplar esto.

**IMPORTANTE**: antes de tocar `env_free`, leerla con atención: tal como está, desmapea todo el espacio de usuario del environment y libera su _page directory_. Aplicada sobre un _thread_, que comparte el `env_pgdir` con su proceso, destruiría la memoria del proceso y del resto de los _threads_. Revisar también el _refcount_ de la página del _page directory_ (`pp_ref`, ver cómo lo maneja `env_setup_vm`) al compartirlo entre varios environments.
{:.alert .alert-warning}

#### Librería de usuario
{: #threads-lib}

El archivo `lib/thread.c` contiene el stub de `thread_create`:

```c
envid_t thread_create(void (*func)(void *), void *arg);
```

Esta función debe:
1. Alocar páginas para el stack del nuevo _thread_ con `sys_page_alloc`.
2. Preparar el stack de modo que al retornar de `func` se llame a `exit()`.
3. Llamar a `sys_thread_create(func, stack_top)`.

Se debe decidir la dirección donde alocar el _stack_ y documentar el criterio. Tener en cuenta que la página `[USTACKTOP - PGSIZE, USTACKTOP)` ya está ocupada por el stack del _thread_ principal (ver `load_icode` en `kern/env.c`), y que los stacks de los _threads_ no deben solaparse entre sí.

#### Consideraciones
{: #threads-considerations}

- ¿Qué ocurre si dos _threads_ modifican la misma variable simultáneamente? Mostrar un ejemplo concreto de condición de carrera en el informe (aunque no es necesario implementar sincronización).
- ¿Qué pasa con el _page directory_ cuando un _thread_ termina pero el padre sigue vivo? ¿Y al revés?
- ¿Puede un _thread_ hacer `fork`? ¿Qué heredaría el hijo?
- La variable global `thisenv` la inicializa `libmain` (y la corrige `fork`) para el proceso, pero un _thread_ arranca directamente en `entry` y comparte esa variable con el resto del proceso. ¿Qué implica eso para un _thread_ que quiera conocer su propio `env_id`? ¿Cómo lo resolvería un sistema real?

<div class="alert alert-primary" markdown="1">
**Tarea**
  - Implementar `sys_thread_create` en `kern/syscall.c`.
  - Modificar `env_free` en `kern/env.c` para destruir los _threads_ hijos al destruir el proceso padre.
  - Completar `thread_create` en `lib/thread.c` (el prototipo ya está en `inc/lib.h`).
  - Completar `user/threadtest.c` (esqueleto provisto): crear al menos 3 _threads_ que compartan una variable global e impriman su progreso.
  - Demostrar que los _threads_ ven la misma memoria (variable compartida modificada por un _thread_ es visible para los demás).
  - **Informe**:
    - Explicar la diferencia entre el _page directory_ en `sys_exofork` vs `sys_thread_create`.
    - Mostrar un ejemplo de condición de carrera entre dos _threads_.
    - Describir cómo se decidió dónde alocar los stacks de los _threads_ y por qué.
    - Comparar el costo de creación de un _thread_ vs un proceso (`fork`): ¿cuántas páginas se copian en cada caso?
</div>

[labguide]: https://pdos.csail.mit.edu/6.828/2017/labguide.html


## Bibliografía útil

A continuación se presentan algunos enlaces y bibliografía útiles como referencia.
  - OSTEP, capítulo 7: [_Scheduling: Introduction_][ostep-cap7] (PDF)
  - OSTEP, capítulo 8: [_Scheduling: The Multi-Level Feedback Queue_][ostep-cap8] (PDF)
  - OSTEP, capítulo 9: [_Scheduling: Proportional Share_][ostep-cap9] (PDF)
  - OSTEP, capítulo 26: [_Concurrency: An Introduction_][ostep-cap26] (PDF)
  - Manuales de Intel: [_Intel® 64 and IA-32 Architectures Software Developer's Manual Volume 3A: System Programming Guide, Part 1_][intel] (PDF)

[ostep-cap7]: https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf
[ostep-cap8]: https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf
[ostep-cap9]: https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-lottery.pdf
[ostep-cap10]: https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched-mlfq.pdf
[ostep-cap26]: https://pages.cs.wisc.edu/~remzi/OSTEP/threads-intro.pdf
[intel]: https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html

{% include anchors.html %}
{% include footnotes.html %}
