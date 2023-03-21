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

Durante la cursada, se proporciona a cada estudiante un repositorio privado
en la organización [fiubatps] de GitHub. Este repositorio suele tener la forma
_sisop_año_apellido_, por ejemplo: `github.com:fiubatps/sisop_2020a_mendez`.

Para los trabajos prácticos en grupo se proporciona un segundo repositorio privado
siguiendo el esquema `sisop_año_numgrupo`, por ejemplo: `github.com:fiubatps/sisop_2020a_g8`.

### Descarga inicial
{:#clone}

Cada estudiante deberá clonar sus repositorios privados en su computadora personal,
bien vía _http_ (con contraseña), o _ssh_ (con clave privada),
siendo esta última la más recomendada y fácil de utilizar:

```
# Por HTTP
$ git clone https://github.com/fiubatps/sisop_2020a_mendez

# Por SSH
$ git clone git@github.com:fiubatps/sisop_2020a_g8
```

Se le puede agregar un segundo parámetro a _git clone_
para usar un nombre de directorio más corto,
por ejemplo “labs” o “tps”, respectivamente:

```
$ git clone git@github.com:fiubatps/sisop_2020a_simo labs
$ git clone git@github.com:fiubatps/sisop_2020a_g8 tps
```

Cada operación _git clone_ resulta en un **repositorio local** que es copia del repositorio remoto alojado en GitHub. Para poder conectarse en futuras operaciones, **Git se guarda la dirección del repositorio remoto bajo el alias _origin_.** Pueden estar presentes otros repositorios remotos bajo otros alias.
{:.alert .alert-info}

[fiubatps]: https://github.com/fiubatps/


### Elección de rama local
{:#branching}

Tanto los _labs_ individuales, como los _trabajos prácticos_ grupales, son
independientes entre sí. Esto simplifica enormemente, el desarrollo de los mismos,
haciendo que se puedan desarrollar en ramas separadas.

Lo más simple es que, al comenzar el desarrallo de cada trabajo, se creen dos ramas:
`base_xx` y `entrega_xx`. Estas serán luego, las que se utilicen para crear el
_pull request_ marcando así la entrega del trabajo.

Las instrucciones exactas para crear las ramas se proporcionan más adelante, en la sección [Integración del código](#skel-merge){:.alert-link}.
{:.alert .alert-success}

Por otra parte, se recomienda subir de manera periódica el código a _GitHub_
para que el repositorio remoto actúe como copia de seguridad del trabajo:

```
$ git push
```

**Además, para realizar consultas sobre el código,
se debe subir siempre la última versión al repositorio privado.**
De esta manera los docentes pueden consultarlo sin que sea necesario enviarlo por correo.


### Autenticación sin contraseña
{:#passwordless}

En la configuración estándar, Git pedirá una contraseña
cada vez que se comunique con el repositorio remoto en GitHub
(ya sea para _push_, _pull_, o _clone_). Hay dos maneras de evitar esto:

  - usar el protocolo _ssh_ en combinación con la herramienta `ssh-agent` del
    sistema, tal y como se explica en la documentación oficial de GitHub, [Connecting to GitHub with
    SSH][gh-ssh-en] (o, en castellano: [Conectar a GitHub con SSH][gh-ssh-es]).

  - utilizar el protocolo _https_ y usar el “ayudante de credenciales” del
    sistema operativo para almacenar la contraseña: [Caching your GitHub
    password in Git][gh-https-en] (o, en castellano: [Guardar en caché tu
    contraseña de GitHub en Git][gh-https-es]).

[gh-ssh-en]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh
[gh-ssh-es]: https://docs.github.com/es/authentication/connecting-to-github-with-ssh
[gh-https-en]: https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git
[gh-https-es]: https://docs.github.com/es/get-started/getting-started-with-git/caching-your-github-credentials-in-git


## Esqueleto del TP
{:#skel}


### Ubicación pública
{:#skel-repo}

Para cada lab y trabajo práctico se proporcionará un repositorio público
con un esqueleto sobre el que _obligatoriamente_ realizar la implementación.
En cada enunciado, se proporcionará la siguiente información:

  - la dirección del _repositorio_ donde se aloja el esqueleto
  - la _rama_ exacta que contiene el esqueleto a usar para un lab o TP
    en particular
  - si el esqueleto debe integrarse sobre una rama ya existente, o
    si se parte de él desde cero

<div class="alert alert-secondary" markdown="1">
En general, todos los labs individuales alojan su código inicial
en ramas distintas de un mismo repositorio. Por ejemplo, los dos labs
de un cuatrimestre podrían llamarse _athena_ y _apollo,_ y estar alojados en:

  - lab _athena:_ repositorio `github.com/fisop/olympians`, rama **_athena_**
  - lab _apollo:_ repositorio `github.com/fisop/olympians`, rama **_apollo_**

Los trabajos prácticos suelen hacer lo mismo, en un segundo repositorio; por ejemplo:

  - _tp1:_ repositorio `github.com/fisop/titans`, rama **_tp1_**
  - _tp2:_ repositorio `github.com/fisop/titans`, rama **_tp2_**
</div>


### Agregar _remote_
{:#skel-remote}

El paso previo a la descarga del esqueleto es agregar su repositorio público
como un “remote” distinto de _origin_. Así, para agregar los repositorios
de ejemplo descritos en la sección anterior, se debería hacer:

```
$ cd labs
$ git remote add olympians https://github.com/fisop/olympians

$ cd tps
$ git remote add titans https://github.com/fisop/titans
```

_Nota al margen:_ los alias de estos remotes (`olympians`, `titans`) son arbitrarios; podría haberse usado cualquier otro nombre sea `skel_labs`, `catedra_labs` o, meramente, algo bien breve como `labs`.
{:.small}


### Integración del código
{:#skel-merge}

Para realmente tener el código inicial en el repositorio local,
se debe descargar mediante `git fetch`, e **integrar en la rama local adecuada**
para que aparezca en el directorio de trabajo.

Los pasos siempre son:

1.  pararse en el repositorio privado (individual o grupal):
    - moverse al directorio físico: **`cd labs`** o **`cd tps`**
    - pararse en la rama _main_:  **`git checkout main`**
2.  realizar la integración del esqueleto:
    - crear una rama base: **`git checkout -b base_athena`**
    - subir la rama base al _remote_: **`git push -u origin base_athena`**
    - bajarse los últimos cambios del esqueleto: **`git fetch --all`**
    - integrar el esqueleto allí: **`git merge olympians/athena`**
      - para los _trabajos prácticos_ será necesario usar el flag `--allow-unrelated-histories`
    - subir los cambios en del esqueleto al _remote_: **`git push origin base_athena`**
3.  configurar la rama de trabajo:
    - crear una nueva rama para la entrega: **`git checkout -b entrega_athena`**
    - subir la rama para la entrega al _remote_: **`git push -u origin entrega_athena`**
4.  se trabaja **siempre** en la rama de la _entrega_

#### Labs

Por ejemplo, si _athena_ y _apollo_ son _labs_ que se implementan
de manera independiente en las semanas 2 y 6:

```
(Semana 1: clone)
$ git clone git@github.com:fiubatps/sisop_2020a_simo labs

(Semana 2: Lab Athena, en su propia rama)
$ cd labs
$ git remote add olympians https://github.com/fisop/athena
$ git checkout main

$ git checkout -b base_athena
$ git push -u origin base_athena
$ git fetch --all
$ git merge olympians/athena
$ git push origin base_athena

$ git checkout -b entrega_athena
$ git push -u origin entrega_athena

(Semana 6: Lab Apollo, en su propia rama)
$ cd labs

$ git checkout main

$ git checkout -b origin base_apollo
$ git push -u origin base_apollo
$ git fetch --all
$ git merge olympians/apollo
$ git push origin base_apollo

$ git checkout -b entrega_apollo
$ git push -u origin entrega_apollo
```

#### Trabajos prácticos

Al igual que los _labs_, los trabajos prácticos son independientes entre sí.
Supongamos, entonces, dos TPs, en la semanas 8 y 11.

```
(Semana 8: TP1: shell)
$ git clone git@github.com:fiubatps/sisop_2020a_g10 tps
$ cd tps

$ git remote add shell git@github.com:fisop/shell.git
$ git fetch --all

$ git checkout main
$ git checkout -b base_shell
$ git merge shell/main --allow-unrelated-histories
$ git push -u origin base_shell

$ git checkout -b entrega_shell
$ git push -u origin entrega_shell

(Semana 11: TP2: malloc)

$ cd tps

$ git remote add malloc git@github.com:fisop/malloc.git
$ git fetch --all

$ git checkout main
$ git checkout -b base_malloc
$ git merge malloc/main --allow-unrelated-histories
$ git push -u origin base_malloc

$ git checkout -b entrega_malloc
$ git push -u origin entrega_malloc
```

Es obligatorio hacer **`git merge`** del esqueleto en el repositorio privado (no es suficiente meramente copiar los archivos). **No se corregirán trabajos que no compartan el historial de Git con el repositorio público.**
{:.alert .alert-danger}


### Directorio de trabajo
{:#workdir}

Esta sección es relevante cuando se realiza más de un lab de manera concurrente en la cursada.
{:.alert .alert-success}

Una consecuencia de usar ramas independientes para los labs es que no es posible trabajar,
en un mismo directorio, en más de un lab a la vez. Esto, en general, no constituye un problema,
pues no se suele trabajar en más de un lab al mismo tiempo; pero puede resultar molesto
en caso de sí necesitar realizar cambios en dos labs (o trabajos prácticos) de manera concurrente.

La manera estándar de trabajar con ramas independientes sería usar `git checkout`
para alternar entre ellas. Así, si se estuviera trabajando en el lab _apollo_ y
se desease realizar algún cambio en el lab anterior _(athena)_, el procedimiento sería:

```
$ git checkout lab_athena
# realizar los cambios...
$ git commit
$ git checkout lab_apollo
```

Para que esto funcione, el directorio de trabajo debe estar previamente “limpio” (esto es, que _git status_ no reporte cambios). Además, los archivos del lab _apollo_ desaparecerían temporalmente del directorio de trabajo hasta que se volviese a la rama original, lo cual puede desorientar al editor o IDE.
{:.small}

*[IDE]: Integrated development environment

**Una alternativa es asignar un directorio distinto a cada lab (o trabajo práctico),
pero sin hacer `git clone` de nuevo.** Git ofrece esta funcionalidad mediante
el concepto de _worktree._ A través de ellos es posible tener varias “copias de trabajo”
de un mismo repositorio, siempre que a cada copia se le asigne una rama distinta.

Para usar _worktrees_ con las ramas de los labs, solamente
sería necesario sustituir la orden _git checkout -b_ explicada arriba por:

```
(Semana 2.)
...
$ git worktree add -b lab_athena ../athena esqueleto_athena/main

(Semana 6.)
...
$ git worktree add -b lab_apollo ../apollo esqueleto_apollo/main
```

donde `../athena` y `../apollo` representan las rutas donde se alojarán
las copias de trabajo adicionales (una por cada lab).

Así, en caso de usar _worktrees_ el directorio principal del repositorio
quedaría siempre en la rama _main_ y, siguiendo el ejemplo de arriba,
se crearían sendos directorios adicionales al mismo nivel que el repositorio principal:

```
$ ls
athena   apollo   labs
```


## Entrega vía _pull request_
{:#entregas}

Una vez terminado el lab o TP, se debe crear un _pull request_ en Github,
bien visitando de manera directa `https://github.com/fiubatps/sisop_<año>_<apellido>/compare`;
bien con la opción **_New pull request_** en la pestaña _Pull requests_ del repositorio.[^prinfo]

[^prinfo]: Para más información sobre _pull requests_, consultar la documentación de GitHub, [About pull requests][gh-pull-en], o en castellano: [Acerca de las solicitudes de extracción][gh-pull-es].

[gh-pull-en]: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests
[gh-pull-es]: https://docs.github.com/es/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests

Allí aparecerá una interfaz de creación de _pull request_ en la que se deberá hacer tres cosas:

1.  elegir la rama _base_ (a la izquierda)
2.  elegir la rama _compare_ (a la derecha)
3.  una vez elegidas, hacer click en _Create pull request_ y completar los
    siguientes campos: Título, _Reviewers_ y _Assignees_.


### Ramas _base_ y _compare_
{:#prcompare}

Tanto para los _labs_ como para los _TPs_, la metodología es siempre la misma.
Si por ejemplo, tenemos el lab _athena_, con las ramas `base_athena` y `entrega_athena`,
entonces, los valores de **base** y **compare** son, respectivamente,
`base_athena` y `entrega_athena`.

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

         [sisop] malloc – g8 (Méndez/Simó)
         [sisop] sched – g9 (Fresia/Raik)

  - _assignees:_ dado que los _pull requests_ se crean desde una sola cuenta de
    GitHub, si el trabajo es grupal se debe incluir en este campo al resto de
    integrantes del grupo, a fin de que les lleguen las notificaciones sobre la
    corrección.

  - _reviewers:_ en caso de ya contar con un docente asignado para las
    correcciones, se le deberá incluir en este campo. En caso contrario, se
    deberá seleccionar se puede dejar en blanco.

### Mecanismo de corrección
{:#codereview}

La corrección se realizará a través del _pull request_ creado en el paso anterior.
Así, el docente asignado revisará el código y realizará las correcciones
y comentarios oportunos. Estos comentarios se recibirán por correo electrónico,
y se podrán consultar también a través de las interfaces web de GitHub
y (en su caso) Reviewable. **Se recomienda la lectura de la corrección a través
de las interfaces web, pues:**

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

En caso de no estar claro qué es lo que se está pidiendo,
se puede pedir una aclaración con el mecanismo de respuestas
en el propio _pull request_, descrito arriba.

#### Reviewable

Algunos docentes utilizan [Reviewable](https://docs.reviewable.io)
para revisar el código, en lugar de la interfaz de GitHub.
En ese caso, se recomienda leer las correcciones en el sitio web de Reviewable,
y no a través de GitHub (pues no aparecerían los comentarios con todo el contexto necesario).
Para acceder a Reviewable, no es necesario crear una cuenta;
es suficiente con usar la opción _Sign in with GitHub_.

Asimismo, los comentarios de Reviewable tienen un concepto de “prioridad”
(o _disposition_, en inglés). En general, el significado en las
correcciones de la materia es:

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

En caso de realizarse cambios (para aprobar o para subir nota),
estos deben enviarse vía el mismo _pull request_ que fue creado para la entrega,
y **no a a través de uno nuevo**. Esto se consigue haciendo _git push_
a la rama _compare_ del pull request:

```
$ git checkout entrega_malloc
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

1.  Revisar la lista **completa** de comentarios realizados por el docente, y
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

**Como comentario final**, es especialmente importante realizar las correcciones
con gran atención al detalle, para que no suceda que una reentrega empeora
la situación (esto es, que en una reentrega dejen de funcionar aspectos
que sí funcionaban antes).

{% include footnotes.html %}

{::options parse_block_html="true" /}
