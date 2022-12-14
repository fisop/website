# Manejo de Git para descargas y entregas

En esta página se explica el mecanismo de entrega para los labs y TPs de la materia. Los puntos clave son:

1.  a cada estudiante y a cada grupo se les asigna **un repositorio privado
    donde realizar el trabajo y subir el código;**

2.  una vez completado el trabajo, **la entrega se realiza desde ese mismo
    repositorio mediante un _pull request;_**

3.  para cada lab o TP, hay un repositorio público donde **se proporciona un
    esqueleto o código inicial sobre el que basar la implementación.**

El método aquí descrito es el único mecanismo de entregas válido, y es obligatorio seguir todos los pasos para garantizar que la entrega sea aceptada.
{:.alert .alert-primary}


## Índice
{:.no_toc}
* TOC
{:toc .sidetoc}


## Repositorio privado
{: #repos}

Durante la cursada, se proporciona a cada estudiante un repositorio privado en la organización [fiubatps] de GitHub. Este repositorio suele tener la forma _sisop_año_apellido_, por ejemplo: `github.com:fiubatps/sisop_2020a_mendez`.

Para los trabajos prácticos en grupo se proporciona un segundo repositorio privado siguiendo el esquema _sisop_año_numgrupo_apellidos_, por ejemplo: `github.com:fiubatps/sisop_2020a_g8_mendez_simo`.

### Descarga inicial
{:#clone}

Cada estudiante deberá clonar sus repositorios privados en su computadora personal, bien vía _http_ (con contraseña), o _ssh_ (con clave privada):

```
# Por HTTP
$ git clone https://github.com/fiubatps/sisop_2020a_mendez

# Por SSH
$ git clone git@github.com:fiubatps/sisop_2020a_g8_mendez_simo
```

Se le puede agregar un segundo parámetro a _git clone_ para usar un nombre de directorio más corto, por ejemplo “labs” o “tps”, respectivamente:

```
$ git clone git@github.com:fiubatps/sisop_2020a_simo labs
$ git clone git@github.com:fiubatps/sisop_2020a_g8_mendez_simo tps
```

Cada operación _git clone_ resulta en un **repositorio local** que es copia del repositorio remoto alojado en GitHub. Para poder conectarse en futuras operaciones, **Git se guarda la dirección del repositorio remoto bajo el alias _origin_.** Pueden estar presentes otros repositorios remotos bajo otros alias.
{:.alert .alert-info}

[fiubatps]: https://github.com/fiubatps/


### Elección de rama local
{:#branching}

En la materia se realizan dos tipos de trabajos distintos:

  - los labs individuales, que son relativamente independientes entre sí
  - los trabajos prácticos grupales, que suelen implementarse cada uno sobre el
    anterior

Así, para cada uno de los repositorios privados (individual y grupal) se trabaja de manera distinta en cuanto a ramas se refiere:

  - **para los trabajos prácticos, la implementación se realiza directamente
    sobre la rama principal** (normalmente llamada _master_ o _main_). Esto quiere decir
    que tras realizar _git clone_, ya se está en la rama adecuada para integrar
    el esqueleto.

  - en cambio, para los labs no es deseable ver la solución de uno de ellos
    mezclada en el pull request de otro (pues son independientes); siendo así,
    **la implementación de cada lab se realizará en una rama local distinta,
    que deberá ser creada con _git checkout -b_**.

Las instrucciones exactas para crear las ramas se proporcionan más adelante, en la sección [Integración del código](#skel-merge){:.alert-link}.
{:.alert .alert-success}

Por otra parte, se recomienda subir de manera periódica el código a GitHub para que el repositorio remoto actúe como copia de seguridad del trabajo:

```
$ git push
```

**Además, para realizar consultas sobre el código, se debe subir siempre la última versión al repositorio privado.** De esta manera los docentes puedan consultarlo sin que sea necesario enviarlo por correo.


### Autenticación sin contraseña
{:#passwordless}

En la configuración estándar, Git pedirá una contraseña cada vez que se comunique con el repositorio remoto en GitHub (ya sea para _push_, _pull_, o _clone_). Hay dos maneras de evitar esto:

  - usar el protocolo _ssh_ en combinación con la herramienta `ssh-agent` del
    sistema, tal y como se explica en la documentación oficial de GitHub, [Connecting to GitHub with
    SSH][gh-ssh-en] (o, en castellano: [Conectar a GitHub con SSH][gh-ssh-es]).

  - utilizar el protocolo _https_ y usar el “ayudante de credenciales” del
    sistema operativo para almacenar la contraseña: [Caching your GitHub
    password in Git][gh-https-en] (o, en castellano: [Guardar en caché tu
    contraseña de GitHub en Git][gh-https-es]).

[gh-ssh-en]: https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh
[gh-ssh-es]: https://help.github.com/es/github/authenticating-to-github/connecting-to-github-with-ssh
[gh-https-en]: https://help.github.com/en/github/using-git/caching-your-github-password-in-git
[gh-https-es]: https://help.github.com/es/github/using-git/caching-your-github-password-in-git


## Esqueleto del TP
{:#skel}


### Ubicación pública
{:#skel-repo}

Para cada lab y trabajo práctico se proporcionará un repositorio público con un esqueleto sobre el que _obligatoriamente_ realizar la implementación. En cada enunciado, se proporcionará la siguiente información:

  - la dirección del _repositorio_ donde se aloja el esqueleto
  - la _rama_ exacta que contiene el esqueleto a usar para un lab o TP
    en particular
  - si el esqueleto debe integrarse sobre una rama ya existente, o
    si se parte de él desde cero

<div class="alert alert-secondary" markdown="1">
En general, todos los labs individuales alojan su código inicial en ramas distintas de un mismo repositorio. Por ejemplo, los dos labs de un cuatrimestre podrían llamarse _athena_ y _apollo,_ y estar alojados en:

  - lab _athena:_ repositorio `github.com/fisop/olympians`, rama **_athena_**
  - lab _apollo:_ repositorio `github.com/fisop/olympians`, rama **_apollo_**

Los trabajos prácticos suelen hacer lo mismo, en un segundo repositorio; por ejemplo:

  - _tp1:_ repositorio `github.com/fisop/titans`, rama **_tp1_**
  - _tp2:_ repositorio `github.com/fisop/titans`, rama **_tp2_**
</div>


### Agregar _remote_
{:#skel-remote}

El paso previo a la descarga del esqueleto es agregar su repositorio público como un “remote” distinto de _origin_. Así, para agregar los repositorios de ejemplo descritos en la sección anterior, se debería hacer:

```
$ cd labs
$ git remote add olympians https://github.com/fisop/olympians

$ cd tps
$ git remote add titans https://github.com/fisop/titans
```

_Nota al margen:_ los alias de estos remotes (_olympians_, _titans)_ son arbitrarios; podría haberse usado cualquier otro nombre sea _skel_labs_, _catedra_labs_ o, meramente, algo bien breve como _labs_.
{:.small}


### Integración del código
{:#skel-merge}

Para realmente tener el código inicial en el repositorio local, se debe descargar mediante _git fetch_, e **integrar en la rama local adecuada** para que aparezca en el directorio de trabajo.

En 2020/2, el lab _kern0_ no precisa de la integración aquí descrita, ya que los repositorios creados ya incluyen el esqueleto.
{:.alert .alert-warning}

<!-- TODO: para los labs deberíamos dejar de requerir la rama base_xxx, y hacer
los pull requests siempre contra main. Es más o menos lo que decidimos en
2020/1 tras ver lo complicado que resultaba el setup para los labs. -->

Los pasos siempre son:

1.  pararse en el repositorio privado (individual o grupal) **usando _cd_**
2.  realizar la integración del esqueleto en _main_. Para ello:
    - bajarse los últimos cambios del esqueleto con **_git fetch --all_**
    - pararse en la rama _main_ con **_git checkout_**
    - integrar el esqueleto allí con **_git merge_**
    - crear una rama de referencia a esta integración con **_git branch_**
3.  _(solamente para labs independientes)_{:.text-danger} crear, a partir del
    esqueleto, una rama de trabajo separada con **_git checkout -b_**
4.  enviar las nuevas ramas al repositorio remoto con **_git push_**

Por ejemplo, si _athena_ y _apollo_ son labs que se implementan de manera independiente en las semanas 2 y 6, se haría (ver abajo para un ejemplo alternativo con JOS):

```
(Semana 1: clone)
$ git clone git@github.com:fiubatps/sisop_2020a_simo labs

(Semana 2: Lab Athena, en su propia rama)
$ cd labs

$ git fetch --all
$ git checkout main
$ git merge --no-ff -m "Integrar esqueleto del lab athena" olympians/athena
$ git branch base_athena

$ git checkout -b lab_athena olympians/athena

$ git push -u origin --all

(Semana 6: Lab Apollo, en su propia rama)
$ cd labs

$ git fetch --all
$ git checkout main
$ git merge --no-ff -m "Integrar esqueleto del lab apollo" olympians/apollo
$ git branch base_apollo

$ git checkout -b lab_apollo olympians/apollo

$ git push -u origin --all
```

Los TPs grupales típicamente se implementan uno sobre el otro, por lo que los pasos serían similares, excepto que se eliminaría la necesidad de
`git checkout -b`. Esto ocurre en los TPs de JOS:
{:.alert .alert-success}

```
(Semana 8: TP1 de JOS, en main)
$ cd tps

$ git remote add jos https://...
$ git fetch --all
$ git checkout main
$ git merge --no-ff -m "Integrar esqueleto del TP1" jos/tp1
$ git branch base_tp1

$ git push -u origin --all
```

Es obligatorio hacer _git merge_ del esqueleto en el repositorio privado (no es suficiente meramente copiar los archivos). **No se corregirán trabajos que no compartan el historial de Git con el repositorio público.**
{:.alert .alert-danger}


### Directorio de trabajo
{:#workdir}

Esta sección es relevante cuando se realiza más de un lab de manera concurrente en la cursada. No es el caso en 2020/2.
{:.alert .alert-success}

Una consecuencia de usar ramas independientes para los labs es que no es posible trabajar, en un mismo directorio, en más de un lab a la vez. Esto, en general, no constituye un problema, pues no se suele trabajar en más de un lab al mismo tiempo; pero puede resultar molesto en caso de sí necesitar realizar cambios en dos labs de manera concurrente.

La manera estándar de trabajar con ramas independientes sería usar _git checkout_ para alternar entre ellas. Así, si se estuviera trabajando en el lab _apollo_ y se desease realizar algún cambio en el lab anterior _(athena)_, el procedimiento sería:

```
$ git checkout lab_athena
# realizar los cambios...
$ git commit
$ git checkout lab_apollo
```

Para que esto funcione, el directorio de trabajo debe estar previamente “limpio” (esto es, que _git status_ no reporte cambios). Además, los archivos del lab _apollo_ desaparecerían temporalmente del directorio de trabajo hasta que se volviese a la rama original, lo cual puede desorientar al editor o IDE.
{:.small}

*[IDE]: Integrated development environment

**Una alternativa es asignar un directorio distinto a cada lab, pero sin hacer _git clone_ de nuevo.** Git ofrece esta funcionalidad mediante el concepto de _worktree._ A través de ellos es posible tener varias “copias de trabajo” de un mismo repositorio, siempre que a cada copia se le asigne una rama distinta.

Para usar _worktrees_ con las ramas de los labs, solamente sería necesario sustituir la orden _git checkout -b_ explicada arriba por:

```
(Semana 2.)
...
$ git worktree add -b lab_athena ../athena esqueleto_athena/main

(Semana 6.)
...
$ git worktree add -b lab_apollo ../apollo esqueleto_apollo/main
```

donde `../athena` y `../apollo` representan las rutas donde se alojarán las copias de trabajo adicionales (una por cada lab).

Así, en caso de usar _worktrees_ el directorio principal del repositorio quedaría siempre en la rama _main_ y, siguiendo el ejemplo de arriba, se crearían sendos directorios adicionales al mismo nivel que el repositorio principal:

```
$ ls
athena   apollo   labs
```


## Entrega vía _pull request_
{:#entregas}

Una vez terminado el lab o TP, se debe crear un _pull request_ en Github, bien visitando de manera directa `https://github.com/fiubatps/sisop_<año>_<apellido>/compare`; bien con la opción **_New pull request_** en la pestaña _Pull requests_ del repositorio.[^prinfo]

[^prinfo]: Para más información sobre _pull requests_, consultar la documentación de GitHub, [About pull requests][gh-pull-en], o en castellano: [Acerca de las solicitudes de extracción][gh-pull-es].

[gh-pull-en]: https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests
[gh-pull-es]: https://help.github.com/es/github/collaborating-with-issues-and-pull-requests/about-pull-requests

Allí aparecerá una interfaz de creación de _pull request_ en la que se deberá hacer tres cosas:

1.  elegir la rama _base_ (a la izquierda)
2.  elegir la rama _compare_ (a la derecha)
3.  una vez elegidas, hacer click en _Create pull request_ y completar los
    siguientes campos: Título, _Reviewers_ y _Assignees_.


### Ramas _base_ y _compare_
{:#prcompare}

Para los labs, la rama _base_ es siempre `main`, y la rama _compare_ aquella a donde se subió el código (típicamente, una rama con el mismo nombre que el lab).

Para los TPs, la rama _base_ es siempre la rama que fue creada en el momento de integración del esqueleto, concretamente en el paso: “crear una rama de referencia a esta integración con _git branch_”. Por ejemplo, _base_tp1_.

En cuanto a la rama _compare_, para los TPs se la crea a mano en el momento, a partir de la rama principal:

    $ git push origin main:refs/heads/entrega_tp2

Así, en este ejemplo la rama _compare_ sería _entrega_tp2;_ **no se debe usar _main_ para este propósito.**


### Campos a completar
{:#prfields}

Antes de finalizar la creación del _pull request_, se deben completar los siguientes campos:

 - _título:_
   - para entregas individuales, el nombre del lab más el apellido, siguiendo
     el formato:

         [sisop] lab shell – Méndez
         [sisop] lab virt – Simó

   - para entregas grupales, se agrega el número de grupo a los apellidos,
     siguiendo el formato:

         [sisop] jos tp3 – g8 (Méndez/Simó)
         [sisop] xv6 alloc – g9 (Fresia/Raik)

  - _assignees:_ dado que los _pull requests_ se crean desde una sola cuenta de
    GitHub, si el trabajo es grupal se debe incluir en este campo al resto de
    integrantes del grupo, a fin de que les lleguen las notificaciones sobre la
    corrección.

  - _reviewers:_ en caso de ya contar con un docente asignado para las
    correcciones, se le deberá incluir en este campo. En caso contrario, se
    deberá seleccionar _fiubatps/sisop-adm_.

<!-- TODO: make format, make test, make entrega. -->

### Mecanismo de corrección
{:#codereview}

La corrección se realizará a través del _pull request_ creado en el paso anterior. Así, el docente asignado revisará el código y realizará las correcciones y comentarios oportunos. Estos comentarios se recibirán por correo electrónico, y se podrán consultar también a través de las interfaces web de GitHub y (en su caso) Reviewable. **Se recomienda la lectura de la corrección a través de las interfaces web, pues:**

  - aparecerán los comentarios junto con el código a que estos se refieren
    (en otras palabras, los comentarios aparecerán con el contexto exacto
    en que se realizaron)

  - se podrá contestar a los comentarios de manera directa desde la
    interfaz web, lo cual permite saber con exactitud a qué comentario o
    corrección se refiere cada respuesta (cosa que no ocurre con igual
    facilidad por correo)

Junto con los comentarios, la corrección recibida indicará claramente:

  - si el TP está aprobado o no;
  - en caso de no estar aprobado, qué cambios es imprescindible realizar
    para poder aprobarlo;
  - en caso de estar aprobado, si el corrector aceptaría cambios
    adicionales para mejorar la nota, y cuáles son estos cambios.

En caso de no estar claro qué es lo que se está pidiendo, se puede pedir una aclaración con el mecanismo de respuestas en el propio _pull request_, descrito arriba.

#### Reviewable

Algunos docentes utilizan [Reviewable](https://docs.reviewable.io) para revisar el código, en lugar de la interfaz de GitHub. En ese caso, se recomienda leer las correcciones en el sitio web de Reviewable, y no a través de GitHub (pues no aparecerían los comentarios con todo el contexto necesario). Para acceder a Reviewable, no es necesario crear una cuenta; es suficiente con usar la opción _Sign in with GitHub_.

Asimismo, los comentarios de Reviewable tienen un concepto de “prioridad” (o _disposition_, en inglés). En general, el significado en las correcciones de la materia es:

  - _blocking_ (marcados con el ícono de prohibición <span class="fa
    fa-minus-circle"></span>): si el TP no está aprobado, se debe corregir
    este ítem para poder aprobar; si está aprobado, para subir la nota se
    deben corregir TODOS los ítems marcados de esta manera.

  - _discussing_ (marcados con el círculo vacío <span class="far
    far-circle"></span>): el ítem merece la pena ser corregido, pero no
    es obligatorio hacerlo.

  - _informing_ (marcados con el ícono de información <span class="fa
    fa-info-circle"></span>): corrección menor o informativa.


### Reentregas y mejoras
{:#reentregas}

En caso de realizarse cambios (para aprobar o para subir nota), estos deben enviarse vía el mismo pull request que fue creado para la entrega, y no a a través de uno nuevo. Esto se consigue haciendo _git push_ a la rama _compare_ del pull request:

```
$ git checkout entrega_tp2
# ...
$ git commit ...
$ git push
```

O bien, si se usan _worktrees:_

```
$ cd shell
# ... realizar cambios
$ git commit ...
$ git push
```


Una vez enviados los cambios con _git push_, se deben hacer dos cosas:

1.  Revisar la lista completa de comentarios realizados por el docente, y
    responder a los mismos indicando si se realizaron los cambios solicitados
    (en los casos más triviales es suficiente con responder _Done_ o _Hecho_;
    Reviewable proporciona un botón para esto); si corresponde, responder
    también a cualquier pregunta que hubiera hecho el docente, e indicar qué
    decisiones se tomaron en la (re-)implementación.[^commreply]

    {:.small}
    - En el caso de Reviewable, es importante que no queden “pending
      conversations”, esto es, que al enviar los comentarios, Reviewable
      no diga “waiting on ... nombre del alumne”.

2.  Una vez respondidos los comentarios, hacer explícito en el _pull request_
    que éste está listo para ser revisado de nuevo. Esto se puede hacer al
    tiempo que se envían los comentarios, incluyendo el acrónimo PTAL en la
    respuesta (estándar en la industria anglosajona con significado _Please
    take another look_).

    Como alternativa, este paso también se puede realizar vía la interfaz
    de GitHub, con el ícono de solicitud de revisión (dos flechas en
    círculo <span class="fa fa-sync-alt"></span>) que aparece al lado del
    campo _Reviewer_.

[^commreply]: Esta práctica es estándar en la industria, de manera que se le haga fácil a la persona que revisa la nueva versión saber qué se hizo y qué no, esto es, con qué se va a encontrar.

**Como comentario final**, es especialmente importante realizar las correcciones con gran atención al detalle, para que no suceda que una reentrega empeora la situación (esto es, que en una reentrega dejen de funcionar aspectos que sí funcionaban antes).

{% include footnotes.html %}

{::options parse_block_html="true" /}
