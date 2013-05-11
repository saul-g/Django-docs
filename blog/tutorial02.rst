===============================================
Activando la interfaz administrativa en Django:
===============================================

La interfaz de administración es una de las características más
sobresalientes de Django, nos ahorra bastante trabajo y tiempo
ya que crea automáticamente una **descripción visual de los campos** que
tengamos definidos en nuestros modelos, podemos utilizarla de inmediato
para introducir contenido ala base de datos, es muy flexible  y sencilla
de personalizar como veremos continuación.

La interfaz administrativa  no se activa por default, para usarla hay que
seguir esto tres pasos:

**1.-** En el archivo ``settings.py`` descomentamos ``django.contrib.admin``
en la variable ``INSTALLED_APPS``.

**2.-** Editamos el archivo ``urls.py``, la  URLconf creada automáticamente
cuando iniciamos el proyecto, descomentamos las 3 lineas que hacen referencia
al ``admin`` así::

    from django.conf.urls import patterns, include, url

    # Descomenta las siguientes dos lineas para usar el admin
    from django.contrib import admin
    admin.autodiscover()

    urlpatterns = patterns('',
       # Examples:
       # url(r'^$', 'blog.views.home', name='home'),
       # url(r'^blog/', include('blog.foo.urls')),

       # Uncomment the admin/doc line below to enable admin documentation:
       # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

       # Descomenta la siguiente linea para usar el admin
       url(r'^admin/', include(admin.site.urls)),
    )

**3.-** Sincronizamos nuestra base de datos para crear las tablas de la
interfaz administrativa así:

.. code-block::  bash

    python manage.py syncdb

Ejecutamos el servidor de desarrollo para acceder a nuestra recién creada
interfaz administrativa:

.. code-block::  bash

    python manage.py runserver

Nos dirigimos a la dirección http://127.0.0.1:8000/admin del servidor local,
donde nos encontraremos con una pagina de login como esta, donde introducimos el
nombre de usuario y contraseña  que creamos anteriormente en la terminal.

.. image:: img/login.png

* Imagen 02 Pagina de autentificación de Django o ``Login``.

.. admonition:: El servidor de desarrollo

   Django incluye un servidor web ligero que puedes usar mientras estás
   desarrollando tu sitio, rápidamente, sin tener que lidiar con
   configuraciones  de servidores web como: Apache,
   hasta que estés  listo para la producción. Este servidor de
   desarrollo vigila tu código a la espera de cambios y se reinicia
   automáticamente, ayudándote a hacer algunos cambios en tu proyecto sin
   necesidad de reiniciar nada.

Después de autentificarnos  nos encontramos con una pagina de índice que
Django construyo automáticamente con los modelos que son incluidos por
default en django como: ``grupos``, ``usuarios`` y ``sitios``:

.. image:: img/admin01.png

* Imagen 03 Pagina de índice de la ``Interfaz Administrativa``.

Nuestro aplicación ``blog`` aun no aparece registrada en la interfaz administrativa,
para ello devemos primero registrarla y esto lo hacemos creando una
archivo llamado ``admin.py`` que colocaremos en nuestra aplicación blog en el
mismo nivel que ``models.py`` que creamos anteriormente  de esta forma:

blog/admin.py::

    from django.contrib import admin
    from blog.models import Entrada

    class AdminEntrada(admin.ModelAdmin):
        pass

    admin.site.register(Entrada, AdminEntrada)

Reiniciamos el servidor y ya podemos ver registrada la entrada ``blog``,
con una tabla llamada ``Entradas`` que Django creo automáticamente por nosotros:

.. image:: img/admin02.png

* Imagen 04 Registrando la tabla ``Entradas``


Ahora vamos a registrar la tabla para las ``Categoria`` así::

blog/admin.py::

    from django.contrib import admin
    from blog.models import Entrada, Categoria

    class AdminEntrada(admin.ModelAdmin):
        pass

    class AdminCategoria(admin.ModelAdmin):
        pass

    admin.site.register(Entrada, AdminEntrada)
    admin.site.register(Categoria,AdminCategoria)

.. image:: img/admin03.png

* Imagen 05  Registrando la tabla de ``Categorias``

Personalizar la interfaz administrativa de Django
-------------------------------------------------

Una de las mas poderosas características de Django es la interfaz administrativa,
esta lee los metadatos de nuestros modelos y crea una interface donde muestra
una serie de widgets adecuados a cada tipo de campos contenidos en el modelo
automáticamente.
La interfaz de administración de Django brilla especialmente cuando usuarios
no técnicos necesitan ser capaces de ingresar datos; ese es el propósito
detrás de esta característica. Desde este momento podemos empezar a insertar
datos ala base de datos.

.. admonition:: Nota:

    La interfaz de administración de Django está diseñada para una
    sola actividad: Usuarios confiables editando contenido estructurado.

Podemos personalizar el aspecto y la forma en que la interfaz de
administración se comporta de varias maneras, para indicar que campos
queremos ver en la pagina del admin.

Modificamos el archivo  ``admin.py``  que creamos anteriormente
de esta forma::

    from django.contrib import admin
    from blog.models import Entrada,Categoria

    class AdminEntrada(admin.ModelAdmin):

        list_display = ('titulo', 'autor', 'fecha', 'estatus')
        prepopulated_fields = {"slug": ("titulo",)}
        list_filter = ('titulo', 'fecha')
        ordering = ('-fecha',)
        search_fields = ('titulo',)

    admin.site.register(Entrada, AdminEntrada)

La opción ``list_display`` controla que columnas aparecen en la tabla de la
lista. Por defecto, la lista de cambios muestra una sola columna que
contiene la representación en cadena de caracteres del objeto.
Aquí podemos cambiar eso para mostrar el título, el autor, la fecha y el estatus
de nuestras entradas  publicadas.

Con ``prepopulated_fields`` django de forma automaticamenete  rellena el
campo ``slug`` de acuerdo al titulo, quitando espacios y caracteres no
permitidos.

La opción ``list_filter`` crea una barra de filtrado del lado derecho de
la lista. Lo que nos permite filtrar objetos por fecha y titulo  (para
que podamos ver las entradas publicados la última semana, mes, etc.).

Podemos indicarle a la interfaz de administración que filtre por cualquier
campo,los filtros aparecen cuando tenemos al menos 2 valores de dónde elegir.

La opción ``ordering`` controla el orden en el que los objetos son
presentados en la interfaz de administración. Es simplemente una lista
de campos con los cuales ordenar el resultado; anteponiendo un signo
menos a un campo se obtiene el orden reverso. En este ejemplo,
ordenamos por fecha de publicación con los más recientes al principio.

Finalmente, la opción ``search_fields`` crea un buscador que permite buscar
texto por  algún campo especifico. En nuestro caso, buscará el texto en el
campo título (entonces podrías ingresar Django para mostrar todos las
entradas que lleven en el titulo la palabra ``Django``).

Para registrar las ``Categorias`` en la interfaz administrativa solo le
pasamos ala variable ``list_display`` los nombres de los campos a visualizar
de este modo:

..  code-block:: python

    class AdminCategoria(admin.ModelAdmin):
        list_display= ('nombre','slug')

    admin.site.register(Categoria,AdminCategoria)

.. image::  img/admin04.png

* Imagen 06 Vista de campos en la interfaz administrativa

En el archivo ``model.py``  creamos un campo ``estatus`` que nos permite
guardar una  entrada que todavía no queramos publicar como un borrador.
Tenemos una OPCION con dos variables ``p`` y ``b`` vamos a crear un ``action``
para poder publicar varias entradas  desde la pagina índice de ``Entradas`` de la interfaz
administrativa cuando estas sean borradores.

Creando un action
-----------------
Ahora vamos a crear una pequeña función en la interfaz administrativa una
``accion`` django incluye una por default que nos permite borrar
objetos de forma individual o todos a la vez en la interfaz, vamos a crear
una ``accion`` que nos permita publicar varias entradas que tengamos como
borrador desde el ``admin`` estas funciones son llamadas ``actions``, un
``action`` es simplemente una función que actúa sobre una lista de objetos
en la lista de pagina de cambios de un determinado objeto.
Modificamos el archivo ``admin.py`` y creamos nuestra función así:

``blog/admin.py``::

    from django.contrib import admin
    from blog.models import Entrada, Categoria

    #Actualizar un queriset
    def publicar_entrada(modeladmin, request, queryset):
        queryset.update(estatus='p')

    class AdminEntrada(admin.ModelAdmin):

        list_display = ('titulo', 'autor', 'fecha', 'estatus')
        prepopulated_fields = {"slug": ("titulo",)}
        list_filter = ('titulo', 'fecha')
        ordering = ('-fecha',)
        search_fields = ('titulo', 'texto')
        actions = [publicar_entrada] #Agregamos el action

    class AdminCategoria(admin.ModelAdmin):
        list_display= ('nombre','slug')

    admin.site.register(Entrada, AdminEntrada)
    admin.site.register(Categoria,AdminCategoria)

Los ``actions`` necesitan de 3 cosas:

* Importar ``ModelAdmin`` los modelos del administrador.
* Una peticion  ``HttpRequest``.
* Un ``queryset`` que contenga el set de objetos seleccionados por el usuario.

Creamos la función ``publicar_entrada`` y actualizamos el campo ``estatus``
de borrador a publicado, importamos los modelos del admin, request y el
queryset lo filtramos por el campo ``estatus``.
Eso es todo lo que necesitamos para crear un ``actions``, Opcionalmente
podemos darle un toque personal al titulo que sera mostrado con:
``short_description`` así::

    def publicar_entrada(modeladmin, request, queryset):
        queryset.update(estatus = 'p')
    publicar_entrada.short_description = "Marcar Entradas como publicadas"

Y podemos agregar un mensaje en la parte superior que diga que varias
entradas han sido publicadas con ``self`` y metiendo nuestro función
ala clase ``AdminEntrada`` así:

.. code-block:: python

    from django.contrib import admin
    from blog.models import Entrada, Categoria

    class AdminEntrada(admin.ModelAdmin):

        list_display = ('titulo', 'autor', 'fecha', 'estatus')
        prepopulated_fields = {"slug": ("titulo",)}
        list_filter = ('titulo', 'fecha')
        ordering = ('-fecha',)
        search_fields = ('titulo', 'texto')
        actions = ['publicar_entradas']

        def publicar_entradas(self, request, queryset):
	    rows_updated = queryset.update(estatus='p')
            if rows_updated == 1:
                message_bit = "1 entrada fue"
            else:
                message_bit = "%s entradas fueron" % rows_updated
            self.message_user(request, "%s publicadas exitosamente." % message_bit)

    class AdminCategoria(admin.ModelAdmin):
        list_display= ('nombre','slug')

    admin.site.register(Entrada, AdminEntrada)
    admin.site.register(Categoria,AdminCategoria)

Si no especificamos un nombre para los ``actions`` django lo remplazara
con el nombre de la función usada, agregando espacios en lugar  de guiones
bajos así: ``publicar_entradas`` a ``Publicar entradas``.
y con esto ya  tenemos una nueva opción en nuestra interfaz administrativa
que nos da la opción de borrador y publicado.

De la misma forma en que marcamos varias entradas como publicadas también
podemos hacer lo contrario marcar varias entradas como borrador de la
misma forma en que lo hicimos arriba volvemos a escribir otra pequeña
función muy parecida ala anterior así:

blog/admin.py:

.. code-block:: python

    from django.contrib import admin
    from blog.models import Entrada, Categoria

    class AdminEntrada(admin.ModelAdmin):

        list_display = ('titulo', 'autor', 'fecha', 'estatus' )
        prepopulated_fields = {"slug": ("titulo",)}
        list_filter = ('titulo', 'fecha')
        ordering = ('-fecha',)
        search_fields = ('titulo', 'texto')
        actions = ['publicar_entradas', 'marcar_como_borrador']

        def publicar_entradas(self, request, queryset):
            rows_updated = queryset.update(estatus='p')
            if rows_updated == 1:
                message_bit = "1 entrada fue"
            else:
                message_bit = "%s entradas fueron" % rows_updated
            self.message_user(request, "%s publicadas exitosamente." % message_bit)
        publicar_entradas.short_description = "Marcar como publicadas"

        def marcar_como_borrador(self, request, queryset):
            rows_updated = queryset.update(estatus='b')
            if rows_updated == 1:
                message_bit = "1 entrada fue"
            else:
                message_bit = "%s entradas fueron" % rows_updated
            self.message_user(request, "%s marcadas como borrador." % message_bit)
        marcar_como_borrador.short_description="Marcar como borradores"

    admin.site.register(Entrada, AdminEntrada)

    class AdminCategoria(admin.ModelAdmin):
        list_display= ('nombre','slug')
        prepopulated_fields = {"slug": ("nombre",)}

    admin.site.register(Categoria,AdminCategoria)

.. image::  img/actions.png

* Imagen 07 Personalizar la  interfaz administrativa:

* 1.-Mensaje(self.message_user())
* 2.-Filtros(list_filter)
* 3.-Buscador(search_fields)
* 4.-Actions(actions = ['publicar_entradas', 'marcar_como_borrador'])

Ahora ya podemos ``publicar`` y marcar entradas como ``borrador`` desde
la pagina indice de la interfaz administrativa.

Creando una Vista:
------------------

Una función de vista, o vista en pocas palabras(views.py), es una simple
función de Python que toma como argumento una petición Web y retorna una
respuesta Web, la respuesta puede se una pagina en html, un redireccionamiento,
un error 404 o cualquier otro documento.
La vista en sí contiene toda la información lógica necesaria para retornar
esa respuesta.

Vamos a crear un vista que se encargara de mostrar nuestra pagina principal.
Modificamos el archivo ``views.py`` de esta forma.

blog/views.py::

    from blog.models import Entrada, Categoria
    from django.shortcuts import render_to_response

    def entradas(request):
        return render_to_response('blog/entradas.html', {
            'categorias': Categoria.objects.all(),
            'entradas': Entrada.objects.filter(estatus='p')[:5]
        })


* Importamos nuestros modelos ``Entrada`` y ``Categoria`` de nuestro blog

* Importamos el atajo ``render_to_response`` de ``django.shortcuts`` para
  renderizar la vista.

* Primero, importamos la clase ``render_to_response``, la cual toma como
  primer argumento un request, le asignamos una plantilla, como elemento opcional
  un diccionario que retornara las categorias y las entradas publicadas.

* Creamos un diccionario con un queryset ``categorias`` que retornara
  todas las categorias de la clase ``Categoria``.
  Y un queryset ``entradas`` para filtrar las entradas marcadas como
  ``p`` publicadas en la tabla de ``estatus``, que mostrara las ultimas
  cinco entradas.

* Un ``queryset`` es una colección de objetos con cero, uno, o muchos filtros

.. admonition:: Nota:

    Un ``contexto`` es un mapeo entre nombres y valores
    similar a un diccionario de Python que es pasado a una plantilla.
    Un ``diccionario`` de Python es un mapeo entre llaves conocidas y valores
    de variables.

Configurando una url.py:
------------------------
Una url es como una tabla de contenidos para nuestro sitio web hecho con
Django. Básicamente, es un mapeo entre los patrones URL y las funciones
de vista que deben ser llamadas por esos patrones URL. Es como decirle
a Django, “Para esta URL, llama a este código, y para esta otra URL,
llama a este otro código”.

.. admonition:: Nota:

   Django procesa los patrones de URL en orden de arriba para abajo y
   usa la primera URL que concuerde con la petición.

Agregamos una nueva entrada a nuestro archivo urls.py para indicarle
que estamos usando un vista de esta forma:

urls.py:

.. code-block:: python

    from django.conf.urls import patterns, include, url
    from django.contrib import admin
    admin.autodiscover()

    urlpatterns = patterns('',
       # Examples:
       # url(r'^entradas/$', 'blog.views.home', name='home'),
       # url(r'^blog/', include('blog.foo.urls')),
       # Uncomment the admin/doc line below to enable admin documentation:
       # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
       # Uncomment the next line to enable the admin:
       url(r'^entradas/$', 'blog.views.entradas'),
       url(r'^admin/', include(admin.site.urls)),
    )

* La primera línea importa todos los objetos desde el módulo
  ``django.conf.urls.defaults``, incluyendo una función llamada patterns de django.

* La segunda linea importa el área administrativa.

* La variable urlpatterns llama a la función ``patterns()`` y guarda el resultado
  en una variable llamada ``urlpatterns``.

* La función ``patterns()`` sólo recibe un argumento (la cadena de
  caracteres vacía).

* Las demás líneas están comentadas(#).

* Importamos la vista que creamos anteriormente llamada ``entradas`` de el
  archivo ``views.py`` de la aplicación ``blog`` para crear la primera url,
  accedemos ala función de vistas como accederíamos a cualquier función
  de un modulo python con el punto.

* ``r'^$'`` es una expresión regular o regex  la  ``r``  significa que '^$' es una
  cadena de caracteres en crudo, esto permite que las expresiones regulares
  sean escritas sin demasiadas sentencias de escape.
  El carácter acento circunflejo (^) significa que requiere que el patrón
  concuerde con el inicio de la cadena de caracteres.
  El carácter signo de dólar ($) significa que el patrón concuerde
  con el fin de la cadena.

* Notemos que pasamos la función de vista entrada como un objeto sin tener
  que llamar a la función, las funciones son objetos de primera clase,
  lo que significa que podemos pasarlas como cualquier otra variable.

Expresiones Regulares:
----------------------

Las Expresiones Regulares (o regexes) son la forma compacta de especificar
patrones en un texto. Aunque las URLconfs de Django permiten el uso
de regexes arbitrarias para tener un potente sistema de definición de
URLs, probablemente en la práctica no utilices más que un par de
patrones regex.
Esta es una pequeña selección de patrones comunes:

+----------------------------+--------------------------------------------------------------------------------------------------+
|       Símbolo              |                        Coincide con                                                              |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``. (punto)``        |  Cualquier carácter                                                                              |
+----------------------------+--------------------------------------------------------------------------------------------------+
|        ``\d``              |  Cualquier dígito                                                                                |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``[A-Z]``            |  Cualquier carácter, A-Z (mayúsculas)                                                            |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``[a-z]``            |  Cualquier carácter, a-z (minúsculas)                                                            |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``[A-Za-z]``         |  Cualquier carácter, a-z (no distingue entre                                                     |
|                            |	mayúscula y minúscula)                                                                          |
+----------------------------+--------------------------------------------------------------------------------------------------+
|        ``+``               | Una o más ocurrencias de la expresión anterior (ejemplo, ``\d+``                                 |
|                            | coincidirá con uno o más dígitos)                                                                |
+----------------------------+--------------------------------------------------------------------------------------------------+
|        ``[^/]+``           | Todos los caracteres excepto la barra.                                                           |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``*``                | Cero o más ocurrencias de la expresión anterior (ejemplo, ``\d*``                                |
|                            | coincidirá con cero o más dígitos)                                                               |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``{1,3}``            | Entre una y tres (inclusive) ocurrencias de la expresión anterior                                |
+----------------------------+--------------------------------------------------------------------------------------------------+
|     ``^``                  |      Requiere que el patrón concuerde con el inicio de la cadena de caracteres                   |
+----------------------------+--------------------------------------------------------------------------------------------------+
|       ``$``                |   Requiere que el patrón concuerde con el final la cadena de caracteres                          |
+----------------------------+--------------------------------------------------------------------------------------------------+

Creando una Plantilla:
----------------------

Una plantilla de Django (template) es una cadena de texto que pretende separar la
presentación de un documento de sus datos. Una plantilla define rellenos
y diversos bits de lógica básica (esto es, bloques y etiquetas de plantillas)
que regulan cómo debe ser mostrado el documento. Normalmente, las
plantillas son usadas para producir HTML, pero las plantillas de
Django son igualmente capaces de generar cualquier formato basado
en texto.

Como con la mayoría de los programas, la mejor documentación es el
propio código, en este caso nada mejor que echar un vistazo a las
plantillas originales (que se encuentran en django/contrib/admin/templates/)
para darnos una idea general de su funcionamiento y estructura.

.. admonition::  ¿Que es una plantilla?

    Una plantilla es un documento de texto, o un string normal de Python
    marcado con la sintaxis especial del lenguaje de plantillas
    de Django. Una plantilla puede contener etiquetas de bloque
    (block tags) y variables.

Ahora vamos a crear la plantillas para la vista que creamos anteriormente
creamos una carpeta ala que llamaremos ``templates``  en el mismo nivel
que ``models.py`` en la carpeta blog esta carpeta sera  la encargada
de contener las plantillas de  nuestro sitio, dentro de esta crearemos otra llamada
blog, siempre es mejor separar las plantillas en carpetas diferentes para
mantener un buen diseño y distribución, para que Django encuentre
nuestras plantillas modificamos la variable ``TEMPLATE_DIRS`` en el
archivo ``settings.py`` y le damos la ruta de nuestro carpeta
de plantillas ``templates`` como sigue:

Modificamos ``settings.py``::

    import os
    TEMPLATE_DIRS = (
        # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
        # Always use forward slashes, even on Windows.
        # Don't forget to use absolute paths, not relative paths.
        # Podemos usar una ruta  dinamica
        os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
        # O una ruta fija
        #/home/saul/plantillas/,
        )

Es más sencillo usar rutas absolutas (esto es, las rutas a los directorios
comienzan desde la raíz del sistema de archivos).
Si quieres ser un poco más flexible e independiente, también, puedes
tomar el hecho de que el archivo de configuración de Django ``settings.py``
es sólo código  Python y construir la variable ``TEMPLATE_DIRS``
dinámicamente, por ejemplo::

    import os.path

    TEMPLATE_DIRS = (
        os.path.join(os.path.dirname(__file__), 'templates').replace('\\','/'),
    )

Si estás en Windows, incluye tu letra de unidad y usa el estilo de Unix
usando  barras invertidas, como sigue::

    TEMPLATE_DIRS = ('C:/users/django/templates/',)

Dentro de la carpeta  blog creamos nuestra primera plantilla ala que hemos llamado
``base.html``  que sera la encargada de ser el esqueleto de nuestras
plantillas, contendrá bloques  que las plantillas hijas podrán sobreescribir.

templates/blog/base.html:

.. code-block:: html+django

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
        <head>
            <meta charset="utf-8">
            <title>{% block cabezera %}Bienvenidos a mi blog{% endblock %}</title>
        </head>
            <body>
                <h1>{% block titulo %} Bienvenidos a mi Blog {% endblock %}</h1>
                    {% block contenido %} {% endblock %}
            </body>
                    {% block pie_pagina %} {% endblock %}
    </html>

Herencia de Plantillas
----------------------
Usamos la herencia de plantillas en Django para indicar que una plantilla
padre puede ser sobrescrita por una plantilla hija esto lo hacemos gracias
alas etiquetas de bloques que sirven como estructuras de control, podemos
usar tantos bloques como queramos.

Hemos definido cuatro bloques, cada bloque puede contener un nombre para
que sea  fácil de identificar los bloques tiene la forma de:
``{% block titulo %}`` ``{% endblock %}`` que indica donde termina, los bloques
serán sobreescritos por los bloques que contengan las plantillas hijas
Usamos la etiqueta ``{% extends 'blog/base.html' %}`` para referirnos a
nuestra plantilla base.

Ahora creamos la pagina de índice de nuestro blog de esta forma:

templates/blog/entradas.html:

.. code-block:: html+django

    {% extends 'blog/base.html' %}
    {% block titulo %} MiBlog {% endblock %}

    {% block contenido %}

    <h2>Entradas</h2>
        {% if entradas %}
            <ul>{% for entrada in entradas %}<li>
                <a href="{{ entrada.get_absolute_url }}">{{ entrada.titulo }}</a>
            </li>{% endfor %}</ul>
                {% else %}
                    <p>No hay Entradas.</p>
                {% endif %}

    <h2>Categorias</h2>

        {% if categorias %}
            <ul>{% for categoria in categorias %}<li>
                <a href="{{categoria.get_absolute_url }}">{{ categoria.nombre }}</a>
                </li>{% endfor %}</ul>
                    {% else %}
                        <p>No hay Categorias</p>
        {% endif %}

    {% endblock %}

Esta plantilla es un HTML básico con algunas variables y etiquetas de
plantillas agregadas. Veamos paso a paso algunas definiciones:

* En la plantilla base definimos varios bloques para que una plantilla
  hija pueda sobreescribir ese bloque devemos pasarle  el nombre que
  previamente hayamos definido así: ``{% block titulo %}`` sobreescribirá
  el bloque titulo de la plantilla base.

* Cualquier texto encerrado por un par de llaves (por ej. ``{{ entrada.titulo }}``)
  es una variable. Esto significa “insertar el valor de la variable a
  la que se dio ese nombre”.

* Cualquier texto que esté rodeado por llaves y signos de porcentaje
  (por ejemplo  ``{% for entrada in entradas %}`` es una etiqueta de plantilla.
  Una etiqueta sólo le indica al sistema de plantilla “haz algo”.

* Esta  plantilla contiene un bucle ``for``:  la etiqueta
  ``{% for entradas in entradas %}``  Una etiqueta ``for`` actúa como
  un simple constructor  de bucle, dejándonos recorrer cada uno de los
  items de  una secuencia.

* La plantillas de Django usan la herencia de plantillas esto quiere
  decir que podemos crear una plantilla base con bloques que pueda ser sobreescritos
  por las plantillas hijas con la etiqueta ``{% extends 'blog/base.html' %}``
  A traves de bloques.

Dirigimos nuestro navegador a la dirección local http://localhost:8000/entradas
y ahora podemos ver una sencilla  pagina de bienvenida que muestra las entradas
y categorias de nuestro blog así:

.. image::  img/bienvenidos.png

* Imagen 08 Entradas y Categorias del blog.

En la parte siguiente crearemos un par de vistas mas y conoceremos un poco
sobre las vistas genericas.
