HTTP Cache notes
====

.. contents:: Table of Contents

Contributing
-----

All contributions to this document are welcome. If you want to contribute, use the *fork-change-pr approach*.

Currently all information is contained in this document. If the Cookbook_ section is growing too large, I may separate it into different sub documents.

License
-----

Copyright (C) 2015 Tom Van Herreweghe

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

HTTP Cache Headers
-----

Response Headers
____

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

The value of the ``expires`` header should be a valid `RFC 7231`_ HTTP-date. In PHP, you can use this DateTime constant: ``DateTime::RFC1123``

.. code:: console

    Expires: Thu, 01 Dec 1994 16:00:00 GMT

For more information on the ``expires`` header, go to `the specification`_.

.. _`the specification`: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21

ETag
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

Last-Modified
....

The ``Last-Modified`` header informs the client when the requested resource was last modified. This header can be used together with the If-Modified-Since_ Request header for cache invalidation. The value of the ``Last-Modified`` header is  a valid `RFC 7231`_ HTTP-date on which the resource was last modified.

.. code:: console

    Last-Modified: Thu, 01 Dec 1994 16:00:00 GMT

.. _`RFC 7231`: https://tools.ietf.org/html/rfc7231#section-7.1.1.1

Request Headers
____

If-Modified-Since
....

A client may send an ``If-Modified-Since`` header if it wants to be informed of the validity of its cache of the requested resource. If the requested resource was not modified since the specified datetime, a ``304 Not Modified`` status code should be returned, informing the client that the resource has not changed. The value of the ``If-Modified-Since`` header is a valid `RFC 7231`_ HTTP-date.

Inspect this header if you want to use `Cache Invalidation With Last-Modified`_ on resources.

.. code:: console

    If-Modified-Since: Thu, 01 Dec 1994 16:00:00 GMT

If-None-Match
....

Each resource content can uniquely defined by an ETag_ header. A client may send an ``If-None-Match`` request to verify that the resource content is still valid. When that is the case, a ``304 Not Modified`` status code should be returned, informing the client that the resource content has not changed. 

Inspect this header if you want to use `Cache Invalidation With ETag`_ on resources.`

.. code:: console

    If-None-Match: 0800fc577294c34e0b28ad2839435945


Cookbook
-----

Cache Invalidation
____

Cache invalidation is the act of informing a client that its cache of a requested resource is not valid anymore, thus prompting the client to refresh its cache, effectively transmitting the resource content again.

The key approach here is that resources are cached as long as they are valid. On each request to the resource, the client sends the appropriate headers to verify the cache validity. While the cache is valid, the response will inform the client that nothing has changed with a ``304 Not Modified`` status code. The client will then keep its cache of the resource.

The main benefit of this approach is that you are able to quickly serve new content for the requested resource. Responsability for a correct implementation lies both with the client who should send the correct headers as with the receiving end which should correctly respond to the provided `Request Headers`_.

Disadvantages are the extra overhead on each request by transmitting headers. Both parties also have to correctly implement their headers.

Cache Invalidation should be used on resource content which has a more dynamic nature. You want to prevent having to process each request fully on the server side, yet benefit from correctly updated content when necessary.

Cache Invalidation With Last-Modified
....

1. Initial Request Headers
####

    No caching headers should be transmitted to the receiving end.

2. Initial Response Headers
####

.. code:: console

    Last-Modified: Thu, 01 Dec 1994 16:00:00 GMT
    Status: 200 OK

3. Subsequent Request Headers
####

.. code:: console

    If-Modified-Since: Thu, 01 Dec 1994 16:00:00 GMT

4. Possible Response Headers
####

4.1 Response Headers if resource hasn't changed

.. code:: console

    Status: 304 Not Modified

4.2 Response Headers if resource **has** changed

.. code:: console

    Last-Modified: Thu, 01 Dec 1994 16:05:00 GMT
    Status: 200 OK

Cache Invalidation With ETag
....

**TODO**


Cache Validation (or Cache Expiration)
____

**TODO**
