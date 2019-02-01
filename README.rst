.. image:: https://img.shields.io/badge/chat-join%20now-blue.svg
   :target: https://gitter.im/python-trio/general
   :alt: Join chatroom

.. image:: https://img.shields.io/badge/docs-read%20now-blue.svg
   :target: https://async-generator.readthedocs.io/en/latest/?badge=latest
   :alt: Documentation Status

.. image:: https://travis-ci.org/python-trio/async_generator.svg?branch=master
   :target: https://travis-ci.org/python-trio/async_generator
   :alt: Automated test status

.. image:: https://ci.appveyor.com/api/projects/status/af4eyed8o8tc3t0r/branch/master?svg=true
   :target: https://ci.appveyor.com/project/python-trio/trio/history
   :alt: Automated test status (Windows)

.. image:: https://codecov.io/gh/python-trio/async_generator/branch/master/graph/badge.svg
   :target: https://codecov.io/gh/python-trio/async_generator
   :alt: Test coverage

The async_generator library
===========================

Many Python programmers already know and love `generators
<https://medium.freecodecamp.org/how-and-why-you-should-use-python-generators-f6fb56650888>`__,
because they're a super convenient way to write your own iterators
that you can loop over using ``for``. Python 3.6 added a new thing:
`async generators <https://www.python.org/dev/peps/pep-0525/>`__. Just
like regular generators, they're super convenient; the difference is
that now you loop over them using ``async for``, and this means you
can use ``await`` inside them. If you're not doing async programming,
you probably don't care about this at all. But if you *are* doing
async programming, this is *super awesome*. If you're not convinced,
check out this `5-minute demo
<https://youtu.be/PulzIT8KYLk?t=24m30s>`__ (warning: contains
Nathaniel's first ever attempt to live-code on stage).

But what if you aren't using the latest and greatest Python, but still
want that async generator goodness? That's where this package comes
in. It gives you async generators and ``@asynccontextmanager`` on
Python 3.5+.

**Example 1:** suppose you want to **write your own async generator**.
The goal is that you can use it like:

.. code-block:: python3

   async for json_obj in load_json_lines(stream_reader):
       ...

If you're using Python 3.6+, you could implement `load_json_lines`
using native syntax:

.. code-block:: python3

   async def load_json_lines(stream_reader):
       async for line in stream_reader:
           yield json.loads(line)

Or, you could use ``async_generator``, and now your code will work on
Python 3.5+:

.. code-block:: python3

   from async_generator import async_generator, yield_

   @async_generator
   async def load_json_lines(stream_reader):
       async for line in stream_reader:
           await yield_(json.loads(line))


**Example 2:** let's say you want to **write your own async context
manager**. The goal is to be able to write:

.. code-block:: python3

   async with background_server() as ...:
       ...

If you're using Python 3.7+, you don't need this library; you can
write something like:

.. code-block:: python3

   from contextlib import asynccontextmanager

   @asynccontextmanager
   async def background_server():
       async with trio.open_nursery() as nursery:
           value = await nursery.start(my_server)
           try:
               yield value
           finally:
               # Kill the server when the scope exits
               nursery.cancel_scope.cancel()

This is the same, but works on 3.5+:

.. code-block:: python3

   from async_generator import async_generator, yield_, asynccontextmanager

   @asynccontextmanager
   @async_generator
   async def background_server():
       async with trio.open_nursery() as nursery:
           value = await nursery.start(my_server)
           try:
               await yield_(value)
           finally:
               # Kill the server when the scope exits
               nursery.cancel_scope.cancel()

(And if you're on 3.6, so you have built-in async generators but no
built-in ``@asynccontextmanager``, then don't worry, you can use
``async_generator.asynccontextmanager`` on native async generators.)


Let's do this
=============

* Install: ``python3 -m pip install -U async_generator`` (or on Windows,
  maybe ``py -3 -m pip install -U async_generator``

* Manual: https://async-generator.readthedocs.io/

* Bug tracker and source code: https://github.com/python-trio/async_generator

* Real-time chat: https://gitter.im/python-trio/general

* License: MIT or Apache 2, your choice

* Contributor guide: https://trio.readthedocs.io/en/latest/contributing.html

* Code of conduct: Contributors are requested to follow our `code of
  conduct
  <https://trio.readthedocs.io/en/latest/code-of-conduct.html>`__ in
  all project spaces.


How come some of those links talk about "trio"?
===============================================

`Trio <https://trio.readthedocs.io>`__ is a new async concurrency
library for Python that's obsessed with usability and correctness – we
want to make it *easy* to get things *right*. The ``async_generator``
library is maintained by the Trio project as part of that mission, and
because Trio uses ``async_generator`` internally.

You can use ``async_generator`` with any async library. It works great
with ``asyncio``, or Twisted, or whatever you like. (But we think Trio
is pretty sweet.)
