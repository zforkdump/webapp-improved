.. _caching_toplevel:

========
Caching
========

Any template or component can be cached using the ``cache``
argument to the ``<%page>``, ``<%def>`` or ``<%block>`` directives:

.. sourcecode:: mako

    <%page cached="True"/>
 
    template text
 
The above template, after being executed the first time, will
store its content within a cache that by default is scoped
within memory. Subsequent calls to the template's :meth:`~.Template.render`
method will return content directly from the cache. When the
:class:`.Template` object itself falls out of scope, its corresponding
cache is garbage collected along with the template.

Caching requires that the ``beaker`` package be installed on the
system.

The caching flag and all its options can be used with the
``<%def>`` tag.

.. sourcecode:: mako

    <%def name="mycomp" cached="True" cache_timeout="30" cache_type="memory">
        other text
    </%def>

... and equivalently with the ``<%block>`` tag, anonymous or named:

.. sourcecode:: mako

    <%block cached="True" cache_timeout="30" cache_type="memory">
        other text
    </%block>

Cache arguments
================

The various cache arguments are cascaded from their default
values, to the arguments specified programmatically to the
:class:`.Template` or its originating :class:`.TemplateLookup`, then to those
defined in the ``<%page>`` tag of an individual template, and
finally to an individual ``<%def>`` tag within the template. This
means you can define, for example, a cache type of ``dbm`` on your
:class:`.TemplateLookup`, a cache timeout of 60 seconds in a particular
template's ``<%page>`` tag, and within one of that template's
``<%def>`` tags ``cache=True``, and that one particular def will
then cache its data using a ``dbm`` cache and a data timeout of 60
seconds.

The options available are:

* ``cached="False|True"`` - turn caching on
* ``cache_timeout`` - number of seconds in which to invalidate the
  cached data. after this timeout, the content is re-generated
  on the next call.
* ``cache_type`` - type of caching. ``memory``, ``file``, ``dbm``, or
  ``memcached``.
* ``cache_url`` - (only used for ``memcached`` but required) a single
  IP address or a semi-colon separated list of IP address of
  memcache servers to use.
* ``cache_dir`` - In the case of the ``file`` and ``dbm`` cache types,
  this is the filesystem directory with which to store data
  files. If this option is not present, the value of
  ``module_directory`` is used (i.e. the directory where compiled
  template modules are stored). If neither option is available
  an exception is thrown.
 
  In the case of the ``memcached`` type, this attribute is required
  and it's used to store the lock files.
* ``cache_key`` - the "key" used to uniquely identify this content
  in the cache. the total namespace of keys within the cache is
  local to the current template, and the default value of "key"
  is the name of the def which is storing its data. It is an
  evaluable tag, so you can put a Python expression to calculate
  the value of the key on the fly. For example, heres a page
  that caches any page which inherits from it, based on the
  filename of the calling template:
 
.. sourcecode:: mako

    <%page cached="True" cache_key="${self.filename}"/>

    ${next.body()}
 
    ## rest of template
 
Accessing the Cache
===================

The :class:`.Template`, as well as any template-derived namespace, has
an accessor called ``cache`` which returns the ``Cache`` object
for that template. This object is a facade on top of the Beaker
internal cache object, and provides some very rudimental
capabilities, such as the ability to get and put arbitrary
values:

.. sourcecode:: mako

    <%
        local.cache.put("somekey", type="memory", "somevalue")
    %>
 
Above, the cache associated with the ``local`` namespace is
accessed and a key is placed within a memory cache.

More commonly the ``cache`` object is used to invalidate cached
sections programmatically:

.. sourcecode:: python

    template = lookup.get_template('/sometemplate.html')
 
    # invalidate the "body" of the template
    template.cache.invalidate_body()
 
    # invalidate an individual def
    template.cache.invalidate_def('somedef')
 
    # invalidate an arbitrary key
    template.cache.invalidate('somekey')
 
API Reference
==============

.. autoclass:: mako.cache.Cache
    :members:
    :show-inheritance: