========================================
Creando formularios a partir de modelos
========================================

.. module:: django.forms.models
   :synopsis: ModelForm and ModelFormset.

.. currentmodule:: django.forms

``ModelForm``
=============
.. class:: ModelForm

Si estas construyendo una aplicación con una bases de datos, lo más probable es
que tendrás que usar formularios para asignarlos conjuntamente a los modelos de
Django. Por ejemplo, podrías tener un aplicación con un modelo ``BlogComment``
y si deseas crear un formulario que le permita a las personas enviar sus
comentarios. En este caso, sería redundante definir los tipos de campos en el
formulario, porque estos ya se han definido en los campos del modelo.

Por esta razón, Django proporciona una clase auxiliar que nos permite crear un
``Form`` (formulario) mediante una clase de un modelo de Django.

Por ejemplo::

    >>> from django.forms import ModelForm

    # Creamos la clase del formulario.
    >>> class ArticleForm(ModelForm):
    ...     class Meta:
    ...         model = Article

    # Creamos una forma para agregar un articulo.
    >>> form = ArticleForm()

    # Creamos una forma para cambiar un articulo existente.
    >>> article = Article.objects.get(pk=1)
    >>> form = ArticleForm(instance=article)

Tipos de campos
---------------

Cada formulario generado por una clase tendrá un campo de formulario para todos
los campos del modelo. cada modelo de campo tiene su correspondiente campo de
formulario predeterminado. Por ejemplo, un campo ``CharField`` en un modelo es
representado como un campo ``CharField`` en un formulario. Un campo
``ManyToManyField`` de un modelos se representa como un campo ``MultipleChoiceField``.
Aquí está la lista completa de conversiones:

===============================  ========================================
Campo en un modelo                     Campo en un formulario o forma
===============================  ========================================
``AutoField``                    No se representa en un formulario

``BigIntegerField``              ``IntegerField`` with ``min_value`` set
                                 to -9223372036854775808 and ``max_value``
                                 set to 9223372036854775807.

``BooleanField``                 ``BooleanField``

``CharField``                    ``CharField`` with ``max_length`` set to
                                 the model field's ``max_length``

``CommaSeparatedIntegerField``   ``CharField``

``DateField``                    ``DateField``

``DateTimeField``                ``DateTimeField``

``DecimalField``                 ``DecimalField``

``EmailField``                   ``EmailField``

``FileField``                    ``FileField``

``FilePathField``                ``CharField``

``FloatField``                   ``FloatField``

``ForeignKey``                   ``ModelChoiceField`` (see below)

``ImageField``                   ``ImageField``

``IntegerField``                 ``IntegerField``

``IPAddressField``               ``IPAddressField``

``GenericIPAddressField``        ``GenericIPAddressField``

``ManyToManyField``              ``ModelMultipleChoiceField`` (see
                                 below)

``NullBooleanField``             ``CharField``

``PositiveIntegerField``         ``IntegerField``

``PositiveSmallIntegerField``    ``IntegerField``

``SlugField``                    ``SlugField``

``SmallIntegerField``            ``IntegerField``

``TextField``                    ``CharField`` with
                                 ``widget=forms.Textarea``

``TimeField``                    ``TimeField``

``URLField``                     ``URLField``
===============================  ========================================

Como es de esperarse, los modelos de campo ``ForeignKey`` y ``ManyToManyField``
se manejan como casos especiales:

* ``ForeignKey`` es representado por ``django.forms.ModelChoiceField``,
  que es un  ``ChoiceField`` en el cual las opciones son un modelo ``QuerySet``.

* ``ManyToManyField`` se representa  por un campo ``django.forms.ModelMultipleChoiceField``,
  que es un campo ``MultipleChoiceField`` cuyas opciones son un modelo ``QuerySet``.:
  Además, cada campo de formulario generado por tiene establecido algún atributo de la siguiente manera:

* Si el modelo de campo cuenta con un ``blank = True``, entonces es necesaria usar
  ``False`` en el campo de formulario. De lo contrario, ``required = True``.

* Si la etiqueta del campo de formulario es ``verbose_name`` del modelo del campo,
  con el primer carácter en mayúscula.

* El campo de formulario ``help_text`` sera igual ``help_text`` del modelo de campo.

* Si el modelo de campo cuenta con una opción ``choices``, entonces  el widget
  del campo de formulario se establecerá en ``Select``, con opciones que vienen
  del campo el modelo de ``choices``. Las opciones incluirán normalmente la
  opción en blanco, que es seleccionada por defecto. Si se requiere el campo,
  esto obliga al usuario a hacer una selección. La elección en blanco no se
  incluirá si el modelo del campo contiene un ``blank = False`` y un explicito valor
  por ``default`` (el valor predeterminado se seleccionará inicialmente en su lugar).

Por último, ten en cuenta que se pueden reemplazar los campo de formularios
utilizado para un determinado modelo o campo. Puedes consultar:
`Reemplazando  los tipos de campos predeterminados o widgets` _ a continuación.

Un ejemplo completo
-------------------

Considera este conjunto de modelos::

    from django.db import models
    from django.forms import ModelForm

    TITLE_CHOICES = (
        ('MR', 'Mr.'),
        ('MRS', 'Mrs.'),
        ('MS', 'Ms.'),
    )

    class Author(models.Model):
        name = models.CharField(max_length=100)
        title = models.CharField(max_length=3, choices=TITLE_CHOICES)
        birth_date = models.DateField(blank=True, null=True)

        def __unicode__(self):
            return self.name

    class Book(models.Model):
        name = models.CharField(max_length=100)
        authors = models.ManyToManyField(Author)

    class AuthorForm(ModelForm):
        class Meta:
            model = Author

    class BookForm(ModelForm):
        class Meta:
            model = Book

Con estos modelos, la subclase ``ModelForm``  anterior sería más o menos
equivalente a este (la única diferencia es el metodo ``save ()``, el cual
vamos a discutir en un momento.) ::

    from django import forms

    class AuthorForm(forms.Form):
        name = forms.CharField(max_length=100)
        title = forms.CharField(max_length=3,
                    widget=forms.Select(choices=TITLE_CHOICES))
        birth_date = forms.DateField(required=False)

    class BookForm(forms.Form):
        name = forms.CharField(max_length=100)
        authors = forms.ModelMultipleChoiceField(queryset=Author.objects.all())

.. _modelform-is-valid-and-errors:


El metodo ``is_valid()`` y el metodo ``errors``
---------------------------------------------

La primera vez que se llama al metodo ``is_valid ()`` o accedemos a un atributo
``errores`` de un ``ModelForm`` disparamos un :ref:`form validation <form-and-field-validation>`
así como un :ref:`model validation <validating-objects>`. Esto tiene el efecto secundario
de que limpia el modelo que pasamos como constructor ``ModelForm``. Por ejemplo,
si llamamos al metodo ``is_valid ()`` en el formulario  este convertirá cualquier
campo de fecha del modelo a objetos actualizados. Si falla la validación del
formulario, sólo algunos de los cambios pueden ser aplicados. Por esta razón,
es probable que desees evitar reutilizar la  instancia del modelo que  pasamos a
la forma, sobre todo si la validación falla.

El metodo ``save()``
---------------------

Cada forma producida por ``ModelForm`` también contiene un metodo ``save ()``. Este
método crea y guarda un objeto en la base de datos a partir de los datos
que pasamos a la forma. Una subclase de ``ModelForm`` puede aceptar una existente
instancia con un el argumento de palabra clave,``instance``, si no
suministramos, un metodo ``save ()`` se actualizará dicha instancia. Si no se proporciona,
el metodo ``save ()`` se  crea una nueva instancia del modelo especificado:

.. code-block:: python

    # Creamos una forma a través de una instancia de datos POST.
    >>> f = ArticleForm(request.POST)

    # Guardamos un nuevo objeto Article object de los datos de la forma.
    >>> new_article = f.save()

    # Creamos una forma para editar un articulo existente, pero usando
    # datos POST que pasamos ala forma.
    >>> a = Article.objects.get(pk=1)
    >>> f = ArticleForm(request.POST, instance=a)
    >>> f.save()

Ten en cuenta que si la forma: :ref:`hasn't been validated
<modelform-is-valid-and-errors>`, se llamara a ``save ()`` mediante la comprobación de
``form.errors``. Un ``ValueError`` se incrementará si los datos en el formulario
no son validos - es decir, si ``form.errors`` evalúa como ``True``.

El método ``save ()`` acepta un argumento opcional ``commit``, que
acepta tanto  ``True`` o ``False``. Si se llama a  el metodo ``save ()`` con el
argumento ``commit = False``, entonces se devolverá un objeto que aún no se ha guardado en
la base de datos. En este caso, necesitamos llamar a el metodo ``save ()`` explicitamente, en
la misma instancia del modelo. Esto es útil si queremos hacer  un procesamiento
personalizado en algún objeto antes de guardarlo  o si deseamos utilizar algunas
opciones especiales antes de guardar el objeto :ref:`model saving options <ref-models-force-insert>`.
de otra forma ``commit`` es ``True`` de forma predeterminada.

Otro efecto secundario de usar ``commit = False`` se aprecia cuando el modelo tiene
una relación muchos-a-muchos con otro modelo. Si el modelo tiene una relación muchos-a-muchos
y especificamos ``commit = False`` al guardar el formulario, Django no puede
inmediatamente guardar los datos del formulario para el campo muchos-a-muchos. Esto es porque
no es posible guardar un campo muchos-a-muchos  para una instancia hasta que la instancia
exista en la base de datos.

Para solucionar este problema, cada vez que guardemos un formulario utilizando
``commit = False``, Django agrega un metodo ``save_m2m ()`` a la  subclase ``ModelForm``.
Después que haya guardado manualmente la instancia producida por el formulario,
se puede invocar  el método ``save_m2m ()`` para guardar el campo muchos-a-muchos
(many to many) de  datos del formulario. Por ejemplo ::

    # Creamos  una instancia de un formulario con datos POST.
    >>> f = AuthorForm(request.POST)

    # Creamos, pero no guardamos la instancia del nuevo autor.
    >>> new_author = f.save(commit=False)

    # Modificamos de alguna forma el autor.
    >>> new_author.some_field = 'some_value'

    # Guardamos la nueva instancia.
    >>> new_author.save()

    # Ahora, guardamos la forma y el campo many-to-many.
    >>> f.save_m2m()

Llamar a un metodo ``save_m2m ()`` sólo es necesario si utilizamos ``save (commit = False)``.
Cuando se utiliza una simple metodo ``save ()`` en un formulario, todos los datos - incluyendo
el campo muchos-a-muchos - se guardan sin la necesidad de ninguna llamadas a métodos adicionales.
Por ejemplo ::

    # Creamos  una instancia de un formulario con datos POST.
    >>> a = Author()
    >>> f = AuthorForm(request.POST, instance=a)

    # Creamos y guardamos la nueva instancia autor. No hay necesidad de hacer otra cosa.
    >>> new_author = f.save()

Aparte de los métodos ``save ()`` y ``save_m2m ()`` , un  ``ModelForm`` funciona
exactamente de la misma forma que cualquier otro formulario ``forms``. Por ejemplo,
el metodo ``is_valid ()``  se utiliza para comprobar la validez, del metodo `` is_multipart () ``
que se usa para determinar si una forma requiere subir múltiples archivos
(y por lo tanto, se requiere pasar a la forma un ``request.FILES``), puedes ver
:ref:`binding-uploaded-files`  para más información.

Usando un subconjunto de campos en un formulario
-------------------------------------------------

En algunos casos, es posible que no queremos que todos los campos de algún modelo
aparecerán en el formulario  generado. Hay tres maneras de decirle a ``ModelForm``
que  utiliza sólo un subconjunto de campos del modelo:

1. Usa ``editable = False`` en el modelo del campo. Como resultado, *cualquier forma*
creada a partir del modelo vía ``ModelForm`` no incluirá este campo.

2. Usa los campos internos del atributo ``ModelForm`` tal como la clase ``Meta``
Estos atributos, si se dan, deben ser una lista de nombres de campos
incluidos en el formulario. El orden en que se especifican los nombres de campos
en esa lista son respetados cuando la forma se renderiza.

3. Usa el atributo ``exclude`` del formulario ``ModelForm`` y la clase interna
``Meta``. Este atributo, si se da, debe ser una lista de nombres de campos
excluidos de la forma.

Por ejemplo, si queremos que en la forma el modelo ``Author`` (definido arriba)
incluya únicamente los campos  ``name`` y ``birth_date``, solo devemos especificar
``fields`` o ``exclude`` de esta forma::

    class PartialAuthorForm(ModelForm):
        class Meta:
            model = Author
            fields = ('name', 'birth_date')

    class PartialAuthorForm(ModelForm):
        class Meta:
            model = Author
            exclude = ('title',)


Desde  que el modelo Author tiene 3 campos 'name', 'title', y
'birth_date', el formulario de arriba contendra exactamente los mismos campos.

.. note::

    Si se especifican ``fields`` o ``exclude`` al crear un formulario con
    ``ModelForm``, los campos que no están en la forma resultante
    no será establecidos por el metodo ``save ()`` del formulario. Además, si
    agregamos manualmente los campos excluidos de nuevo a la forma, no podrán
    ser inicializados desde la instancia del modelo.

    Django evitará cualquier intento de guardar un modelo incompleto, por lo que si
    el modelo no permite que los campos que faltan estén vacíos y no proporcionamos
    un valor por defecto para los campos que faltan, cualquier intento de llamar a
    el metodo ``save ()`` a  través de un ``ModelForm`` con los campos que faltan
    fallará, para evitar este error, se debe crear una  instancia de el modelo con
    valores iniciales para los campos que excluimos pero que son requeridos::

        author = Author(title='Mr')
        form = PartialAuthorForm(request.POST, instance=author)
        form.save()

    Alternativamente podemos usar  ``save(commit=False)`` y  manualmente ingresar
    valores extra requeridos a los campos::

        form = PartialAuthorForm(request.POST)
        author = form.save(commit=False)
        author.title = 'Mr'
        author.save()

    Puedes consultar la sección: `section on saving forms`_  para mas detalles usando
    ``save(commit=False)``.

.. _section on saving forms: `The save() method`_

Reemplazar los tipos de campo predeterminados o widgets
--------------------------------------------------------

Los tipos de campos predeterminados, como se describe en los tipos de campo `Field types`_  de
la tabla anterior, son parámetros por defecto. Si deseamos usar  un ``DateField``
en un modelo, lo más probable es que necesitemos representar este campo como
un campo  ``DateField`` en el formulario, pero un ``ModelForm`` nos da la
suficiente  flexibilidad para cambiar el tipo de campo de un  formulario y
el widget para un campo determinado del modelo.

Para especificar un widget personalizado para un campo, utilizamos el atributo
``widgets`` de la clase interior ``Meta``. Este debería ser un diccionario
con los nombres de los campos de mapeo entre las clases o instancias.

Por ejemplo, si queremos tener un  ``CharField``  para un campo ``name`` del modelo
``Author`` este  debe ser representado por una instancia de un ``<textarea>`` en lugar
del que se provee por defecto ``<input type="text">``, podemos  anular el campo del
Widget así::

    from django.forms import ModelForm, Textarea

    class AuthorForm(ModelForm):
        class Meta:
            model = Author
            fields = ('name', 'title', 'birth_date')
            widgets = {
                'name': Textarea(attrs={'cols': 80, 'rows': 20}),
            }


El diccionario de el ``widgets`` acepta una instancia de un widget  (e.g.,
``Textarea(...)``) o una  clase (e.g., ``Textarea``).


Si queremos personalizar un campo --incluyendo el tipo, la etiqueta, etc -- podemos
usar para ello una declaración especificando los campos en una forma ``Form``
Declarando los campos sobrescribimos los parámetros por default, una vez generado
usamos el atributo  ``model``.

Por ejemplo. si queremos usar un ``MyDateFormField`` para el campo ``pub_date``
necesitamos hacer lo siguiente::

    class ArticleForm(ModelForm):
        pub_date = MyDateFormField()

        class Meta:
            model = Article

Si deseamos reemplazar un ``label``  predeterminado de un campo, necesitamos
especificar a  continuación, el parámetro ``label`` al declarar el campo del formulario ::


   >>> class ArticleForm(ModelForm):
   ...     pub_date = DateField(label='Publication date')
   ...
   ...     class Meta:
   ...         model = Article

.. note::

   Si creamos una instancia explícitamente de  un campo de un formulario como este,
   Django asume que deseamos cambiar completamente su comportamiento, por lo tanto,
   los atributos por defecto (por ejemplo, ``max_length``  o ``required``) no
   estarán presentes en los correspondientes modelos, si queremos mantener el comportamiento
   especifico en el modelo, debemos establecer los argumentos pertinentes de manera
   explícita al declarar el campo de formulario.

   Por ejemplo, si el modelo ``Article`` se ve así::

        class Article(models.Model):
            headline = models.CharField(max_length=200, null=True, blank=True,
                                        help_text="Use puns liberally")
            content = models.TextField()

    Si queremos hacer alguna validación personalizada para el  ``headline``, manteniendo
    los valores ya especificados ``blank`` y``help_text``, necesitamos definir
    ``ArticleForm`` así::

        class ArticleForm(ModelForm):
            headline = MyFormField(max_length=200, required=False,
                                   help_text="Use puns liberally")

            class Meta:
                model = Article

Debemos asegurarnos  de que el tipo de campo de el formulario se puede utilizar
para establecer el contenido del campo correspondiente a el modelo. Cuando no son
compatibles, se obtendrá un error del tipo  ``ValueError`` y ninguna conversión
implícita se realiza.

Puedes consultar :doc:`form field documentación </ref/forms/fields>` para obtener
más información sobre los campos y sus argumentos.

Cambiar el orden de los campos
-------------------------------

Por defecto, un ``ModelForm`` renderizara los campos del formulario en el mismo
orden en que son definidos en el modelo, con los campos ``ManyToManyField`` al
ultimo, si deseamos cambiar el orden en que se representan los campos, se puede
utilizar el atributo interno de la clase ``Meta``.

Los atributos de un campo  o ``fields`` definen  el subconjunto de campos del modelo
que serán presentados y el orden en que van a ser renderizados. Por ejemplo dado este
modelo::

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=100)

El campo ``author`` sera renderizado primero. Si queremos que el titulo del campo
sea renderizado primero, necesitamos especificarlo en el  ``ModelForm`` así::

    >>> class BookForm(ModelForm):
    ...     class Meta:
    ...         model = Book
    ...         fields = ('title', 'author')

.. _overriding-modelform-clean-method:

Reemplazando el método clean()
------------------------------

Podemos reemplazar el método clean() en un formulario de un  modelo para proporcionar
validación adicional de la misma manera en que lo hacemos  en una forma normal.

En este sentido, las formas de un modelo tienen dos características específicas
cuando se compara con formas:

Por defecto, el método ``clean()`` valida los campos que son marcados como
``unique``, ``unique_together`` o ``unique_for_date|month|year`` en un modelo.
Por lo tanto, si necesitamos anular el  metodo ``clean ()`` y mantener la
validación por defecto, debemos  llamar al metodo padre de la clase ``clean ()``.

Además, una instancia del modelo del formulario  enlazada a un objeto del modelo,
incluirá un atributo ``self.instance`` que da acceso al metodo de la forma del modelo especifico.

Herencia en formularios
-----------------------

Al igual que con los formularios básicos, podemos extender y volver a
utilizar ``ModelForms`` usando la herencia. Esto es útil si tenemos que declarar
campos adicionales o métodos adicionales en una  clase padre para su uso en un
número de formas derivadas de los modelos. Por ejemplo, utilizando la clase
anterior  ``ArticleForm``::

    >>> class EnhancedArticleForm(ArticleForm):
    ...     def clean_pub_date(self):
    ...         ...

Esto crea un formulario que se comporta igual que un ``ArticleForm``, excepto que no
tiene alguna validación adicional y limpieza para el campo ``pub_date``.

También podemos usar una subclase padre interna ``Meta``  si queremos cambiar las
listas ``Meta.fields`` o ``Meta.excludes``::

    >>> class RestrictedArticleForm(EnhancedArticleForm):
    ...     class Meta(ArticleForm.Meta):
    ...         exclude = ('body',)

Esto agrega un método extra a la ``EnhancedArticleForm`` y modifica
el campo original ``ArticleForm.Meta`` para eliminar un campo.

Hay un par de cosas a tener en cuenta, sin embargo.

* Se aplican reglas de resolución de nombres Python normales. Si tenemos alguna
  base de múltiples clases que se declaran en una clase interna hija  ``Meta``,
  sólo el primero será  utilizado. Esto significa que el hijo ``Meta``, si es que
  existe, de lo contrario se usar el primer  ``Meta``, etc

* Por razones técnicas, una subclase no puede heredar de ambos: ``ModelForm``
  y ``Form`` simultáneamente.

Es probable que estas notas no te afectarán a a menos que estés tratando de hacer algo
difícil con subclases.

Interacción con la validación de un modelo
------------------------------------------

Como parte del proceso de validación de  ``ModelForm`` se llamara al metodo
``clean ()`` para cada campo en el modelo que tiene un campo correspondiente en
el formulario. Si se ha excluido los campos del modelo, no se llevará a cabo la
validación de los campos. Véase la documentación sobre validación de formularios en
:doc:`form validation </ref/forms/validation>` para mas información sobre cómo
limpiar y el trabajo de validación. Además, el modelo del metodo ``clean ()``
se llama antes de que se chequen que son únicos, puedes  ver
:ref:`Validating objects <validating-objects>`   para obtener más información sobre
como anclar el metodo ``clean ()`` a un modelo.

Funciónes de fábrica ModelForm
-------------------------------

Podemos crear formularios de un modelo determinado, utilizando la función independiente
:func:`~django.forms.models.modelform_factory`,  en lugar de utilizar una clase
que hayamos definido. Esto puede ser más conveniente si no tenemos muchas personalizaciones
para hacer ::

    >>> from django.forms.models import modelform_factory
    >>> BookForm = modelform_factory(Book)

Esto puede ser usado para hacer simples modificaciones a una forma existente, por
ejemplo especifcando que campos seran mostrados::

    >>> Form = modelform_factory(Book, form=BookForm, fields=("author",))

... o que campos seran excluidos::

    >>> Form = modelform_factory(Book, form=BookForm, exclude=("title",))

Podemos usar especificamente un  widgets para usar una campo dado::

    >>> from django.forms import Textarea
    >>> Form = modelform_factory(Book, form=BookForm, widgets={"title": Textarea()})

.. _model-formsets:

Conjunto de formularios o formsets
==================================

.. class:: models.BaseModelFormSet

Tal como :doc:`regular formsets </topics/forms/formsets>`, Django provee un par de
mejoradas clases para un conjunto de formularios, para trabajar mas fácilmente con
modelos Django, vamos a utilizar el modelo ``Author`` que usamos arriba:

Esto creará un conjunto de formularios que sera capaz de trabajar con los datos asociados
del modelo ``Autor``. Estos trabajan justo como un conjunto de formularios regulares ::

   >>> formset = AuthorFormSet()
    >>> print(formset)
    <input type="hidden" name="form-TOTAL_FORMS" value="1" id="id_form-TOTAL_FORMS" />
    <input type="hidden" name="form-INITIAL_FORMS" value="0" id="id_form-INITIAL_FORMS" />
    <input type="hidden" name="form-MAX_NUM_FORMS" id="id_form-MAX_NUM_FORMS" />
    <tr><th><label for="id_form-0-name">Name:</label></th><td>
    <input id="id_form-0-name" type="text" name="form-0-name" maxlength="100" /></td></tr>
    <tr><th><label for="id_form-0-title">Title:</label>
    </th><td><select name="form-0-title" id="id_form-0-title">
    <option value="" selected="selected">---------</option>
    <option value="MR">Mr.</option>
    <option value="MRS">Mrs.</option>
    <option value="MS">Ms.</option>
    </select></td></tr>
    <tr><th><label for="id_form-0-birth_date">Birth date:</label></th><td>
    <input type="text" name="form-0-birth_date" id="id_form-0-birth_date" />
    <input type="hidden" name="form-0-id" id="id_form-0-id" /></td></tr>

.. note::

    :func:`~django.forms.models.modelformset_factory` usa ``formset_factory`` para
    generar el conjunto de formularios o formsets.Esto significa que un modelo de
    de un formeset es sólo una extensión de un conjunto de formularios básico
    que sabe cómo interactuar con un modelo particular.

Cambiando el queryset
---------------------

Por defecto, cuando se crea un conjunto de formularios en un modelo, el conjunto
de formularios usará un queryset que incluye todos los objetos del modelo (por ejemplo,
``Author.objects.all ()``). Podemos modificar este comportamiento a través de el
argumento ``querySet``::

    >>> formset = AuthorFormSet(queryset=Author.objects.filter(name__startswith='O'))

Alternativamente podemos crear una subclase estableciendo  ``self.queryset`` en
``__init__``::

    from django.forms.models import BaseModelFormSet

    class BaseAuthorFormSet(BaseModelFormSet):
        def __init__(self, *args, **kwargs):
            super(BaseAuthorFormSet, self).__init__(*args, **kwargs)
            self.queryset = Author.objects.filter(name__startswith='O')

Al pasar la clase, ``BaseAuthorFormSet`` ala funcion factory::

    >>> AuthorFormSet = modelformset_factory(Author, formset=BaseAuthorFormSet)


Si queremos retornar un formset que no incluya *ninguna* instancia pre-existente
en el modelo, podemos especificar un QuerySet vacio::

   >>> AuthorFormSet(queryset=Author.objects.none())

Controlar que campos son usados con ``fields`` y ``exclude``
------------------------------------------------------------

De forma predeterminada, un modelo de un formset utiliza todos los campos del
modelo que no han sido marcados con ``editable = False``. Sin embargo, esto puede
puede ser reemplazado  en el nivel del formset::

>>> AuthorFormSet = modelformset_factory (Autor, campos = ('nombre', 'title'))

Usando ``fields``  restringimos el conjunto de formularios a utilizar sólo a los
campos indicados. Alternativamente, podemos tomar un enfoque de "opt-out", que
especifica que campos vamos a excluir ::

    >>> AuthorFormSet = modelformset_factory(Author, exclude=('birth_date',))

Proporcionar valores iniciales
-------------------------------

.. versionadded:: 1.4

Al igual que con formsets regulares, es posible: :ref:`specify initial data
<formsets-initial-data>` para el conjunto de formularios mediante la especificación
de un parametro  ``initial``  al crear instancias de el conjunto de clases de
formularios  del modelo este devuelve :func:`~django.forms.models.modelformset_factory`.
Sin embargo, con el modelo formsets, los valores iniciales se aplican solamente a las
formas adicionales, a las que no están vinculados a a una instancia de objeto existente.

.. _saving-objects-in-the-formset:

Guardar objetos en el formset
-----------------------------

Al igual que con un ``ModelForm``, se puede guardar los datos como un objeto modelo.
Esto se hace con el  método ``save()``::

    # Creamos una instancia formset con datos POST.
    >>> formset = AuthorFormSet(request.POST)

    # Asumimos que todo es valido, guardamos los datos.
    >>> instances = formset.save()

El método ``save ()`` devuelve las instancias que se han guardado en la
base de datos. Si los datos de una instancia dada no cambiaron con los datos vinculados, la
instancia no se guardará en la base de datos y no se incluirá en el retorno
del valor ( ``instances``, en el ejemplo anterior).

Cuando los campos no figuran en el forma (por ejemplo, debido a que tienen o
sido excluidos), estos campos no se definirán por el  metodo ``save ()``.
Se puede encontrar más información sobre esta restricción, que también es válida
para ``ModelForms`` puedes ver `Using a subset of fields on
the form`_.

Pasamos ``commit=False`` para retornar una instancia de un modelo no guardada::

    # No guardamos la base de datos
    >>> instances = formset.save(commit=False)
    >>> for instance in instances:
    ...     # Hacemos algo con la instancia
    ...     instance.save()

Esto da la posibilidad de adjuntar datos a las instancias antes de guardarlos
a la base de datos. Si el formset contiene un campo  ``ManyToManyField``, además
deberá llamarse a  ``formset.save_m2m ()`` para asegurar que se guarde correctamente
se guardan correctamente.

.. _model-formsets-max-num:

Limitar el número de objetos editables
---------------------------------------

Al igual que con formsets regulares, podemos utilizar lo parametros adicionales
``max_num`` y ``extra`` de  :func:`~django.forms.models.modelformset_factory`
para limitar el número de formularios adicionales que se  muestran,
``max_num`` no previe que un objeto existente sea mostrado::

    >>> Author.objects.order_by('name')
    [<Author: Charles Baudelaire>, <Author: Paul Verlaine>, <Author: Walt Whitman>]

    >>> AuthorFormSet = modelformset_factory(Author, max_num=1)
    >>> formset = AuthorFormSet(queryset=Author.objects.order_by('name'))
    >>> [x.name for x in formset.get_queryset()]
    [u'Charles Baudelaire', u'Paul Verlaine', u'Walt Whitman']

Si el valor ``max_num`` es mayor que el numero de relaciones existentes en los objetos,
hasta ``extra`` adicionalmente se agregararan  formas al formulario formas adicionales en blanco
valor de `` `` MAX_NUM es mayor que el número de relacionados existentes
objetos, hasta un `` `` adicional se agregarán formularios en blanco al formset,
siempre que el número total de formularios no exceda el ``max_num`` ::

    >>> AuthorFormSet = modelformset_factory(Author, max_num=4, extra=2)
    >>> formset = AuthorFormSet(queryset=Author.objects.order_by('name'))
    >>> for form in formset:
    ...     print(form.as_table())
    <tr><th><label for="id_form-0-name">Name:</label></th><td>
    <input id="id_form-0-name" type="text" name="form-0-name"
    value="Charles Baudelaire" maxlength="100" /><input type="hidden"
    name="form-0-id" value="1" id="id_form-0-id" /></td></tr>
    <tr><th><label for="id_form-1-name">Name:</label></th><td><input
     id="id_form-1-name" type="text" name="form-1-name" value="Paul Verlaine"
     maxlength="100" /><input type="hidden" name="form-1-id" value="3"
     id="id_form-1-id" /></td></tr>
    <tr><th><label for="id_form-2-name">Name:</label></th><td><input
     id="id_form-2-name" type="text" name="form-2-name" value="Walt Whitman"
      maxlength="100" /><input type="hidden" name="form-2-id" value="2"
      id="id_form-2-id" /></td></tr>
    <tr><th><label for="id_form-3-name">Name:</label></th><td><input
    id="id_form-3-name" type="text" name="form-3-name" maxlength="100" />
    <input type="hidden" name="form-3-id" id="id_form-3-id" /></td></tr>

Un valor ``max_num`` o ``None`` (el valor por  default) tiene un limite en el numero
de formar ha mostrar(1000). En la practica este equivalente no tiene limites.

Usando un modelo formset en una vista
-------------------------------

Los modelos formsets son muy similares a formsets. Digamos que deseamos presentar un
formSet par  editar  una instancia de nuestro modelo ``Author``::

    def manage_authors(request):
        AuthorFormSet = modelformset_factory(Author)
        if request.method == 'POST':
            formset = AuthorFormSet(request.POST, request.FILES)
            if formset.is_valid():
                formset.save()
                # Hacemos algo.
        else:
            formset = AuthorFormSet()
        return render_to_response("manage_authors.html", {
            "formset": formset,
        })

Como se puede ver, la lógica de la vista de un modelo formset no es drásticamente diferente
que el de un formset "normal". La única diferencia es que llamamos
almetodo ``formset.save ()`` para guardar los datos en la base de datos.
(Esto se describió anteriormente, en: in :ref:`saving-objects-in-the-formset`.)

Sobrescribir el metodo ``clean ()`` en un ``model_formset``
------------------------------------------------------------

Al igual que con ``ModelForms``, por defecto el metodo ``clean ()`` de un
``model_formset`` validará que ninguno de los elementos en el formset violen
las restricciones únicas de el modelo (ya sean ``unique``, ``unique_together`` o
``unique_for_date|month|year``). Si deseamos reemplazar el metodo ``clean()``
en un ``model_formset`` y mantener esta validación, debemos llamar a los padres
de la clase del método ``clean``::

    class MyModelFormSet(BaseModelFormSet):
        def clean(self):
            super(MyModelFormSet, self).clean()
            #  Ejemplo de validación personalizada a través de formas en el formset:
            for form in self.forms:
                # la validación personaliza del formset

Usando un queryset personalizado
-----------------------

Como se señaló anteriormente, se puede reemplazar el queryset predeterminado
utilizado por el modelo formset ::

    def manage_authors(request):
        AuthorFormSet = modelformset_factory(Author)
        if request.method == "POST":
            formset = AuthorFormSet(request.POST, request.FILES,
                                    queryset=Author.objects.filter(name__startswith='O'))
            if formset.is_valid():
                formset.save()
                # Hacemos algo.
        else:
            formset = AuthorFormSet(queryset=Author.objects.filter(name__startswith='O'))
        return render_to_response("manage_authors.html", {
            "formset": formset,
        })

Observe que pasamos el argumento ``queryset`` tanto en la llamada a ``POST`` y ``GET``
en este ejemplo.

Utilizando el formset en las plantillas
---------------------------------

.. highlight:: html+django

Existen tres formas de renderizar un formseten una plantilla de Django.

Primero, dejemos que el formset haga su trabajo::

    <form method="post" action="">
        {{ formset }}
    </form>

En segundo lugar, podemos manualmente renderizar el formset, para presentar
el formulario en si mismo::

    <form method="post" action="">
        {{ formset.management_form }}
        {% for form in formset %}
            {{ form }}
        {% endfor %}
    </form>

Al procesar manualmente los formularios, debemos asegúrarnos de que menejamos el
renderizado de las formas como se muestra arriba. Consulta :ref:`management form documentation
<understanding-the-managementform>`, para mayor información.

En tercer lugar, podemos renderizar cada campo manualmente::

    <form method="post" action="">
        {{ formset.management_form }}
        {% for form in formset %}
            {% for field in form %}
                {{ field.label_tag }}: {{ field }}
            {% endfor %}
        {% endfor %}
    </form>

Si optamos por usar el tercer metodo y no queremos iterar sobre cada campo con 
un bucle ``{% for %}``, necesitamos renderizar el campo primario.Por ejemplo si queremos
renderizar los campos ``name`` y ``age`` de nuestro modelo::

    <form method="post" action="">
        {{ formset.management_form }}
        {% for form in formset %}
            {{ form.id }}
            <ul>
                <li>{{ form.name }}</li>
                <li>{{ form.age }}</li>
            </ul>
        {% endfor %}
    </form>    
   
   
Observa cómo necesitamos de forma explícita renderizar un ``{{form.id}}``. Esto 
asegura que los modelo del formset, en los casos ``POST``, funcionará correctamente.
(En este ejemplo se asume una clave principal llamad ``id``. Si se ha definido
explícitamente una clave principal esta no sera llamada con ``id``, devemos
asegurarnos que esta sea renderizada).    

.. highlight:: python

.. _inline-formsets:


formsets en línea
=================

Los formsets en linea son una pequeña capa de abstracción por encima del modelo
formsets modelo, estos  casos simplifican, el  trabajar con relaciones de objetos 
a través de una clave externa. Supongamos que tenemos estos dos modelos ::

    class Author(models.Model):
        name = models.CharField(max_length=100)

    class Book(models.Model):
        author = models.ForeignKey(Author)
        title = models.CharField(max_length=100)

Si deseamos crear una conjunto de formularios o ``formsets`` que nos permita
editar libros pertenecientes a un autor en particular, podríamos hacer lo 
siguiente::


    >>> from django.forms.models import inlineformset_factory
    >>> BookFormSet = inlineformset_factory(Author, Book)
    >>> author = Author.objects.get(name=u'Mike Royko')
    >>> formset = BookFormSet(instance=author)

.. note::

    :func:`~django.forms.models.inlineformset_factory` uses
    :func:`~django.forms.models.modelformset_factory` and marks
    ``can_delete=True``.

.. seealso::

    :ref:`Manually rendered can_delete and can_order <manually-rendered-can-delete-and-can-order>`.
    
Más de un foreign key  para un mismo modelo
----------------------------------------------

Si el modelo contiene más de un campo foreign key en el mismo modelo, necesitamos
resolver la ambigüedad manualmente usando  ``fk_name``. Por ejemplo, considera
el siguiente modelo::    


    class Friendship(models.Model):
        from_friend = models.ForeignKey(Friend)
        to_friend = models.ForeignKey(Friend)
        length_in_months = models.IntegerField()

Para resolver esto, necesitamos usar la función: ``fk_name`` para
:func:`~django.forms.models.inlineformset_factory`::

    >>> FriendshipFormSet = inlineformset_factory(Friend, Friendship, fk_name="from_friend")    
    
Utilizando un formset en una vista en línea
--------------------------------------------

Es posible que quieras proporcionar una vista que permite al usuario editar las
relaciones de los objetos de un modelo. Así es como puedes hacerlo ::   


    def manage_books(request, author_id):
        author = Author.objects.get(pk=author_id)
        BookInlineFormSet = inlineformset_factory(Author, Book)
        if request.method == "POST":
            formset = BookInlineFormSet(request.POST, request.FILES, instance=author)
            if formset.is_valid():
                formset.save()
                # Hacemos algo. En general usamos una redirección. Por ejemplo:
                return HttpResponseRedirect(author.get_absolute_url())
        else:
            formset = BookInlineFormSet(instance=author)
        return render_to_response("manage_books.html", {
            "formset": formset,
        })   

Observa como pasamos una  ``instance`` en ambos casos  ``POST`` y ``GET``.
