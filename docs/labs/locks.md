# Primitivas de sincronización

Se proponen varios ejercicios para estudiar la semántica de las siguientes primitivas de sincronización, y las diferencias entre ellas:

  - spin lock
  - mutex (o _sleeping lock_)
  - condition variable


## Índice
{:.no_toc}

* TOC
{:toc .sidetoc}

Los ejercicios marcados con ★ no son obligatorios, pero suman cada uno 1pt adicional.


## Spin locks
{: #spin-locks}

El siguiente programa utiliza un spin-lock de la biblioteca _pthreads_ para sincronizar dos hilos de ejecución que incrementan continuadamente un mismo contador. (Es un ejemplo artificial de mucha contención, lo cual se discute más adelante.)

```c
#define _POSIX_C_SOURCE 200809

#include <pthread.h>
#include <stdio.h>

static unsigned counter;
static pthread_spinlock_t lock;

static void *work(void *arg) {
    for (int i = 0; i < 1e7; i++) {
        pthread_spin_lock(&lock);
        counter++;
        pthread_spin_unlock(&lock);
    }
    printf("%s done\n", (char *) arg);
    return NULL;
}

int main(void) {
    pthread_t t1, t2;
    pthread_spin_init(&lock, PTHREAD_PROCESS_PRIVATE);

    pthread_create(&t1, NULL, work, "A");
    pthread_create(&t2, NULL, work, "B");

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("main done (counter = %u)\n", counter);
}
```


### Ej: spin-time
{: #ej-time}

Compilar el programa anterior y medir sus tiempos de ejecución. Para medir, usar el programa `/usr/bin/time` (no “time” a secas), para este ejercicio con la opción _-p_.

Ejemplo:

```
$ /usr/bin/time -p ./spin
A done
B done
main done (counter = 20000000)
real 0.69
user 1.32
sys 0.00
```

Se pide:

  1. Mostrar los resultados de la ejecución.

  2. Explicar qué significa cada uno de los tiempos (_real_, _user_ y _sys_)

  3. Mostrar los tiempos si se usan tres hilos en lugar de dos, y explicar los cambios observados.

  4. Explicar por qué _sys_ siempre es 0 con este programa.


### Ej: spin-xchg
{: #ej-xchg}

Suponiendo una arquitectura x86, se pide implementar en el archivo `spin-x86.c` un spin lock usando la instrucción [XCHG]. Remplazar _pthread_spinlock_t_ por esta implementación, y medir de nuevo los tiempos.

Esqueleto y primitivas assembler:[^esq]

[^esq]: En la entrega, incluir solamente los tiempos, y el archivo _spin-x86.c_.

```c
// lock.h
struct lock {
    volatile int flag;
};

void init(struct lock *lk);
void acquire(struct lock *lk);
void release(struct lock *lk);

// x86.h
static inline int xchg(volatile int *addr, int newval) {
    int result;

    __asm__ volatile("xchgl %0, %1"
                     : "+m"(*addr), "=a"(result)
                     : "1"(newval)
                     : "cc");

    return result;
}

static inline void clear(volatile int *addr) {
    __asm__ volatile("movl $0, %0" : "+m"(*addr) :);
}

// spin-x86.c
#include "lock.h"
#include "x86.h"

void init(struct lock *lk) { ... }

void acquire(struct lock *lk) { ... }

void release(struct lock *lk) { ... }
```

[xchg]: https://en.wikibooks.org/wiki/X86_Assembly/Data_Transfer#Data_swap


### Ej: spin-c11
{: #ej-c11}

Una implementación portable no debería usar instrucciones específicas de la arquitectura. En el caso de spin locks, se pueden implementar de manera genérica si el lenguaje expone primitivas atómicas de manejo de datos.

El estándar C11 introdujo soporte para tipos atómicos en C. Se pide implementar una versión del ejercicio anterior sin usar instrucciones específicas de la arquitectura, y medir de nuevo los tiempos.

Funciones recomendadas: `atomic_flag_test_and_set()`, `atomic_flag_clear()`.

Esqueleto:

```c
#include <stdatomic.h>

struct lock {
    atomic_flag flag;
};

void init(struct lock *lk);
void acquire(struct lock *lk);
void release(struct lock *lk);
```


### Ej: sys-clone ★
{: #ej-clone}

En Linux, la llamada al sistema `clone(2)` permite crear nuevos hilos de ejecución con alto nivel de configurabilidad:

> Unlike `fork()`, `clone()` allows the child process to share parts  of its execution context with the calling process, such as the memory space, the table of file descriptors, and the table of signal  handlers.

Al ser tan flexible, tanto `fork()` como `pthread_create()` se pueden implementar (o, de hecho, se implementan) en términos de `clone()`.

Se pide remplazar el uso de pthreads en el ejercicio anterior con llamadas a `clone()`, `wait()` y nuestro spin lock en C11. No es necesario usar todos los flags de `clone()` que usaría `pthread_create()`; es suficiente con aquellos que permitan resolver el caso de uso de _counter.c_.

Ayuda 1: No pasar por alto esta parte de la documentación de `clone()`:

> The low byte of flags contains the number of the termination signal sent to the parent when the child dies. If this signal is  specified  as anything other than `SIGCHLD`, then the parent process must specify the `__WALL` or `__WCLONE` options when waiting for the child with  `wait(2)`. If no signal is specified, then the parent process is not signaled when the child terminates.

Ayuda 2: Como el número de hilos es fijo en tiempo de compilación, no es necesario llamar a `malloc()` para crear cada stack. Simplemente, se puede reservar como BSS:[^stackprotect]

    #define STACK_SIZE (1 << 12)  // 4 KiB per thread.

    static unsigned char stack_space[NUM_THREADS][STACK_SIZE];

Responder:

  - asumiendo que se llamó a `clone()` sin el flag `CLONE_THREAD`: ¿qué se haría diferente, o no funcionaría, de haberlo usado?

[^stackprotect]: No habría, haciéndolo así, “guard pages” para el stack de cada thread—aunque tampoco los habría usando `malloc()`. Como ejercicio, se podrían configurar _guard pages_  combinando una llamada a `mmap(ANON)` y un par de llamadas a `mprotect(NONE)`.

## Sleeping lock (mutex)
{: #sleeping-locks}

El ejemplo anterior es un ejemplo extremo de contención, pues ambos hilos ejecutan un ciclo continuo _lock-increment-unlock_.

El modelo _productor-consumidor_ es un caso más realista: uno o más hilos de ejecución producen resultados que otros hilos procesan independientemente. El programa siguiente simula este comportamiento, con un productor y un número variable de consumidores.

En los siguientes ejercicios se pedirá medir los tiempos de distintas versiones del programa. Se debe agrupar en una tabla común los tiempos de cada versión con 1, 2, 3 y 4 consumidores. Cada celda debe incluir los tiempos reales, de sistema y usuario, más el número de cambios de contexto voluntarios e involuntarios:

    R: 17.0
    U: 32.2
    S: 0.20
    C: 2/32

Se pueden obtener los números en este formato con la siguiente invocación de `time(1)`:

    $ /usr/bin/time -f "R: %e\nU: %U\nS: %S\nC: %w/%c\n"

El programa es:

```c
// gcc -O2 -std=c99 consumers.c -lpthread -DCONSUMERS=<1|2|3|4>

#define _POSIX_C_SOURCE 200809

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>  // rand_r
#include <time.h>    // nanosleep
#include <unistd.h>  // sleep

#ifndef ITEMS
#define ITEMS 15
#endif

#ifndef CONSUMERS
#define CONSUMERS 2
#endif

static unsigned item;
static unsigned available;
static pthread_spinlock_t lock;

static void *producer(__attribute__((unused)) void *arg) {
    unsigned seed = 17;
    struct timespec ts = {};

    for (int i = 0; i < ITEMS; i++) {
        // Simulate work for producing an item.
        ts.tv_nsec = 1e8 + (rand_r(&seed) % 9) * 1e8;
        nanosleep(&ts, NULL);

        // Produce next item, marking it as available.
        pthread_spin_lock(&lock);
        available++;
        pthread_spin_unlock(&lock);
    }
    return NULL;
}

static void *consumer(void *arg) {
    while (1) {
        pthread_spin_lock(&lock);

        if (item == ITEMS) {
            pthread_spin_unlock(&lock);
            break;  // All items consumed.
        }

        if (!available) {
            pthread_spin_unlock(&lock);
#ifdef RESPIN_WAIT
            sleep(1);
#endif
            continue;
        }

        // Simulate getting the item
        printf("%s: consumed item %u\n", (char *) arg, ++item);
        available--;

        // Release the lock and simulate some work.
        pthread_spin_unlock(&lock);
        sleep(2);
    }
    return NULL;
}

int main(void) {
    pthread_t prod, consumers[CONSUMERS];
    char *names[] = {"A", "B", "C", "D", "E",
                     "F", "G", "H", "I", "J"};

    pthread_spin_init(&lock, PTHREAD_PROCESS_PRIVATE);

    for (int i = 0; i < CONSUMERS; i++) {
        pthread_create(&consumers[i], NULL, consumer, names[i]);
    }

    pthread_create(&prod, NULL, producer, "Producer");

    for (int i = 0; i < CONSUMERS; i++) {
        pthread_join(consumers[i], NULL);
    }
}
```


### Ej: queue-spin
{: #ej-queue}

Medir los tiempos de _consumer.c_ para 1, 2, 3 y 4 consumidores. Se puede usar la opción `-D` para definir el valor de `CONSUMERS` desde la línea de comandos:

```
$ for i in `seq 1 4`; do
    gcc -O2 -std=c99 consumer.c -Wall -DCONSUMERS=$i -lpthread
    /usr/bin/time -f ...
done
```


### Ej: queue-mutex
{: #ej-mutex}

Modificar _consumer.c_ para que use un `pthread_mutex_t` (inicializado con los atributos por omisión) en lugar de un spin lock. Volver a medir los tiempos.

Explicar las diferencias en tiempos y cambios de contexto respecto a la implementación con spin lock.

Lectura recomendada: por ejemplo, esta entrada en StackOverflow sobre la [diferencia entre mutex y spin lock][dif-mtx-spin].

[dif-mtx-spin]: https://stackoverflow.com/a/5870415/848301


### Ej: queue-respin-wait
{: #ej-wait}

En un intento de mejorar el consumo de CPU, los consumidores deciden dormir por un tiempo si, tras adquirir el spin lock, no hay ítems que consumir. Se puede definir, en el código original, la macro `RESPIN_WAIT` para activar este comportamiento.

Compilar y medir tiempos con la macro definida (`gcc -DRESPIN_WAIT`), y comparar los tiempos reales de ejecución con los de CPU, en esta y versiones anteriores.


### Ej: sys-futex ★
{: #ej-futex}

Al igual que la creación de threads se implementa sobre `clone(2)`, en Linux las implementaciones eficientes de sleeping locks (mutex) se implementan sobre `futex(2)` (ver la [bibliografía adicional](#futex-biblio)).

Se pide implementar `init/acquire/release` usando `futex(2)`, y medir los tiempos de nuevo, en dos versiones:

  - usando 1 como parámetro a `futex_wait()`
  - usando `INT_MAX`

Razonar las diferencias entre ambas versiones, y cómo comparan a la implementación de _mutex_ de pthreads.

Ayuda: basar la implementación en `spin-x86.c`, pues solo se deben agregar llamadas a `FUTEX_WAIT` y `FUTEX_WAKE` en los momento adecuados.

Nótese que no hay wrapper directo de glibc para `futex(2)`; se recomienda encapsular el uso de `syscall(2)` tras prototipos tales como:

    static int futex_wait(volatile int *uaddr, int val);
    static int futex_wake(volatile int *uaddr, int count);


## Condition variables
{: #condition-variables}

Bibliografía: En el capítulo 30 de **\[ARP]** se estudia en detalle el uso de condition variables para el problema _producer-consumer_, incluyendo el uso de dos variables.

### Ej: queue-condvar
{: #ej-condvar}

Añadir una variable de tipo `pthread_cond_t` en el programa, y usarla para coordinar productor y consumidor.

Completar la tabla de tiempos del lab con esta versión, y explicar las diferencias tanto en tiempo como cambios de contexto.

¿Qué dos versiones tardan menos tiempo real, y por qué? ¿Cuál tarda más, y por qué?

### Ej: cond-unlock
{: #ej-xchg}

¿Por qué toma `pthread_cond_wait()` un mutex como segundo parámetro? Describir qué ocurriría si el consumidor usara la supuesta secuencia:

```
... {
  pthread_mutex_unlock(&lock);
  pthread_cond_wait(&cond);
}
```

## Bibliografía adicional sobre _futex_
{: #futex-biblio}

Los _futex_ (cuyo nombre proviene de “fast user-space mutex”) son una llamada al sistema específica de Linux que permite una implementación más eficiente de varias primitivas de sincronización.

Los dos artículos canónicos son:

  - H. Franke et al.: “Fuss, Futexes and Furwocks: Fast Userlevel Locking” (2002). [PDF][futex1]

  - U. Drepper: “Futexes Are Tricky” (2011). [PDF][futex2][^russell]

Los siguientes enlaces pueden servir como introducción más genérica:

  - La página de manual [`futex(7)`][futex7].
  - LWN: [A futex overview and update][futexlwn].


[futex1]: https://www.kernel.org/doc/ols/2002/ols2002-pages-479-495.pdf
[futex2]: https://www.akkadia.org/drepper/futex.pdf
[futex7]: http://man7.org/linux/man-pages/man7/futex.7.html
[futexlwn]: https://lwn.net/Articles/360699/

[^russell]: Algunos de los detalles técnicos de Franke et al. (2002) son deficientes; se recomienda leer este segundo artículo con mayor atención.

{% include anchors.html %}
{% include footnotes.html %}
