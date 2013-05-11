==================================
Creación de un blog con Django 1.4
==================================

Django es un framework de desarrollo Web que ahorra tiempo y hace que el 
desarrollo Web sea rápido y divertido, Utilizando Django puedes crear y
mantener aplicaciones Web de alta calidad con un mínimo de esfuerzo, Django
esta escrito enteramente en  python, así que lo primero que necesitamos
es instalar python, una versión 2.6 o 2.7 trabajara bien, Django no
funciona con python 3.0 por el momento, puedes descargar python de
http://www.python.org, dependiendo del sistema operativo operativo que
estés usando.

.. admonition:: Que es un framework:

   Es una infraestructura digital, con una estructura conceptual y
   tecnológica de soporte definido, normalmente con módulos de software
   concretos, con base a la cual otro proyecto de software puede ser
   más fácilmente organizado y desarrollado. Puede incluir soporte de
   programas y bibliotecas par unir los diferentes componentes de un proyecto.

Instalar Django
---------------

Para instalar el lanzamiento oficial más reciente descarga de
http://www.djangoproject.com/download/ la ultima versión. Django usa el
método distutils estándar de instalación de Python, que en el mundo de
Linux es así:

* Baja el tarball, que se llamará algo así como Django-versión.tar.gz
  descomprimelo con ``tar`` xzvf Django.tar.gz.
* Te cambias al directorio que acabas de descomprimir con ``cd`` y ejecutas 
  el comando:  ``sudo python setup.py install``
* En Windows, podemos usar 7-Zip para manejar archivos comprimidos de todo
  tipo, incluyendo .tar.gz. Puedes bajar 7-Zip de 
  http://www.djangoproject.com/r/7zip/.

.. admonition:: Nota: 

    Antes de instalar una nueva versión primero desinstala la versión
    anterior.  
  
Cambiate a algún otro directorio e inicia python en una terminal. Si todo
está funcionando bien, deberías poder importar el módulo Django así:

    >>> import django
    >>> django.VERSION
    (TU VERSION)

Ahora que lo tenemos instalado vamos a crear un sencillo blog para
darnos una idea de su funcionamiento en general:

Django-admin.py
---------------

``django-admin.py``  es la linea de comandos de Django para realizar
diversas tareas administrativas tales como iniciar un proyecto, iniciar
el servidor de desarrollo, crear las bases de datos, etc. Podemos ver la
ayuda con ``django-admin.py  --help``.

Ejecuta el comando ``django-admin.py startproject miblog`` para crear 
el proyecto  ``miblog`` en el directorio actual.

.. code-block::  bash

   django-admin.py startproject miblog 
 
Esto fue lo que ``django-admin.py startproject miblog`` creo por
nosotros; 2 directorios y 5 archivos::
    .
    `-- miblog
        |-- manage.py
        `-- miblog
            |-- __init__.py
            |-- settings.py
            |-- urls.py
            `-- wsgi.py
         
* :file:`miblog/`: El Directorio externo que contiene nuestro projecto.
* :file:`miblog/miblog`: El directorio interno que sera el nombre que
  usaremos para importar el paquete.   
* :file:`manage.py`: Una utilidad de línea de comandos para interactuar
  con nuestro proyecto(crear las tablas, iniciar el servidor...).
* :file:`miblog/__init__.py`: Un archivo vació requerido para que Python
  trate a  este directorio como un paquete.
* :file:`miblog/settings.py`: Configuraciones para este proyecto de Django.
* :file:`miblog/urls.py`: La "tabla de contenidos" de nuestro proyecto.
* :file:`miblog/wsgi.py`: El archivo  compatible con el servidor web.       

``django-admin`` crea una estructura de directorios para el nombre de el
proyecto dado, en el directorio actual, pone el paquete de tu proyecto en sys.path
establece la variable de entorno ``DJANGO_SETTINGS_MODULE`` para que apunte
al archivo ``settings.py`` de tu proyecto.

Nos cambiamos al directorio ``miblog`` que hemos creado y arrancamos el
servidor de desarrollo con el siguiente comando:

.. code-block::  bash

   ./manage.py runserver
   
Tambien podemos usar:

.. code-block::  bash

   python manage.py runserver

Dirigimos nuestro navegador ala pagina  http://127.0.0.1:8000
y veremos una pagina de bienvenida en color azul pastel como esta:

.. image:: img/bienvenido.png

* Imagen 01 Pagina de bienvenida de Django.
Para detener la ejecución del servidor de desarrollo usamos las teclas
``CONTROL-C``
  
Configurar un proyecto
----------------------

El archivo ``setting.py`` contiene las variables de configuración de un
proyecto  django, en cualquier caso  necesitamos agregar algunos datos
iniciales como el tipo de ``base de datos`` a usar, el ``idioma``, la ``zona``, etc.
Para este proyecto en  particular vamos a usar como base de datos ``sqlite3``
ya que se instala por defecto  con versiones superiores a python2.5. 

Editamos el archivo ``settings.py``::

   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.sqlite3', # Tipo base de datos
           'NAME': 'blog.db',   #  Nombre de la  base de datos
           'USER': '',      # No requerido con sqlite3 
           'PASSWORD': '',  # No requerido con sqlite3 
           'HOST': '',      # No requerido con sqlite3 
           'PORT': '',      # No requerido con sqlite3 
       }
   }

   TIME_ZONE = 'America/Mexico_City' # Zona horario
   LANGUAGE_CODE = 'es' # Idioma

Una vez configuradas las primeras variables de nuestro  proyecto, vamos
a crear una aplicación para usar la base de datos. Vale la pena explicar
la terminología aquí, porque esto es algo un poco confuso:

.. admonition:: ¿cuál es la diferencia entre un proyecto y una aplicación?

   Un ``proyecto`` es una instancia de un cierto conjunto de aplicaciones
   de Django, más las configuraciones de esas aplicaciones.
   Técnicamente, el único requerimiento de un proyecto es que este
   suministre un archivo de configuración(`settings.py`), el cual define
   la información hacia la conexión a la base de datos, la lista de
   las aplicaciones instaladas, la variable ``TEMPLATE_DIRS``, y así sucesivamente.
   Una ``aplicación`` es un conjunto portable de una funcionalidad de Django,
   típicamente incluye modelos y vistas, que conviven en un solo paquete 
   de Python.

En la linea de comandos ejecutamos:

.. code-block::  bash

   django-admin.py startapp blog
     
Este comando creara una aplicación dentro de nuestro proyecto que sera la
encargada de manejar las entradas de nuestro blog, esto es lo que
``django-admin.py startapp``  creo::

   blog
       |-- __init__.py
       |-- models.py
       |-- tests.py
       `-- views.py
      
* :file:`__init__` :Un archivo vació requerido para que Python trate este
  directorio como un paquete.
* :file:`models.py`:Un archivo para crear una descripción de los tipos
  de campos y tablas para usar en la base de datos.
* :file:`view.py`:Una simple función de Python que toma como argumento 
  una petición Web y retorna una respuesta Web (una vista). 
* :file:`test.py`: Un archivo para testear  nuestra aplicación.

Creando un Modelo
-----------------
Django es apropiado para crear sitios web interactivos que manejen 
información y consultas sobre una base de datos, ya que incluye una 
manera fácil pero poderosa de realizar consultas a bases de datos 
utilizando Python.
La parte más importante de un modelo  y la única parte requerida es la
lista de campos de la base de datos que defina, solo devemos tener
cuidado de dos cosas:

* Un nombre de campo no puede ser una palabra reservada de python.
* No puede contener dos o mas guiones bajos consecutivos debido ala forma
  en que trabaja  la sintaxis de búsqueda.
  
.. admonition:: Nota:

    Un modelo de Django es una descripción abstracta de datos para crear 
    la base de datos, representada como código de Python. 

Tipos de campos:
----------------
Para crear los modelos necesitamos crear una clase que defina cada una de los
tipos de campos que vamos a usar, esta es una pequeña descripción de los
que vamos a usar:

``CharField``: Un campo para cadenas cortas o largas, requiere un argumento extra:
``max_lenght``, que es la longitud máxima de caracteres del campo, ``choices``:una lista
o tupla de objetos para usar como opciones de este objeto.(``titulo, autor, nombre, estatus``).

``DateTimeField``: Un campo de fecha y hora, acepta argumentos extras como ``auto_now``
al momento de guardar un objeto asigna automáticamente un valor, ``auto_now_add`` que
asigna automáticamente al campo un valor al momento en que se crea(``fecha``).

``SlugField``:Un campo para una etiqueta corta, que contiene números, letras y guiones bajos,
que se utiliza comunmente en las URL, acepta como opciones: ``max_lenght`` (``slug``).

``TextField``: Un campo para la entrada de texto de forma ilimitada(``texto``).

``ManyToManyField``:Un campo para la relación muchos a muchos, requiere un argumento
posicional, el modelo al cual es relacionado(``Categoria``).     
   
Una vez definidos nuestros modelos, editamos el archivo ``models.py`` de nuestra
aplicación blog que creamos anteriormente así:

models.py::

    from django.db import models
    
    ESTADO = (
        ('b', 'Borrador'),
        ('p', 'Publicado'),
        )
    
    class Categoria(models.Model):
        nombre = models.CharField(max_length =100, db_index = True)
        slug = models.SlugField(max_length =100, db_index = True)
  
	
    class Entrada(models.Model):
        
        titulo = models.CharField(max_length=100, unique = True)
        slug = models.SlugField(max_length=100, unique = True)
        autor = models.CharField(max_length = 100)
        texto = models.TextField()
        fecha = models.DateTimeField(db_index=True, auto_now_add = True)
        estatus = models.CharField(max_length=1, choices = ESTADO)
        categoria = models.ManyToManyField(Categoria)
        
Que hicimos:

* La primera cosa a notar es que cada modelo es representado por una clase Python 
  que es una subclase de ``django.db.models.Model``
* Declaramos la variable ``ESTADO`` que usaremos en el campo estatus para separar
  las entradas publicadas de los borradores.
* Cada modelo corresponde a una tabla única de la base de datos, y cada atributo 
  de un modelo generalmente corresponde a una columna de esa tabla.
  Creamos dos clases: ``Categoria`` y ``Entrada``, cada una con sus respectivos
  campos en el caso de ``Categoria`` tiene dos campos ``nombre`` y ``slug``, el 
  nombre acepta texto como máximo 100 y un slug que vamos a usar para 
  mostrar la dirección absoluta de de cada categoria.
* El campo ``Entrada`` contiene siete campos, autor, titulo, texto, fecha, un
  campo estatus que mostrara si una entrada esta publicada o no, un slug para 
  crear la dirección de cada entrada, este no debe ser repetido y un campo 
  ``muchos a muchos``  que esta relacionado a el campo ``Categoria``

* Podemos usar opciones para los campos tales como: ``max_lenght`` longitud
  maxima de caracteres, ``db_index`` Crea un índice para esta columna.    
  
.. admonition:: Que es un Slug:

    Una etiqueta corta para algo, que sólo contiene letras, números o guiones
    Estás generalmente se utiliza en las direcciones  URLs. Por ejemplo, 
    en una URL de un  blog esta seria una entrada típica:
    https://www.djangoproject.com/weblog/2008/apr/12/spring/
    la ultima palabra ``spring`` seria el slug.

    Un SlugField implica que debe tener un índice ``db_index=True``
    debido a que este campo se usa principalmente en  búsquedas sobre la base de datos.
    
El siguiente paso es agregar nuestra aplicación ``blog`` al archivo de configuración
``settings.py`` ala variable ``INSTALLED_APPS`` para que Django cree las
tablas de nuestros modelos asi::

    INSTALLED_APPS = (
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'blog', #<-----No se te olvide la coma al ultimo-------------*
        # Descomenta la siguiente linea para habilitar el admin:
        # 'django.contrib.admin',
        # Descomenta la siguiente linea para habilitar la  documentación:
        # 'django.contrib.admindocs',
    )
    
Verificamos si no tiene errores nuestro archivo ``models.py`` con el
siguiente comando:

.. code-block::  bash

    python manage.py validate
       
Luego ejecutamos el  siguiente comando:

.. code-block::  bash

    python manage.py sqlall blog

Y veremos una salida como la siguiente si todo salio bien, este comando
solo muestra por pantalla la salida del comando pero no efectúa cambios en la
base de datos:

.. code-block:: sql

    BEGIN;
    CREATE TABLE "blog_categoria" (
        "id" integer NOT NULL PRIMARY KEY,
        "nombre" varchar(100) NOT NULL,
        "slug" varchar(100) NOT NULL
    )
    ;
    CREATE TABLE "blog_entrada_categoria" (
        "id" integer NOT NULL PRIMARY KEY,
        "entrada_id" integer NOT NULL,
        "categoria_id" integer NOT NULL REFERENCES "blog_categoria" ("id"),
        UNIQUE ("entrada_id", "categoria_id")
    )
    ;
    CREATE TABLE "blog_entrada" (
        "id" integer NOT NULL PRIMARY KEY,
        "titulo" varchar(100) NOT NULL UNIQUE,
        "slug" varchar(100) NOT NULL UNIQUE,
        "autor" varchar(100) NOT NULL,
        "texto" text NOT NULL,
        "fecha" datetime NOT NULL,
        "estatus" varchar(1) NOT NULL
    )
    ;
    CREATE INDEX "blog_categoria_73a2967e" ON "blog_categoria" ("nombre");
    CREATE INDEX "blog_categoria_56ae2a2a" ON "blog_categoria" ("slug");
    CREATE INDEX "blog_entrada_27b32b8f" ON "blog_entrada" ("fecha");
    COMMIT;
    
    
.. Admonition:: Nota:

    Django deriva automáticamente el nombre de la tabla de la
    base de datos a partir del nombre de la clase modelo y la aplicación
    que la contiene(blog_entrada, blog_entrada_categoria, blog_categoria).


Creamos las tablas con el comando:

.. code-block::  bash
 
    python manage.py syncdb

Nos preguntara si queremos instalar un superusuario le decimos que si,
y nos pide una contraseña y un correo  (podemos mas tarde instalarlo ).

Introducción ala API De la Base de Datos:
-----------------------------------------

Una vez que hemos sincronizado nuestro modelo, Django provee automáticamente una API Python 
de alto nivel para trabajar con estos modelos en una terminal, podemos usar
la API en cualquier momento para ello usamos el comando:

.. code-block:: bash

    python manage.py shell 
    
E importamos los modelos así:

.. code-block:: python 
   
   #Importamos los modelos
    >>>from blog.models import Categoria, Entrada
    # No tenemos Categorias ni Entradas
    >>>Categoria.objects.all()
    []
    >>>Entrada.objects.all()
    []
    #Creamos una Categoria
    >>>a1=Categoria(nombre="python",slug="Python")
    #Necesitamos añadir una función ``__unicode__``
    >>>a1
    <Categoria: Categoria object>
    #La guardamos con:
    >>>a1.save
    #Salimos con:
    >>>exit()

Django agrega un manager a cada modelo automaticamente para poder
acceder a cada queriset.

* Necesitamos añadir una función ``__unicode__`` ala clase ``Categoria``
  para poder ver una salida mas legible del modelo así. ::

    def __unicode__(self):
        return '%s' % self.nombre
        
* De igual forma agregamos una funcion  ``__unicode__``  ala clase
  ``Entrada``  tambien ::

    def __unicode__(self):
        return '%s' % self.titulo
          
Agregar metodos ``__unicode__`` a nuestros modelos es un buen habito, de esta
manera conseguimos una salida legible.  Podemos ordenar la salida de la clase
``Categoria`` con respecto al nombre y la clase ``Entrada`` con respecto al
``titulo`` que serán mostradas en la interfaz administrativa en el mismo
método unicode().
    
Cuando recuperamos algún objeto de la base de datos, estamos construyendo
un QuerySet usando el Manager del modelo, que  por default es ``objects``.
a menos que explícitamente asignemos uno.
Este QuerySet sabe como ejecutar SQL y retornar los objetos solicitados,
cada modelo tiene por lo menos un Manager.

.. admonition:: Que es un QuerySet:

    Un ``queryset`` es una colección de objetos de la base de datos.
    Puede tener cero, uno, o muchos filtros. En términos de ``SQL`` un
    QuerySet se compara a una declaración ``SELECT`` y un filtro es una
    cláusula de limitación como por ejemplo ``WHERE`` o ``LIMIT``.

.. admonition:: Que es un Manager:

    Un manager es la interfaz a través de la cuál se proveen las operaciones
    de consulta  de la base de datos (por default es ``objects``). 
   
Volvemos a entrar en el interprete interactivo con ``python manage.py shell``:
   
.. code-block:: python

   #importamos Categoria y Entrada de nuestros modelos.
    >>> from blog.models import Categoria, Entrada
    >>> import datetime
    # Con el metodo __unicode__ ya se puede leer.
    >>>Categoria.objects.all()
    [<Categoria: python>]
    #Creamos otra categoria:
    >>>a1=Categoria(nombre="perl",slug="Perl") 
    # La guardamos explícitamente con
    >>>a1.save()
    # Mostramos todas las categorias
    >>>Categoria.objects.all()
    [<Categoria: python>,<Categoria: perl>]
    #Podemos filtrar una categoria
    >>>Categoria.objects.filter(id=1)
    [<Categoria: python>]
    #Podemos acceder a elementos individuales con:
    >>>a=Categoria.objects.get(id=1)
    >>> a.nombre
    u'python'
    >>> a.slug
    u'Python'
    >>>a.id
    1
    #Borramos con delete()
    >>>a=Categoria.objects.all()
    >>>a.delete()
    >>>a = Categoria.objects.all() 
    >>>a
    []
    # Creamos otra Categoria 
    >>>a1=Categoria(nombre='python',slug='Python')
    #Debemos llamar al metodo save() para guardar.
    >>>a1.save
    <bound method Categoria.save of <Categoria: python>>
    #Al guardar no emite señal
    >>>a1.save()
    >>>a1
    <Categoria: python>
    #Creamos una Entrada
    >>>a2=Entrada(titulo='Mi primer post',
                  autor='Saul',
                  slug='Mi_primer_post',
                  texto='Hola Mundo este es mi primer post',
                  fecha='datetime.datetime.now',estatus='p')
    # La guardamos                  
    >>> a2.save()
    # Retorna el titulo de la Entrada(lo definimos en el metodo __unicode__)
    >>>a2
    <Entrada: Mi primer post>
    #Accedemos individualmente a cada uno de sus atributos.
    >>>a2.fecha
    datetime.datetime(2012, 8, 5, 9, 38, 54, 86292, tzinfo=<UTC>)
    >>> a2.autor
    'Saul'
    >>>a2.texto
    'Hola Mundo este es mi primer post'
    >>> a2.fecha
    datetime.datetime(2012, 8, 5, 9, 38, 54, 86292, tzinfo=<UTC>)
    >>> a2.estatus
    'p'
    >>>a=Entrada.objects.get(id=1)
    # Podemos filtrar un objeto y retornar el ultimo objeto de la tabla
    >>> Entrada.objects.latest('fecha')
    >>> a
    <Entrada: Mi primer post>
    # Django asigna una clave primaria(pk)
    >>> a.pk
    1
    # Podemos contar los objetos de la tabla
    >>> Entrada.objects.count()
    # Retornar las ultimas 5 entradas
    >>>Entrada.objects.all()[:5]
    # Podemos filtrar por año
    >>> Entrada.objects.filter(fecha__year=2012)
    [<Entrada: Mi primer post>]
    #Por mes
    >>>Entrada.objects.filter(fecha__month=09)
    [<Entrada: Mi primer post>]
    #Tenemos acceso al campo ManyToMany
    >>>Entrada.objects.filter(categoria__nombre='python')
    >>>entradas=Entrada.objects.get(pk=1)
    #Salimos de la API
    >>>exit()

Para crear un objecto primero creamos una instancia usando argumentos
clave y llamando al metodo ``save()``.

Las clases Meta:
---------------

Necesitamos agregar una cosa mas a nuestro modelo una ``clase Meta``.
Los modelos de Django utiliza la ``clase Meta`` para contener información
adicional sobre el modelo.
Los metadatos de un modelo pertenecen ala clase ``class Meta`` definida
en el cuerpo del modelo, los ``metadatos`` son cualquier cosa que no sea un
campo, como ``ordering``, obtener por un nombre, ``verbose_name``, el
nombre plural, ``db_table`` el nombre de la base de  datos a usar.

Tambien necesitamos agregar un método ``get_absolute_url``.

Que es ``get_absolute_url``
---------------------------

``get_absolute_url`` es un método de los modelos, que define como se
calcula la URL canónica  para un objeto dado. La cadena que retorna
``get_absolute_url`` debe contener caracteres ASCII solamente.
Es buena práctica utilizar ``get_absolute_url ()`` en los modelos, en vez de
codificar la  URL directamente en los objetos. Por ejemplo, este código es malo::

    <a href="/blog/{{ object.id }}/">{{ object.name }}</a>

es mejor usar este::

    <a href="{{ object.get_absolute_url }}">{{ object.name }}</a>
    


El decorador ``permalink``
--------------------------
La manera que escribimos el ``get_absolute_url()`` en el ejemplo anterior
es una clara violación al principio de **DRY** (No te repitas), la url de
este objeto esta definida en los modelos y en el archivo urlConf, para
poder desacoplar los modelos usamos el decorador ``permalink()``.
Este decorador toma el nombre de un patron url y una lista de argumentos
clave para construir el patron correcto de la url y desacoplarla. 


Nuestro modelo quedara así:

models.py:

.. code-block:: python

    from django.db import models
    from django.db.models import permalink

    ESTADO = (
        ('b', 'Borrador'),
        ('p', 'Publicado'),
        )

    class Categoria(models.Model):
        nombre = models.CharField(max_length=100, db_index=True)
        slug = models.SlugField(max_length=100, db_index=True)
    
        class Meta:
            verbose_name_plural='Categorias'
            
        def __unicode__(self):
            return '%s' % self.nombre

        @permalink
        def get_absolute_url(self):
            return ('categorias', None, { 'slug': self.slug })            

    
    class Entrada(models.Model):

        titulo = models.CharField(max_length=100, unique=True)
        slug = models.SlugField(max_length=100, unique=True)
        autor = models.CharField(max_length=100)
        texto = models.TextField()
        fecha = models.DateTimeField(db_index=True, auto_now_add=True)
        estatus = models.CharField(max_length=1, choices=ESTADO)
        categoria = models.ManyToManyField(Categoria) 
   
        class Meta:
            ordering = ['-fecha']
            verbose_name_plural = 'Entradas' 
		
        def __unicode__(self):
            return '%s' % self.titulo

        @permalink
        def get_absolute_url(self):
            return ('entradas', None, { 'slug': self.slug }) 

Eso es todo lo que necesitamos para definir nuestro modelo, ahora necesitamos
activar la interfaz administrativa para acceder visualmente a cada uno de
los objetos que hemos creado.      
