HTTP Cache notes
====

.. contents:: Table of Contents

HTTP Cache Headers
-----


Cache-control
.....

"Default" header for cache behavior. Structure:

.. code:: console

    cache-control:<directives>

The value for this cache header is a ``,``-separated list of cache response directives. Each directive is lower-cased.

Below is a list of most-used directives. For more information, go see the `RFC 2616 spec`_

Public / Private
+++++

This cache response directive lets the caching software know that the response is specific (``private``) for given end-user or not (``public``).

A response marked as ``private`` can still be cached, for instance by the browser of the end-user. Intermediary caches will not cache the request however if the ``private`` directive is set.

.. code:: console

    cache-control:private


No-cache
+++++

The ``no-cache`` directive let's the caches know that the given response is not to be cached, and that the response is to be revalidated on each request.

.. code:: console

    cache-control:no-cache

Max-age
+++++

Determines the maximum cache validity for the current request. For intermediary caches, you should use the ``s-maxage`` directive.

``Max-age`` overrides the Expires_ header.

The value of the ``max-age`` directive is in "deltaseconds", or the amount of seconds the cache remains valid.

.. code:: console

    cache-control:max-age=3600

S-maxage
+++++

Determines the maximum cache validity for intermediary (or shared) caches. This directive also overrides the Expires_ header.

When no ``s-maxage`` directive is available for the intermediarey cache, it will fall back to the ``max-age`` directive.

.. code:: console

    cache-control:s-maxage=7200

.. _`RFC 2616 spec`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9

Expires
....

**TODO**

E-tag
....

**TODO**

Pragma
....

**TODO**

Vary
....

**TODO**

Cookbook
-----

**TODO**

