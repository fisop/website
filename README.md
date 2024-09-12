Repositorio para FIUBA 75.08
============================

De momento existen los siguientes directorios:

  • docs - sitio Jekyll que se publica en https://fisop.github.io/website

    Se pueden añadir cualquier tipo de contenido aquí: Markdown, HTML,
    PDF, incluso código o diapositivas

Instrucciones GitHub Pages
==========================

Cualquier commit en el directorio ‘docs’ se auto-publica en la página al
hacer push a la rama ‘master’. GitHub se encarga de ese proceso.

Para visualizar los cambios de manera local, se debe instalar _Jekyll_. El
archivo docs/Gemfile lo hace bastante fácil.

Setup inicial
-------------

```bash
$ cd docs
$ sudo ../install-deps.sh
```

Visualizar al editar
--------------------

```bash
$ cd docs
$ ../run.sh
```

