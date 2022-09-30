# Bibliografía de la materia

A continuación se lista la bibliografía **principal** y la _complementaria_. Cada ítem tiene una abreviatura que usaremos en nuestras referencias, por ejemplo: **[DAH] §2.3**.

Para los trabajos prácticos, consultar tambien nuestro [kit de supervivencia](kit.md).

## Bibliografía principal

  - Thomas Anderson, Michael Dahlin, _Operating Systems: Principles and Practice_ (2.ª ed.), Recursive Books (2014). **\[DAH]**

  - Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau, _Operating Systems: Three Easy Pieces_ (v0.91), Arpaci-Dusseau Books (2015). **\[ARP]**

    Disponible en línea de manera gratuita en <http://ostep.org>.


## Bibliografía complementaria

  - Randal E. Bryant, David R. O’Hallaron, _Computer Systems: A Programmer’s Perspective_ (2.ª ed.), Prentice Hall (2011). **\[BRY2]**[^bry32]

    [^bry32]:
        Se debe usar la edición de 2011. En la 3.ª edición (2016) la arquitectura de referencia deja de ser x86 (32-bits) en favor de x86_64 (64-bits), que no usamos en la materia aún.

  - Michael Kerrisk, _The Linux Programming Interface: A Linux and UNIX System Programming Handbook_ (1.ª ed.), No Starch Press (2010). **\[KERR]**


## Casos de estudio

  - Russ Cox, Frans Kaashoek, Robert Morris, _[xv6]: a simple, Unix-like teaching operating system_ (draft, 2016-09-05), MIT PDOS Group [en línea][xv6b]. **\[xv6]**

  - Maurice Bach, _The Design of the Unix Operating System_, Prentice-Hall (1986). **\[BACH]**

<!--
  - Douglas Comer, _Operating System Design: The Xinu Approach_ (2.ª ed.), CRC Press (2015). **\[XINU]**
-->

  - Robert Love, _Linux Kernel Development_ (3.ª ed.), Addison-Wesley (2010). **\[LOV]**

[xv6]: https://pdos.csail.mit.edu/6.828/xv6
[xv6b]: https://pdos.csail.mit.edu/6.828/2016/xv6/book-rev9.pdf


## Programación en C

  - Brian Kernighan, Dennis Ritchie, _The C Programming Language_ (2.ª ed.), Prentice Hall (1988). **\[K&R]**

  - Richard Reese, _Understanding and Using C Pointers_, O’Reilly Media (2013). **\[REES]**


## Material de referencia

  - The Linux _man-pages_ project: [kernel.org/doc/man-pages][manpages].

    - Instalable en Debian/Ubuntu:

      `$ sudo apt-get install manpages-dev manpages-posix-dev`{:.wrap}

    - Abreviatura: **open(2)** o **malloc(3)** (nombre de la página y [sección]).

  - _Intel 64 and IA-32 Architectures Software Developer Manuals_, Intel Corporation (marzo de 2017), [en línea][intel-sdm]. **\[IA32]** \\
    En particular:

      - Volume 1: _Basic Architecture_, [pdf][ia32-1].
      - Volume 3A: _System Programming Guide, part 1_, [pdf][ia32-3a].

[manpages]: https://www.kernel.org/doc/man-pages/
[sección]: https://unix.stackexchange.com/a/3587/
[intel-sdm]: https://software.intel.com/en-us/articles/intel-sdm
[ia32-1]: https://software.intel.com/sites/default/files/managed/a4/60/253665-sdm-vol-1.pdf
[ia32-3a]: https://software.intel.com/sites/default/files/managed/7c/f1/253668-sdm-vol-3a.pdf

{% include footnotes.html %}
