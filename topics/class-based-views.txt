==================================
Vistas genéricas basadas en clase
==================================


.. versionchanged:: 1.3

.. note::

    A partir de Django 1.3, las vistas genéricas basadas en funciones han quedado en
    desuso en favor de un enfoque basado en clases, que se describe aquí.
    Para detalles sobre implementaciones previas puedes ver el documento
    :doc:`topic guide </topics/class-based-views>` y la :doc:`detailed reference </ref/class-based-views>`.

Escribir aplicaciones web pueden ser monótono, ya que repetimos ciertos patrones
una y otra vez. Django intenta quitar algo de esa monotonía en el modelo
y en la capa de plantillas, pero los desarrolladores web también experimentan ese
aburrimiento a nivel de vistas.

Las Visitas genéricas en  *Django* fueron desarrolladas  para aliviar ese dolor. Toman ciertas
expresiones y patrones comunes encontrados en el desarrollo de vista y lo abstraen de modo que
podamos  escribir vistas comunes de datos  sin tener que escribir demasiado código.

Podemos reconocer ciertas tareas comunes, como mostrar una lista de objetos y
escribir el código que muestra una lista de objetos *cualquiera*. El modelo en
cuestión puede ser pasado como un argumento adicional  a través de la URLconf.

Django viene con vistas genéricas que pueden hacer lo siguiente:

* Realizar tareas "simples" y comunes: redirigir a una página diferente y renderizar una plantilla dada.

* Mostrar paginas de listado y  detalles para un único objeto. Si estuviéramos
  creando un  aplicación para gestionar conferencias tendríamos una vista ``talk_list``
  y también una ``registered_user_list`` estas vista serían ejemplos de ``ListView`` (listado).
  Una simple página de discusión es un ejemplo de lo que llamamos una vista de ``DetailView`` (detalle).

* Presentar objetos basados en datos en paginas del tipo fecha,  año/mes/día y
  su detalle asociado, y las paginas "nuevas". Los archivo del blog web de Django
  (https://www.djangoproject.com/weblog/) son archivos del tipo  año, mes y día
  y se construyen de esta forma, como sería un típico archivo de un periódico
  
* Permitir a los usuarios crear, actualizar y borrar objetos -con o sin autorización-.

Tomados en conjunto, estas vistas proporcionan interfaces fáciles para realizar los
trabajos más comunes que puedan encontrar los desarrolladores.

Simple uso
===========

Las vistas basados ​​en clases genéricas (y cualquier vista basada ​​en clases  heredaran
los atributos de las clases base que proporcionan Django) se puede configurar de dos
maneras: creando una subclase y/o pasando directamente los argumentos en la URLconf.

Cuando sobreescribimos algunos atributos en una subclase de una vista basada en
(Tal como el ``template_name``) o algun  método (tal como ``get_context_data``)
en la subclase para proporcionar nuevos valores o métodos. Consideremos, por ejemplo,
una vista que sólo muestra una plantilla ``about.html``. Django posee una
vista genérica para hacer esto - :class:`~django.views.generic.base.TemplateView` -
para que podamos simplemente pasarla como una subclase y sobreescribir
el nombre de la plantilla::

    # alguna_app/views.py
    from django.views.generic import TemplateView

    class AboutView(TemplateView):
        template_name = "about.html"

Luego, sólo tenemos que añadir esta nueva  vista a nuestra URLconf. Como
las clases basadas en vistas son en en  sí mismo  clases, necesitamos apuntar
el método  ``as_view``  ala URL  de la  clase en su lugar, que es el punto de
entrada para las vistas basadas en  clases::

    # urls.py
    from django.conf.urls import patterns, url, include
    from some_app.views import AboutView

    urlpatterns = patterns('',
        (r'^about/', AboutView.as_view()),
    )

Alternativamente, si sólo está cambiando algunos pequeños  atributos en un
vista basada en clases, sólo necesitamos pasar los nuevos atributos en la llamada
al método ``as_view`` en si mismo::

    from django.conf.urls import patterns, url, include
    from django.views.generic import TemplateView

    urlpatterns = patterns('',
        (r'^about/', TemplateView.as_view(template_name="about.html")),
    )

Un patrón predominantemente similar se puede utilizar para usar un atributo en la ``url``
en la clase  :class:`~django.views.generic.base.RedirectView`, otra sencilla
vista genérica.


Vistas genéricas de objetos
===========================

El método :class:`~django.views.generic.base.TemplateView`  es ciertamente útil,
pero las vistas genéricas de Django realmente brillan cuando se trata de presentar
vistas sobre el contenido de tu base de datos. Porque Es una tarea común,
Django viene con un puñado de vistas genéricas incorporadas
que hacen la generación de vistas de listado y detalle de objetos increíblemente fácil.

Echemos un vistazo a una de estas vistas genéricas: la vista "object list". Vamos a usar
estos modelos::

    # models.py
    from django.db import models

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

        class Meta:
            ordering = ["-name"]

        def __unicode__(self):
            return self.name

    class Book(models.Model):
        title = models.CharField(max_length=100)
        authors = models.ManyToManyField('Author')
        publisher = models.ForeignKey(Publisher)
        publication_date = models.DateField()

Para crear una pagina de lista o un listado de todos los publishers, usaremos la URLconf bajo estas lineas::

    from django.conf.urls import patterns, url, include
    from django.views.generic import ListView
    from books.models import Publisher

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            model=Publisher,
        )),
    )

Este es todo el código Python que necesitamos escribir. Sin embargo, todavía
tenemos que escribir una plantilla. Sin embargo podríamos decirle explícitamente ala vista
que plantilla debe  utilizar mediante la inclusión de un argumento  ``template_name``
en el método ``as_view``, pero en la ausencia de una plantilla explícita
Django inferirá una del nombre del objeto.
En este caso, la plantilla  inferida sera ``"books/publisher_list.html"``
- la parte "books" proviene del nombre de la aplicación que define el modelo,
mientras que el "publisher" es solo la versión del nombre del modelo en  minúsculas.

.. note::
    Por lo tanto, cuando (por ejemplo) :class:`django.template.loaders.app_directories.Loader` el
    cargador de plantillas este habilitado en :setting:`TEMPLATE_LOADERS`, la ubicación de la plantilla sería::

                /path/to/project/books/templates/books/publisher_list.html


.. highlightlang:: html+django

Esta plantilla será renderizada en un contexto que contiene una variable llamada
``object_list`` la cual contiene todos los objetos publisher. Una plantilla muy simple
podría verse como la siguiente::

    {% extends "base.html" %}

    {% block content %}
        <h2>Publishers</h2>
        <ul>
            {% for publisher in object_list %}
                <li>{{ publisher.name }}</li>
            {% endfor %}
        </ul>
    {% endblock %}

Eso es realmente todo en lo referente al tema. Todas las geniales características
de las vistas genéricas provienen de cambiar el diccionario “info” pasado a
la vista genérica. El  documento :doc:`generic views reference</ref/class-based-views>`
describe  todas las vistas genéricas y todas  sus opciones en detalle;
el resto de este capítulo considerará algunas de las formas comunes en que tú
puedes personalizar y extender las vistas genéricas.

Extender las vistas genéricas
=============================

.. highlightlang:: python

No hay duda de que el uso de las vistas genéricas puede acelerar el desarrollo
sustancialmente. En la mayoría de los proyectos, sin embargo, llega un momento en
el que las vistas genérica no son suficientes. De hecho, la pregunta más común que
se hacen los desarrolladores de Django  es cómo hacer que las vistas genéricas
manejen un rango  más amplio de situaciones.

Esta es una de las razones del por que las vistas genéricas fueron rediseñadas
para el lanzamiento de la versión  1,3 -  anteriormente, solo eran funciones con
una desconcertante variedad de opciones;
Ahora, en lugar de pasar una gran cantidad de configuraciones en la URLconf,
el método recomendado para extender las vistas genéricas es crear una subclase
de ellos, y anular y sobreescribir sus atributos o métodos.


Haciendo contextos de plantilla "amigables"
-------------------------------------------

Tal vez hayas notado que el ejemplo de la plantilla publisher list almacena
todos los books en una variable llamada ``object_list``. Aunque que esto funciona
bien, no es una forma “amistosa” para los autores de plantillas: ellos sólo
tienen que “saber” aquí que están trabajando con books. Un nombre mejor para
esa variable sería ``publisher_list`` ; el contenido de esa variable es bastante
obvio.

Si Bien, si estamos tratando con un objeto de modelo, esto ya  está hecho
por nosotros. Cuando se trata de un objeto o queryset, Django es capaz
de rellenar  el contexto con el nombre ``verbose``, en
el caso la lista de objetos) del objeto que sera mostrado.
Esto sera  proporcionado por defecto como ``object_list``, pero
contiene exactamente los mismos datos.

Si el nombre ``verbose``  no concuerda todavía,
se puede configurar manualmente el nombre de la variable de contexto. El
atributo ``context_object_name``  de una vista genérica especifica la
variable de contexto de su uso. En este ejemplo, vamos a reemplazar el de la
URLconf, con  un simple cambio:

.. parsed-literal::

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            model=Publisher,
            **context_object_name="publisher_list",**
        )),
    )



Proporcionar un útil ``context_object_name`` es siempre una buena idea.

Podemos cambiar el nombre de esa variable fácilmente con el argumento ``template_object_name``:
Tus compañeros de trabajo que diseñan las plantillas te lo agradecerán.

Agregar un contexto extra
-------------------------
A menudo simplemente necesitamos presentar alguna información extra aparte de la
proporcionada por la vista genérica. Por ejemplo, piensa en mostrar una lista
de todos los otros publisher en cada página de detalle de un publisher. La vista
genérica :class:`~django.views.generic.detail.DetailView`  provee a el publisher
el contexto, pero parece que no hay
forma de obtener una lista de todos los publishers en esa plantilla.

Pero sí la hay: esta la proporciona la subclase :class:`~django.views.generic.detail.DetailView` ,
``extra_context``. Este es un diccionario de objetos extra que serán agregados
y provee una implementación del método ``get_context_data`` .La implementación
por viene con la clase :class:`~django.views.generic.detail.DetailView`  simplemente
agregamos el objeto que vamos a mostrar en la plantilla, pero podemos
sobrescribir mas::

    from django.views.generic import DetailView
    from books.models import Publisher, Book

    class PublisherDetailView(DetailView):

        context_object_name = "publisher"
        model = Publisher

        def get_context_data(self, **kwargs):
            # Llamamos ala implementacion para traer un primer context
            context = super(PublisherDetailView, self).get_context_data(**kwargs)
            # Agregamos un QuerySet de todos los books
            context['book_list'] = Book.objects.all()
            return context

Mostrar subconjuntos de objetos
===============================

Ahora echemos un vistazo más de cerca el argumento ``model`` que hemos estado
usando todo el tiempo. El argumento ``model``, que especifica el modelo de la base de datos
sobre la que la vista operará, está disponible en todos los
vistas genéricas que operan en un solo objeto o en una colección de
objetos. Sin embargo, el argumento ``model``  no es la única manera de
especificar los objetos que la vista que operan - también se pueden
especificar la lista de objetos utilizando el argumento ``QuerySet`` ::

    from django.views.generic import DetailView
    from books.models import Publisher, Book

    class PublisherDetailView(DetailView):

        context_object_name = "publisher"
        queryset = Publisher.objects.all()


Especificando ``model = Publisher``  en realidad es una atajo  para decir
``queryset = Publisher.objects.all()``. Sin embargo, mediante el uso de ``queryset``
para definir una lista filtrada de objetos que pueden ser más específico sobre la
objetos que serán visibles en la vista (ver :doc:`/topics/db/queries`
Para obtener más información acerca de los objetos :class:`QuerySet`, y puedes ver
:doc:`class-based views reference </ref/class-based-views>` para detalles
completos.

Para escoger un ejemplo sencillo, queremos  ordenar una lista de libros por
su fecha de publicación, con el más reciente al principio usaríamos la siguiente
forma en el archivo URLconf::

    urlpatterns = patterns('',
        (r'^publishers/$', ListView.as_view(
            queryset=Publisher.objects.all(),
            context_object_name="publisher_list",
        )),
        (r'^books/$', ListView.as_view(
            queryset=Book.objects.order_by("-publication_date"),
            context_object_name="book_list",
        )),
    )


Este es un ejemplo bastante simple, pero ilustra muy bien la idea. Por supuesto,
tú usualmente querrás hacer más que sólo reordenar objetos. Si quieres presentar
una lista de books de un publisher en particular, puedes usar la misma técnica
(en este caso, lo ilustramos mediante la creación de una  subclase en lugar pasar los argumentos
en la URLconf))::

    from django.views.generic import ListView
    from books.models import Book

    class AcmeBookListView(ListView):

        context_object_name = "book_list"
        queryset = Book.objects.filter(publisher__name="Acme Publishing")
        template_name = "books/acme_list.html"


Nota que también estamos filtrando un ``queryset`` , y estamos usando un nombre de
plantilla personalizado. Si no lo hiciéramos, la vista genérica usaría la misma
plantilla que la lista de objetos “genérica”, que puede no ser lo que queremos.

También nota que ésta no es una forma muy elegante de encontrar los publisher-specific books.
Si queremos agregar otra página publisher, necesitamos otro puñado de líneas en
la URLconf, y más de unos pocos publishers no será razonable.
Enfrentaremos este problema en la siguiente sección.

.. note ::

       Si obtienes un error 404 cuando solicitas ``/books/acme/``, para estar seguro,
       verifica que en realidad tienes un Publisher con el nombre 'ACME Publishing'.
       Las vistas genéricas tienen un parámetro ``allow_empty`` para estos casos.
       Mira en  :doc:`class-based-views reference</ref/class-based-views>`
       para mas detalles.



Filtrado dinámico
-----------------

Otra necesidad común es filtrar los objetos que se muestran en una página listado
por alguna clave en la URLconf. Anteriormente codificamos el nombre del publisher
en la URLconf, pero ¿qué pasa si queremos escribir una vista que muestre todos
los books por algún publisher arbitrario?.

Convenientemente tenemos una vista ``ListView`` que tiene un método
:meth:`~django.views.generic.detail.ListView.get_queryset` que podemos sobreescribir.
Anteriormente, solo retornamos los atributos del valor del  ``queryset``
, pero ahora podemos añadir más lógica.

La parte clave para hacer este trabajo es que cuando  vistas basadas en clases son llamadas
se  almacenas  varias cosas útiles ``self``; si bien la petición (``self.request``)
que incluye la posición (``self.args``) y los nombres basados en argumentos (``self.kwargs``)
capturados de acuerdo ala URLconf.

Aquí tenemos una URLconf con un simple  grupo capturado::

    from books.views import PublisherBookListView

    urlpatterns = patterns('',
        (r'^books/(\w+)/$', PublisherBookListView.as_view()),
    )


A continuación, escribiremos la vista ``PublisherBookListView`` ::

    from django.shortcuts import get_object_or_404
    from django.views.generic import ListView
    from books.models import Book, Publisher

    class PublisherBookListView(ListView):

        context_object_name = "book_list"
        template_name = "books/books_by_publisher.html"

        def get_queryset(self):
            publisher = get_object_or_404(Publisher, name__iexact=self.args[0])
            return Book.objects.filter(publisher=publisher)

Como se puede ver, es muy fácil agregar más lógica al queryset seleccionado;
si quisiéramos, podríamos usar ``self.request.user`` para filtrar el
usuario actual , o podemos usar alguna lógica un poco más compleja.

También podemos añadir el ``publisher`` en el mismo contexto, para poder
utilizarlo en la plantilla::

    class PublisherBookListView(ListView):

        context_object_name = "book_list"
        template_name = "books/books_by_publisher.html"

        def get_queryset(self):
            self.publisher = get_object_or_404(Publisher, name__iexact=self.args[0])
            return Book.objects.filter(publisher=self.publisher)

        def get_context_data(self, **kwargs):
            # Llamamos ala implementacion primero del  context
            context = super(PublisherBookListView, self).get_context_data(**kwargs)
            # Agregamos el publisher
            context['publisher'] = self.publisher
            return context


Realizar trabajo extra
----------------------

El último patrón común que veremos involucra realizar algún trabajo extra antes
o después de llamar a la vista genérica.

Imagina que tenemos un campo ``last_accessed`` en nuestro objeto ``Author`` que
estuvimos usando para tener un registro de la última vez que alguien vio
ese author::

    # models.py

    class Author(models.Model):
        salutation = models.CharField(max_length=10)
        first_name = models.CharField(max_length=30)
        last_name = models.CharField(max_length=40)
        email = models.EmailField()
        headshot = models.ImageField(upload_to='/tmp')
        last_accessed = models.DateTimeField()


La vista genérica object_detail, por supuesto, no sabría nada sobre
este campo, pero una vez más podríamos fácilmente escribir una vista
personalizada para mantener ese campo actualizado::

    from books.views import AuthorDetailView

    urlpatterns = patterns('',
        #...
        (r'^authors/(?P<pk>\d+)/$', AuthorDetailView.as_view()),
    )

La clase genérica  ``DetailView``, por supuesto, no sabe nada de este campo,
pero una vez más, fácilmente podríamos escribir una vista personalizada para
mantener ese campo actualizado.

En primer lugar, tendríamos que añadir los  detalles del  autor en la URLconf
para que apunte a una vista personalizada:

.. parsed-literal::

    from books.views import AuthorDetailView

    urlpatterns = patterns('',
        #...
        **(r'^authors/(?P<pk>\\d+)/$', AuthorDetailView.as_view()),**
    )

A continuación, necesitamos escribir la función  -  ``get_object``  que es el
método que recupera un objeto - por lo que sólo tenemos que sobrescribir  y
envolver la llamada::

    import datetime
    from books.models import Author
    from django.views.generic import DetailView
    from django.shortcuts import get_object_or_404

    class AuthorDetailView(DetailView):

        queryset = Author.objects.all()

        def get_object(self):
            # Llamamos ala superclase
            object = super(AuthorDetailView, self).get_object()
            # Grabamos el ultimo acceso ala base de datos
            object.last_accessed = datetime.datetime.now()
            object.save()
            # Retornamos el objeto
            return object


.. note::
        Este código en realidad no funcionará a menos que escribamos la plantilla ``books/author_detail.html``

.. note::

        La URLconf aquí usa el nombre del grupo ``pk`` este nombre es el valor por
        defecto que usa ``DetailView`` para encontrar el valor primario que
        se usa para filtrar el queryset.

Si deseamos cambiarlo, necesitamos crear una llamada a el método``get()`` en un
``self.queryset`` usando el nuevo nombre del parámetro de ``self.kwargs``.


Mas que solo HTML
-----------------

Hasta ahora, nos hemos centrado en renderizar  plantillas para generar
respuestas html. Sin embargo, eso no es todo lo que las vistas genéricas puede hacer.

Cada vista genérica esta compuesta de una  una serie de mixins, y cada uno de esos
mixin contribuye con una pequeña parte a toda la vista. Algunos de estos
mixins - Son tales como
:class:`~django.views.generic.base.TemplateResponseMixin` --son
específicamente diseñado para representar el contenido de una respuesta HTML usando una
plantilla. Sin embargo, podemos escribir nuestros  propios mixins para que
devuelvan un contenido diferente.

Por ejemplo, un simple mixin JSON podría ser algo como esto::

    from django import http
    from django.utils import simplejson as json

    class JSONResponseMixin(object):
        def render_to_response(self, context):
            "Returns a JSON response containing 'context' as payload"
            return self.get_json_response(self.convert_context_to_json(context))

        def get_json_response(self, content, **httpresponse_kwargs):
            "Construct an `HttpResponse` object."
            return http.HttpResponse(content,
                                     content_type='application/json',
                                     **httpresponse_kwargs)

        def convert_context_to_json(self, context):
            "Convert the context dictionary into a JSON object"
            # Note: This is *EXTREMELY* naive; in reality, you'll need
            # to do much more complex handling to ensure that arbitrary
            # objects -- such as Django model instances or querysets
            # -- can be serialized as JSON.
            return json.dumps(context)

Entonces podríamos escribir una vista que nos retorne una respuesta  JSON
:class:`~django.views.generic.detail.DetailView` con los tipos de mixing
:class:`JSONResponseMixin` con la
:class:`~django.views.generic.detail.BaseDetailView` -- (la
:class:`~django.views.generic.detail.DetailView` antes de que la plantilla sea
renderizada con el mixin )::

    class JSONDetailView(JSONResponseMixin, BaseDetailView):
        pass

Este tipo de vista se puede  implementar de igual forma que las otras vista
:class:`~django.views.generic.detail.DetailView` con el mismo comportamiento
--exepto el formato de respuesta--

Si somos realmente aventureros podemos usar una mezcla de la clase
:class:`~django.views.generic.detail.DetailView` para que sea capaz de
de devolver *ambos* contenidos HTML y JSON dependiendo de alguna propiedad
de la solicitud HTTP tal como un argumento o una cabecera  HTTP.Solamente
mezclando la clase :class:`JSONResponseMixin` y la clase
:class:`~django.views.generic.detail.SingleObjectTemplateResponseMixin`,
sobreescribiendo la implementacion ala función :func:`render_to_response()`
para aplazar ala subclase apropiada  dependiendo del tipo de respuesta que el
usuario pidió::

    class HybridDetailView(JSONResponseMixin, SingleObjectTemplateResponseMixin, BaseDetailView):
        def render_to_response(self, context):
            # Look for a 'format=json' GET argument
            if self.request.GET.get('format','html') == 'json':
                return JSONResponseMixin.render_to_response(self, context)
            else:
                return SingleObjectTemplateResponseMixin.render_to_response(self, context)

Devido ala forma en que python resuelve el método de sobrecarga, la implementacion
local del método  ``render_to_response()`` sobrescribe la versión proporcionada por la clase
:class:`JSONResponseMixin` y la clase :class:`~django.views.generic.detail.SingleObjectTemplateResponseMixin`.

Decorando clases basadas en vistas
===================================

.. highlightlang:: python

Para extender el uso de clases genéricas no estamos limitado solamente a el uso de
mixins.También podemos usar decoradores.

Decorando una URLconf
---------------------

La forma mas simple de decorar una clase genérica es decorando el resultado
del método :meth:`~django.views.generic.base.View.as_view`
El mejor lugar para utilizar un decorador es en la clase del método en si mismo
:meth:`~django.views.generic.base.View.dispatch`.


Un método en una clase no es lo mismo que una función independiente, por lo que
no se puede simplemente aplicar un decorador ala función del método - es necesario
transformarlo primero en un método decorado. El método  decorador ``method_decorator``
transforma una función en un método decorado que se puede utilizar en un método de
una de instancia. Por ejemplo::

    from django.contrib.auth.decorators import login_required
    from django.utils.decorators import method_decorator
    from django.views.generic import TemplateView

    class ProtectedView(TemplateView):
        template_name = 'secret.html'

        @method_decorator(login_required)
        def dispatch(self, *args, **kwargs):
            return super(ProtectedView, self).dispatch(*args, **kwargs)

En este ejemplo, cada instancia de ``ProtectedView`` tendrá protección de login,
con tan solo licarle el decorador.

.. note::

     El método  ``method_decorator`` pasa ``*args`` y ``**kwargs``
     como parámetros al método representativo de la clase. Si el método no acepta un
     conjunto compatible de parámetros sera lanzada una excepción del tipo ``TypeError``.

