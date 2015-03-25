HTTP Cache notes
====

.. contents:: Table of Contents

Contributing
-----

**TODO**

HTTP Cache Headers
-----


Cache-control
.....

"Default" header for cache behavior. Structure:

.. code:: console

    Cache-Control:<directives>

The value for this cache header is a ``,``-separated list of cache response directives. Each directive is lower-cased.

Below is a list of most-used directives. For more information, go see the `RFC 2616 spec`_

Public / Private
+++++

This cache response directive lets the caching software know that the response is specific (``private``) for given end-user or not (``public``).

A response marked as ``private`` can still be cached, for instance by the browser of the end-user. Intermediary caches will not cache the request however if the ``private`` directive is set.

.. code:: console

    Cache-Control:private


No-cache
+++++

The ``no-cache`` directive let's the caches know that the given response is not to be cached, and that the response is to be revalidated on each request.

.. code:: console

    Cache-Control:no-cache

Max-age
+++++

Determines the maximum cache validity for the current request. For intermediary caches, you should use the ``s-maxage`` directive.

``Max-age`` overrides the Expires_ header.

The value of the ``max-age`` directive is in "deltaseconds", or the amount of seconds the cache remains valid.

.. code:: console

    Cache-Control:max-age=3600

S-maxage
+++++

Determines the maximum cache validity for intermediary (or shared) caches. This directive also overrides the Expires_ header.

When no ``s-maxage`` directive is available for the intermediarey cache, it will fall back to the ``max-age`` directive.

.. code:: console

    Cache-Control:s-maxage=7200

.. _`RFC 2616 spec`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9

Expires
....

The ``expires`` header used to be **the** standard way of defining cache validity. On most modern caching systems, the ``cache-control`` header takes precedence over ``expires``.

Some older systems may still use the ``expires`` header though, so it's always a good idea to provide it along with the ``cache-control`` headers for compatability purposes.

The value of the ``expires`` header should be a valid `RFC 1123`_ date format. In PHP, you can use this DateTime constant: ``DateTime::RFC1123``

.. code:: console

    Expires: Thu, 01 Dec 1994 16:00:00 GMT

For more information on the ``expires`` header, go to `the specification`_.

.. _`RFC 1123`: http://tools.ietf.org/html/rfc1123#page-55
.. _`the specification`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21

E-tag
....

An ``etag`` or ``entity-tag`` in full is a unique identifier for a requested resource. It usually is a hash of resource content, or a hash of the last time the resource was updated.

``Etag`` headers can be used by the client to request a given resource, if the ``etag`` is different than the one it already has. It's up to the server to correctly generate an ``etag`` for the requested resource.

.. code:: console

    ETag: 0800fc577294c34e0b28ad2839435945

For more information on the ``expires`` header, go to `the etag specification`_.

.. _`the etag specification`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.19

Vary
....

The ``vary`` header is used to inform the caching mechanism(s) which header is used to differentiate a given cache.

Imagine you have a want to cache a given resource, but each client should have a different cache, then you can specify the ``User-Agent`` in the ``vary`` header. As a result, the caching layer will create a new cached version of the response for each different User Agent.

This comes in handy if you want to cache certain parts of your response differently because they depend for instance on a logged in user.

.. code:: console

    Vary: Cookie

For more information on the ``vary`` header, go to `the vary specification`_.

.. _`the vary specification`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.44

Cookbook
-----

**TODO**

