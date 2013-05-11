========================================
Usando las baterías Incluidas en Django:
========================================
Una de las características mas apreciadas  de  Python, es su filosofía
de ``baterías incluidas``. Cuando instalamos  Python, viene con una
amplia biblioteca de paquetes que se puedes utilizar  inmediatamente,
sin necesidad de descargar nada más. Django trata de seguir esta filosofía,
e incluye su propia biblioteca  de paquetes comunes al desarrollo web,
algunos de ellos son:

* ``comments``: Para crear comentarios.
* ``flatpages``:Paginas estáticas.
* ``admin``: El sitio de administración.
* ``auth``: El framework de autenticación.
* ``syndication``: un framework para generar feeds.
* ``sites``: Operar múltiples sitios.
* ``humanize``: Un conjunto de filtros de plantillas.
* ``markup``: Lenguajes de marcado (rst, markdown, textile)

La biblioteca estándar de Django vive en el paquete ``django.contrib``.
Dentro de cada sub-paquete hay una pieza aislada de funcionalidad para
agregar. Estas piezas no están necesariamente relacionadas entre si,
pero algunos sub-paquetes de ``django.contrib`` pueden requerir a otros.
No hay grandes requerimientos para los tipos de funcionalidad que hay
en ``django.contrib``. Algunos de los paquetes incluyen modelos
(y por lo tanto requieren que instales sus tablas en tu base de datos),
pero otros consisten solamente de middleware o de etiquetas
de plantillas (template tags).


Activar el sistema de comentarios:
-----------------------------------------
Ya tenemos una pequeña aplicación funcionando, vamos agregarle un 
sistema de comentarios.
Django incluye en recientes versiones un personalizable sistema de 
comentarios incluidos en ``django.contrib.comments`` que podemos usar
para crear de manera rapida un sistema de comentarios para nuestro blog,
solo devemos seguir unos cuantos pasos para activarlos:

Instalar el framework comments que  se encuentra en ``django.contrib.comments``  
en nuestro archivo ``settings.py`` de nuestro proyecto así::

    INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
    'django.contrib.comments',  #<-----Agregamos comentarios----------->
    # Uncomment the next line to enable the admin:
    'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    )  

Ejecutamos manage.py syncdb para crear las tablas de comentarios.::

    python manage.py syncdb 

Agregamos la entrada a nuestra urls.py::

    urlpatterns = patterns('',
    ...
    (r'^comments/', include('django.contrib.comments.urls')),
    ...
    )

Ahora modificamos nuestra plantilla base y detalles.py para agregarle un nuevo
bloque con las etiquetas siguientes:

detalles.html

.. code-block:: html+django

    {% block  comentarios %}
    {% load comments %}

    {% if entradas  %}
        <h3>Comentarios:</h3>
  
        <p>{% render_comment_list for entradas %}</p>
    
        </ul>   
    {% else %}
    
        <p>Se el primero en dejar tu comentario.</p>
   
       {% endif %}
       
       <h4>Deja un comentario</h4>
    {% render_comment_form for entradas %}
 
    {%endblock%}


Esto nos permite traer los comentarios ala vista actual.    
    
Y agregamos el bloque a nuestra plantilla base.html asi:

.. code-block:: html+django

    {% block comentarios %}
    
    {% endblock %}

.. admonition:: Bloques

    Podemos agregar tantos bloques vacíos  como necesitemos a la plantilla
    base siempre es mejor que sobren.
    
Agregando Paginas Estaticas:
---------------------------------------

La aplicación ``flatpages`` de Django, la cual reside en el paquete
``django.contrib.flatpages`` nos permite crear  paginas estáticas
usando el sistema de plantillas de django detrás de escena usa modelos
Django, lo que significa que almacena las páginas en una base de datos,
de la misma manera que el resto de los datos y puedes acceder a
las flatpages con la API de bases de datos estándar de Django.
Con flatpages podemos crear una pagina ``acerca de``, Contactos etc.
Las flatpages son identificadas por su URL y su sitio. Cuando
creas una flatpage, especificas con cual URL está asociada,
junto con  cuál(es) sitio(s).

Usando flatpages:

Para instalar la aplicación flatpages, sigue estos pasos:

* Agrega ``django.contrib.flatpages`` a tu archivo de configuración
  ``settings.py`` ala variable INSTALLED_APPS.
  ``django.contrib.flatpages`` depende de ``django.contrib.sites``, asi que
  asegúrate de que ambos paquetes se encuentren en ``INSTALLED_APPS``.
  
* Agrega ``django.contrib.flatpages.middleware.FlatpageFallbackMiddleware``
  a tu variable de configuración ``MIDDLEWARE_CLASSES``.
  
* Ejecuta el comando ``python manage.py syncdb`` para instalar las dos tablas
  necesarias en tu base de datos.
  La aplicación ``flatpages`` creara  dos tablas en tu base de datos:
  
  * ``django_flatpage`` y ``django_flatpage_sites``.

  * ``django_flatpage`` simplemente mantiene una correspondencia entre URLs
    y títulos más contenido de  texto.
    
  * ``django_flatpage_sites`` es una tabla muchos a muchos que asocia una
    flatpage con uno o más sitios.

Podemos ver lo que ``manage.py syncdb`` creo con el comando::

    python manage.py sqlall flatpages
    
.. code-block:: sql

    BEGIN;
    CREATE TABLE "django_flatpage_sites" (
        "id" integer NOT NULL PRIMARY KEY,
        "flatpage_id" integer NOT NULL,
        "site_id" integer NOT NULL REFERENCES "django_site" ("id"),
        UNIQUE ("flatpage_id", "site_id")
    )
    ;
    CREATE TABLE "django_flatpage" (
        "id" integer NOT NULL PRIMARY KEY,
        "url" varchar(100) NOT NULL,
        "title" varchar(200) NOT NULL,
        "content" text NOT NULL,
        "enable_comments" bool NOT NULL,
        "template_name" varchar(70) NOT NULL,
        "registration_required" bool NOT NULL
    )
    ;
    CREATE INDEX "django_flatpage_a4b49ab" ON "django_flatpage" ("url");
    COMMIT;

Y si nos dirigimos nuevamente a http://localhost:8000/admin/ tenemos
una nueva entrada llamada paginas estáticas:

.. image:: img/flatpages.png

Como con cada nueva aplicación instalada es necesario agregar su respectiva
entrada a nuestro archivo ``url.py``, agregamos la entrada justo debajo
de la entrada del administrador de otro forma no podremos acceder a ella. ::

    (r'', include('django.contrib.flatpages.urls')),

Ahora creamos una carpeta llamada ``flatpages`` en nuestra carpeta de
plantillas y dentro de ella creamos un archivo llamado ``default.html``
que sera el encargado de mostrar nuestras paginas estáticas.

templates/flatpages/default.html:

.. code-block:: html+django

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN"
    "http://www.w3.org/TR/REC-html40/loose.dtd">
    <html>
    <head>
        <title>{{ flatpage.title }}</title>
    </head>
    <body>
        {{ flatpage.content }}
    </body>
    </html>


Creando un Generador de  Feeds
--------------------------------
Django viene con un framework que nos permite crear feeds RSS y ATOM
de manera sencilla lo único que necesitamos hacer es crear una clase python
y podemos crear tantos feeds como queramos
Los feeds son visores basados en urlCONF, y están en el modulo
``django.contrib.syndication.views.Feed``.

Creando un Feed:
-----------------

Django incluye un framework para la generación y sindicación de feeds
de alto nivel que permite crear feeds RSS y Atom de manera sencilla.
Para crear un feed, necesitas escribir una clase Feed y hacer
referencia a la misma en la URLconf
Un feed puede ser simple (p. ej. “noticias del sitio”, o una
lista de las últimas entradas del blog como en este caso) o algo más
complejo (por ejemplo mostrar todas las entradas de una categoría en especial).

Que es RSS
----------
RSS son las siglas de ``Really Simple Syndication``, un
formato XML para sindicar o compartir contenido en la
web. Se utiliza para difundir información actualizada
frecuentemente a usuarios que se han suscrito a la
fuente de contenidos. El formato permite distribuir
contenidos sin necesidad de un navegador, utilizando
un software diseñado para leer estos contenidos RSS
(agregador). A pesar de eso, es posible utilizar el mismo
navegador para ver los contenidos RSS. Las últimas
versiones de los principales navegadores permiten leer
los RSS sin necesidad de software adicional.

Creamos un archivo llamdo ``feeds.py`` en nuestra carpeta blog en el mismo
nivel que ``models.py`` para mostrar las ultimas entradas de nuestro
blog, con el siguiente contenido:

feed.py::

    from django.contrib.syndication.views import Feed
    from blog.models import Entrada
    from django.utils.feedgenerator import Atom1Feed


    class RssEntradas(Feed):
	 
        title = "Miblog"
        link = "/blog/"
        description = "Las Ultimas entradas de mi Miblog."

        def items(self):
            return Entrada.objects.all().order_by('-fecha')[:5]
		
        def item_title(self, item):
            return item.titulo
 	
        def item_description(self, item):
            return item.texto
	
    class AtomSiteNewsFeed(RssEntradas):
        feed_type = Atom1Feed
        subtitle = RssEntradas.description


* La clase Feed es una subclase de  ``django.contrib.syndication.views.Feed``
* ``title``, ``link`` y ``description`` corresponden al estandar RSS  <title>,
  <link>, <description>  respectivamente.
* item() es un simple metodo que retorna una lista de objetos que deben ser
  incluidos en el feed como <item> elementos.
* Cada item tiene un  <title>, <link>, <description> nosotros debemos decirle
  al framework que datos poner en cada uno.
* Para especificar el contenido de <link>, hay dos opciones.
  Por cada ítem en items(), Django primero tratará de ejecutar el
  método get_absolute_url() que previamente definimos en nuestro modelo.
  Si dicho método no existe,  entonces trata de llamar al método
  item_link() en la clase Feed, pasándole un único parámetro, item,
  que es el objeto en sí mismo.
* De igual forma pasamos los argumentos de los feeds para mostrar el otro
  tipo de sindicación ``atom``  

  
Registramos la entrada en nuestro archivo url.py asi::

    from django.conf.urls import patterns, include, url
    from django.contrib import admin
    admin.autodiscover()
    from blog.feeds import RssEntradas, AtomSiteNewsFeed


    urlpatterns = patterns('',
        # ...
        (r'^blog/feed/$', RssEntradas()),
        (r'^blog/atom/$', AtomSiteNewsFeed()),
    
        # ...
    )
Ahora nos dirigimos ala dirección http://localhost:8000/blog/feed/ para
ver el sitio de sindicatión feed y a  http://localhost:8000/blog/atom/

.. image:: img/feed.png

Imagen de una entrada de sindication feed en Django. 

Creando un sitemap
---------------------------

Django nos proporciona un framework de alto nivel para crear un sitemap
en XML (un mapa del sitio) de manera sencilla.

Un Sitemap es un archivo XML de un sitio web que les dice alos indexadores
o buscadores de internet que tan a menudo cambian ciertas paginas y cuales
son mas importantes respecto  a otras, para crear un sitemap lo unico que
debemos hacer es crear una clase y apuntarla a nuestro urlCONF.

Instalación:

Para instalar sitemaps seguimos los siguientes pasos:

* Agregamos ``django.contrib.sitemaps`` a ``INSTALLED_APPS`` en nuestro archivos
  de configuración ``settings.py``.

* Asegurarnos que ``django.template.loaders.app_directories.Loader``
  este activada en ``INSTALLED_APPS`` ya que no se instala por default.

* Asegurarnos tener instalado  sites framework en ``INSTALLED_APPS``.

.. admonition:: Nota:

   La aplicación sitemap no instala ninguna tabla en nuestra base de
   datos lo único que busca es que las plantillas puedan hallar el Loader().


En el archivo urlCONF necesitamos agregar su respectiva entrada asi::

    (r'^sitemap\.xml$', 'django.contrib.sitemaps.views.sitemap', {'sitemaps': sitemaps})

Esto le dice a django que construya un sitemap cuando alguien acceda a
/sitemap.xml.
El nombre de sitemap no  es importante, pera al ubicación si, los buscadores
únicamente indexan  links en el sitio, para el actual nivel de la URL,
por esa razón /sitemap.xml deve estar en el directorio root que pueda
referenciar cualquier url del sitio.

El visor del sitemap toma un argumento  extra: {'sitemaps': sitemaps}.
sitemaps deve ser un diccionario que mapee una corta sección.

modificamos la url.py asi::

   
    from django.contrib.sitemaps import FlatPageSitemap, GenericSitemap
   

    info_dict = {
        'queryset':Entrada.objects.filter(estatus='p'),
        'date_field': 'fecha',
       }

    sitemaps = {
        'flatpages': FlatPageSitemap,
        'blog': GenericSitemap(info_dict, priority=0.6),
    }


    urlpatterns = patterns('',
       (r'^sitemap\.xml$', 'django.contrib.sitemaps.views.sitemap', {'sitemaps': sitemaps}),
    )


Agregando Django Ckeditor
--------------------------

``Ckeditor`` es un editor del tipo WYSIWYG ("Lo que ves es lo que obtienes")
muy vistoso especialmente diseñado para introducir texto en aplicaciones
web. Django-Ckeditor esta especialmente adaptado para proporcionar un
editor de textos para los campos TexField en  Django lo podemos usar de 
dos formas diferentes:

* Podemos insertarlo en nuestro modelos con ``RichTextField``.
* Crear un widget con   ``CKEditorWidget``.
 
Modo de instalación:

* Lo primero es instalar  ckeditor hay varias formas de hacerlo la mas
  sencilla es con ``pip``.::

    sudo pip install django-ckeditor ó
    easy_install django-ckeditor
    

* Una ves que lo tenemos instalado lo siguiente que necesitamos hacer es
  registrar nuestra aplicación en nuestro archivo ``settings.py`` asi:

settings.py::

    CKEDITOR_MEDIA_PREFIX = "/static/ckeditor/" #copiar ``media/ckeditor``
    CKEDITOR_UPLOAD_PATH = "/media/"# Subir archivos 

    INSTALLED_APPS = (
    ckeditor, # Agregamos a nuestras aplicaciones
     )
     
* Agregarle una entrada en nuestro archivo urlconf.

url.py::

    (r'^ckeditor/', include('ckeditor.urls')),

* Copiar el directorio ``media/ckeditor`` a nuestro directorio static.    

* Modificamos el campo texto de nuestros modelos asi::

models.py:

.. code-block:: python

    from django.db import models
    from ckeditor.fields import RichTextField


    STATUS_CHOICES = (
        ('d', 'Borrador'),
        ('p', 'Publicado'),
        ) 

    class Entrada(models.Model):

        titulo=models.CharField(max_length=100)
        autor=models.CharField(max_length=100)
        texto=RichTextField()  # Usamos  RichTextField()
        fecha=models.DateTimeField() 
        status = models.CharField(max_length=1, choices=STATUS_CHOICES)
    
       ...

Primero importamos ckeditor con ``from ckeditor.fields import RichTextField``
y luego modificamos el campo texto para agregar el campo para mostrar
ckeditor ``texto=RichTextField()``.
     
* Sincronizar nuestra aplicación con::

    manage.py syncdb

.. image:: img/ckeditor.png
Mas configuraciones para ckeditor las puedes encontrar aqui:
http://docs.cksource.com/ckeditor_api/symbols/CKEDITOR.config.html
Siqueremos personalizar CKeditor podemos usar las siguientes
opciones en el archivo settings.py::

    CKEDITOR_CONFIGS=  {
      'default': {
          'toolbar': 'Full',
          # 'toolbar': 'Basic',
	      'height': 200,
	      'width': 800,
	      #'skin': 'kama',
	      'skin': 'office2003', # Temas
	      #'skin':  'v2'
              
        },
    }

Para cambiar de tema solo descomentamos otra opcion.

En la siguiente parte le daremos color a nuestro blog agregando las
hojas de estilo(css) e imagenes con ``static``

       
    





 

 
  


   
