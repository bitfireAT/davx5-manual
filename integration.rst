=================
Integration / API
=================

This section is about other apps can interact with DAVx⁵.


Launching the DAVx⁵ login screen
================================

You can use either

1. an explicit Intent (= directed directly to DAVx⁵) or
2. an implicit Intent (= directed to all apps which support these URLs, especially DAVx⁵)

to launch the DAVx⁵ login screen with pre-filled URL and credentials.

Explicit Intent
---------------

If you want to explicitly open DAVx⁵ (and no other app)::

    val intent = Intent().apply {
        setClassName("at.bitfire.davdroid", "at.bitfire.davdroid.ui.setup.LoginActivity")
        putExtra("url", "https://example.com/path/")
        putExtra("username", user.name)
        putExtra("password", user.app_password)
    }

You can set URL, username and password as extras. All of those are optional*.

+------------+---------+-----------------------------------------------------------------+
| extra name | type    | description                                                     |
+============+=========+=================================================================+
| url        | String* | CalDAV/CardDAV base URL                                         |
+------------+---------+-----------------------------------------------------------------+
| username   | String* | pre-filled username                                             |
+------------+---------+-----------------------------------------------------------------+
| password   | String* | pre-filled password                                             |
|            |         | (generate an app-specific password for DAVx⁵)                   |
+------------+---------+-----------------------------------------------------------------+

.. versionadded:: 2.6
   Instead of providing URL, username and password as an extra, you can also use the
   `Nextcloud Login Flow <https://docs.nextcloud.com/server/latest/developer_manual/client_apis/LoginFlow/index.html>`_
   method:

+------------------+---------+---------------------------------------------------------------------------+
| Intent           | type    | description                                                               |
+==================+=========+===========================================================================+
| data             | URI     | login flow entry point (``<server>/index.php/login/flow``).               |
+------------------+---------+---------------------------------------------------------------------------+
| extra: loginFlow | Int     | set to 1 to indicate Login Flow                                           |
+------------------+---------+---------------------------------------------------------------------------+
| extra: davPath   | String* | CalDAV/CardDAV base URL; will be appended to server URL returned by Login |
|                  |         | Flow without further processing (e.g. ``/remote.php/dav``)                |
+------------------+---------+---------------------------------------------------------------------------+



Implicit Intent
---------------

If you want to open *any CalDAV/CardDAV app that supports CalDAV/CardDAV URLs, including DAVx⁵*,
you can launch a ``caldav(s)://`` or ``carddav(s)://`` Intent::

    val intent = Intent(Intent.ACTION_VIEW, Uri.parse("caldavs://server.example.com/path/"))
    # caldav://server.example.com/path/   will be rewritten to http://server.example.com/path/
    # caldavs://server.example.com/path/  will be rewritten to https://server.example.com/path/
    # carddav://server.example.com/path/  will be rewritten to http://server.example.com/path/
    # carddavs://server.example.com/path/ will be rewritten to https://server.example.com/path/
    # davx5://server.example.com/path/    will be rewritten to https://server.example.com/path/

You can specify username and password either as extras (see explicit Intent above) or encoded it
directly in the URL (example: ``davx5://user:password@server.example.com/path/``). This is
primarily intended for links from Web pages (for instance, an Intranet page that links to a
``davx5://`` setup URL) or QR codes that can then be scanned and opened with a compatible QR code
scanner app (like `Binary Eye <https://github.com/markusfisch/BinaryEye>`_).

For instance, you can generate a QR code of the URL ``davx5://user@server.example.com/path/``
and print it. Then scan it with a compatible QR code scanner app on the Android device to open
DAVx⁵ with the URL ``https://server.example.com/path/`` and username ``user`` (without password).

.. warning:: Username and password should not be encoded in the URL programatically. Use the
   ``username`` and ``password`` extras instead.


.. _extended_event_properties:

Extended event properties
=========================

There are iCalendar properties which are not covered by the `Android Calendar provider <https://developer.android.com/guide/topics/providers/calendar-provider>`_,
like the ``URL`` of an event and attachments. DAVx⁵ and calendar apps can't exchange such properties in a
standardized way, so editing/viewing these properties in calendar apps and synchronizing them won't work.

To overcome these limitations, it's necessary to agree on a common format for these properties. The following
formats are designed for interaction with calendar apps. We hope that calendar apps start to accept
these formats in order to allow users to get even better CalDAV integration on Android.

Currently only ``URL`` is supported.

URL
---

.. versionadded:: 3.3.10

`Extended property <https://developer.android.com/reference/kotlin/android/provider/CalendarContract.ExtendedProperties>`_ with those fields:

  * ``NAME`` = ``vnd.android.cursor.item/vnd.ical4android.url``
  * ``VALUE`` = the `iCalendar URL <https://tools.ietf.org/html/rfc5545#section-3.8.4.6>`_ (e.g. ``https://example.com``)

Calendar apps that are known to support this property:

  * aCalendar+ (since 2.5)

See also `KDoc <https://bitfireat.gitlab.io/ical4android/dokka/ical4android/at.bitfire.ical4android/-android-event/-m-i-m-e-t-y-p-e_-u-r-l.html>`_.

