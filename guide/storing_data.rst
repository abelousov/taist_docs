Storing data
============

Taist allows to store data in different ways:

User data locally
-----------------
``taistApi.localStorage`` allows to store data using ``localStorage`` - it's just a wrapper around ``localStorage`` that allows storing objects:

* ``set(key, value)`` - stores value in ``localStorage``; can store objects
* ``get(key)`` - gets previously stored value;
* ``delete(key)`` - deletes value

User and company data on server
-------------------------------
Data can also be stored on Taist server: 

* user-level data is processed via ``userData``
* company-level data (shared between all company members) is processed via ``companyData``
* addon-level data (shared between all company members in single addon) is processed via ``addonData``

``userData`` and ``companyData`` have the same basic interface:

* ``set(key, value, callback)`` - stores ``value`` on Taist server
* ``get(key, callback)`` - gets previously stored data
* ``delete(key, callback)`` - deletes ``value``; throws an error if there is no data stored with given ``key``

**params**:
* ``key``, ``value`` - scalar value or JSON-like object
* ``function callback(error, result)`` - ``result`` is set only for ``get`` method

Company data: working with parts of values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``companyData`` also allows to change or retrieve parts of data:

* ``getPart(key, partKey, callback)`` - gets field ``partKey`` of an object previously stored with key = ``key``
* ``setPart(key, partKey, value, callback)`` - sets field ``partKey`` of an object previously stored with key = ``key``

Example:

.. code-block:: javascript

  taistApi.companyData.set('foo', {bar: 1, baz: '2'}, function(setErr){
    taistApi.companyData.getPart('foo', 'bar', function(getPartErr, part){

      taistApi.log('foo.bar =', part);

      taistApi.companyData.setPart('foo', 'baz', {newBaz: true}, function(setPartErr){

        taistApi.companyData.get('foo', function(getErr, whole){

          taistApi.log('foo =', whole);
        })
      })
    })
  })

  /*
  output:

  foo.bar = 1
  foo = {"bar":1,"baz":{"newBaz":true}}
  */

Model data: working with repository of models
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``models`` allows to define and get repository of stored on server models:

* ``define(modelName, schema)`` - defines model ``modelName`` with schema ``schema``. Schema implements types ``String``, ``Number``, ``Object``, ``Boolean``
* ``getRepository(modelName)`` - returns repository of models stored as ``modelName`` that can be used to retrieve, save or delete ``modelName`` models

Example:

.. code-block:: javascript

  taistApi.models.define('myModel', { name: 'String', age: 'Number' })

  var repository = taistApi.models.getRepository('myModel')

  /*
  output:

  repository = {getAll: function, create: function, save: function, delete: function}
  */

``repository`` allows to add or retrieve (all or filtered by criteria) models, stored on server:

All ``repository`` methods except ``create`` is asynchronous and return promise object.

* ``create(data)`` - creates ``model`` object. Then ``model`` can be added to repository using ``save``. Generates unique id if it's not passed in ``data``
* ``save(model)`` - add ``model`` to ``repository`` and save it to server. Saved ``model`` can be retrieved by ``getAll`` and ``find``. Saves only fields that defined in ``schema``
* ``getAll()`` - get all models stored in ``repository``
* ``find(criteria)`` - find ``model`` by ``criteria``. ``Criteria`` is mongo-like find query object
* ``delete(model)`` - delete ``model`` from repository. ``Model`` will be immediately and forever deleted from server.

Example:

.. code-block:: javascript

  taistApi.models.define('myModel', { name: 'String', age: 'Number' });

  var repository = taistApi.models.getRepository('myModel');

  taistApi.log('repository =', repository);
  
  var model = repository.create({ name: 'Alex', age: '16', notInSchema: 'willNotBeSaved' });
  
  taistApi.log('model =', model);
  
  repository.save(model)
      .then(function() {
          return repository.getAll()
      })
      .then(function(savedModels) {
          taistApi.log('savedModels =', savedModels);
  
          repository.save(repository.create({ name: 'Kate', age: '17' }))
      })
      .then(function() {
          return repository.find({ name: 'Alex' })
      })
      .then(function(filteredModels) {
          taistApi.log('filteredModels =', filteredModels);
  
          return repository.delete(filteredModels)
      })
      .then(function() {
          return repository.getAll()
      })
      .then(function(modelsAfterDelition) {
          taistApi.log('modelsAfterDelition =', modelsAfterDelition);
      });


  /*
  output:

  repository = {getAll: function, create: function, save: function, delete: function}

  model = {name: 'Alex', age: '16', undefinedField: 'undefined', id: '21EC2020-3AEA-4069-A2DD-08002B30309D'}

  savedModels = [{name: 'Alex', age: '16', id: '21EC2020-3AEA-4069-A2DD-08002B30309D'}]

  filteredModels = [{name: 'Alex', age: '16', id: '21EC2020-3AEA-4069-A2DD-08002B30309D'}]

  modelsAfterDelition = [{name: 'Kate', age: '17', id: 'D790D359-AB3D-4657-A864-FA89FACB3E99'}]


  */
