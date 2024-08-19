{% assign main = site.data.main %}
{% assign links = site.data.links %}

Este es el sitio de la materia _Sistemas Operativos de FIUBA_ ({{ main.codigos }}). La información que se presenta corresponde a la cursada {{ main.cursada }}.

  - [Contacto](contacto.md)
  - [Docentes](docentes.md)

Se pide **completar** [el formulario de alta][alta]{:.alert-link} (que incluye la
suscripción a la lista de correo, y el servidor de _Discord_ a través de los cuales se comunica el curso).
{:.alert .alert-primary}

También, para los trabajos prácticos **grupales** deben inscribirse completando [este formulario][grupos]{:.alert-link}.

*[FIUBA]: Facultad de Ingeniería, Universidad de Buenos Aires

## Horarios de clase

Se dicta clase los siguiente días y horarios teniendo en cuenta la modalidad híbrida:
- **Teórica**:
  - _Día_: {{ main.teorica.dia }}
  - _Modalidad_: **{{ main.teorica.modalidad }}**
  - _Horario_: {{ main.teorica.horario }}
  - _Aula_: {{ main.teorica.aula }}
- **Práctica**:
  - _Día_: {{ main.practica.dia }}
  - _Modalidad_: **{{ main.practica.modalidad }}**
  - _Horario_: {{ main.practica.horario }}

Las clases virtuales se realizan vía **Google Meet**, y se graban las mismas cada semana.

Los enlaces para acceder a **Meet**, así como las grabaciones de clases anteriores, se acceden exclusivamente [a través del calendario de la materia][calendario]{:.alert-link}, accesible desde cuentas G Suite (`@fi.uba.ar`).
{:.alert .alert-danger}

[alta]: {{ links.alta }}
[grupos]: {{ links.grupos }}
[calendario]: {{ links.calendar }}

{% include footnotes.html %}
