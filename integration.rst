=================
Integration / API
=================

This section is about other apps can interact with DAVx⁵.


Launching the DAVx⁵ login screen
================================

You can use an explicit Intent to launch the DAVx⁵ login screen with pre-filled credentials::

    val intent = Intent()
    intent.setClassName("at.bitfire.davdroid", "at.bitfire.davdroid.ui.setup.LoginActivity")

.. warning:: Always use an explicit intent with hardcoded package name for security reasons. Keep
   in mind that an attacker could trick users into installing a malicious app with the same package
   name (when third-party app sources are allowed).

You can set URL, username and password as extras. All of those are optional.

+------------+--------+---------------------------------------------+
| extra name | type   | description                                 |
+============+========+=============================================+
| url        | String | CalDAV/CardDAV base URL                     |
+------------+--------+---------------------------------------------+
| username   | String | pre-filled username                         |
+------------+--------+---------------------------------------------+
| password   | String | pre-filled password (usage not recommended) |
+------------+--------+---------------------------------------------+

.. versionadded:: 2.6
   Alternatively, you can use the `Nextcloud Login Flow <https://docs.nextcloud.com/server/latest/developer_manual/client_apis/LoginFlow/index.html>`_ method:

Intent data (URI): login flow entry point (``<server>/index.php/login/flow``). Intent extras (\*… optional):

+------------+---------+---------------------------------------------------------------------------+
| extra name | type    | description                                                               |
+============+=========+===========================================================================+
| loginFlow  | Int     | set to 1 to indicate Login Flow                                           |
+------------+---------+---------------------------------------------------------------------------+
| davPath    | String* | CalDAV/CardDAV base URL; will be appended to server URL returned by Login |
|            |         | Flow without further processing (e.g. ``/remote.php/dav``)                |
+------------+---------+---------------------------------------------------------------------------+

For compatibility with old DAVx⁵ versions, you can use both methods at the same time. Old DAVx⁵ versions will
use the ``url``, ``username``, ``password`` extras, while new versions will see the ``loginFlow``
extra and switch to the Login Flow method.


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

See also `KDoc <https://bitfireat.gitlab.io/ical4android/dokka/ical4android/at.bitfire.ical4android/-android-event/-m-i-m-e-t-y-p-e_-u-r-l.html>`_.

