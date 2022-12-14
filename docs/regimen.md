# Régimen de cursada y evaluaciones

### Índice
{:.no_toc}

* TOC
{:toc .sidetoc}

## Instancias evaluatorias
{: #eval}

La cursada comprenderá tres tipos distintos de instancias evaluatorias:

  - 2 entregas individuales - denominadas "labs"
  - 3 entregas grupales (de tres/cuatro personas) - denominadas "trabajos prácticos"
  - 1 parcial - teórico-práctico, individual

Cada una de esas instancias tendrá una calificación numérica entre 0 y 10 puntos
(con excepción de los [_labs_](#labs)).

La nota final de la cursada, será una composición ponderada de las notas de cada
una de las instancias de evaluación de acuerdo a la siguiente fórmula:

```
promedio_tps = (tp1 + tp2 + tp3) / 3

nota_cursada = (0.4 * parcial + 0.6 * promedio_tps)

if (lab1.state == 'APROBADO' && lab2.state == 'APROBADO') {
	nota_cursada = round_up(nota_cursada)
} else {
	nota_cursada = trunc(nota_cursada)
}
```

- Ejemplo 1:

```
fork = 'APROBADO'
shell = 'APROBADO'

nota_cursada = 7.20

nota_cursada => 8
```

- Ejemplo 2:

```
fork = 'APROBADO'
shell = 'APROBADO'

nota_cursada = 7.80

nota_cursada => 8
```

- Ejemplo 3:

```
fork = 'APROBADO'
shell = 'REGULAR'

nota_cursada = 7.20

nota_cursada => 7
```

- Ejemplo 4:

```
fork = 'APROBADO'
shell = 'REGULAR'

nota_cursada = 7.80

nota_cursada => 7
```

Además, existen instancias evaluadoras complementarias, denominadas [*challenges*](#challenges)
o desafíos, que exceden al régimen de evaluación regular pero contribuyen para
el régimen de [**final alternativo**](#final) (el cual se explica más adelante).

### Descripción de los labs
{: #labs}

Los _labs_ consisten en una serie de ejercicios guiados a realizar de manera
individual al inicio de la cursada. Estos ejercicios, toman, en general, la
forma de tareas de programación, respuestas en prosa, búsqueda y lectura de
información en internet, o algún ejemplo de interacción con la terminal.

El propósito de los _labs_ es de preparar a los estudiantes de cara a los trabajos
prácticos, de manera que los integrantes de cada grupo hayan cubierto una
cantidad de material y conocimientos previos similar.

Por su carácter formativo, se recomienda leer mucho, dedicarles tiempo, y
preguntar en clase y en la [lista](index.md#contacto) tanto como sea
necesario, hasta aclarar cualquier duda posible.

Los **estados** posibles para un _lab_ corregido son:

**APROBADO**: pasan correctamente todas las pruebas automáticas
(y las respuestas en prosa son correctas)
{:.alert .alert-success}

**REGULAR**: no pasan todas las pruebas por errores menores en el código (_no_ conceptuales)
{:.alert .alert-warning}

**DESAPROBADO**: no pasan todas las pruebas por errores conceptuales en el código
{:.alert .alert-danger}

### Descripción de los trabajos prácticos
{: #tps}

Los _trabajos prácticos_ grupales consisten en la implementación de
funcionalidad relacionada con un componente importante de un sistema operativo.

Si bien las consignas también son guiadas, al tratarse de grupos de
tres/cuatro personas, también se dejará lugar al diseño y modularización de su solución.
Siendo ésto una parte importante de la nota final de cada _trabajo práctico_.

### Parcialitos
{: #parcialitos}

Cada uno de los _trabajos prácticos_ tendrá una instancia de evaluación _individual_ asociada,
en la semana de la entrega. Estas instancias se denominan "parcialitos".

Los _parcialitos_ serán evaluaciones cortas (de no más de 30/40 minutos), de carácter
_choice_ o respuestas muy breves en prosa. Los mismos serán
durante las clases prácticas, a la semana siguiente de la entrega de los _trabajos
prácticos_. Los temas a evaluar estarán relacionados con el trabajo de turno a
entregar.

Los parcialitos **no son recuperables**, y su resultado será lo que permanezca
para el cómputo de la calificación.
{:.alert .alert-danger}

Los _parcialitos_ representan un **30% de la calificación** del _trabajo práctico_.
{:.alert .alert-primary}

### Descripción del parcial
{: #parcial}

El parcial constituirá en una única evaluación a mitad de la cursada, contemplando
los temas dados en la primera mitad de la materia (kernel, memoria y scheduling).

El parcial podrá ser recuperado en dos oportunidades;
con fechas a determinar durante la cursada.

### Ejercicios opcionales
{: #challenges}

Tanto en los _labs_ como en los _trabajos prácticos_ habrá ejercicios marcados como
opcionales. Están para explorar con más profundidad algunos temas y representan un
desafío en cuanto a complejidad e investigación respecto a los ejercicios
obligatorios.

Éstos ejercicios contribuyen al régimen de [**final alternativo**](#final).

Cabe mencionar que las consignas de estos ejercicios suelen ser mucho menos
específicas, y quedará definir con el corrector asignado qué es suficiente para
aprobar los mismos.

## Criterios de aprobación
{: #aprob}

Para aprobar la cursada es necesario:

  - Aprobar el parcial, en primera instancia o en recuperatorio
  - Aprobar todos los _trabajos prácticos_ y _labs_
  - Los _labs_ **deben** estar en los estados `APROBADO` o `REGULAR`,
    **nunca** en `DESAPROBADO`

Con estas condiciones, se podrán anotar a rendir el coloquio.

## Instancias recuperatorias
{: #recup}

Tanto los _trabajos prácticos_ como los parcialitos **no admiten reentrega**.
Esto quiere decir que la entrega inicial será la válida
para computar la nota.
{:.alert .alert-danger}

Los _labs_ admiten **una** reentrega cuando están como **REGULAR**.
Asimismo, es **obligatoria** la reentrega en caso
de estar **DESAPROBADO**
{:.alert .alert-danger}

En casos excepcionales, el docente a cargo de las correcciones podrá indicar
cambios específicos permitidos y/o necesarios como segunda entrega de un
trabajo, pudiendo imponer un techo en la calificación máxima del mismo. En caso de ser
correcciones _necesarias_ para la aprobación, las mismas deberán llevarse a cabo
en el tiempo y forma acordados.

El criterio estricto de la entrega de los _trabajos prácticos_ y _labs_ es que a la
fecha de entrega original se haya _entregado algo_ que demuestre un avance
sustancial.

**No se aceptarán** prórrogas de la fecha de entrega si no se ha mostrado
avance/interés. En tales condiciones, la regularidad se perderá automáticamente.
Aún así, cualquier retraso o demora en la entrega deberá ser previamente
consensuada con el equipo docente.

En caso de desaprobar el parcial, el mismo podrá ser recuperado durante las
fechas dedicadas para tal fin (sobre el final del cuatrimestre).

## Criterio de promoción
{: #promocion}

Aquellos/as alumnos/as que califiquen con las siguientes condiciones, podrán, si
así lo desean, apelar a la promoción. En caso de cumplir con los requisitos de
promoción, el/la alumno/a quedaría exento de rendir el coloquio-final, quedándole
como calificación final de la materia la nota de cursada redondeada al entero
más cercano.

El criterio de promoción es el siguiente:
  - Nota final de cursada `>= 8`
  - Aprobar el parcial en primera instancia, con calificación `>= 8`
  - Aprobar todos los _labs_, con estado **APROBADO** (ya sea en primera instancia
    o en la reentrega)
  - Aprobar todos los trabajos grupales, con calificación `>= 7` en cada uno
  - Aprobar todos los parcialitos, con calificación `>= 5` en cada uno

## Final alternativo
{: #final}

Independientemente del criterio de promoción, para la evaluación integradora
cualquiera puede optar por reemplazar el coloquio-final con _trabajos prácticos_
adicionales.

A lo largo de la cursada, se expondrán distintos [desafíos](#challenges) como parte adicional
de los _trabajos prácticos_. Éstos son de carácter no obligatorio y serán
claramente señalizados.

Quienes opten por el final alternativo deberán cumplir lo siguiente:
  - Aprobar la cursada, de forma regular
  - Elegir 3 desafíos opcionales de los presentados. Realizarlos y entregarlos, a
    más tardar, hasta la **segunda fecha de final**.

Los desafíos van a estar visibles tan pronto como se llegue al trabajo práctico
de la materia al que estén asociados, y no hay limitaciones con que se realicen
_durante_ la cursada (si el/la alumno/a quiere optar por este final
alternativo). Lo importante es que los mismos serán evaluados _luego_ de la
cursada, durante las mismas fechas de final.

{::options toc_levels="2" /}
{% include footnotes.html %}
