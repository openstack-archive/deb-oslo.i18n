=====================================================
 How to Use oslo.i18n in Your Application or Library
=====================================================

Installing
==========

At the command line::

    $ pip install oslo.i18n

.. _integration-module:

Creating an Integration Module
==============================

To use oslo.i18n in a project, you will need to create a small
integration module to hold an instance of
:class:`~oslo_i18n.TranslatorFactory` and references to
the marker functions the factory creates.

::

    # app/i18n.py

    import oslo_i18n

    _translators = oslo_i18n.TranslatorFactory(domain='myapp')

    # The primary translation function using the well-known name "_"
    _ = _translators.primary

    # Translators for log levels.
    #
    # The abbreviated names are meant to reflect the usual use of a short
    # name like '_'. The "L" is for "log" and the other letter comes from
    # the level.
    _LI = _translators.log_info
    _LW = _translators.log_warning
    _LE = _translators.log_error
    _LC = _translators.log_critical

Then, in the rest of your code, use the appropriate marker function
for each message:

.. code-block:: python

    from myapp.i18n import _, _LW

    # ...

    LOG.warn(_LW('warning message: %s'), var)

    # ...

    try:

        # ...

    except AnException1:

        # Log only
        LOG.exception(_LE('exception message'))

    except AnException2:

        # Raise only
        raise RuntimeError(_('exception message'))

    else:

        # Log and Raise
        msg = _('Unexpected error message')
        LOG.exception(msg)
        raise RuntimeError(msg)

.. note::

   Libraries probably do not want to expose the new integration module
   as part of their public API, so rather than naming it
   ``mylib.i18n`` it should be called ``mylib._i18n`` to indicate that
   it is a private implementation detail, and not meant to be used
   outside of the library's own code.

.. warning::

    The old method of installing a version of ``_()`` in the builtins
    namespace is deprecated. Modifying the global namespace affects
    libraries as well as the application, so it may interfere with
    proper message catalog lookups. Calls to
    :func:`gettextutils.install` should be replaced with the
    application or library integration module described here.

Handling hacking Objections to Imports
======================================

The OpenStack style guidelines prefer importing modules and accessing
names from those modules after import, rather than importing the names
directly. For example:

::

    # WRONG
    from foo import bar

    bar()

    # RIGHT

    import foo

    foo.bar()

The linting tool hacking_ will typically complain about importing
names from within modules. It is acceptable to bypass this for the
translation marker functions, because they must have specific names
and their use pattern is dictated by the message catalog extraction
tools rather than our style guidelines. To bypass the hacking check
for imports from the integration module, add an import exception to
``tox.ini``.

For example::

    # tox.ini
    [hacking]
    import_exceptions =
      app.i18n

.. _hacking: https://pypi.python.org/pypi/hacking

.. _lazy-translation:

Lazy Translation
================

Lazy translation delays converting a message string to the translated
form as long as possible, including possibly never if the message is
not logged or delivered to the user in some other way. It also
supports logging translated messages in multiple languages, by
configuring separate log handlers.

Lazy translation is implemented by returning a special object from the
translation function, instead of a unicode string. That special
message object supports some, but not all, string manipulation
APIs. For example, concatenation with addition is not supported, but
interpolation of variables is supported. Depending on how translated
strings are used in an application, these restrictions may mean that
lazy translation cannot be used, and so it is not enabled by default.

To enable lazy translation, call :func:`enable_lazy`.

::

    import oslo_i18n

    oslo_i18n.enable_lazy()

Translating Messages
====================

Use :func:`~oslo_i18n.translate` to translate strings to
a specific locale. :func:`translate` handles delayed translation and
strings that have already been translated immediately. It should be
used at the point where the locale to be used is known, which is often
just prior to the message being returned or a log message being
emitted.

::

    import oslo_i18n

    trans_msg = oslo_i18n.translate(msg, desired_locale=my_locale)

if desired_locale is not specified then the default locale is used.

Available Languages
===================

Only the languages that have translations provided are available for
translation. To determine which languages are available the
:func:`~oslo_i18n.get_available_languages` is provided. Since different languages
can be installed for each domain, the domain must be specified.

::

    import oslo_i18n

    avail_lang = oslo_i18n.get_available_languages('myapp')

.. seealso::

   * :doc:`guidelines`
