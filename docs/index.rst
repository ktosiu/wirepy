wirepy
======

`Wirepy` aims to remedy the disastrous situation of packet-dissection-libraries
available to the Python programming language. It is a foreign function
interface to use `Wireshark`_ within Python as implemented by `CPython`_ and
`PyPy`_.

The currently available options are either painfully slow or lack
features. `Wireshark`_ provides support for more than 1.300 protocols, more
than 125.000 fields within those protocols and more than 1.500.000 defined
values and is actively maintained.

Get the code from `GitHub`_.

.. note::
    This library is created out of pure necessity. I dont' know know where it
    is headed or even feasible to create a direct binding to ``libwireshark``.
    The best current source of documentation are the unittests.


.. toctree::
   modules

Installation
------------

Requirements
~~~~~~~~~~~~

* CPython_ 3.x or later
* CFFI_ 0.6 or later
* Wireshark_ 1.10 or later
* GLib2_ 2.16.0 or later
* nose_ and tox_ are used for testing


Configuring Wireshark
~~~~~~~~~~~~~~~~~~~~~

* If you are using a Linux distribution, `CPython`_-, `Wireshark`_ and their
  headers can be usually be installed from the package repository (e.g. via
  yum).
* Otherwise you may configure and build a **minimal** Wireshark library like
  this::

    ./configure -q --prefix=$HOME/wireshark --disable-wireshark --disable-packet-editor --disable-editcap --disable-capinfos --disable-mergecap --disable-text2pcap --disable-dftest --disable-airpcap --disable-rawshark --without-lua
    make -sj9
    make install


Configuring wirepy
~~~~~~~~~~~~~~~~~~

#. Run ``./configure`` to configure `Wirepy`'s sourcecode:

   * Running ``./configure`` as it is should work if you have wireshark
     installed through `pkg-config`.
   * **Otherwise** you need to specify the paths to wireshark's and glib's
     header files yourself. You may also want to use a locally installed
     version of wireshark. The command may look something like this::

        DEPS_CFLAGS="-I/path/to/wireshark-headers -I/path/to/glib-2.0-headers" DEPS_LIBS="-L/path/to/wireshark/lib" ./configure

     Executing may look like this::

        LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/wireshark/lib PATH=$PATH:/path/to/wireshark/bin make

#. Take a look at the ``Makefile`` and use ``make``.


Development
-----------

`Wirepy` uses CFFI_ to create an interface to ``libwireshark``, ``libwsutil``
and ``libwiretap``. Class-based representations of the defined C-structs and
-types are used to bind behavior, state and documentation. Instead of
returning error values, all functions raise exceptions of appropriate type.
Memory is handled by python's garbage collector most of the time.

The entire wireshark-interface can be found in ``/lib``; one may need special
knowledge about how to use classes there. Once things quiet down in ``/lib``, a
more pythonic API is to be created outside of ``/lib``.


* What (at least in part) works:

  + Enumerating live interfaces and their capabilities.
  + Reading packets from live interfaces.
  + Reading packet dumps using the wiretap library.
  + Compiling and using display-filters to filter the resulting frame data.
  + Inspection of the resulting protocol-tree (``epan.ProtoTree``),

    - inspection of it's fields (``ftypes.FieldType``).
    - and their values (``epan.FieldValue``).

  + Working with columns, including ``COL_CUSTOM``.

* What does not:

  + Putting it all together.

    - We probably want to create class-based representations of protocols,
      fields and their known values; one might create a class factory that uses
      the functions from ``/lib`` to create classes for protocols as they pop
      into existence in a proto-tree and keep a weakref to those.
    - It should be fairly easy to use the above for class-based comparision of
      values and create a simple compiler for display-filter strings (e.g.
      "``DisplayFilter(IP.proto==IP.proto.IPv4)``").
    - A ``FieldType`` should have it's own subclass that is able to interpret
      common python objects, preserving it's type as closely as possible.

      - A ``INT8`` should do arithmetic mod 2**8
      - A ``IPv4`` or ``IPv6`` may take values from the ``ipaddr``-module
      - etc

    This should live outside of ``/lib``.

  + Writing packet dumps through ``wtap_dump...``.
  + Taps and the other ~95% of the more useful functions of wireshark.
  + Plugins will not load because they expect the symbols from ``libwireshark``
    in the global namespace. We hack this situation by flooding the namespace
    with a call to dlopen().
  + A backport to Python 2.x (using a compat module) should be easy.

* To be considered:

  + There are many ways in which ``libwireshark`` handles memory allocation.
    From within Python, everything should be garbage-collected though;
  + There are many ways in which ``libwireshark`` handles memory deallocation.
    Once some or the other function is called or state is reached, memory
    represented by reachable objects becomes invalid garbage.
  + The raw C-API very much expects C-like behavior from it's user; there are
    many de-facto global states and carry-on-your-back variables. Hide those


Contact
-------

Via lukas.lueg@gmail.com. Please use github to report problems and submit
comments (which are very welcome). Patches should conform to :pep:`8` and have
appropriate unittests attached.

Indices and tables
------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

Generated |today|.

.. _GitHub: https://github.com/lukaslueg/wirepy
.. _CFFI: https://cffi.readthedocs.org
.. _`Wireshark`: https://www.wireshark.org
.. _`CPython`: http://www.python.org
.. _`PyPy`: http://www.pypy.org
.. _Wireshark: https://www.wireshark.org
.. _nose: https://nose.readthedocs.org/en/latest/
.. _tox: http://tox.readthedocs.org/en/latest/
.. _Glib2: https://developer.gnome.org/glib/
