=================
Integration / API
=================

This section is about other apps can interact with DAVx⁵.


Launching the DAVx⁵ login screen
================================

You can use either

1. an explicit Intent (= directed directly to DAVx⁵) or
2. an implicit Intent (= directed to all apps which support these URLs, especially DAVx⁵)
3. an implicit Intent together with a configuration file

to launch the DAVx⁵ login screen with pre-filled URL and credentials.

Do not rely on the ``Activity`` result because when OAuth is used, the flow is complex and
involves the browser redirecting back to DAVx⁵. So it's best to just launch the ``Intent``
(with ``FLAG_ACTIVITY_NEW_TASK``) and, if desired, register an ``OnAccountsUpdateListener``
to get informed about new accounts.

Explicit Intent
---------------

If you want to explicitly open DAVx⁵ (and no other app)::

    val intent = Intent().apply {
        setClassName("at.bitfire.davdroid", "at.bitfire.davdroid.ui.setup.LoginActivity")
        putExtra("url", "https://example.com/path/")
        putExtra("username", user.name)
        putExtra("password", user.app_password)
        addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
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



Using a configuration file together with an implicit Intent (QR code scan)
--------------------------------------------------------------------------

Preconfigured Settings Format
---------------

DAVx⁵ can be pre-configured by pointing it at a JSON file hosted on a
server. The URL (for example ``https://example.com/config.json``) of that file can be:

- entered manually in the login screen,
- encoded in a QR code as ``davx5://setup?config=<url>`` and scanned with the
  in-app scanner, or
- passed from another app via an Android intent with the URI
  ``davx5://setup?config=<url>``.

JSON format
-----------

.. code-block:: json

   {
     "baseRoot": "https://dav.example.com",
     "basicAuth": {
       "username": "ash@example.com",
       "password": "generated-app-pw"
     }
   }

.. note::

   Unknown fields are ignored for compatibility with future versions.

Field reference
---------------

``baseRoot`` (string, URI — **required**)
   Base root URI of the CalDAV/CardDAV server.

``basicAuth`` (object — **required**)
   HTTP Basic Authentication credentials.

   ``username`` (string — **required**)
      Username.

   ``password`` (string — **required**)
      Generated app password for this login attempt. The server should generate this password on the fly and revoke it
      if it is not used within a brief period of time. Once used, it must remain valid for subsequent logins.

Example of the final URL
---------------

- create a QR code from an URI like this ``davx5://setup?config=https://example.com/config.json`` and scan it with a capable QR scan app or directly from within DAVx⁵ (scan capability can be limited if you use the -ose version of DAVx⁵)
- manually enter the URL of the config file (for example ``https://example.com/config.json``) directly in the approprate field in DAVx⁵'s dedicated QR/connection profile setup option.

Currently *not* yet implemented
---------------

These additional field will be handled in future versions of DAVx⁵

.. code-block:: json

   "customization": {
     "productName": "Example Mail",
     "description": "Sync your contacts, calendars and tasks - brought to you by DAVx⁵",
     "logoImage": "data:image/gif;base64,R0lGODd[...]IQA7",
     "headerImage": "data:image/png;base64,R0lGODd[...]IQA7"
   },
   "supportInfos": {
     "linkDestination": "https://example.com/support",
     "linkTitle": "Open a ticket / get support",
     "description": "In case of any problem with DAVx5 or the calendar / contacts synchronization please reach out to us directly via our online helpdesk"
   }



Open DAVx⁵ for a specific account
=================================

You can use an explicit Intent (= directed directly to DAVx⁵) to launch the DAVx⁵ account screen for a specific account::

    val intent = Intent(Intent.ACTION_MAIN)
    intent.setClassName(DAVX5_PACKAGE_NAME,"${DAVX5_PACKAGE_NAME}.ui.account.AccountActivity")
    intent.addFlags(FLAG_ACTIVITY_NEW_TASK)
    intent.putExtra("account", Account("accountname", "bitfire.at.davdroid"))
    

You have to set the Account as an extra.

+------------+-------------+-----------------------------------------------------------------+
| extra name | type        | description                                                     |
+============+=============+=================================================================+
| account    | Parcelable  | CalDAV/CardDAV base URL                                         |
+------------+-------------+-----------------------------------------------------------------+



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
  * Etar (since 1.0.35)

See also `KDoc <https://bitfireat.github.io/ical4android/ical4android/at.bitfire.ical4android/-android-event/-companion/-m-i-m-e-t-y-p-e_-u-r-l.html>`_.

