Usage
=====

Config
------

- /api/conf/graphql.php
- Models (_plural_ for multiple records and _singular_ for single record)

Query
-----

Selection
^^^^^^^^^

Basic
"""""

.. code-block:: console

  query {
    frames {
      idFrame
      name
    }
  }

Alias
"""""

.. code-block:: console

  query {
    LexicalUnit:lus {
        luName: name
    }    
  }


Flattened association 
"""""""""""""""""""""

.. code-block:: console

  query {
    frames {
        name
        lu(field: "lus.name")
    }
  }


Nested association
""""""""""""""""""

.. code-block:: console

  query {
    frames {
        name
        fes {
            name
        }
    }
  }


Expression
""""""""""

.. code-block:: console

  query {
    frames {
        fe (expr: "concat(name, ' - ', fes.name)")
    }
  }


Operators
---------

Id
^^

.. code-block:: console

  query {
    frames (id: 1) {
         idFrame
         name
    }
  }


Limit & Offset
^^^^^^^^^^^^^^

.. code-block:: console				

  query {
    frames(limit: 10, offset: 200) {
        idFrame
        name
    }
  }


Order By
^^^^^^^^

.. code-block:: console

  query {
    lus (order_by: [
            {asc: "frame.name"}
            {desc: "name"}
        ], 
        limit: 5
    ){
        frame (field: "frame.name")
        name
    }
  }


Group
^^^^^

.. code-block:: console

  query {
    lus(
        group_by: ["idLanguage"]
    ){
        idLanguage
      	cnt(expr: "count(idLU)")
    }
  }


Forced Join
^^^^^^^^^^^

LEFT, RIGHT, INNER (in uppercase)
"""""""""""""""""""""""""""""""""

.. code-block:: console

  query {
    frames(
        join: [{LEFT: "lus"}]
        where: {
          __condition: [
            {field:"lus.idLU", is_null: true}
          ]
        }    
    ) {
        name
    }
  }


Conditions
----------

Generic operators
^^^^^^^^^^^^^^^^^

- eq : =
- neq : <>
- gt : >
- lt  : <
- gte : >=
- lte : <=
- in : IN
- nin : NOT IN
- {is_null: true} : IS NULL
- {is_null: false} : IS NOT NULL 
 
Textual operators
^^^^^^^^^^^^^^^^^

- like : LIKE
- nlike : NOT LIKE
- starts_with : string starts with 
- regex : RLIKE

.. code-block:: console

  query {
    lus (
        where: {
            name: {like: "%action%"}, 
            idLU: {gt: 1800}, 
            idLanguage: {in: [1,2]}
    }) {
        name
        idLanguage
    }
  }


Insert
------

Single
^^^^^^

.. code-block:: console

  mutation {
    insert_colors (
        object: {
            name: "sample1",
            rgbFg: "000000",
            rgbBg: "000000"
        }
    ) {
        idColor
        name
    }
  }

Multiple
^^^^^^^^ 

.. code-block:: console

  mutation {
    insert_colors (
        objects: [{
            name: "sample1",
            rgbFg: "000000",
            rgbBg: "000000"
         },{
            name: "sample2",
            rgbFg: "000000",
            rgbBg: "000000"
         },{
            name: "sample3",
            rgbFg: "000000",
            rgbBg: "000000"
         }]
    ) {
        idColor
        name
    }
  }


Update
------

Object
^^^^^^

.. code-block:: console

  mutation {
    update_colors (
         id: 116,
         set: {
            name: "sample4",
            rgbFg: "000000",
            rgbBg: "000000"
         }
    ) {
        idColor
        name
    }
  }


Multiple
^^^^^^^^ 

.. code-block:: console

  mutation {
    update_colors(
        where: {name: {like: "sample%"}}
        set: {name: "testcolor"}
    ) {
        idColor
        name
    }
  }


Delete
------

Object
^^^^^^

delete mutations only accept \"id\" operator

.. code-block:: console

  mutation {
    delete_colors(
      id:119
    )
  }


Services
--------

name: service_method([parameters])

.. code-block:: console

  mutation {
    service_AuthUser_registerLogin(userInfo: $userInfo) {
        result
    }
  }


Advanced
--------

Subquery - Query
^^^^^^^^^^^^^^^^ 

.. code-block:: console

  query {
    colors(
        __condition:[
            {name: {startswith: "blue"}}
        ]
    ) {
        idColor
    }
    
    fes(
		distinct: true
        where: {idColor: {in:{ __subquery: "colors" field: "idColor"}}}
    ) {
        name
    }
  }


Fragment
^^^^^^^^ 

.. code-block:: console

  fragment FrameInfo on Frames {
        name
  }

  query {
    frames {
        ...FrameInfo
   }
  }


Examples
--------

.. code-block:: console

  query listDocuments($corpus: String, $document: String, $ids: Array, $idLanguage: Int) {
    documents (
      order_by: [
         {asc: "corpusName"}
         {asc: "name"}
      ]
      where: {
         idDocument: {in:$ids}
         name: {starts_with:$document}
         idLanguage: {eq:$idLanguage}
         __condition: [
            {field:"corpus.name" starts_with:$corpus}
            {field:"corpus.idLanguage" starts_with:$idLanguage}
         ]
      }
    ){
      idDocument
      name
      idCorpus
      corpusName(field:"corpus.name")
    }
  }


.. code-block:: console

  query total {
    imagemm (
      database: "charon"
      offset: 0
      limit: 0
    ){
      idImageMM
    }
    __total (query: "imagemm")
  }

.. code-block:: console

  query listAll ($offset:Int $limit:Int) {
    objectmm (
      database: "charon"
      order_by: [
        {asc: "idObjectMM"}
      ]
      offset: $offset
      limit: $limit
    ){
      idObjectMM
      name
      startFrame
      endFrame
      startTime
      endTime
      status
      origin
      idDocumentMM
      idFrameElement
      idFlickr30k
      idImageMM
      idLemma
      idLU
    }
  }

