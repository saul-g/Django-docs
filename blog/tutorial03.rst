========================
Creando vistas en Django
========================
Ya tenemos una pagina de inicio que actualmente no hace nada, vamos a cambiar
eso, como siguiente paso necesitamos crear una vista que se encargue de
mostrar cada entrada de forma individual agregamos una nueva función a
nuestro archivo ``views.py`` de esta forma:
 
views.py::

    from django.shortcuts import render_to_response, get_object_or_404
    from blog.models import Entrada, Categoria
    
    def entradas(request):
        return render_to_response('blog/entradas.html', {
            'categorias': Categoria.objects.all(),
            'entradas': Entrada.objects.filter(estatus='p')[:5]
        })

    def detalles(request, slug):   
        return render_to_response('blog/detalles.html', {
            'entradas': get_object_or_404(Entrada, slug=slug)
        })
Las vistas no son más que funciones python que recoge los argumentos
de una petición web http y devuelve una respuesta o un error.
        
* Como primer argumento para ``render_to_response()`` le pasamos el nombre de la 
  plantilla a utilizar. El segundo argumento, si es pasado, debe ser un
  diccionario para usarse en la creación de un Context para esa
  plantilla.
  Si no se le pasa un segundo argumento, ``render_to_response()``
  utilizará un diccionario vacío.

* Creamos dos querysets una para mostrar las categorias y otro para que
  filtre las entradas publicadas por el campo ``estatus`` y la opción ``p``.   

* Del modulo django.shortcuts importamos la función ``get_object_or_404``
  para mostrar una pagina 404 (No existe pagina).
  ``get_object_or_404`` llamamos a get() desde un mánager para que lanze
  un ``Http404`` si el objeto no existe asi:

.. image::  img/404.png

* Imagen 08 Pagina 404 No existe la pagina.

Estas pagina solo es mostrada en modo ``DEBUG=True``, para mostrar nuestra
propia pagina 404 tenemos que crear una en la raiz de la carpeta ``templates``

Como siguiente paso necesitamos crearemos  otra entrada en nuestro archivo
``url.py`` que se encargara de mostrar cada entrada de forma individual.

blog/url.py::
    
    url(r'^blog/(?P<slug>[^\.]+).html', 'blog.views.detalles',name='entradas'),
    
    
* Una cosa a notar en esta URL conf es que usamos los paréntesis para capturar
  el slug de la entrada para pasarle como argumento ala vista
  agregamos al final la típica extensión .html que sera
  agregada a cada pagina para mostrarse en el navegador.
* Le pasamos ala url la función ``detalles`` directamente y el argumento
  ``entradas`` para llamar al método ``get_absolute_url`` que definimos en el
  archivo ``models.py`` anteriormente.

         
Ahora necesitamos crear otra plantilla ``blog/detalles.html`` que sera la encargada
de mostrar una pagina individual por cada entrada en la carpeta blog de
nuestros plantillas de esta forma:

templates/blog/detalles.html:

.. code-block:: html+django


    {% extends 'blog/base.html' %}
    {% block cabezera %}miblog|{{ entradas.titulo }}{% endblock %}

    {% block contenido %}
   
        <h1>{{entradas.titulo}}</h2>
        <h2>Autor:{{entradas.autor}}</h2>
        <h3>Fecha:{{entradas.fecha}}</h3>
        <p>{{ entradas.texto|safe }}</p>
        
        Categoria:   
    {% for categoria in entradas.categoria.all %}

        <a href="{{ categoria.get_absolute_url }}"> {{ categoria}}</a></ul>

    {% endfor %}
    
    {% endblock %}

Creando una tercera vista:
---------------------------

Aun nos queda crear una tercera vista para poder mostrar las entradas por
tipo de categoria(s), clasificarlas y crear un enlace  para ver de forma  individual
las entradas para nuestro blog, como hicimos anteriormente modificamos de
nuevo el archivo view.py así:

.. code-block:: python

    from django.shortcuts import render_to_response, get_object_or_404
    from blog.models import Entrada, Categoria
       
    def entradas(request):
        return render_to_response('blog/entradas.html', {
            'categorias': Categoria.objects.all(),
            'entradas': Entrada.objects.filter(estatus='p')[:5]
        })
    
    def detalles(request, slug):   
        return render_to_response('blog/detalles.html', {
            'entradas': get_object_or_404(Entrada, slug=slug)
        })

    def categorias(request, slug):
        categoria = get_object_or_404(Categoria, slug=slug)
        return render_to_response('blog/categorias.html', {
            'categorias': categoria,
            'entradas': Entrada.objects.filter(categoria=categoria)[:5]
        })

* Definimos una tercera función ala que llamamos ``categorias``
  que nos pide 2 argumentos como la anterior función un request y un slug
  que utilizaremos para traer el objeto. Creamos un par de diccionarios
  para crear dos querysets y traerlos ala vista.

* El método ``get_object_or_404`` nos muestra una pagina 404 si no haya ningún
  objeto(pagina no encontrada).   

Creamos otra entrada en nuestro archivo ``urls.py`` que manejara las categorias
así::

    url(r'^blog/categoria/(?P<slug>[^\.]+)', 'blog.views.categorias', name='categorias'),
  
Creamos la plantilla que llamamos  templates/blog/categorias.html así:

.. code-block:: html+django

    {% extends 'blog/base.html' %}
    {% block cabezera %} Ver categorias|{{ categorias.nombre }}{% endblock %}

    {% block contenido %}
    <h3>Categoria:{{ categorias.nombre }}</h3>

    {% if entradas %}
        <ul>
            {% for entrada in entradas %}
                <li><a href="{{ entrada.get_absolute_url }}">{{ entrada.titulo }}</a></li>
            {% endfor %}
                </ul>

            {% else %}
           <p>No hay Entradas en la Categoria {{ categorias.nombre }} </p>

            {% endif %}
    {% endblock %}
 

Dirigimos nuestro navegador a la dirección http://127.0.0.1:8000/entradas y ya podremos movernos
por las categorias, por las entradas y ver de forma individual cada uno  de
los elementos de la pagina.

El acoplamiento débil
-------------------------

Una parte clave de la filosofía detrás de  Django en general es  el principio
de ``acoplamiento débil`` (loose coupling): Es una manera de diseñar software
aprovechando el valor de la importancia de que se puedan cambiar  y
reutilizar las piezas. Si dos piezas de código están débilmente acopladas
los cambios realizados sobre una de dichas piezas va a tener poco
o ningún efecto sobre la otra, de esta forma podemos utilizar una misma
aplicación en varios proyectos sin cambios de ningún tipo.

Las URLconfs de Django son un claro ejemplo de este principio en
la práctica. En una aplicación Web de Django, la definición de la URL
y la función de vista que se llamará están débilmente acopladas;
de esta manera, la decisión sobre cuál debe ser la URL para una función,
y la implementación de la función misma, residen en dos lugares separados.
Esto permite el desarrollo de una pieza sin afectar a la otra.

Vamos a desacoplar nuestras  urlconfs:

En la carpeta de nuestra aplicación ``blog`` creamos un archivo nuevo
llamado ``urls.py`` con el siguiente contenido:

/blog/urls.py::

    from django.conf.urls import patterns, include, url

    urlpatterns = patterns('',
    
        #Las entradas y categorias de nuestro blog
        url(r'^entradas/$', 'blog.views.entradas'),
        url(r'^blog/(?P<slug>[^\.]+).html', 'blog.views.detalles',name='entradas'),
        url(r'^blog/categoria/(?P<slug>[^\.]+)', 'blog.views.categorias', name='categorias'),

    )

Tenemos las tres urls que hemos creado para las vistas en un archivo aparte,
ahora editamos nuestra ``urls.py`` que se creo con nuestro proyecto para
borrar estas tres entradas y agregarle  una nueva entrada que cargara 
la urlCONF que hemos creado  así.

url.py::

    from django.conf.urls import patterns, include, url
    from django.contrib import admin
    admin.autodiscover()

    urlpatterns = patterns('',
    
       #Usamos include para  cargamos la url de nuestro blog 
       (r'', include('blog.urls')), 
       url(r'^admin/', include(admin.site.urls)),
      )

* Usamos ``include`` para decirle a Django que deve  incluir y cargar una url
  que se encuentra en otro lugar (blog.urls).
  
Vistas Genericas basadas en clases:
-----------------------------------

Después de Django 1.3 las vistas genéricas dejan de ser funciones para
convertirse en clases, para mantener un mejor nivel de personalización y
reutilización de código.   

Las vistas representan un caso común del desarrollo web, obtener información
de una base de datos de acuerdo a un parámetro pasado ala URL, buscar una plantilla
y renderizar esa plantilla con los datos, Django proporciona una forma de
simplificar ese proceso con las nuevas clases basadas en vistas genéricas
``generic views``.

Django proporciona vistas genéricas para los siguientes casos:

* Realizar tareas simples como redirigir a otra pagina.
* Mostrar una lista y detalles de forma individual de un objeto.
* Presentar objetos por fecha, día, mes y año.
* Permitir a los usuarios crear y borrar objetos con o sin autorización.

Las vistas genéricas se pueden usar de dos formas diferentes:

* Pasando los argumentos ala URLconf directamente.
* Creando una clase e importándola ala URLconf.

Solo necesitamos agregar ``as_view`` en la URLconf para que apunte al
método de la clase que estamos usando.
Cuando usamos vistas genéricas como clases podemos remplazar atributos
como: ``template_name``, ``contex_object_name`` o sobreescribir
métodos como ``get_context_data`` o ``get_queryset``.


Creando Vistas Genérica:
------------------------
Las vistas genéricas basadas en ``ListDetail`` son muy similares alas basadas
en datos con la única diferencia que solo posee dos vistas:

* ``ListView`` Una pagina que representa una lista de objetos.
* ``DetailView`` Una pagina para mostrar un objeto de forma individual.

Estas vistas aceptan los siguientes argumentos:

* ``model`` El modelo sobre el que actua la vista. 
* ``context_object_name``: El nombre del contexto a renderizar.
* ``queryset`` El objeto a pasar ala vista
* ``template_name``: El nombre de la plantilla.
  Si no especificamos un nombre de plantilla para la variable template_name
  la vista utilizara  <nombre_aplicación>/<nombre_modelo>_list.html.
* ``mimetype`` El tipo de formato a usar para el documento resultante.
* ``paginate_by`` El paginador, numero de objetos a mostrar en la vista.

Borramos las vistas que habíamos creado anteriormente en el archivo ``views.py``
y creamos una nueva de este modo:

``blog/views.py``:

.. code-block:: python

    from blog.models import Entrada
    from django.views.generic import ListView

    class ListaEntradas(ListView):
	    '''Entradas del blog'''
	    queryset=Entrada.objects.filter(estatus='p')
	    context_object_name='entradas'
            template_name='blog/entradas.html'
            paginate_by=5


* Importamos la vista genérica ``ListView`` del modulo ``django.views.generic``
  como dijimos anteriormente podemos pasar los valores en la urlCONF
  directamente o como en este caso crear una clase solo con los valores
  que vamos a usar.

Ahora solo le pasamos la vista ala urlCONF directamente así:

.. code-block:: python

    from django.conf.urls import patterns, include, url
    from blog.models import Entrada
    from blog.views import ListaEntradas

    urlpatterns = patterns('',

        url(r'^blog/$', ListaEntradas.as_view()),
    )

.. admonition:: ¿ Donde quedaron las Categorias: ?

    Hemos omitido las categorias en esta vista porque mas adelante las
    traeremos por medio de una etiqueta de una forma muy sencilla.

Ahora vamos a crear la vista para ver de forma individual las entradas,
para ello vamos a usar la vista genérica ``DetailView`` del modulo
``django.views.generic`` así:

.. code-block:: python

    from blog.models import Entrada
    from django.views.generic import ListView,DetailView

    class Detalles(DetailView): 
         ''' Ver entradas de forma individual '''	
	    model=Entrada
	    context_object_name='entradas'
            template_name='blog/detalles.html'

* Como en la vista anterior solo asignamos valores alas variables
  predefinidas que acepta la vista.

Creamos la entrada apropiada en la urls.py de esta forma:

blog/urls.py

.. code-block:: python

    from django.conf.urls import patterns, include, url
    from blog.models import Entrada
    from blog.views import ListaEntradas,Detalles

    urlpatterns = patterns('',

        url(r'^blog/$', ListaEntradas.as_view()),
        url(r'^blog/entradas/(?P<slug>[^\.]+).html', Detalles.as_view(),
            name='entradas'),
    )

Un par de detalles a notar en esta urls:

* Capturamos con los parentesis el ``slug`` de cada entrada para pasarlos
  ala vista.

* Agregamos una variable mas ``name='entradas'`` para traer el método ``get_absolute_url``
  que previamente definimos en nuestro modelo, para que lo utilize la vista.

Estas dos vistas son bastante simples como dijimos anteriormente podemos
definirlas en la propia urlCONF y evitarnos crear las dos clases así:

.. code-block:: python

    from django.conf.urls import patterns, include, url
    from blog.models import Entrada
    from django.views.generic import ListView,DetailView 

    urlpatterns = patterns('',

        url(r'^blog/$', ListView.as_view(
            model=Entrada,
	    context_object_name='entradas',
            template_name='blog/entradas.html',
	    paginate_by=5,
    
        )),
        url(r'^blog/entradas/(?P<slug>[^\.]+).html', DetailView.as_view(
            model=Entrada,
	    context_object_name='entradas',
            template_name='blog/detalles.html',
        ),
            name='entradas'),
    )

El unico inconveniente de este método es que algunas veces es necesario
sobreescribir algún método como ``get_queryset`` o ``get_context_data``
para pasarle opciones extras alas vistas, en esto radica el verdadero poder
y flexibilidad que proveen las vistas genéricas que proven mixins es decir
pequeñas funciones para extender la clase y de esta manera proporcionar
una mayor reutilización de código, nada mejor para explicar que un ejemplo,
vamos a crear la vista genérica que se encargue de mostrar las entradas
por tipo de categoria. 

blog/views.py

.. code-block:: python

    from blog.models import Entrada,Categoria
    from django.views.generic import ListView,DetailView
    from django.shortcuts import  get_object_or_404

    class ListaCategorias(ListView):
        """Retorna las Entradas por tipo de Categorias """
        template_name = 'blog/categorias.html'
            
        def get_queryset(self):
            self.categoria=get_object_or_404(Categoria, slug__exact=self.kwargs['slug'])
	    return Entrada.objects.filter(categoria=self.categoria)

        def get_context_data(self, **kwargs):
            context = super(ListaCategorias, self).get_context_data(**kwargs)
	    context['entradas']= Entrada.objects.filter(categoria=self.categoria)
	    context['categorias']= Categoria.objects.get(nombre=self.categoria)
	    return context
	
Que hicimos:

* Importamos  la vista genérica ``ListView``  del modulo ``django.views.generic``        
* Importamos de nuestro modelo las clases  ``Entrada`` y ``Categoria``
* Un argumento obligatorio de la vista es definir un queryset en este caso
  necesitamos traer dos y filtrar las Entradas por Categorias.
* Sobreescribimos la función ``get_context_data`` para agregar los dos
  querysets que vamos a mostrar en la vista.
* Sobreescribimos el método ``get_queryset`` para filtrar las entradas
  por el slug y el tipo de categoria de forma  individual y pasarle la
  opción extra a ``get_context_data``.
* Asignamos una  plantilla ala variable ``template_name`` y  sobreescribimos
  el contexto con ``get_context_data`` para asignar un diccionario con los dos
  queryset que usaremos en la vista.
 
Ahora necesitamos importar la vista a nuestra URLconf de esta forma:

blog/urls.py::

    from django.conf.urls import patterns, include, url
    from blog.models import Entrada
    from blog.views import ListaEntradas,Detalles,ListaCategorias

    urlpatterns = patterns('',

        url(r'^blog/$', ListaEntradas.as_view()),
        url(r'^blog/entradas/(?P<slug>[^\.]+).html', Detalles.as_view(),
            name='entradas'),
        url(  r'^blog/categorias/(?P<slug>[^\.]+).html', ListaCategorias.as_view(),
            name='categorias'),
    )
  
Crear una etiqueta en Django:
-----------------------------
Mencionamos anteriormente que crearíamos una etiqueta para mostrar las
categorias  en nuestro blog.
Para crear una etiqueta lo primero que necesitamos es crear una
estructura de directorios para alojarlas, podemos crear una aplicación 
exclusivamente para ello o podemos ponerlas en otra aplicación que tengamos
registrada en ``INSTALLED_APPS`` solo tenemos que crear una carpeta
``templatetags`` en el mismo nivel que ``models.py`` y crear un
archivo vacio ``__init.py__`` dentro así::

    blog/
        __init__.py
        models.py
        templatetags/
            __init__.py
            mis_etiquetas.py
        views.py

Creando una etiqueta personalizada
-------------------------------------
Las etiquetas son más complejas que los filtros porque las etiquetas
pueden implementar prácticamente cualquier funcionalidad.
Funcionan en dos etapas:

* Compilación
* Renderizado

Cuando Django compila una plantilla, divide el texto crudo de la plantilla
en nodos, cada nodo tiene un método render().

Etiquetas simples
-------------------
Muchas etiquetas reciben un solo argumento una cadena o una variable para
procesar.Para facilitar la creación de esos tipos de etiquetas, Django provee una
función auxiliar: ``simple_tag``. Esta función, que es un método de
``django.template.Library``, recibe una función que acepta un argumento,
y lo encapsula en una función render  así::

    def hora_actual(format_string):
        return datetime.datetime.now().strftime(format_string)

    register.simple_tag(current_time)

templatetags/hora_actual.py::

    from django import template
    import datetime

    register = template.Library()

    def hora_actual(format_string):
        return datetime.datetime.now().strftime(format_string)

    register.simple_tag(hora_actual)

    
Y la podemos usar de la siguiente forma en la plantilla base.html
para que nos muestre la fecha actual::

    {% load hora_actual%}
    <p>Fecha: {% hora_actual "%Y-%m-%d Hora: %I:%M %p" %}.</p>
  
Lo que hace esta etiqueta es mostrar la fecha de acuerdo a alguno parámetros
que le pasemos ala plantilla(Esta etiqueta es parecida a la etiqueta ``now``
que viene con django.).

* Primero cargamos la etiqueta con ``{% load hora_actual %}``.
* Le pasamos algunos parámetros ``{% hora_actual "%Y-%m-%d Hora: %I:%M %p" %}``.

Etiquetas de inclusión
-------------------------

Este tipo de etiquetas de plantilla  visualiza ciertos datos
renderizando otra plantilla.
Primero definimos la función que toma el argumento y produce un
diccionario de datos con los resultados, nos basta un diccionario
y no necesitamos retornar nada más complejo.
Esto será usado como el contexto para el fragmento de plantilla:

La etiquetas para nuestro blog quedaran así:

templatetags/ultimas_categorias.py

.. code-block:: python 

    from django import template
    from blog.models import Categoria

    register =template.Library()
 
    @register.inclusion_tag('blog/tags/ultimas_categorias.html')
    def ultimas_categorias(n=5):
        """Retorna 'n' categorias"""
        return {'template': template,
            'ultimas_categorias': Categoria.objects.filter()[:n]}

templatetags/ultimas_entradas.py

.. code-block:: python

    from django import template
    from blog.models import Entrada

    register =template.Library()        

    @register.inclusion_tag('blog/tags/ultimas_entradas.html')
    def ultimas_entradas(n=5):
        """Retorna las ultima 5 entradas"""
        return {'template': template,
            'ultimas_entradas':Entrada.objects.filter(estatus='p')[:n]}

* Le pasamos un argumento extra a nuestras etiquetas la opción ``n`` para
  indicar cuantos objectos queremos mostrar por default muestra 5.
          
Luego creamos las plantillas  para renderizar la salida de las etiquetas,
creamos una carpeta llamada ``tags`` dentro de templates para colocar
nuestras etiquetas, las plantillas quedarian así:

templatetags/ultimas_categorias.html:

.. code-block::  html+django

        <ul>
    {% for categoria in ultimas_categorias %}
        <li> <a href="{{categoria.get_absolute_url }}">{{ categoria }}</a></li>
    {% endfor %}
        </ul>

templatetags/ultimas_entradas.html:

.. code-block::  html+django

        <ul>
    {% for entrada in ultimas_entradas %}
        <li> <a href="{{entrada.get_absolute_url }}">{{ entrada }}</a></li>
    {% endfor %}
       </ul>

Usamos las etiquetas de esta forma:

* Para usar una etiqueta como primer paso necesitamos cargarla y esto lo
hacemos con ``{% load ultimas_categorias%}`` o ``{% load ultimas_entradas %}``

* Después solo la llamamos con su nombre así ``{{ultimas_categorias 5}}`` las
  etiquetas aceptan un parámetro extra el número de objetos a mostrar que
  muestra por default 5, ``n=5``.

Modificamos nuestra plantilla ``entradas.html`` que habíamos creada previamente
ya que  no sera necesaria para mostrar las categorias como las habíamos
definido antes ya que las mostraremos por medio de una etiqueta.

blog/entradas.html

.. code-block::  html+django

    {% extends 'blog/base.html' %}
    {% block titulo %} MiBlog {% endblock %}

    {% block contenido %}

    {% load hora_actual%}
       Fecha: {% hora_actual "%Y-%m-%d Hora: %I:%M %p" %}

    {% load ultimas_categorias%}
    {% ultimas_categorias 5 %}

    {% load ultimas_entradas%}
    {% ultimas_entradas 5 %}

    {% if entradas %}
    {% for entrada in entradas %}

     	<h2>{{ entrada.titulo}}</h2>
        <span> Autor {{ entrada.autor}}</span>
        <span> Fecha de Publicacion{{ entrada.fecha}}</span>
        <p>  {{entrada.texto|truncatewords:50|safe}}</p>
       <div class="readmore"><a href="{{ entrada.get_absolute_url }}">seguir leyendo</a>
    
    {% endfor %}
        </ul>
       
    {% else %}
        <p>No hay Entradas.</p>
    {% endif %}

    {% endblock %}

En esta plantilla podemos ver varios tipos de etiquetas y filtros:

* La etiqueta ``{% for %}`` Itera sobre cada uno de los elementos de un lista. En
  este caso itera sobre las ``entradas`` para traerlas todas alas vista,
  para cerrar la etiqueta ``{% for %}`` debemos usar ``{% endfor %}``
  
* La etiqueta ``{% if %}`` evalúa una variable. Si la variable se evalúa
  como una expresión “verdadera” (que el valor exista, no esté vacía y
  no es False), se muestra el contenido del bloque, se cierra con la
  etiqueta ``{% endif %}``.

* La etiqueta ``{% if %}`` puede tener un bloque opcional ``{% else %}``
  que se mostrará en el caso de que la evaluación del bloque sea  falso.

Como se crea un filtro
----------------------
Los filtros toman uno o dos argumentos:

* El valor de la variable de entrada
* Un argumento 
 
Los filtros siempre deben retornar algo, no deben arrojar excepciones y
deben fallar silenciosamente, si existe un error pueden retornar una cadena vacía
o la entrada original.::

     <p>  {{entrada.texto|truncatewords:50|safe}}</p>

  
Los filtro se usan para  alterar la  salida de una variable:

* En este ejemplo usamos el filtro ``truncatewords`` en la variable
  ``{{entrada.texto|truncatewords:"50"|safe}}`` estamos pasando a la
  variable ``entrada.texto``  el filtro  ``truncatewords:"50"``
  el cual corta el texto y solo nos muestra las primeras 50 palabras del
  bloque, este a su ves lo pasamos por el filtro ``safe``.
  Los filtros se encadenan mediante el uso de un carácter pipe ``(|)``,
  como una referencia a las tuberías de Unix.
 
En la siguiente parte crearemos un calendario para mostrar el mes actual
y las entradas publicadas por día.
