# Régimen de cursada y evaluaciones

### Índice
{:.no_toc}

* TOC
{:toc .sidetoc}

## Instancias evaluatorias
{: #eval}

La cursada comprenderá tres tipos distintos de instancias evaluatorias:

  - 1 "_lab_" [**individual**]
  - 3 "_trabajos prácticos_" [**grupal** (de cuatro personas)]
  - 1 "_parcial_" - teórico-práctico [**individual**]

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
preguntar en clase y en la [lista](../contacto)/_Discord_ tanto como sea
necesario, hasta aclarar cualquier duda posible.

Los **estados** posibles para un _lab_ corregido son:

**APROBADO**: pasan correctamente todas las pruebas automáticas
y la solución, al igual que respuestas en prosa (en caso de corresponder)
son conceptualmente correctas.
{:.alert .alert-success}

**REGULAR**: no pasan las pruebas por errores en el código (_no_ conceptuales)
y/o la solución no sigue _buenas prácticas de programación_.
{:.alert .alert-warning}

**DESAPROBADO**: no pasan todas las pruebas por errores _conceptuales_ en el código
{:.alert .alert-danger}

### Descripción de los trabajos prácticos
{: #tps}

Los _trabajos prácticos_ grupales consisten en la implementación de
funcionalidad relacionada con un componente importante de un sistema operativo.

Si bien las consignas también son guiadas, al tratarse de grupos de
cuatro personas, también se dejará lugar al diseño y modularización de su solución.
Siendo ésto una parte importante de la nota final de cada _trabajo práctico_.

### Parcialitos
{: #parcialitos}

Cada uno de los _trabajos prácticos_ tendrá una instancia de evaluación _individual_ asociada.
Estas instancias se denominan "parcialitos".

Los _parcialitos_ serán evaluaciones cortas (de no más de 30/40 minutos), de carácter
_choice_ o respuestas muy breves en prosa. Los mismos serán
durante las clases prácticas, en la semana siguiente de la entrega de los _trabajos
prácticos_. Los temas a evaluar estarán relacionados con el trabajo de turno a
entregar.

**No son recuperables**, y su resultado será lo que permanezca
para el cómputo de la calificación.
{:.alert .alert-danger}

Representan un **30% de la calificación** del _trabajo práctico_.
{:.alert .alert-primary}

### Descripción del parcial
{: #parcial}

El parcial se constituye de una única evaluación cerca del final de la cursada,
contemplando todos los temas de la materia (kernel, memoria, _scheduling_ y _file system_).

El parcial podrá ser recuperado en dos oportunidades;
con fechas a determinar durante la cursada.

## Cálculo de notas
{: #notas}

Cada una de estas instancias tendrá una calificación numérica entre 0 y 10 puntos
(con **excepción** de los [_labs_](#labs)).

La nota de los parcialitos, se computa automáticamente (al ser _choice_),
pero la nota final de cada uno es:

```
nota_parcialito_final = round_up(nota_parcialito)
```

- Ejemplo 1

```
nota_parcialito = 7.3

nota_parcialito_final = round_up(7.3)
nota_parcialito_final = 8
```

- Ejemplo 2

```
nota_parcialito = 7.7

nota_parcialito_final = round_up(7.7)
nota_parcialito_final = 8
```

La nota de cada _trabajo práctico_, se calcula de la siguiente forma:

```
nota_tp_i = round_near(nota_grupal_tp_i * 0.7 + nota_parcialito_tp_i * 0.3)
```

- Ejemplo 1

```
nota_grupal_tp_1 = 7.5

nota_parcialito_tp_1 = 10

nota_tp_1 = round_near(7.5 * 0.7 + 10 * 0.3)
nota_tp_1 = round_near(8.25)
nota_tp_1 = 8
```

- Ejemplo 2

```
nota_grupal_tp_2 = 8

nota_parcialito_tp_2 = 10

nota_tp_1 = round_near(8 * 0.7 + 10 * 0.3)
nota_tp_1 = round_near(8.60)
nota_tp_1 = 9
```

La nota final de la cursada, será una composición ponderada de las notas de cada
una de las instancias de evaluación de acuerdo a la siguiente fórmula:

```
promedio_tps = (tp1 + tp2 + tp3) / 3

nota_cursada = (0.4 * parcial + 0.6 * promedio_tps)

nota_cursada_final = lab.state == 'APROBADO'
	? nota_cursada = round_up(nota_cursada)
	: nota_cursada = trunc(nota_cursada)
```

- Ejemplo 1:

```
fork = 'APROBADO'

nota_cursada = 7.20

nota_cursada_final => 8
```

- Ejemplo 2:

```
fork = 'APROBADO'

nota_cursada = 7.80

nota_cursada_final => 8
```

- Ejemplo 3:

```
fork = 'REGULAR'

nota_cursada = 7.20

nota_cursada_final => 7
```

- Ejemplo 4:

```
fork = 'REGULAR'

nota_cursada = 7.80

nota_cursada_final => 7
```

## Criterios de aprobación
{: #aprob}

Para aprobar la cursada/materia es necesario:

  - Aprobar el parcial, en primera instancia o en recuperatorio
  - Aprobar el _lab_ y todos los _trabajos prácticos_
  - El _lab_ **debe** estar en los estados `APROBADO` o `REGULAR`,
    **nunca** en `DESAPROBADO`
  - Cumplir con el 75% de asistencia a las clases teóricas

## Instancias recuperatorias
{: #recup}

Los parcialitos **no admiten reentrega**.
Esto quiere decir que la entrega inicial será la válida para computar la nota.
{:.alert .alert-danger}

El _lab_ admite **una** reentrega cuando esté **REGULAR**.
Asimismo, es **obligatoria** la reentrega en caso
de estar **DESAPROBADO**
{:.alert .alert-danger}

En casos excepcionales, el docente a cargo de las correcciones podrá indicar
cambios específicos permitidos y/o necesarios como segunda entrega de un
trabajo, pudiendo imponer un techo en la calificación máxima del mismo. En caso de ser
correcciones _necesarias_ para la aprobación, las mismas deberán llevarse a cabo
en el tiempo y forma acordados.

El **criterio estricto** de la entrega de los _trabajos prácticos_ y _lab_ es que a la
fecha de entrega original se haya **entregado algo** que demuestre un avance
sustancial.
{:.alert .alert-danger}

**No se aceptarán** prórrogas de la fecha de entrega si no se ha mostrado
avance/interés. En tales condiciones, la regularidad se perderá automáticamente.
Aún así, cualquier retraso o demora en la entrega deberá ser previamente
consensuada con el equipo docente.

En caso de desaprobar el parcial, el mismo podrá ser recuperado durante las
fechas dedicadas para tal fin (sobre el final del cuatrimestre).

## Ejercicios opcionales (_challenges_)
{: #challenges}

**IMPORTANTE**: avisar de su implementación a la casilla [fisop-doc@googlegroups.com][doc]. El _asunto_ del mail debería ser: **Entrega challenge: [nombre del challenge]**.
{:.alert .alert-danger}

**REQUERIDO**: realizar la implementación en una rama separada del _lab_ o _trabajo práctico_.
{:.alert .alert-warning}

La idea de estos ejercicios opcionales es que sirvan para
mejorar la nota final de aquellos que así lo deseen.

Cada desafío entregado (y _aprobado_) sumará **1 punto**
a la nota final de la cursada.

Tanto en el _lab_ como en los _trabajos prácticos_ habrá ejercicios marcados como
opcionales. Están para explorar con más profundidad algunos temas y representan un
desafío en cuanto a complejidad e investigación respecto a los ejercicios
obligatorios.

Cabe mencionar que las consignas de estos ejercicios suelen ser mucho menos
específicas, y quedará definir con el corrector asignado qué es suficiente para
aprobar los mismos.

A lo largo de la cursada, se expondrán distintos _desafíos_ como parte adicional
de los _trabajos_ (individual y grupales). Éstos son de carácter no obligatorio y serán
claramente señalizados.

Los desafíos van a estar visibles tan pronto como se llegue al lab/trabajo práctico
de la materia al que estén asociados, y no hay limitaciones con que se realicen
_durante_ la cursada.
Lo importante es que los mismos serán evaluados _luego_ de la
cursada, durante las mismas fechas de final.

[doc]: mailto:fisop-doc@googlegroups.com

{::options toc_levels="2" /}
{% include footnotes.html %}
