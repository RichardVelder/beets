Web Plugin
==========

The ``web`` plugin is a very basic alternative interface to beets that
supplements the CLI. It can't do much right now, and the interface is a little
clunky, but you can use it to query and browse your music and---in browsers that
support HTML5 Audio---you can even play music.

While it's not meant to replace the CLI, a graphical interface has a number of
advantages in certain situations. For example, when editing a tag, a natural CLI
makes you retype the whole thing---common GUI conventions can be used to just
edit the part of the tag you want to change. A graphical interface could also
drastically increase the number of people who can use beets.

Install
-------

The Web interface depends on `Flask`_. To get it, just run ``pip install
flask``. Then enable the ``web`` plugin in your configuration (see
:ref:`using-plugins`).

.. _Flask: http://flask.pocoo.org/

If you need CORS (it's disabled by default---see :ref:`web-cors`, below), then
you also need `flask-cors`_. Just type ``pip install flask-cors``.

.. _flask-cors: https://github.com/CoryDolphin/flask-cors
.. _CORS: http://en.wikipedia.org/wiki/Cross-origin_resource_sharing


Run the Server
--------------

Then just type ``beet web`` to start the server and go to
http://localhost:8337/. This is what it looks like:

.. image:: beetsweb.png

You can also specify the hostname and port number used by the Web server. These
can be specified on the command line or in the ``[web]`` section of your
:doc:`configuration file </reference/config>`.

On the command line, use ``beet web [HOSTNAME] [PORT]``. Or the configuration
options below.

Usage
-----

Type queries into the little search box. Double-click a track to play it with
`HTML5 Audio`_.

.. _HTML5 Audio: http://www.w3.org/TR/html-markup/audio.html

Configuration
-------------

To configure the plugin, make a ``web:`` section in your
configuration file. The available options are:

- **host**: The server hostname. Set this to 0.0.0.0 to bind to all interfaces.
  Default: Bind to 127.0.0.1.
- **port**: The server port.
  Default: 8337.
- **cors**: The CORS allowed origin (see :ref:`web-cors`, below).
  Default: CORS is disabled.

Implementation
--------------

The Web backend is built using a simple REST+JSON API with the excellent
`Flask`_ library. The frontend is a single-page application written with
`Backbone.js`_. This allows future non-Web clients to use the same backend API.

.. _Flask: http://flask.pocoo.org/
.. _Backbone.js: http://documentcloud.github.com/backbone/

Eventually, to make the Web player really viable, we should use a Flash fallback
for unsupported formats/browsers. There are a number of options for this:

* `audio.js`_
* `html5media`_
* `MediaElement.js`_

.. _audio.js: http://kolber.github.com/audiojs/
.. _html5media: http://html5media.info/
.. _MediaElement.js: http://mediaelementjs.com/

.. _web-cors:

Cross-Origin Resource Sharing (CORS)
------------------------------------

The ``web`` plugin's API can be used as a backend for an in-browser client. By
default, browsers will only allow access from clients running on the same
server as the API. (You will get an arcane error about ``XMLHttpRequest``
otherwise.) A technology called `CORS`_ lets you relax this restriction.

If you want to use an in-browser client hosted elsewhere (or running from
a different server on your machine), first install the `flask-cors`_ plugin by
typing ``pip install flask-cors``. Then set the ``cors`` configuration option
to the "origin" (protocol, host, and optional port number) where the client is
served. Or set it to ``'*'`` to enable access from all origins. Note that
there are security implications if you set the origin to ``'*'``, so please
research this before using it.

For example::

    web:
        host: 0.0.0.0
        cors: 'http://example.com'


JSON API
--------


``GET /item/``
++++++++++++++

Responds with a list of all tracks in the beets library. ::

    {
      "items": [
        {
          "id": 6,
          "title": "A Song",
          ...
        }, {
          "id": 12,
          "title": "Another Song",
          ...
        }
        ...
      ]
    }


``GET /item/6``
+++++++++++++++

Looks for an item with id *6* in the beets library and responds with its JSON
representation. ::

    {
      "id": 6,
      "title": "A Song",
      ...
    }

If there is no item with that id responds with a *404* status
code.


``GET /item/6,12,13``
+++++++++++++++++++++

Response with a list of tracks with the ids *6*, *12* and *13*.  The format of
the response is the same as for `GET /item/`_. It is *not guaranteed* that the
response includes all the items requested. If a track is not found it is silently
dropped from the response.


``GET /item/query/querystring``
+++++++++++++++++++++++++++++++

Returns a list of tracks matching the query. The *querystring* must be a valid query as described in :doc:`/reference/query`. ::

    {
      "results": [
        { "id" : 6,  "title": "A Song" },
        { "id" : 12, "title": "Another Song" }
      ]
    }


``GET /item/6/file``
++++++++++++++++++++

Sends the  media file for the track. If the item or its corresponding file do
not exist a *404* status code is returned.


Albums
++++++

For albums, the following endpoints are provided:

* ``GET /album/``

* ``GET /album/5``

* ``GET /album/5,7``

* ``GET /album/query/querystring``

The interface and response format is similar to the item API, except replacing
the encapsulation key ``"items"`` with ``"albums"`` when requesting ``/album/``
or ``/album/5,7``. In addition we can request the cover art of an album with
``GET /album/5/art``.


``GET /stats``
++++++++++++++

Responds with the number of tracks and albums in the database. ::

    {
      "items": 5,
      "albums": 3
    }