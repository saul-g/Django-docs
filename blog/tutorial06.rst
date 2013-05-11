Creando un blogroll
===================
Siguiendo con nuestro blog vamos a crear  un blogroll que muestre
enlaces en la parte lateral de nuestro blog.


.. admonition:: Que es un blogroll:

   Un blogroll es una colección de enlaces de blogs,
   normalmente presentado en una columna lateral o sidebar de una página
   web o un blog.

Creamos una aplicación con el comando siguiente:
 
.. code-block:: bash

    python manage.py startapp blogroll

En un proyecto Django pueden convivir varias aplicaciones, que manejan
diferentes funciones, con su propia  separación de codigo.    


Creamos los modelos
-------------------

Un modelo es simplemente una representación de  datos representados en
codigo python, este contiene los campos esenciales y maneja el
comportamiento de los datos que estamos almacenando.

.. code-block:: python

    # -*-coding: utf-8 -*-
    
    from django.db import models
    from django.utils.translation import ugettext_lazy as _
    from blogroll import managers

    class Categoria(models.Model):
        """
        La Categoria para los  links.
        """
        titulo = models.CharField(_('Titulo'), max_length=100)
        slug = models.SlugField(max_length=100, unique=True)
        es_activo = models.BooleanField(_('Esta Activo'), default=True)

        objects = models.Manager()
        actives = managers.CategoriaActiveManager()

        class Meta:
            verbose_name = _('Categoria')
            verbose_name_plural = _('Categorias')
            ordering = ('titulo', )

        def __unicode__(self):
            return self.titulo

    class Link(models.Model):
        """
        Entrada para los  link para el blogrolls y similar.
        """
        categoria = models.ForeignKey(Categoria, verbose_name=_('Categoria'))
        titulo = models.CharField(_(u'Titulo'), max_length=100)
        descripcion = models.TextField(_(u'Descripción'), blank = True, null = True)
        href = models.URLField(_(u'Dirección'), verify_exists = False)
        es_activo = models.BooleanField(_('Esta Activo'), default=True)

        objects = models.Manager()
        actives = managers.LinkActiveManager()

        class Meta:
            verbose_name = _('Link')
            verbose_name_plural = _('Links')
            ordering = ('categoria__titulo', 'titulo', )

        def __unicode__(self):
            return self.titulo
            
Los tipos de campos
-------------------
Los modelos son definido en  la clase ``django.db.models.fields``  pero por
conveniencia son importados con ``django.db.models`` por convención
usamos ``from django.db import models`` y nos referimos a los campos como
``models.<Foo>Field``

* CharField:  Un campo para pequeñas y largas cadenas.
* SlugField: Un campo para una etiqueta corta usada generalmente en URL
* BooleanField: Un campo para Cierto o Falso.
* ForeignKey: Un campo para relaciones uno a muchos, requiere un argumento
  posicional la clase del modelo ala qu esta relacionada .
* TextField: Un campo para textos largos.
* URLField: Un CharField para manejar URL.

Con la opción ``max_length`` Limitamos la entrada de texto a un número
restringido de palabras que hayamos definido.

Los siguientes son argumentos avaluables para todos los tipos de campos:

* verbose_name: Un nombre legible para el campo, si no especificamnos uno Django
  usa el nombre del campo y sus atributos, conviertye los espacios a guiones bajos. 

* blank: Si es ``True`` el campo puede quedar en blanco por default es ``False``
  verify_exists.

* default: El valor por default del campo.

* unique: Si es ``True`` el campo deve ser unico en la tabla.

Importamos el modulo ``django.utils.translation``, usado para
traducir las cadenas de texto en Django a otros idiomas,
la función   ``ugettext_lazy()`` es usada para llamar cadenas unicode,
usamos un alias ala función con ``as _`` para poder traducir las
cadenas marcadas en el modelo. 

La clase Meta:
-----------------

La clase Meta sirve como una clase interna para agregar metadatos a
nuestros modelos, los metatadatos pueden sert cualquier cosa menos un
``campo``, no es requerida agregar una clase Meta es algo opcional,
sin embargo proporciona opciones extra que hacen mas facil la salida y el
manejo de datos en nuestr modelo usamos las siguientes ``META`` opciones:

* ordering: El orden por default del objeto, ideal para obtener listas de objetos.
  ordering = ('categoria__titulo', 'titulo', )
  ordering = ('titulo', )
  
* verbose_name: El nombre del  objeto en singular.
  verbose_name = _('Categoria')
  verbose_name = _('Link')
  
* verbose_name_plural: El nombre del objeto en Plurar.
  verbose_name_plural = _('Categorias')
  verbose_name_plural = _('Links') 


Metodos de los Modelos:
-----------------------

El metodo __unicode__() es usado por Django por  para mostrar un objecto en la
interfaz administrativa, como el valor insertado en la plantilla  cuando
muestra el objeto de una forma que sea legible.
 

            
Managers
---------

* Creamos los ``managers``:

Un manager es una interface a traves de la cual recuperamos información
de la base de datos proporcionada por Django, Deve existir al menos un
``manager`` en cada aplicación, por default Django agrega un ``manager``
con el nombre ``objects`` a cada clase del modelo, si queremos usar un
nombre distinto a ``objects``  devemos crear una clase que renombre el
``manager`` con el metodo ``models.Manager()``   asi::

    objects = models.Manager()
    actives = managers.CategoriaActiveManager()

Para sobreescribir un ``manager``  devemos sobrescribir el metodo
``Manager.get_query_set()`` con en nuevo ``get_query_set()`` que retorne
las propiedades del queryset que necesitemos.

Creamos un archivo llamado ``managers.py`` en la aplicación blogroll para
renombrar los manager y perzonalizarlos  asi:

blogroll/managers.py

.. code-block:: python

    from django.db import models

    class CategoriaActiveManager(models.Manager):
        def get_query_set(self, *args, **kwargs):
            """
            Filtra un  queryset solo si esta activado.
            """
            qs = super(CategoriaActiveManager, self).get_query_set(*args, **kwargs)
            return qs.filter(es_activo=True)


    class LinkActiveManager(CategoriaActiveManager):
        def get_query_set(self, *args, **kwargs):
            """
            Filtra un  queryset si una categoria  del modelo es True.
            """
            qs = super(LinkActiveManager, self).get_query_set(*args, **kwargs)
            return qs.filter(categoria__es_activo=True)

Usando los Modelos
------------------

Una vez  que hemos definido nuestros modelos, necesitamos decirle a Django
que use estos modelos, para hacerlo editamos el archivo ``settings.py``
modificamos la variable ``INSTALLED_APPS`` agregandole la aplicación que
contiene los modelos de la aplicación que vamos a registrar en este
caso ``blogroll`` asi::

    INSTALLED_APPS = (
        #...
        'blogroll',
        #...
    )

Cuando una nueva aplicación ha sido agregada a ``INSTALLED_APPS``,
devemos ejecur ``manage.py syncdb`` para crear las nuevas tablas de la
base de datos.

.. code-block:: bash

    python manage.py syncdb

Creando Querys:

Una vez creado los modelos Django proporciona abstracción a nuestra
base de datos, mediante una API de alto nivel nos permite crear,
recuperar, actualizar y borrar objetos.
Para entrar ala API desde la terminal usamos el comando::

    python manage.py shell
    
    # Como primer paso importamos los modelos   
    >>>from blogroll.models import Categoria, Link
    >>>Categoria.objects.all()
    # Vacio  
    []
    >>>Link.objects.all()
    []
    # Creamos una Categoria
    >>>a=Categoria(titulo='Python', slug='python')
    # La guardamos con el metodo save()
    >>>a.save()
    # Ahora ya podemos acceder alas categorias
    >>>a
    [<Categoria: Python>]
    # Atravez del manager objects
    >>> Categoria.objects.all()
    [<Categoria: Python>]
    # Podemos obtener las Categorias activas con el manager actives
    >>> Categoria.actives.all()
    [<Categoria: Python>]
   

Registramos la aplicación en la interfaz administrativa
---------------------------------------------------------

Creamos un archivo ``admin.py`` en  el blogroll de esta forma:

blogroll/admin.py

.. code-block:: python

    from django.contrib import admin
    from blogroll.models import Categoria, Link

    class CategoriaAdmin(admin.ModelAdmin):
        list_display = ('titulo', 'es_activo', )
        search_fields = ('titulo', )
        prepopulated_fields = {'slug': ('titulo', )}

    class LinkAdmin(admin.ModelAdmin):
        list_display = ('titulo', 'href', 'categoria', 'es_activo', )
        search_fields = ('titulo', 'descripcion', )
        list_filter = ('categoria', 'es_activo', )

    admin.site.register(Categoria, CategoriaAdmin)
    admin.site.register(Link, LinkAdmin)

Creamos Un Etiqueta
--------------------
Las etiquetas y filtros de una aplicación Django deven encontrarse
en un modulo llamado ``templatetags`` para Django pueda encontrarlos,
creamos una carpeta llamada ``templatetags`` en el mismo nivel que ``models.py``
dentro de ella creamos un archivo vacio __init__.py para que este
directorio sea tratado como un paquete y creamos un archivo ``lista_links.py``

templatetags/lista_links.py

.. code-block:: python

    import re
    from django import template
    from blogroll.models import Link

    register = template.Library()

    class LinkListNode(template.Node):
        def __init__(self, categoria_slug, links_var_name, categoria_var_name):
            self.categoria_slug = categoria_slug
            self.links_var_name = links_var_name
            self.categoria_var_name = categoria_var_name

        def render(self, context):
            links = (Link.actives.filter(categoria__slug=self.categoria_slug)
                .select_related())

            if links:
                categoria = links[0].categoria
            else:
                categoria = None

            context[self.links_var_name] = links
            context[self.categoria_var_name] = categoria
            return ''

    @register.tag(name='lista_links')
    def do_link_list(parser, token):
        """
        Usando el slug de las  categorias como
        parametro retorna una lista de links y categorias.

        Uso:: 

            {% load lista_links %}
            {% lista_links "blogroll" as links, categoria %}

            <h3>{{ categoria.titulo }}<h3>
            <ul>
            {% for link in links %}
            <li><a href="{{ link.get_absolute_url }}">
                {{ link.titulo }}</a></li>
            {% endfor %}
            </ul>
        """
        try:
            tag_name, arg = token.contents.split(None, 1)
        except ValueError:
            raise (template.TemplateSyntaxError("%r La etiqueta requiere argumentos"\
                % token.contents.split()[0]))

        m = re.search(r'"(.*?)" as (\w+),\s?(\w+)', arg)
        if not m:
            raise (template.TemplateSyntaxError("%r Invalidos argumentos para la etiqueta"\
                % tag_name))

        categoria_slug, links_var, categoria_var = m.groups()
        return LinkListNode(categoria_slug, links_var, categoria_var)

Para definir una etiqueta de plantilla personalizable en Django necesitamos
implementar las dos etapas del proceso la compilación y el renderizado,
necesitamos definir amabas etapas.
Cuando Django compila una plantilla, divide el texto crudo de la
plantilla en nodos. Cada nodo es una instancia de ``django.template.Node``
y tiene un método render().

Cuando llamamos a el metodo  render() en una plantilla compilada,
la plantilla llama a render(), en cada Node() de su lista de nodos,
con el contexto proporcionado. Los resultados son unidos para formar la
salida de la plantilla.

Para crear una función, se debe  primero obtener el parámetro y crear
el objeto Node, el segundo paso es definir una subclase de Node que
posea un método render(), las funciones (__init__ y render) se relacionan
directamente con los dos pasos para el proceso de compilación
y renderizado de la plantilla.
La función de inicialización sólo necesitará almacenar la cadena con el
formato deseado, el trabajo real sucede dentro de la función render()
y para registrar la etiqueta llamamos al modulo ``Library`` y la
registramos con un decorador.

El nombre de la etiqueta de la  plantilla deve ser pasado como una cadena,
de lo contrario Django utilizará el nombre de la función de compilación.
también es posible utilizar ``@register.tag()`` como un decorador en
Python 2.4 o posterior:

Por lo tanto, una etiqueta  es simplemente una lista de objetos Node.
Una vez terminado esta es la estructura  que deve tener nuestra aplicación:

.. code-block:: literal

    .
    |-- admin.py
    |-- __init__.py
    |-- managers.py
    |-- models.py
    |-- templatetags
    |   |-- __init__.py
    |   |-- lista_links.py
    |-- tests.py
    `-- views.py

Para usar una etiqueta primero necesitamos llamarla::

    {% load lista_link %}

Para usarla solo devemos usar:

.. code-block:: python 

    {% lista_links "blogroll" as links, categoria %}

        <h3>{{ categoria.titulo }}<h3>
        <ul>
        {% for link in links %}
        <li><a href="{{ link.get_absolute_url }}">
            {{ link.titulo }}</a></li>
        {% endfor %}
        </ul>

        

        
      

