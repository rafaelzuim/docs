Entidades
########

.. php:namespace:: Cake\ORM

.. php:class:: Entity

Enquanto :doc:`/orm/table-objects` representam e promovem acesso à coleções de objetos, entidades representam linhas individuais ou objetos do domínio na sua aplicação. Entidades possuem propriedades persistentes e métodos para manipular e acessar a informação contida nelas.

Entidades são criadas para você, pelo CakePHP cada vez que você usa  ``find()`` em um objeto Tabela.

Criando uma Classe do tipo Entidades
====================================

VocÊ não precisa  criar uma classe do tipo entidade para começar a  utilizar ORM. De qualquer maneira, se você deseja ter uma lógica própria em suas entidades você precisará criar classes. Por conveção, as classes do tipo entity ficam em **src/Model/Entity/**. Se a sua aplicação possui uma tabela chamada ``artigos`` nós precisamos criar a seguinte entidade::
our application had an ``articles`` table we could create the following entity::

    // src/Model/Entity/Artigo.php
    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class Artigo extends Entity
    {
    }

Por hora esta entidade não faz muita coisa. De qualquer maneira,quando nós carregamos dados da tabela artigos, nnós teremos uma instância desta classe.


.. note::
    Se você não definir uma classe do tipo entidade CakePHP vai utilizar uma classe padrão do tipo entidade.    

Criando entidades
=================

Entidades podem ser instânciadas diretamente::

    use App\Model\Entity\Artigos;

    $article = new Article();

When instantiating an entity you can pass the properties with the data you want
to store in them::

    use App\Model\Entity\Article;

    $artigo = new Artigo([
        'id' => 1,
        'title' => 'Novo artigo',
        'created' => new DateTime('now')
    ]);

Uma outra maneira de criar novas entidades é utilizando o método ``newEntity()`` dos objetos do tipo ``Table``::

    use Cake\ORM\TableRegistry;

    $article = TableRegistry::get('Artigos')->newEntity();
    $article = TableRegistry::get('Artigos')->newEntity([
        'id' => 1,
        'title' => 'Novo Artigo',
        'created' => new DateTime('now')
    ]);

Acessando os dados da entidade
=====================

Entidades disponibilizam alguns caminhos para acessar a informação nelas contidas. O mais comum é utilizar a notação de objeto para acessar os dados::

    use App\Model\Entity\Artigo;

    $article = new Artigo;
    $article->title = 'Este é meu primeiro post';
    echo $article->title;

Você também pode utilizar os métodos ``get()`` e ``set()``::

    $article->set('title', 'Este é meu primeiro post');
    echo $article->get('title');

Quando utilizar ``set()`` você pode atualizar várias propriedades ao mesmo tempo usando um array::

    $article->set([
        'title' => 'Meu primeiro post',
        'body' => 'Isto é o melhor de tudo!'
    ]);

.. cuidado::
       
    Ao atualizar entidades com dados de uma requisição, você precisa definir a lista autorizada de quais campos você vai permiti  ser informados via atribuição em massa.    

Acessores & Mutadores
====================

Em adicção a interface simples dos métodos simples get/set, entidades pemitem que você provenha métodos acessores e mutadores. Estes métodos lhe permitem definir como os as propriedades da entidade são escritas ou lidas.

Acessores usam a convenção ``_get`` seguido do nome do campo no padrão CamelCase.

.. php:method:: get($field)

Eles recebem o valor padrão armazenado no array ``_properties`` como seu único argumento. Acessores serão utilizados quando salvarmos as entidades, então tenha cuidado ao definir métodos que formatam os dados, pois eles persistiram formatados no momento de persistir a entidade. Por exemplo::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class Artigos extends Entity
    {
        protected function _getTitle($title)
        {
            return ucwords($title);
        }
    }
O acessor estaria rodando quando pegamos a propriedade através de qualquer um dos caminhos::

    echo $user->title;
    echo $user->get('title');

Você pode customizar como as propriedades são ldias e escritas definindo um mutador:

.. php:method:: set($field = null, $value = null)

Métodos mutadores devem sempre retornar o valor que deve ser salvo na propriedade. Como você pode ver abaixo, você também pode utilizar mutadores para setar outras propriedades calculadas. Quando estiver fazendo isso, seja cuidadoso para não introduzir nenhum laço, pois o CakePHP não previne métodos com laços infinitos..

Mutadores permitem a você converter propriedades como elas são setadas, ou criar dados calculados. Mutadores e acessores são aplicados quando as propriedades são lidas usando a notação de objeto, ou utilizando os métodos ``get()`` e ``set()``. Por exemplo::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;
    use Cake\Utility\Text;

    class Artigo extends Entity
    {

        protected function _setTitle($title)
        {
            $this->set('slug', Text::slug($title));
            return $title;
        }

    }
O mutador seria inicializado quando setarmos a propriedade através de qualquer um dos caminhos a segui::

    $user->title = 'foo'; // slug is set as well
    $user->set('title', 'foo'); // slug is set as well

.. _entities-virtual-properties:

Criando campos virtuais
-----------------------
Definindo acessores você pode ter acesso a campos/propriedades qua na realidade não existem. Por exemplo se a sua tabela de usuários possui os campos ``primeiro_nome`` and ``ultimo_nome`` você pode criar o método para ler o nome completo::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class Usuario extends Entity
    {

        protected function _getNomeCompleto()
        {
            return $this->_properties['primeiro_nome'] . '  ' .
                $this->_properties['ultimo_nome'];
        }

    }
Você pode acessar campos virtuais comose ele existisse realmente na entidade. O nome da propriedade será o nome da método em letras minúsculas e utilizando underscore para serparar as palavras.

    echo $user->nome_completo;

Tenha em mente que campos virtuais não podem ser utilizados para efetuar buscas.


Checando se uma entidade foi modificada
========================================

.. php:method:: dirty($field = null, $dirty = null)

You may want to make code conditional based on whether or not properties have
changed in an entity. For example, you may only want to validate fields when
they change::

    // See if the title has been modified.
    $article->dirty('title');

You can also flag fields as being modified. This is handy when appending into
array properties::

    // Add a comment and mark the field as changed.
    $article->comments[] = $newComment;
    $article->dirty('comments', true);

In addition you can also base your conditional code on the original property
values by using the ``getOriginal()`` method. This method will either return
the original value of the property if it has been modified or its actual value.

You can also check for changes to any property in the entity::

    // See if the entity has changed
    $article->dirty();

To remove the dirty mark from fields in an entity, you can use the ``clean()``
method::

    $article->clean();

When creating a new entity, you can avoid the fields from being marked as dirty
by passing an extra option::

    $article = new Article(['title' => 'New Article'], ['markClean' => true]);

Validation Errors
=================

.. php:method:: errors($field = null, $errors = null)

After you :ref:`save an entity <saving-entities>` any validation errors will be
stored on the entity itself. You can access any validation errors using the
``errors()`` method::

    // Get all the errors
    $errors = $user->errors();

    // Get the errors for a single field.
    $errors = $user->errors('password');

The ``errors()`` method can also be used to set the errors on an entity, making
it easier to test code that works with error messages::

    $user->errors('password', ['Password is required.']);

.. _entities-mass-assignment:

Mass Assignment
===============

While setting properties to entities in bulk is simple and convenient, it can
create significant security issues. Bulk assigning user data from the request
into an entity allows the user to modify any and all columns. When using
anonymous entity classes or creating the entity class with the :doc:`/bake`
CakePHP does not protect against mass-assignment.

The ``_accessible`` property allows you to provide a map of properties and
whether or not they can be mass-assigned. The values ``true`` and ``false``
indicate whether a field can or cannot be mass-assigned::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class Article extends Entity
    {
        protected $_accessible = [
            'title' => true,
            'body' => true
        ];
    }

In addition to concrete fields there is a special ``*`` field which defines the
fallback behavior if a field is not specifically named::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class Article extends Entity
    {
        protected $_accessible = [
            'title' => true,
            'body' => true,
            '*' => false,
        ];
    }

.. note:: If the ``*`` property is not defined it will default to ``false``.

Avoiding Mass Assignment Protection
-----------------------------------

When creating a new entity using the ``new`` keyword you can tell it to not
protect itself against mass assignment::

    use App\Model\Entity\Article;

    $article = new Article(['id' => 1, 'title' => 'Foo'], ['guard' => false]);

Modifying the Guarded Fields at Runtime
---------------------------------------

You can modify the list of guarded fields at runtime using the ``accessible``
method::

    // Make user_id accessible.
    $article->accessible('user_id', true);

    // Make title guarded.
    $article->accessible('title', false);

.. note::

    Modifying accessible fields effects only the instance the method is called
    on.

When using the ``newEntity()`` and ``patchEntity()`` methods in the ``Table``
objects you can customize mass assignment protection with options. Please refer
to the :ref:`changing-accessible-fields` section for more information.

Bypassing Field Guarding
------------------------

There are some situations when you want to allow mass-assignment to guarded
fields::

    $article->set($properties, ['guard' => false]);

By setting the ``guard`` option to ``false``, you can ignore the accessible
field list for a single call to ``set()``.


Checking if an Entity was Persisted
-----------------------------------

It is often necessary to know if an entity represents a row that is already
in the database. In those situations use the ``isNew()`` method::

    if (!$article->isNew()) {
        echo 'This article was saved already!';
    }

If you are certain that an entity has already been persisted, you can use
``isNew()`` as a setter::

    $article->isNew(false);

    $article->isNew(true);

.. _lazy-load-associations:

Lazy Loading Associations
=========================

While eager loading associations is generally the most efficient way to access
your associations, there may be times when you need to lazily load associated
data. Before we get into how to lazy load associations, we should discuss the
differences between eager loading and lazy loading associations:

Eager loading
    Eager loading uses joins (where possible) to fetch data from the
    database in as *few* queries as possible. When a separate query is required,
    like in the case of a HasMany association, a single query is emitted to
    fetch *all* the associated data for the current set of objects.
Lazy loading
    Lazy loading defers loading association data until it is absolutely
    required. While this can save CPU time because possibly unused data is not
    hydrated into objects, it can result in many more queries being emitted to
    the database. For example looping over a set of articles & their comments
    will frequently emit N queries where N is the number of articles being
    iterated.

While lazy loading is not included by CakePHP's ORM, you can just use one of the
community plugins to do so. We recommend `the LazyLoad Plugin
<https://github.com/jeremyharris/cakephp-lazyload>`__

After adding the plugin to your entity, you will be able to do the following::

    $article = $this->Articles->findById($id);

    // The comments property was lazy loaded
    foreach ($article->comments as $comment) {
        echo $comment->body;
    }

Creating Re-usable Code with Traits
===================================

You may find yourself needing the same logic in multiple entity classes. PHP's
traits are a great fit for this. You can put your application's traits in
**src/Model/Entity**. By convention traits in CakePHP are suffixed with
``Trait`` so they can be discernible from classes or interfaces. Traits are
often a good complement to behaviors, allowing you to provide functionality for
the table and entity objects.

For example if we had SoftDeletable plugin, it could provide a trait. This trait
could give methods for marking entities as 'deleted', the method ``softDelete``
could be provided by a trait::

    // SoftDelete/Model/Entity/SoftDeleteTrait.php

    namespace SoftDelete\Model\Entity;

    trait SoftDeleteTrait
    {

        public function softDelete()
        {
            $this->set('deleted', true);
        }

    }

You could then use this trait in your entity class by importing it and including
it::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;
    use SoftDelete\Model\Entity\SoftDeleteTrait;

    class Article extends Entity
    {
        use SoftDeleteTrait;
    }

Converting to Arrays/JSON
=========================

When building APIs, you may often need to convert entities into arrays or JSON
data. CakePHP makes this simple::

    // Get an array.
    // Associations will be converted with toArray() as well.
    $array = $user->toArray();

    // Convert to JSON
    // Associations will be converted with jsonSerialize hook as well.
    $json = json_encode($user);

When converting an entity to an JSON the virtual & hidden field lists are
applied. Entities are recursively converted to JSON as well. This means that if you
eager loaded entities and their associations CakePHP will correctly handle
converting the associated data into the correct format.

Exposing Virtual Properties
---------------------------

By default virtual properties are not exported when converting entities to
arrays or JSON. In order to expose virtual properties you need to make them
visible. When defining your entity class you can provide a list of virtual
properties that should be exposed::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class User extends Entity
    {

        protected $_virtual = ['full_name'];

    }

This list can be modified at runtime using ``virtualProperties``::

    $user->virtualProperties(['full_name', 'is_admin']);

Hiding Properties
-----------------

There are often fields you do not want exported in JSON or array formats. For
example it is often unwise to expose password hashes or account recovery
questions. When defining an entity class, define which properties should be
hidden::

    namespace App\Model\Entity;

    use Cake\ORM\Entity;

    class User extends Entity
    {

        protected $_hidden = ['password'];

    }

This list can be modified at runtime using ``hiddenProperties``::

    $user->hiddenProperties(['password', 'recovery_question']);

Storing Complex Types
=====================

Accessor & Mutator methods on entities are not intended to contain the logic for
serializing and unserializing complex data coming from the database. Refer to
the :ref:`saving-complex-types` section to understand how your application can
store more complex data types like arrays and objects.

.. meta::
    :title lang=en: Entities
    :keywords lang=en: entity, entities, single row, individual record
