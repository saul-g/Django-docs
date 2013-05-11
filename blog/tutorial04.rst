===============================
Creando un Calendario en Django
===============================

Como siguiente paso vamos a crear un sencillo calendario que nos permita
mostrar los días en que hayamos publicado una Entrada en nuestro blog.
Para ello crearemos una etiqueta para traer las entradas por la ``fecha``
haremos uso de las vistas genéricas basadas en datos que muestran los
objetos por tipo de año mes y día.

templatetags/month_cal.py

.. code-block::  python 

    from django import template
    from datetime import date, timedelta
    from blog.models import Entrada
    
    register = template.Library()
    
    def get_last_day_of_month(year, month):
        if (month == 12):
            year += 1
            month = 1
        else:
            month += 1
        return date(year, month, 1) - timedelta(1)


    def cal_mes(year=date.today().year, month=date.today().month):
	
        event_list = Entrada.objects.filter(fecha__year=year, fecha__month=month)
        first_day_of_month = date(year, month, 1)
        last_day_of_month = get_last_day_of_month(year, month)
        first_day_of_calendar = first_day_of_month - timedelta(first_day_of_month.weekday())
        last_day_of_calendar = last_day_of_month + timedelta(7 - last_day_of_month.weekday())

        cal_mes = []
        week = []
        week_headers = []

        i = 0
        day = first_day_of_calendar
        while day <= last_day_of_calendar:
            if i < 7:
                week_headers.append(day)
            cal_day = {}
            cal_day['day'] = day
            cal_day['event'] = False
            for event in event_list:
                if day >= event.fecha.date() and day <= event.fecha.date():
                    cal_day['event'] = True
            if day.month == month:
                cal_day['in_month'] = True
            else:
                cal_day['in_month'] = False  
            week.append(cal_day)
            if day.weekday() == 6:
                cal_mes.append(week)
                week = []
            i += 1
            day += timedelta(1)

        return {'calendar': cal_mes, 'headers': week_headers}

    register.inclusion_tag('tags/cal_mes.html')(cal_mes)

  
Creamos la plantilla tags/cal_mes.html:
  

.. code-block::  html+django

    <table class="cal_month_calendar">
    <tr>
        {% for day in headers %}
        <th>{{ day|date:"D"|slice:":3" }}</hd>
    {% endfor %}
    </tr>
        {% for week in calendar %}
        <tr>
    {% for day in week %}
        <td{% if not day.in_month %} class="cal_not_in_month"{% endif %}>{% if day.event %}<a href="/blog/{{ day.day|date:"Y/m/d" }}">{{ day.day|date:"j" }}</a>{% else %}{{ day.day|date:"j" }}{% endif %}</td>
    {% endfor %}
        </tr>
    {% endfor %}
        </table>

Para usarlo solo tenemos que cargar la etiqueta así::

    {% load cal_mes %}
    {% cal_mes %}

Usando vistas genéricas basadas en datos:
-----------------------------------------
   
Para que funcione nuestro calendario y nos muestre las entradas por el
tipo de fecha, día y año, haremos uso de las vista genéricas basadas en
datos están proporcionan una manera muy sencilla de manejar datos basados en
fechas.

Agregamos otra clase a blog/views.py :

.. code-block:: python

    from django.views.generic import DayArchiveView

    class  EntradasDia( DayArchiveView):	
	    '''Entradas por día'''
	    queryset=Entrada.objects.order_by('fecha')
	    template_name='blog/entradas_dia.html'
            date_field = 'fecha'
    	    context_object_name='entradas'
	    month_format = '%m'


Que hicimos:

* Esta clase sera la encargada de mostrar entradas por día.        
* Importamos ``DayArchiveView`` de ``django.views.generic``.
* Estas vistas son parecidas alas de Lista, Detalles anteriores.
* Podemos crear una clase o definirla en la propia url.
* Estas vistas necesitan un campo  ``date_field`` que maneja la fechas.

Agregamos una entrada mas en el archivo urls.py asi::

    from blog.views import EntradasDia

    url(r'^blog/(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})$',
        EntradasDia.as_view(), name='entradas'),


* Con los parentesis capturamos el año, mes, día y los pasamos como
  argumentos ala vista  ``EntradasDia``.
* Las nuevas  vistas genéricas usan el método ``as_view()`` en la ``urls.py``.

Creamos la plantilla ``blog/entradas_dia.html``:

.. code-block::  html+django

    {% extends 'blog/base.html' %}
    {% block cabezera %}miblog|{{ day|date }}{% endblock %}

    {% block contenido %}
        <h3> Entradas del día:
    {% for entrada in entradas %}

   	{% ifchanged %}{{entrada.fecha|date:"d  M  Y"}}</h3>{% endifchanged %}

        <li> {{ forloop.counter }}<a href="{{entrada.get_absolute_url }}">
    {{ entrada }}</a></li>
        {% endfor %}
        </ul>
    
    {% endblock %}

De igual forma creamos la vista para las entradas por mes:

``blog/entradas_mes``::

    
    from django.views.generic import MonthArchiveView

    class EntradasMes(MonthArchiveView):
	    '''Entradas por mes'''
            queryset=Entrada.objects.order_by('fecha')
    	    template_name='blog/entradas_mes.html'
            date_field = 'fecha'
	    month_format = '%m'
	    context_object_name='entradas'    

La urlCONF quedara:

.. code-block:: python

    from blog.views import EntradasMes
    url(r'^blog/(?P<year>\d{4})/(?P<month>\d{2})/$',
        EntradasMes.as_view(), name='entradas'),

La plantilla entradas_mes.html:

.. code-block::  html+django

    {% extends 'blog/base.html' %}
    {% block cabezera %}miblog|{{ month|date:"F" }}{% endblock %}

    {% block contenido %}
        <h2>  Entradas del Mes de {{ month|date:"F" }} <h2>
    {% for entrada in entradas %}
  
	{% ifchanged %} <h3> {{entrada.fecha|date:"Y/m/d"}}</h3>{% endifchanged %}

        <li> {{ forloop.counter }}<a href="{{entrada.get_absolute_url }}">
    {{ entrada }}</a></li>
    {% endfor %}
    </ul>
    
    {% endblock %}

Ya solo nos queda crear la vista para mostrar las entradas por año:

``blog/entradas_year.html``:

.. code-block:: python

    from django.views.generic import YearArchiveView

    class  EntradasYear(YearArchiveView):	
	    '''Entradas por año'''
	    queryset=Entrada.objects.order_by('fecha')
	    template_name='blog/entradas_año.html'
            date_field = 'fecha'
	    context_object_name='entradas'
	    make_object_list='True'

Python tiene soporte nativo para Unicode y sus encodings más populares.
Si ejecutamos un interprete de Python en un terminal, lo común es que
herede el ``encoding`` por defecto.En este tutorial asumimos que tus
“locales” son ``es_ES.UTF-8``. Puedes comprobarlo con:

    >>> import sys
    >>> sys.stdin.encoding
    'UTF-8'
    >>>

Cualquier cadena Unicode debe estar expresada con un encoding concreto.
Python intentará utilizar siempre su encoding por defecto, que se puede
obtener con:

   >>> sys.getdefaultencoding()
   'ascii'

Lo mejor es declararlo en todos los archivos al inicio de la siguiente forma::

    # -*- coding: utf-8 -*-          
  

La urlCONF quedara:

.. code-block::  html+django

    from blog.views import EntradasYear

    url(r'^blog/(?P<year>\d{4})/$',
        EntradasYear.as_view(), name='entradas'),

La plantilla:

.. code-block::  html+django

    {% extends 'blog/base.html' %}
    {% block cabezera %}miblog|{{ year }}{% endblock %}
    

    {% block contenido %}
    
        <h3> Entradas del año:
          
    {% for entrada in entradas %}
   
	{% ifchanged %}{{entrada.fecha|date:"Y"}}</h3>{% endifchanged %}

       <li> {{ forloop.counter }}<a href="{{entrada.get_absolute_url }}">
    {{ entrada }}</a></li>
    {% endfor %}
       </ul>
    
    {% endblock %}

        

Formularios:
------------

Como todo buen sitio web es necesario crear un buscador para nuestros blog
lo haremos creando un simple formulario para que cuando alguien introduzca
alguna palabra podamos buscar en la base de datos el titulo de alguna entrada,
que concuerde con la búsqueda.

Comenzamos agregando al archivo urlCONF la vista que vamos a utilizar
así::

  (r'^buscar/$', 'blog.views.buscar')

Enseguida escribimos la vista de nuestro buscador:

blog/views.py::

    from django.shortcuts import render_to_response
    from django.db.models import Q

    def buscar(request):
        query = request.GET.get('q', '')
        if query:
            qset = (
                Q(titulo__icontains=query)|
	        Q(texto__icontains=query)
                  
            )
            results = Entrada.objects.filter(qset)
        else:
            results = []
        return render_to_response("blog/search.html", {
            "results": results,
            "query": query
        })

* Agremos un método ``GET`` para enviar datos al servidor.
* Usamos ``query = request.GET.get('q', '')`` buscamos un parámetro
  llamado ``q`` del método ``GET`` y retorna una cadena vacia si no se
  le pasan parámetros.
* Los Objetos ``Q`` se usan para realizar consultas complejas, buscamos
  los títulos que concuerden con la consulta.
* El método  ``icontains`` no distingue mayúsculas de minúsculas.
* Con el método ``Q`` buscamos coicidencias en el titulo y en el texto de
  alguna entrada que coincida con la búsqueda.

Ahora creamos la plantilla:

.. code-block:: html+django

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
    <html lang="en">
    <head>
        <title>Busqueda{% if query %} Resultados{% endif %}</title>
    </head>
    <body>
        <h1>Buscar</h1>
           <form action="." method="GET">
              <label for="q">Buscar: </label>
              <input type="text" name="q" value="{{ query|escape }}">
           <input type="submit" value="Search">
        </form>

    {% if query %}
        <h2>Resultados para "{{ query|escape }}":</h2>

    {% if results %}
        <ul>
    {% for entradas in results %}
        <li><a href="{{ entradas.get_absolute_url }}">{{ entradas|escape }}</l1>
    {% endfor %}
        </ul>
    {% else %}
       <p>No se encontraron Entradas para {{ query|escape }}</p>
    {% endif %}
    {% endif %}
    </body>
    
    </html>

De esta forma con tan solo dirigirnos ala direccion http://127.0.0.1:8000/blog/buscar/
podemos usar nuestro buscador.


Mas adelante ordenaremos la plantilla base de nuestro blog para poner
cada cosa en su lugar usando ``staticfiles``.


Una de las ventajas de usar Vistas genéricas es que podemos usarlas
varias veces con distintas urls, solo tenemos que pasar el nombre de la
plantilla a usar en la url, como por ejemplo ahora vamos a crear la
pagina de bienvenida de nuestro blog asi::

    (r'^$', MyListView.as_view(template_name='blog/bienvenidos.html')),

Agregamos otra entrada a nuestra urlCONF que sera la pagina de bienvenida
solo importamos la vista y le pasamos el nombre de la plantilla que vamos
a usar y por supuesto necesitamos crear la plantilla.

templates/blog/bienvenidos.html

.. code-block:: html+django

    {% extends 'blog/base.html' %}
    {% block titulo %}Bienvenidos a MiBlog {% endblock %}
    
    {% block contenido %} 
    {% if entradas %} 
    <ul>
    {% for e in entradas %} 
   
        <div class="post_meta">
            {{ e.titulo}}
               </div>
	       
           <div>
           {{ e.autor}}
              </div>
           {{ e.fecha}}
        <div class="post_body">
           {{e.texto|truncatewords:50|safe}}
              </div>
           <a href="{{ e.get_absolute_url }}">seguir leyendo</a>

    {% endfor %} 
        </ul>
    {% else %} 
       <p>
            No hay Entradas. 
       </p>
    {% endif %} {% endblock %}

Usando la Paginación en Vistas Genericas
----------------------------------------

Django provee un modulo para manejar y  mostrar la paginación de un objecto
el tipico pie de pagina que dice ``pagina siguiente``, estas clases
se encuentran en el modulo ``django/core/paginator.py``.
Las vista genérica que utilizamos para mostrar las entradas del blog
``ListView`` incluía una variable llamada ``paginate_by`` en la que
podemos definir cuantos objetos en este caso entradas se van a mostrar
por pagina y de este modo calcular cuantas paginas  mostrara con un enlace
a los siguientes objetos que mostrara en otra pagina.

Solo devemos de agregar el siguiente código a nuestra plantilla
``entradas.html``:

.. code-block:: html+django

    {% if is_paginated %}
	
        {% if page_obj.has_previous %}
            <a href="?page={{ page_obj.previous_page_number }}">anterior</a>
        {% endif %}

            <span class="current">
                Pagina {{ page_obj.number }} de {{ paginator.num_pages }}.
            </span>

        {% if page_obj.has_next %}
            <span class="pages"><a href="?page={{ page_obj.next_page_number }}">siguiente</a>
        {% endif %}
           </span>
          </div>

    {% endif %}  

En la siguiente parte usaremos las baterias incluidas de Django.
