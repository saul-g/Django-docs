Creando una nube de Etiquetas en Django
==========================================

Como siguiente paso en la creación de nuestro blog necesitamos crear las
``etiquetas`` para fácilmente separar entradas por el tipo de etiquetas,
para ello vamos a usar ``django-tagging`` un paquete creado especialmente
para manejar etiquetas en django y que además posee una etiqueta para crear
un ``tag cloud`` o una nube de etiquetas de forma  muy sencilla.

Esta aplicación encapsula algunas funcionalidades que nos permite manejar
etiquetas y nubes de etiquetas, es un buen ejemplo de una aplicación ``plugable``,
django toma ventaja del gran ecosistema de terceras aplicaciones, para
implementar un sinnúmero de casos comunes y no tener que reinventar
la rueda.

.. admonition::  ¿Que es una nube de etiquetas?

    Una nube de etiquetas es una representación visual de etiquetas que
    conforman un texto, las etiquetas son palabras clave que identifican
    un objeto. En un sitio web se muestran en un tamaño mayor las
    etiquetas con mayor importancia o mas vistas, aunque también se
    suelen mostrar  con un color, tamaño o fuente diferente.

Lo primero que necesitamos  es instalar ``django-tagging``::

    pip install ``django-tagging``

Vemos la version ejecutando python en la terminal:    

.. code-block:: python

   >>> import tagging
   >>> tagging.VERSION
   (0, 3, 1, 'final', 0)

Existen diversas formas de usar  ``django-tagging``:  como un widget,
como una etiqueta, un campo o registrando un modelo.
En nuestro caso vamos a usar ``django-tagging`` como un campo para
ello necesitamos agregar una campo que incluye ``django-tagging`` llamado
``TagField()`` a nuestro modelo para etiquetar nuestras entradas.
          
Agregamos el campo ``TagField``  a nuestros modelos de esta forma(se han
omitido los demás campos por claridad):

.. code-block:: python 

    from tagging.fields import TagField
    from tagging.models import Tag

    class Entrada(models.Model):

        tags = TagField()

        def etiquetas(self):
            return self.tags.split(' ')

.. admonition:: ¿Y ahora que hago?

    El comando ``manage syncdb`` no crea campos solo tablas así que para
    insertar un nuevo campo tendremos que hacerlo manualmente
    dependiendo de la base de datos que estemos usando variara el
    procedimiento,  en nuestro caso como no tenemos aun datos
    importantes en nuestra base de datos lo mejor sera borrarla y
    crear otra con::

        python manage.py syncdb

Claro que si queremos crear un campo adicional en  una tabla de ``sqlite3``
es muy sencillo de hacer:

Lo primero que necesitamos es ver la salida del comando::

    python manage.py sqlall blog

Para ver el campo que vamos a crear así:     

.. code-block:: sql
    
        CREATE TABLE "blog_entrada" (
         "id" integer NOT NULL PRIMARY KEY,
        "titulo" varchar(100) NOT NULL UNIQUE,
        "slug" varchar(100) NOT NULL UNIQUE,
        "autor" varchar(100) NOT NULL,
        "texto" text NOT NULL,
        "fecha" datetime NOT NULL,
        "estatus" varchar(1) NOT NULL,
        "tags" varchar(255) NOT NULL,
    )
    ;

Podemos ver el nuevo campo **"tags" varchar(255) NOT NULL,** que sera el
que vamos a crear, en la terminal ejecutamos sqlite3 en el mismo nivel
en que se encuentra nuestra base de datos:

.. code-block:: sql

    sqlite3 blog.db
    SQLite version 3.7.3
    Enter ".help" for instructions
    Enter SQL statements terminated with a ";"
    sqlite> ALTER TABLE "blog_entrada"
     ...> ADD tags varchar(255);
    sqlite>.exit

Ahora ya podemos actualizar las tablas.

    
Como instalar Tagging
----------------------

1. Agregamos ``'tagging'`` a nuestro archivo settings.py ala variable 
   ``INSTALLED_APPS`` 

2. Ejecutamos el comando  ``manage.py syncdb`` para crear las 2 tablas::

    python manage.py syncdb  

.. image:: img/tagging.png

Como usar ``django-tagging``
-----------------------------

Ahora como siguiente paso vamos a crear una vista que muestre todas las
etiquetas de nuestro blog como lista con un contador de entradas que
tiene cada una de ellas y un enlace para mostrar las entradas que
correspondan a cada etiqueta. 

blog/views.py:

.. code-block:: python


    from django.shortcuts import render_to_response
    from tagging.models import Tag
 
    def lista_etiquetas(request):
        """El listado de etiquetas con un contador """
        object_list = Tag.objects.usage_for_queryset(Entrada.objects.all(), counts=True)
        return render_to_response("blog/lista_etiquetas.html", {'object_list':object_list})

Veamos paso a paso que hicimos:


* Importamos el atajo ``render_to response``  de Django e importamos  el modelo
  ``Tag`` del paquete ``tagging``.       
 
* Importamos el método ``render_to_response`` de ``django.shortcuts``
  que es un atajo para renderizar una plantilla solo se necesita un diccionario
  con los parámetros que vayamos a pasar ala plantilla en  forma de
  de queryset.
        
* La opción ``counts= True`` es la encargada de mostrar un contador de
  entradas  que vienen predefinidos en ``django-tagging`` y solo muestra
  las etiquetas con alguna entrada .

Ahora mapeamos la vista a nuestro archivo de configuración URLConf así::

    url(r'^etiqueta/$','blog.views.lista_etiquetas'),

Ahora creamos la plantilla para mostrar las etiquetas y los objetos
relacionados con cada una de ellas en forma de lista:

blog/lista_etiquetas.html

.. code-block:: html+django


    {% extends "blog/base.html" %}
    {% load url from future %}
    {% block contenido %}
    <ul>
    {% for etiquetas in object_list %}
    <li>
        <a href="{% url 'etiquetas' etiquetas %}">
        {{ etiquetas }}</a> {{etiquetas.count}} Entradas
    </li>
    {% endfor %}
    </ul>
    {% endblock %}


* Cargamos la etiqueta {% url from future %} para mapear la vista a la url
  que mostrara.
    
También necesitamos crear etiquetas en cada entrada para poder navegar
entre las diferentes entradas por  etiquetas y que nos filtre las
las entradas asociadas a cada etiqueta.
Para ello necesitamos crear una vista que se encargue de mostrar las
entradas asociadas a cada etiqueta.

blog/views.py:

.. code-block:: python

    from django.shortcuts import render_to_response
    from tagging.models import Tag, TaggedItem

    def entradas_por_etiqueta(request, tag, object_id=None, page=1): 
        """Filtramos una entrada con su respectiva etiqueta""" 
        etiquetas = Tag.objects.get(name=tag)
        entradas = TaggedItem.objects.get_by_model(Entrada, etiquetas)
        entradas = entradas.order_by('-fecha') 
 
        return render_to_response("blog/entradas_por_etiqueta.html", 
            dict(tag=tag, entradas=entradas))

Algunas consideraciones:

* Importamos el método ``shortcuts`` que es un atajo para mostrar vistas
  solo necesitamos el nombre de la plantilla a usar y un contexto.
* Creamos  dos querysets uno con las etiquetas y otro con las entradas,
  para que filtre las entradas de una determinada etiqueta.

         
Creamos su respectiva entradas en el archivo url:

.. code-block:: python

    url(r'^tags/(?P<tag>[^/]+)/$','blog.views.entradas_por_etiqueta',name='etiquetas'),
                
Importamos la vista directamente en la urlCONF y le asignamos ala variable
name ``etiquetas`` que usaremos para llamar ala url en la plantilla.
Solo nos falta crear la plantilla.
Para mostrar las etiquetas de forma individual mediante el filtrado de entradas
para ello creamos la siguiente vista:


blog/entradas_por_etiqueta.html

.. code-block:: html+django

    {% extends "blog/base.html" %}

    {% block contenido %}

    <h1><p>Entradas etiquetadas como:<span class="highlight">{{tag}}</span>.</p></h1>

    {% if entradas %}
    {% for entrada in entradas %}
     
    <h2><a href="{{entrada.get_absolute_url}}/">{{entrada.titulo}}</a></h2>
    <h3>{{entrada.fecha}}</h3>
    {{entrada.texto}}
    
    </div>
    {% endfor %}
    {% else %}
    <p>No hay entradas con la etiqueta {{tag}}</p>
    {% endif %}
    {% endblock %}

    
Agregamos el siguiente  bloque a nuestra plantilla ``detalles.html``
justo después de las categorías para mostrar la(s) etiqueta(s)
de cada  entrada:

.. code-block:: python

    <div> Etiquetas:
        <p class="tags">Etiquetas: 
            {% for tag in object.etiquetas %}
            <a href="/etiqueta/{{tag}}/">{{tag}}</a> 
            {%endfor%}
        </p>
    </div>

    
Usando Vistas genéricas basadas en clases nos quedaría así la vista
de listado:

.. code-block:: python

    from django.views.generic import ListView
    from tagging.models import Tag

    class ListaEtiquetas(ListView):
        """Vista que retorna las etiquetas publicadas"""
        template_name = 'blog/lista_etiquetas.html'
        queryset=Tag.objects.all()
        paginate_by = '5'
    
        def get_queryset(self):
            """Sobrescribe el método get_queryset() 
            y retorna las etiquetas con un contador"""
            return Tag.objects.usage_for_queryset(
                Entrada.objects.all(), counts=True)

Que hicimos:

* Importamos la vista generica ``ListView`` que se encarga del trabajo
  común de listar objetos.
* Sobreescribimos el método ``get_queryset`` para refinar nuestro
  queryset y solo nos muestre las etiquetas que tienen alguna entrada y
  no nos muestre las que no tienen entradas.
* Las vistas genéricas basadas en listas tienen una variable llamada
  ``paginate_by`` que es la encargada de manejar la paginación es decir
  le decimos a django  cuantos objetos se van a mostrar por pagina.


Las vistas genéricas podemos usarlas directamente en la urlCONF nuestra
vista anterior quedaría así:

.. code-block:: python

        (r'^etiquetas/$',ListView.as_view(
            queryset= Tag.objects.usage_for_queryset(Entrada.objects.all(), counts=True),
            template_name='blog/lista_etiquetas.html',
            context_object_name='object')),  

La vista genérica que maneja el filtrado de entradas por etiquetas nos
queda de esta forma:

.. code-block:: python

    from tagging.models import Tag, TaggedItem
    from django.http import Http404
    from django.views.generic import ListView
    from tagging.utils import get_tag
                
    class Entradas_por_Etiqueta(ListView):
        """Vista que retorna una lista de etiquetas
        de acuerdo a su etiqueta"""
        model_type = 'tag'
        paginate_by ='5'
        template_name='blog/entradas_por_etiqueta.html'
    
        def get_model_name(self):
            """Aplicamos el filtro slugified alas etiquetas """
            return slugify(self.tag)
    
        def get_queryset(self):
            """Retorna un  queryset de todas las entradas
            de una determinada etiqueta"""
            self.tag = get_tag(self.kwargs['tag'])
            if self.tag is None:
                raise Http404(('No hay entradas con la etiqueta "%s".') % self.tag)
            return TaggedItem.objects.get_by_model(Entrada.objects.all(), self.tag)

        def get_context_data(self, **kwargs):
            """Agregamos al contexto las etiquetas y entradas"""
            context = super(Entradas_por_Etiqueta, self).get_context_data(**kwargs)
            context['tag'] = self.tag
            context['entradas']= TaggedItem.objects.get_by_model(Entrada,self.tag)
            return context

* Al igual que la anterior vista esta es una vista de listado.
* Refinamos el queryset sobreescribiendo  el método ``get_queryset`` para
  traer las entradas con la misma etiqueta.
* Del modulo  ``django.http`` importamos el método  Http404  que lanza
  un error 404(pagina no encontrada) si no hay objetos.
* Creamos el contexto también sobrescribiendo el método  ``get_context_data``
* ¿Porque escribir una clase cuando una función puede hacer el trabajo?

.. admonition:: Nota:

    ``django-tagging`` trae una vista para manejar las etiquetas pero
    esta basada en vistas genéricas basadas en funciones. 
    
Las vista genéricas basadas en clases posen un método de paginación muy
sencillo de usar para mostrar listas solo devemos declarar en la vista el
numero de objetos a mostrar y django automáticamente pagina el objeto,
lo único que tenemos que hacer es agregar la paginación a nuestra
plantilla base así:

blog/base.html

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
    {% endblock %}    


La urlCONF nos quedaria así usando las vista genéricas:

.. code-block:: python

    from blog.views import ListaEtiquetas,Entradas_por_Etiqueta

    url(r'^etiqueta/$',ListaEtiquetas.as_view()),
    url(r'^etiqueta/(?P<tag>[^/]+)/$',Entradas_por_Etiqueta.as_view(),name='etiquetas'),
   
Solo falta agregar una urlCONF mas para manejar la paginación así:

.. code-block:: python

    from blog.views import ListaEtiquetas,Entradas_por_Etiqueta

    url(r'^etiqueta/$',ListaEtiquetas.as_view()),
    url(r'^etiqueta/(?P<tag>[^/]+)/$',Entradas_por_Etiqueta.as_view(),name='etiquetas'),
    url(r'^tags/(?P<tag>[^/]+)/page/$',Entradas_por_Etiqueta.as_view(),name='etiquetas'),

Ahora es un buen momento de crear un archivo urlCONF  para manejar
exclusivamente las etiquetas para mantener ``el acoplamiento débil``
que tanto se hace mención en django, para ello creamos una carpeta ala que
llamaremos ``url`` que se encargara de contener las diferentes configuraciones
de las aplicaciones que tengamos instaladas en nuestro proyecto django,
creamos un archivo vació ``__init__.py`` para decirle a django que trate
esta carpeta como un paquete y así poder importarlo. Dentro  de esta
carpeta creamos  un archivo nuevo llamado ``etiquetas.url`` que contendrá
las urls de nuestro proyecto para mantener una separación entre los
diversos componentes de nuestro blog, de igual forma renombramos el archivo
url.py de nuestro blog a ``entradas.py``  de la aplicación blog y lo
movemos ala carpeta que acabamos de crear.

blog/url/etiquetas.py

.. code-block:: python

    from django.conf.urls import patterns, include, url
    from blog.views import ListaEtiquetas, Entradas_por_Etiqueta

    urlpatterns = patterns('',
 
        url(r'^etiqueta/$',ListaEtiquetas.as_view()),
        url(r'^etiqueta/(?P<tag>[^/]+)/$',Entradas_por_Etiqueta.as_view(),name='etiquetas'),
        url(r'^tags/(?P<tag>[^/]+)/page/$',Entradas_por_Etiqueta.as_view(),name='etiquetas'),

    )


Ahora solo falta importarla desde el modulo de la url del proyecto
así:

miblog/url.py::

    url(r'^', include('blog.urls.entradas')),
    url(r'^etiqueta', include('blog.urls.etiquetas')),

* Solo tenemos que importarla como cualquier otro modulo python.
* De igual forma podemos separar las vistas.

Creamos una Nube de Etiqueta
------------------------------

Una vez que tenemos las vistas y las urls configuradas podemos crear la nube de
etiquetas para mostrar en la barra lateral así:

.. code-block:: python

    {% load tagging_tags %}
    
    <div class="tagbox">
        <div class="header">Nube de Etiquetas:</div>
            <div class="group">
                {% tag_cloud_for_model blog.Entrada as etiquetas with steps=5 min_count=1 distribution=log %}
                    {% for tag in etiquetas %}
                <a class="tag{{ tag.font_size }}" href="/etiqueta/{{ tag }}/">{{ tag }}</a>
                    {% endfor %}
           </div>


* Cargamos la etiqueta con  ``{% load tagging_tags %}`` como cualquier
  otra etiqueta.
* Podemos darle algunas opciones como ``steps`` que controla el número de
  tamaños de las fuentes, ``min_count`` muestra únicamente etiquetas mayores
  que el valor dado y ``distribution`` puede ser logarítmica o lineal.
