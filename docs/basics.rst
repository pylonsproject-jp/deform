.. Basic Usage

基本的な使い方
==============

.. In this chapter, we'll walk through basic usage of Deform to render a
.. form, and capture and validate input.

この章では、フォームをレンダリングしてから、入力を受け取ってバリデーションを
行うまでの、 Deform の基本的な使い方について見ていきましょう。


.. The steps a developer must take to cause a form to be renderered and
.. subsequently be ready to accept form submission input are:

フォームをレンダリングして、それに続くフォーム送信入力を受け取れるように
するために、開発者がしなければならないステップは次のとおりです:


.. - Define a schema

.. - Create a form object.

.. - Assign non-default widgets to fields in the form (optional).

.. - Render the form.

- スキーマを定義する

- フォームオブジェクトを作成する

- フォームのフィールドにデフォルト以外のウィジェットを割り当てる (オプション)

- フォームをレンダリングする


.. Once the form is rendered, a user will interact with the form in his
.. browser, and some point, he will submit it.  When the user submits the
.. form, the data provided by the user will either validate properly, or
.. the form will need to be rerendered with error markers which help to
.. inform the user of which parts need to be filled in "properly" (as
.. defined by the schema).  We allow the user to continue filling in the
.. form, submitting, and revalidating indefinitely.

フォームが表示されると、ユーザはブラウザの中でフォームとやり取りして、
ある時点でそれを送信します。ユーザがフォームを送信すると、ユーザから
提供されたデータは適切にバリデーションされるか、そうでなければエラー
マーカーとともにフォームが再度レンダリングされる必要があるでしょう。
それは、どの部分を (スキーマで定義されているように) 「正しく」再入力する
必要があるかをユーザに通知することをサポートします。ユーザはフォームに
入力して、送信して、再度バリデーションを行うことを無制限に継続すること
ができます。


.. Defining A Schema

スキーマの定義
------------------

.. The first step to using Deform is to create a :term:`schema` which
.. represents the data structure you wish to be captured via a form
.. rendering.  

Deform を使うための第一歩は、フォームのレンダリングを通して受け取ろうとする
データ構造を表わす :term:`schema` を作成することです。


.. For example, let's imagine you want to create a form based roughly on
.. a data structure you'll obtain by reading data from a relational
.. database.  An example of such a data structure might look something
.. like this:

例えば、リレーショナル・データベースから得られるデータ構造におおよそ基づいて
フォームを作成することを想像してください。そのようなデータ構造の一例は
このようなものです:


.. code-block:: python
   :linenos:

   [
   {
    'name':'keith',
    'age':20,
   },
   {
    'name':'fred',
    'age':23,
   },
   ]


.. In other words, the database query we make returns a sequence of
.. *people*; each person is represented by some data.  We need to edit
.. this data.  There won't be many people in this list, so we don't need
.. any sort of paging or batching to make our way through the list; we
.. can display it all on one form page.

言い換えれば、実行されるデータベースクエリは *people* のシーケンスを
返します; それぞれの人は何らかのデータで表わされています。このデータを
編集する必要があります。このリストの中にいる人の数はそれほど多くありません。
そのためリストを辿るのに何らかのページングやバッチは必要ありません;
単一のフォームページにすべて表示することができます。


.. Deform designates a structure akin to the example above as an
.. :term:`appstruct`.  The term "appstruct" is shorthand for "application
.. structure", because it's the kind of high-level structure that an
.. application usually cares about: the data present in an appstruct is
.. useful directly to an application itself.

Deform は、上記の例と類似した構造を :term:`appstruct` として指定しています。
用語 "appstruct" は、「アプリケーション構造」の短縮形です。なぜなら、
それがアプリケーションで通常関心のある高レベルの構造の一種だからです:
appstruct で表されたデータは、アプリケーションそれ自体にとって直接的に
有用です。


.. .. note:: An appstruct differs from other structures that Deform uses
..    (such as :term:`pstruct` and :term:`cstruct` structures): pstructs
..    and cstructs are typically only useful during intermediate parts of
..    the rendering process.

.. note::

   appstruct は、(:term:`pstruct` や :term:`cstruct` 構造のような)
   Deform が使用する他の構造とは異なります: pstruct とcstruct は、
   典型的にはレンダリング処理の間でのみ有用です。


.. Usually, given some appstruct, you can divine a :term:`schema` that
.. would allow you to edit the data related to the appstruct.  Let's
.. define a schema which will attempt to serialize this particular
.. appstruct to a form.  Our application has these requirements of the
.. resulting form:

通常、ある appstruct を与えられると、その appstruct と関係付けられた
データを編集することができる :term:`schema` を推測することができます。
この特定の appstruct をフォームにシリアライズするスキーマを定義しましょう。
このアプリケーションには、生成されるフォームに以下のような要件があります:


.. - It must be possible to add and remove a person.

.. - It must be possible to change any person's name or age after they've
..   been added.

- 人の追加と削除がでなければなりません。

- 人を追加した後で、任意の人の名前あるいは年齢を変更できなければなりません。



.. Here's a schema that will help us meet those requirements:

これらの要件を満たすスキーマはこのようになります:


.. code-block:: python
   :linenos:

   import colander

   class Person(colander.MappingSchema):
       name = colander.SchemaNode(colander.String())
       age = colander.SchemaNode(colander.Integer(),
                                 validator=colander.Range(0, 200))

   class People(colander.SequenceSchema):
       person = Person()

   class Schema(colander.MappingSchema):
       people = People()

   schema = Schema()

       
.. The schemas used by Deform come from a package named :term:`Colander`.  The
.. canonical documentation for Colander exists at
.. http://docs.pylonsproject.org/projects/colander/dev/ .  To compose complex
.. schemas, you'll need to read it to get comfy with the documentation of the
.. default Colander data types.  But for now, we can play it by ear.

Deform が使用するスキーマは、 :term:`Colander` という名前のパッケージ
から来ています。 Colander のための公式のドキュメンテーションは
http://docs.pylonsproject.org/projects/colander/dev/ に存在します。複雑
なスキーマを構成するには、デフォルトの :term:`Colander` データ型の
ドキュメンテーションを読んで慣れておく必要があるでしょう。しかし、今の
ところは、それを即興でやる (play it by ear) ことができます。


.. For ease of reading, we've actually defined *three* schemas above, but
.. we coalesce them all into a single schema instance as ``schema`` in
.. the last step.  A ``People`` schema is a collection of ``Person``
.. schema nodes.  As the result of our definitions, a ``Person``
.. represents:

読んで理解しやすいように、ここでは上記の3つのスキーマを実際に定義
しましたが、最後のステップで ``schema`` としてそれらすべてを単一の
スキーマ・インスタンスに結合しています。 ``People`` スキーマは
``Person`` スキーマノードのコレクションです。この定義の結果、
``Person`` は次のように表現されます:


.. - A ``name``, which must be a string.

.. - An ``age``, which must be deserializable to an integer; after
..   deserialization happens, a validator ensures that the integer is
..   between 0 and 200 inclusive.

- ``name``, これは文字列でなければなりません。

- ``age``, それは整数に逆シリアライズできなければなりません;
  逆シリアライズが起こった後で、バリデータはその整数が 0 以上 200 以下
  (境界含む) であることを保証します。


.. Schema Node Objects

スキーマノード・オブジェクト
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. .. note:: This section repeats and contextualizes the :term:`Colander`
..    documentation about schema nodes in order to prevent you from
..    needing to switch away from this page to another while trying to
..    learn about forms.  But you can also get much the same information
..    at http://docs.pylonsproject.org/projects/colander/dev/

.. note::

   この節では、読者がフォームに関して学習している間このページから別の
   ページに移動する必要がないように、スキーマノードに関する
   :term:`Colander` ドキュメンテーションを文脈に合わせて一部繰り返します。
   しかし、ほとんど同じ情報が
   http://docs.pylonsproject.org/projects/colander/dev/ でも得ることが
   できます。


.. A schema is composed of one or more *schema node* objects, each typically of
.. the class :class:`colander.SchemaNode`, usually in a nested arrangement.
.. Each schema node object has a required *type*, an optional *preparer*
.. for adjusting data after deserialization, an optional
.. *validator* for deserialized prepared data, an optional *default*, an
.. optional *missing*, an optional *title*, an optional *description*,
.. and a slightly less optional *name*.  It also accepts *arbitrary*
.. keyword arguments, which are attached directly as attributes to the
.. node instance.

スキーマは、1つまたは複数の *スキーマノード* ・オブジェクトから構成されます。
それぞれのスキーマノードは、典型的にはクラス
:class:`colander.SchemaNode` のインスタンスで、通常は入れ子状に配置されます。
それぞれのスキーマノード・オブジェクトには、必須の *type*, (逆シリアライズ
後のデータを調節するための) オプションの *preparer*, (prepare 済みのデータを
逆シリアライズするための) オプションの *validator*, オプションの
*default*, オプションの *missing*, オプションの *title*, オプションの
*description*,  ほとんど必須の (slightly less optional) *name* があります。
さらに、それは *任意の* キーワード引数を受け付けます。それらは属性として
ノードインスタンスに直接設定されます。


.. The *type* of a schema node indicates its data type (such as
.. :class:`colander.Int` or :class:`colander.String`).

スキーマノードの *type* は、そのデータ型を示します (例えば
:class:`colander.Int` や :class:`colander.String` などです)。


.. The *preparer* of a schema node is called after
.. deserialization but before validation; it prepares a deserialized
.. value for validation. Examples would be to prepend schemes that may be
.. missing on url values or to filter html provided by a rich text
.. editor. A preparer is not called during serialization, only during
.. deserialization.

スキーマノードの *preparer* は、逆シリアライズの後、バリデーションの前
に呼ばれます; それはバリデーションのために逆シリアライズされた値を用意
します。複数 url 値に省略可能のスキーマを追加すること、リッチテキストエディタ
から提供される html をフィルターすること、などが例として挙げられるでしょう。
preparer はシリアライズ中には呼ばれません。逆シリアライズ中にのみ呼ばれます。


.. The *validator* of a schema node is called after deserialization and
.. preparation ; it makes sure the value matches a constraint.  An example of
.. such a validator is provided in the schema above:
.. ``validator=colander.Range(0, 200)``.  A validator is not called after
.. schema node serialization, only after node deserialization.

スキーマノードの *validator* は逆シリアライズと prepare の後に呼ばれます;
それは、値が制約と一致することを確認します。そのようなバリデーションの
例は上記のスキーマの中に示されています:
``validator=colander.Range(0, 200)`` 。バリデーションはスキーマノードの
シリアライズの後には呼ばれません。ノードの逆シリアライズの後にのみ
呼ばれます。


.. The *default* of a schema node indicates the value to be serialized if
.. a value for the schema node is not found in the input data during
.. serialization.  It should be the deserialized representation.

スキーマノードの *default* は、シリアライズ中に入力データの中に
スキーマノードに対する値が見つからない場合にシリアライズされる値を示します。
それは逆シリアライズ形式で表現されます。


.. The *missing* of a schema node indicates the value to be deserialized
.. if a value for the schema node is not found in the input data during
.. deserialization.  It should be the deserialized representation.  If a
.. schema node does not have a ``missing`` value, a
.. :exc:`colander.Invalid` exception will be raised if the data structure
.. being deserialized does not contain a matching value.

スキーマノードの *missing* は、逆シリアライズ中に入力データの中に
スキーマノードに対する値が見つからない場合に、逆シリアライズされる値を
指定します。それは逆シリアライズ形式で表現されます。スキーマノードが
``missing`` 値を持っていない場合、逆シリアライズされたデータ構造が
一致する値を含んでいなければ :exc:`colander.Invalid` 例外が上げられます。


.. The *name* of a schema node is used to relate schema nodes to each
.. other.  It is also used as the title if a title is not provided.

スキーマノードの *name* は、複数のスキーマノードを互いに関連付けるため
に使用されます。さらに、タイトルが提供されない場合にはタイトルとしても
使用されます。


.. The *title* of a schema node is metadata about a schema node.  It
.. shows up in the legend above the form field(s) related to the schema
.. node.  By default, it is a capitalization of the *name*.

スキーマノードの *title* は、スキーマノードに関するメタデータです。
それは、スキーマノードに関連するフォームフィールド上部の legend に
表示されます。デフォルトでは、これは *name* の先頭を大文字にした文字列です。


.. The *description* of a schema node is metadata about a schema node.
.. It shows up as a tooltip when someone hovers over the form control(s)
.. related to a :term:`field`.  By default, it is empty.

スキーマノードの *description* は、スキーマノードに関するメタデータです。
:term:`field` と関連するフォームコントロールの上にマウスカーソルを
合わせたときにツールチップとして表示されます。デフォルトでは空文字列です。


.. The name of a schema node that is introduced as a class-level
.. attribute of a :class:`colander.MappingSchema`,
.. :class:`colander.TupleSchema` or a :class:`colander.SequenceSchema` is
.. its class attribute name.  For example:

:class:`colander.MappingSchema`, :class:`colander.TupleSchema`,
:class:`colander.SequenceSchema` のクラスレベル属性として定義されている
スキーマノードの名前は、そのクラス属性の名前です。例えば:


.. code-block:: python
   :linenos:

   import colander

   class Phone(colander.MappingSchema):
       location = colander.SchemaNode(colander.String(), 
                                      validator=colander.OneOf(['home','work']))
       number = colander.SchemaNode(colander.String())


.. The name of the schema node defined via ``location =
.. colander.SchemaNode(..)`` within the schema above is ``location``.
.. The title of the same schema node is ``Location``.

上記のスキーマ中で ``location = colander.SchemaNode(..)``
として定義されたスキーマノードの名前は ``location`` です。
同じスキーマノードのタイトルは ``Location`` です。


.. Schema Objects

スキーマ・オブジェクト
~~~~~~~~~~~~~~~~~~~~~~

.. In the examples above, if you've been paying attention, you'll have
.. noticed that we're defining classes which subclass from
.. :class:`colander.MappingSchema`, and :class:`colander.SequenceSchema`.
.. It's turtles all the way down: the result of creating an instance of
.. any of :class:`colander.MappingSchema`, :class:`colander.TupleSchema`
.. or :class:`colander.SequenceSchema` object is *also* a
.. :class:`colander.SchemaNode` object.

上記の例を注意深く見ていれば、 :class:`colander.MappingSchema` と
:class:`colander.SequenceSchema` のサブクラスとしてクラスを定義している
ことに気がついたかもしれません。それは、 "It's turtles all the way down"
です (訳注: 親亀の上に子亀、孫亀が無数に乗っている様子。
上から下まですべて同じものでできていること):
:class:`colander.MappingSchema`, :class:`colander.TupleSchema`
:class:`colander.SequenceSchema` オブジェクトのいずれかのインスタンスを
生成した結果も、 :class:`colander.SchemaNode` オブジェクトになります。


.. Instantiating a :class:`colander.MappingSchema` creates a schema node
.. which has a *type* value of :class:`colander.Mapping`.

:class:`colander.MappingSchema` をインスタンス化すると、
*type* 値 :class:`colander.Mapping` を持つスキーマノードが生成されます。


.. Instantiating a :class:`colander.TupleSchema` creates a schema node
.. which has a *type* value of :class:`colander.Tuple`.

:class:`colander.TupleSchema` をインスタンス化すると、
*type* 値 :class:`colander.Tuple` を持つスキーマノードが生成されます。


.. Instantiating a :class:`colander.SequenceSchema` creates a schema node
.. which has a *type* value of :class:`colander.Sequence`.

:class:`colander.SequenceSchema` をインスタンス化すると、
*type* 値 :class:`colander.Sequence` を持つスキーマノードが生成されます。


.. Creating Schemas Without Using a Class Statement (Imperatively)

クラス構文を使わないスキーマの作成 (命令的)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. See
.. http://docs.pylonsproject.org/projects/colander/dev/basics.html#defining-a-schema-imperatively
.. for information about how to create schemas without using a ``class``
.. statement.

``class`` 構文を使わずにスキーマを作成する方法については、
http://docs.pylonsproject.org/projects/colander/dev/basics.html#defining-a-schema-imperatively
を参照してください。


.. Creating a schema with or without ``class`` statements is purely a
.. style decision; the outcome of creating a schema without ``class``
.. statements is the same as creating one with ``class`` statements.

スキーマを作成するのに ``class`` 構文を使うか使わないかは、純粋に
スタイル上の意思決定です; ``class`` 構文を使わずにスキーマを作成した結果は
``class`` 構文を使って作成した結果と同じです。


.. Rendering a Form

フォームのレンダリング
----------------------

.. Earlier we defined a schema:

以前このようなスキーマを定義しました:


.. code-block:: python
   :linenos:

   import colander

   class Person(colander.MappingSchema):
       name = colander.SchemaNode(colander.String())
       age = colander.SchemaNode(colander.Integer(),
                                 validator=colander.Range(0, 200))

   class People(colander.SequenceSchema):
       person = Person()

   class Schema(colander.MappingSchema):
       people = People()

   schema = Schema()


.. Let's now use this schema to create, render and validate a form.

今度はこのスキーマを使ってフォームを生成して、レンダリングと
バリデーションをしてみましょう。


.. Creating a Form Object

.. _creating_a_form:

フォームオブジェクトの生成
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. To create a form object, we do this:

フォームオブジェクトを生成するために、このようにします:


.. code-block:: python
   :linenos:

   from deform import Form
   myform = Form(schema, buttons=('submit',))


.. We used the ``schema`` object (an instance of
.. :class:`colander.MappingSchema`) we created in the previous section as
.. the first positional parameter to the :class:`deform.Form` class; we
.. passed the value ``('submit',)`` as the value of the ``buttons``
.. keyword argument.  This will cause a single ``submit`` input element
.. labeled ``Submit`` to be injected at the bottom of the form rendering.
.. We chose to pass in the button names as a sequence of strings, but we
.. could have also passed a sequence of instances of the
.. :class:`deform.Button` class.  Either is permissible.

前の節で作成した ``schema`` オブジェクト (:class:`colander.MappingSchema`
のインスタンス) を、 :class:`deform.Form` クラスの最初の位置パラメータとして
使用しました; ``buttons`` キーワード引数の値として値 ``('submit',)``
を渡しました。これによって、フォームのレンダリング結果の下部に ``Submit``
というラベルの付いた単一の ``submit`` 入力要素が差し込まれます。ここで
はボタン名を文字列のシーケンスとして渡すようにしましたが、
:class:`deform.Button` クラスのインスタンスのシーケンスを渡すこともできます。
いずれかが許容されます。


.. Note that the first positional argument to :class:`deform.Form` must
.. be a schema node representing a *mapping* object (a structure which
.. maps a key to a value).  We satisfied this constraint above by passing
.. our ``schema`` object, which we obtained via the
.. :class:`colander.MappingSchema` constructor, as the ``schema``
.. argument to the :class:`deform.Form` constructor

:class:`deform.Form` の最初の位置引数が *mapping* オブジェクト
(キーを値に写像する構造) を表わすスキーマノードでなければならないことに
注意してください。上記の例では、 (:class:`colander.MappingSchema`
コンストラクタによって得られた) ``schema`` オブジェクトを
:class:`deform.Form` コンストラクタに ``schema`` 引数として渡すことで
この制約を満たしました。


.. Although different kinds of schema nodes can be present in a schema
.. used by a Deform :class:`deform.Form` instance, a form instance cannot
.. deal with a schema node representing a sequence, a tuple schema, a
.. string, an integer, etc. as the value of its ``schema`` parameter;
.. only a schema node representing a mapping is permissible.  This
.. typically means that the object passed as the ``schema`` argument to a
.. :class:`deform.Form` constructor must be obtained as the result of
.. using the :class:`colander.MappingSchema` constructor (or the
.. equivalent imperative spelling).

Deform の :class:`deform.Form` インスタンスによって使用されるスキーマの
中に異なる種類のスキーマノードが存在することは可能ですが、フォーム
インスタンスはその ``schema`` パラメータの値としてシーケンス、タプルスキーマ、
文字列、整数などを表わすスキーマノードを扱うことができません; マッピングを
表わすスキーマノードだけが可能です。これは、典型的には
:class:`deform.Form` コンストラクタに対して ``schema`` 引数として渡された
オブジェクトが、 :class:`colander.MappingSchema` コンストラクタ (あるいは
等価な命令的なコマンド) を使用した結果として得られなければならないことを
意味します。


.. Rendering the Form

フォームのレンダリング
~~~~~~~~~~~~~~~~~~~~~~

.. Once we've created a Form object, we can render it without issue by
.. calling the :meth:`deform.Field.render` method: the
.. :class:`deform.Form` class is a subclass of the :class:`deform.Field`
.. class, so this method is available to a :class:`deform.Form` instance.

フォームオブジェクトを生成したら、 :meth:`deform.Field.render` メソッド
を呼ぶことで、問題なくレンダリングすることができます:
:class:`deform.Form` クラスは :class:`deform.Field` クラスのサブクラスです。
したがって、このメソッドは :class:`deform.Form` インスタンスでも利用可能です。


.. If we wanted to render an "add" form (a form without initial
.. data), we'd just omit the appstruct while calling
.. :meth:`deform.Field.render`.

"add" フォーム (初期データのないフォーム) をレンダリングしたいなら、
:meth:`deform.Field.render` を呼ぶ際に単に appstruct を省略します。


.. code-block:: python

   form = myform.render()


.. If we have some existing data already that we'd like to edit using the
.. form (the form is an "edit form" as opposed to an "add form").  That
.. data might look like this:

既にある既存のデータを持っている場合、フォームを使ってそのデータを編集したいと
思うでしょう (そのフォームは "add フォーム" に対して "edit フォーム" です)。
データはこんな感じになります:


.. code-block:: python
   :linenos:

    appstruct = [
        {
            'name':'keith',
            'age':20,
            },
        {
            'name':'fred',
            'age':23,
            },
        ]


.. To inject it into the serialized form as the data to be edited, we'd
.. pass it in to the :meth:`deform.Field.render` method to get a form
.. rendering:

これを編集すべきデータとしてシリアライズされたフォームに含めるために、
そのデータを :meth:`deform.Field.render` メソッドに渡してフォームを
レンダリングします:


.. code-block:: python

   form = myform.render(appstruct)


.. If, finally, instead we wanted to render a "read-only" variant of an edit form
.. using the same appstruct, we'd pass the ``readonly`` flag as ``True``
.. to the :meth:`deform.Field.render` method.

もし最後のところで、代わりに同じ appstruct を使用する edit フォームの
「読み取り専用」の異なるバリエーションをレンダリングしたければ、 ``readonly``
フラグを ``True`` として :meth:`deform.Field.render` メソッドに渡します。


.. code-block:: python

   form = myform.render(appstruct, readonly=True)


.. This would cause a page to be rendered in a crude form without any
.. form controls, so the user it's presented to cannot edit it.

これはフォームコントロールのない未処理の (crude) フォームでページを
レンダリングします。そのため、このフォームを提示されたユーザは、フォームを
編集できません。


.. Once any of the above statements runs, the ``form`` variable is now a
.. Unicode object containing an HTML rendering of the edit form, useful
.. for serving out to a browser.  The root tag of the rendering will be
.. the ``<form>`` tag representing this form (or at least a ``<div>`` tag
.. that contains this form tag), so the application using it will need to
.. wrap it in HTML ``<html>`` and ``<body>`` tags as necessary.  It will
.. need to be inserted as "structure" without any HTML escaping.

上記のいずれかの文が実行されれば、 ``form`` 変数は edit フォームをレンダリング
した HTML を含む Unicode オブジェクトになります (これはブラウザに対して
ページを返すのに便利です)。レンダリングの root タグは、このフォームを表わす
``<form>`` タグ (あるいは少なくとも form タグを含む ``<div>`` タグ)
になります。したがって、それを使用するアプリケーションは、必要に応じて
HTML の ``<html>`` および ``<body>`` タグでそれをラップする必要があります。
HTML エスケープなしで "structure" として挿入される必要があるでしょう。


.. Serving up the Rendered Form

.. _serving_up_the_rendered_form:

レンダリングされたフォームを返す
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. We now have an HTML rendering of a form as the variable named
.. ``form``.  But before we can serve it up successfully to a browser
.. user, we have to make sure that static resources used by Deform can be
.. resolved properly. Some Deform widgets (including at least one we've
.. implied in our sample schema) require access to static resources such
.. as images via HTTP.

ここまでで、フォームをレンダリングする HTML が ``form`` という名前の
変数として手に入りました。しかし、ブラウザユーザに対して無事にそれを提供
できるようになる前に、 Deform による静的 asset が適切に解決されることを
確かめなければなりません。いくつかの Deform ウィジェット (少なくともサンプル
スキーマの中で1つ使われています) は、HTTPによって画像のような静的 asset
へのアクセスを要求します。


.. For these widgets to work properly, we'll need to arrange that files
.. in the directory named ``static`` within the :mod:`deform` package can
.. be resolved via a URL which lives at the same hostname and port number
.. as the page which serves up the form itself.  For example, the URL
.. ``/static/css/form.css`` should be willing to return the
.. ``form.css`` CSS file in the ``static/css`` directory in the
.. :mod:`deform` package as ``text/css`` content and return 
.. ``/static/scripts/deform.js`` as``text/javascript`` content.  
.. How you arrange to do this is dependent on
.. your web framework.  It's done in :mod:`pyramid` imperative
.. configuration via:

これらのウィジェットが適切に動作するためには、 :mod:`deform` パッケージ内の
``static`` という名前のディレクトリ内のファイルが、フォーム自身を出力する
ページと同じホスト名およびポート番号に存在する URL に解決できるように
調整する必要があります。例えば、URL ``/static/css/form.css`` は
:mod:`deform` パッケージの ``static/css`` ディレクトリ内にある
``form.css`` CSS ファイルを ``text/css`` コンテンツとして返す必要があり、
同様に ``/static/scripts/deform.js`` を ``text/javascript`` コンテンツ
として返す必要があります。これをどのように行うかは使用しているウェブ
フレームワークに依存します。 :mod:`pyramid` の命令的な設定の場合は、
以下のように行われます:


.. code-block:: python

  config = Configurator(...)
  ...
  config.add_static_view('static', 'deform:static')
  ...


.. Your web framework will use a different mechanism to offer up static
.. files.

他のウェブフレームワークでは、静的ファイルを返すのに異なるメカニズムを
使用するでしょう。


.. Some of the more important files in the set of JavaScript, CSS files,
.. and images present in the ``static`` directory of the :mod:`deform`
.. package are the following:

:mod:`deform` パッケージの ``static`` ディレクトリにある JavaScript と
CSS ファイルと画像の中で、いくつかの重要なファイルは以下の通りです:


.. ``static/scripts/jquery-1.4.2.min.js``
..   A local copy of the JQuery javascript library, used by widgets and
..   other JavaScript files.

``static/scripts/jquery-1.4.2.min.js``
  jQuery javascript ライブラリのローカルコピー。ウィジェットと他の
  JavaScript ファイルによって使用されます。


.. ``static/scripts/deform.js``
..   A JavaScript library which should be loaded by any template which
..   injects a rendered Deform form.

``static/scripts/deform.js``
  Deform フォームをレンダリングするすべてのテンプレートでロードしなければ
  ならない JavaScript ライブラリ。


.. ``static/css/form.css``
..   CSS related to form element renderings.

``static/css/form.css``
  フォーム要素のレンダリングに関係する CSS 。


.. Each of these libraries should be included in the ``<head>`` tag of a
.. page which renders a Deform form, e.g.:

これらのライブラリそれぞれは、 Deform フォームをレンダリングするページの
``<head>`` タグに含まれているはずです。例えば:


.. code-block:: xml
   :linenos:

   <head>
     <title>
       Deform Demo Site
     </title>
     <!-- Meta Tags -->
     <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
     <!-- CSS -->
     <link rel="stylesheet" href="/static/css/form.css" type="text/css" />
     <!-- JavaScript -->
     <script type="text/javascript"
             src="/static/scripts/jquery-1.4.2.min.js"></script> 
     <script type="text/javascript"
             src="/static/scripts/deform.js"></script>
   </head>


.. The :meth:`deform.field.get_widget_resources` method can be used to
.. tell you which ``static`` directory-relative files are required by a
.. particular form rendering, so that you can inject only the ones
.. necessary into the page rendering.

:meth:`deform.Field.get_widget_resources` メソッドは、 ``static``
ディレクトリからの相対パスでどのファイルが特定のフォームレンダリングに
よって必要とされるかを知るために使用することができます。
その結果、ページのレンダリングに必要なものだけを含めることができます。


.. The JavaScript function ``deform.load()`` *must* be called by the HTML
.. page (usually in a script tag near the end of the page, ala
.. ``<script..>deform.load()</script>``) which renders a Deform form in
.. order for widgets which use JavaScript to do proper event and behavior
.. binding.  If this function is not called, built-in widgets which use
.. JavaScript will not function properly.  For example, you might include
.. this within the body of the rendered page near its end:

JavaScript を使用するウィジェットが適切なイベントと振る舞いのバインディング
を行うために、 Deform フォームをレンダリングする HTML ページで
JavaScript 関数 ``deform.load()`` を (通常はページの最後の方で
``<script..>deform.load()</script>`` のようにして script タグの中で)
*必ず* 呼び出す必要があります。この関数が呼ばれなければ、 JavaScript を
使用する内蔵のウィジェットは正常に機能しません。例えば、レンダリング
されたページの body 内の最後の方に、これがインクルードされます:


.. code-block:: xml
   :linenos:

   <script type="text/javascript">
      deform.load()
   </script>


.. As above, the head should also contain a ``<meta>`` tag which names a
.. ``utf-8`` charset in a ``Content-Type`` http-equiv.  This is a sane
.. setting for most systems.

上記のように、 head はさらに ``Content-Type`` http-equiv に
``utf-8`` charset を指定する ``<meta>`` タグを含む必要があります。
これは、ほとんどのシステムのための良識的な設定です。


.. Validating a Form Submission

フォーム送信のバリデーション
----------------------------

.. Once the user seen the form and has chewed on its inputs a bit, he
.. will eventually submit the form.  When he submits it, the logic you
.. use to deal with the form validation must do a few things:

フォームが表示され、ユーザがある程度の時間をかけて (chew a bit) 入力を
終えたら、ユーザはやがてフォームを送信するでしょう。フォームが送信された
時に、フォームバリデーションを扱うために使用されるロジックはいくつかの
ことをしなければなりません:


.. - It must detect that a submit button was clicked.

- submit ボタンがクリックされたことを検知する。


.. - It must obtain the list of :term:`form controls` from the form POST
..   data.

- フォーム POST データから :term:`form controls` のリストを得る。


.. - It must call the :meth:`deform.Form.validate` method with the list
..   of form controls.

- フォームコントロールのリストと共に :meth:`deform.Form.validate` メソッド
  を呼び出す。


.. - It must be willing to catch a :exc:`deform.ValidationFailure`
..   exception and rerender the form if there were validation errors.

- バリデーションエラーがあった場合、 :exc:`deform.ValidationFailure`
  例外を捕捉して、再度フォームをレンダリングする。


.. For example, using the :term:`WebOb` API for the above tasks, and the
.. ``form`` object we created earlier, such a dance might look like this:

例えば、上記のタスクのために :term:`WebOb` API を使い、以前に作成した
``form`` オブジェクトを使うと、そのようなダンスはこんな感じになります:


.. code-block:: python
   :linenos:

   if 'submit' in request.POST: # detect that the submit button was clicked

       controls = request.POST.items() # get the form controls

       try:
           appstruct = myform.validate(controls)  # call validate
       except ValidationFailure, e: # catch the exception
           return {'form':e.render()} # re-render the form with an exception

       # the form submission succeeded, we have the data
       return {'form':None, 'appstruct':appstruct}


.. The above set of statements is the sort of logic every web app that
.. uses Deform must do.  If the validation stage does not fail, a
.. variable named ``appstruct`` will exist with the data serialized from
.. the form to be used in your application.  Otherwise the form will be
.. rerendered.

上記の一連の文は Deform を使用するすべてのウェブアプリが行わなければ
ならない種類のロジックです。バリデーション段階で失敗しなければ、フォーム
からシリアライズされたデータで ``appstruct`` という名前の変数が作られ、
アプリケーションの中で使用できるようになります。そうでなければ、フォーム
は再度レンダリングされます。


.. Note that by default, when any form submit button is clicked, the form
.. will send a post request to the same URL which rendered the form.
.. This can be changed by passing a different ``action`` to the
.. :class:`deform.Form` constructor.

デフォルトでは、あるフォームの送信ボタンがクリックされた時に、その
フォームと同じ URL に post リクエストが送られることに注意してください。
これは :class:`deform.Form` コンストラクタに異なる ``action`` を渡す
ことで変更できます。


.. Seeing it In Action

動作中のデモ
-------------------

.. To see an "add form" in action that follows the schema in this
.. chapter, visit `http://deformdemo.repoze.org/sequence_of_mappings/
.. <http://deformdemo.repoze.org/sequence_of_mappings/>`_.

この章のスキーマに従う "add フォーム" が実際に動作している状態を見るため、
`http://deformdemo.repoze.org/sequence_of_mappings/
<http://deformdemo.repoze.org/sequence_of_mappings/>`_
を訪れてください。


.. To see a "readonly edit form" in action that follows the schema in
.. this chapter, visit
.. `http://deformdemo.repoze.org/readonly_sequence_of_mappings/
.. <http://deformdemo.repoze.org/readonly_sequence_of_mappings/>`_

この章のスキーマに従う "読み取り専用 edit フォーム" が実際に動作している
状態を見るため、
`http://deformdemo.repoze.org/readonly_sequence_of_mappings/
<http://deformdemo.repoze.org/readonly_sequence_of_mappings/>`_
を訪れてください。


.. The application at http://deformdemo.repoze.org is a :mod:`pyramid`
.. application which demonstrates most of the features of Deform,
.. including most of the widget and data types available for use within
.. an application that uses Deform.  

http://deformdemo.repoze.org にあるアプリケーションは、Deform の
ほとんどの特徴をデモする :mod:`pyramid` アプリケーションです。Deform を
使用するアプリケーション内で使用することのできるほとんどのウィジェットと
データ型を含んでいます。
