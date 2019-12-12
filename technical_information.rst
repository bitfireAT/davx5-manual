
=====================
Technical information
=====================


Synchronization
===============

Synchronization logic
---------------------

These steps are performed when synchronizing a collection:

#. Prepare synchronization: prepare local collection, settings etc.
#. Query capabilities with HTTP ``PROPFIND``:

   * determine whether the server supports Collection Synchronization
   * CardDAV: determine whether the server supports vCard 4
   * fetch current ``CTag`` and ``sync-token``

#. Process locally deleted resources: if a local resource is flagged as deleted,

   * delete it on the server (HTTP ``DELETE`` with ``If-Match`` set to last known ``ETag`` to avoid deleting resources which have been changed on the server in the meanwhile) and
   * then remove it locally

#. Upload locally modified ("dirty") resources:

   * Assign a random UID and resource name to new resources; prepare contact group and recurring events, if necessary
   * If no previous ``ETag`` of the resource is known (i.e. the resource has not been uploaded yet), use HTTP ``PUT`` with ``If-None-Match: *`` to avoid overwriting a possibly existing resource with the same name
   * If a previous ``ETag`` of the resource is known, use HTTP ``PUT`` with ``If-Match`` set to last known ``ETag`` to avoid overwriting changes which happend on the server in the meanwhile
   * remember returned ``ETag`` as last known ``ETag``; otherwise reset last known ``ETag``

#. Choose sync algorithm (``PROPFIND``/``REPORT`` vs. Collection Synchronization):

   * CardDAV: use Collection Synchronization if supported by server, ``PROPFIND`` otherwise
   * CalDAV events: use Collection Synchronization if supported by server and past time event limit is disabled, ``REPORT calendar-query`` otherwise
   * CalDAV tasks: use ``REPORT calendar-query``

#. Check whether further synchronization is needed. Only continue when:

   * modifications (uploads/deletions) have been sent to the server, or
   * the sync state (``CTag``/``sync-token``) of the collection has changed since last sync, or
   * the ``PROPFIND``/``REPORT`` algorithm shall be used and the sync has been initiated manually

#. Continue with chosen sync algorithm (see below).


Sync algorithm: PROPFIND/REPORT
-------------------------------

#. Unset *present remotely* flag for all resources
#. List and process remote resources (only names and ``ETag``) using ``PROPFIND`` or ``REPORT`` (see above)

   * Download resources which have been added/modififed remotely in bunches using ``GET`` or ``REPORT addressbook-multiget/calendar-multiget`` into the local storage.
   * Set *present remotely* flag for all received resources.

#. Locally delete all resources which are not flagged as *present remotely*
#. Post-processing: clean up empty contact groups etc.
#. Save sync state (``CTag``/``sync-token``)


Sync algorithm: Collection Synchronization
------------------------------------------

#. Was a previous *initial sync* aborted and is now being continued? → If yes, set *initial sync*.
#. Do we have a previous ``sync-token``? → If no, set *initial sync*.
#. List and process changes since last ``sync-token`` (or all records if no previous ``sync-token`` is known) using ``REPORT sync-collection``.

   * Download resources which have been added/modififed remotely in bunches using ``GET`` or ``REPORT addressbook-multiget/calendar-multiget`` into the local storage.
   * Set *present remotely* flag for all received resources.

#. If the requested ``sync-token`` was invalid:

   * forget the ``sync-token``
   * reset *present remotely* flags of all local resources
   * set *initial sync* and continue with 3.

#. Save ``sync-token``.
#. Are there further changes on the server (HTTP 507 on collection URL in multiget response)? → If yes, continue with 3.
#. Only for *initial sync*: delete all local resources which are not present remotely.
#. Post-processing: clean up empty contact groups etc.


Conflict handling
-----------------

Conflicts occur when different versions of a resource are available and it's ambigous which one shall be used. For instance:

#. A contact exists on the server and has been synchronized to a mobile phone with DAVx⁵ and to a desktop PC with Gnome Evolution.
#. You modify the contact with Evolution, which immediately uploads it to the server.
#. You modify the same contact on your mobile device, too.
#. DAVx⁵ wants to upload the modified contact and finds that it has been changed on the server in the meanwhile. Now there's a conflict of two different versions of this contact.

How DAVx⁵ handles such conflicts:

* DAVx⁵ relies on HTTP ``ETag`` to determine whether a resource has been changed on the server.
* **The server always wins.** If a local resource can't be uploaded or deleted safely because it has been modified on the server in the meanwhile, local changes are discarded and the server version is used.
* DAVx⁵ doesn't involve the user in resolving conflicts (like asking which version shall be used) because it's supposed to run in the background silently.

At the moment, DAVx⁵ doesn't support `automatic vCard merging as suggested in vCard 4 <https://tools.ietf.org/html/rfc6350#section-7>`_.



Supported fields
================

Supported contact fields
------------------------

.. |vCard_types_mapped| replace::

   Types like private, work etc. are mapped when possible. Not all vCard values have a corresponding Android value and vice versa.
   :ref:`Custom labels are supported. <custom-labels>`

Name
^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.StructuredName <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.StructuredName>`_ records:

   * ``FN`` ↔ display name
   * ``N`` ↔ prefix, given name, middle name, family name, suffix
   * ``X-PHONETIC-FIRST-NAME`` ↔ phonetic given name
   * ``X-PHONETIC-MIDDLE-NAME`` ↔ phonetic middle name
   * ``X-PHONETIC-LAST-NAME`` ↔ phonetic first name

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Nickname <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Nickname>`_ records:

   * ``NICKNAME`` ↔ nick name (types are mapped as ``TYPE`` x-values)

Phone number
^^^^^^^^^^^^

vCard ``TEL`` properties are mapped to `ContactsContract.CommonDataKinds.Phone <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Phone>`_ records (phone number).

|vCard_types_mapped|

Email address
^^^^^^^^^^^^^

vCard ``EMAIL`` properties are mapped to `ContactsContract.CommonDataKinds.Email <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Email>`_ records (email address).

|vCard_types_mapped|

Photo
^^^^^

vCard ``PHOTO`` properties are mapped to `ContactsContract.CommonDataKinds.Photo <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Photo>`_ records.

Because of `Android limitations <https://code.google.com/p/android/issues/detail?id=226875>`_, contact photos with more than 1 MB can't be
stored in the Android contacts provider, so DAVx⁵ has to resize large vCard photos to the values given by
`CONTENT_MAX_DIMENSIONS_URI <https://developer.android.com/reference/android/provider/ContactsContract.DisplayPhoto#CONTENT_MAX_DIMENSIONS_URI>`_.
This limit does not apply in the other direction (Android → vCard).

Organization
^^^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Organization <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Organization>`_ records:

* ``ORG`` ↔ company, department
* ``TITLE`` ↔ (job) title
* ``ROLE`` ↔ job description

Messenger / SIP address
^^^^^^^^^^^^^^^^^^^^^^^

vCard ``IMPP`` properties are mapped to `ContactsContract.CommonDataKinds.Im <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Im>`_
(messenger account) and – if the URI scheme is ``sip:`` – `ContactsContract.CommonDataKinds.SipAddress <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.SipAddress>`_ (SIP address) records.

|vCard_types_mapped|

When importing a vCard, ``X-SIP`` values are treated like ``IMPP:sip:...`` and stored as SIP address.

Note
^^^^

vCard ``NOTE`` properties are mapped to `ContactsContract.CommonDataKinds.Note <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Note>`_ records (note).

Postal address
^^^^^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.StructuredPostal <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.StructuredPostal>`_ records:

* ``ADR`` ↔ street address, p/o box, extended address, locality, region, postal code, country, vCard 4: formatted address
* ``LABEL`` ↔ vCard3: formatted address

|vCard_types_mapped|

If a vCard doesn't contain a formatted address, it will be generated by DAVx⁵ in this format:

.. code-block:: none

   street po.box (extended)
   postcode city
   region
   COUNTRY

Web site
^^^^^^^^

vCard ``URL`` properties are mapped to `ContactsContract.CommonDataKinds.Website <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Website>`_ records (Web site).

|vCard_types_mapped|

Event/date
^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Event <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Event>`_ records:

* ``BDAY`` ↔ birthday
* ``ANNIVERSARY`` ↔ anniversary

Partial dates without year are supported.

Relation
^^^^^^^^

vCard ``RELATED`` properties are mapped to `ContactsContract.CommonDataKinds.Relation <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Relation>`_ records (relation).

Not all vCard values have a corresponding Android value and vice versa. Custom relation names are supported.

.. _contact-groups:

Contact groups
^^^^^^^^^^^^^^

If the *Groups are per-contact categories* method is set in the account settings, DAVx⁵ will match contact groups
to ``CATEGORIES``. For instance, when a contact is in the groups "Friends" and "Family", this property will be added: ``CATEGORIES:Friends,Family``.

If the *Groups are separate vCards* method is set in the account settings, DAVx⁵ will use

* ``KIND`` (or ``X-ADDRESSBOOKSERVER-KIND`` if the server doesn't support VCard 4) to distinguish between contacts and contact groups, and
* ``MEMBER`` (or ``X-ADDRESSBOOKSERVER-MEMBER`` if the server doesn't support VCard 4) to store contact group members.


.. _custom-labels:

Custom labels
^^^^^^^^^^^^^

For some properties, custom labels are supported by vCard property groups. For custom labels, the ``X-ABLABEL`` property is used like that:

.. code-block:: none

   BEGIN:VCARD
   ...
   davdroid1.TEL:+123456
   davdroid1.X-ABLABEL:My Custom Phone
   davdroid2.EMAIL:test@example.com
   davdroid2.X-ABLABEL:My Custom Email Address
   ...
   END:VCARD

In this example, the phone number *+123456* is `grouped together <https://tools.ietf.org/html/rfc6350#page-8>`_ with the custom label "My Custom Phone" and the email address *test@example.com* is labelled "My Custom Email Address".

Unknown properties
^^^^^^^^^^^^^^^^^^

Contact properties which are not processed by DAVx⁵ (like ``X-`` properties) are retained. When importing a vCard, DAVx⁵ saves all unknown properties.
When the respective contact is modified and DAVx⁵ generates the vCard again, it starts with all unknown properties and then adds the known ones.

Protected properties
^^^^^^^^^^^^^^^^^^^^

These vCard properties are processed/generated by DAVx⁵ and cannot be changed by users:

* ``PRODID`` is set to the DAVx⁵ identifier
* ``UID`` is used to identify a vCard (for new vCards, a random UUID will be generated)
* ``REV`` is set to the current time when generating a vCard
* ``SOURCE`` is removed because it doesn't apply anymore as soon as DAVx⁵ generates the vCard
* ``LOGO``, ``SOUND`` are removed because retaining them might cause out-of-memory errors



Supported event fields
----------------------

Events are stored as `CalendarContract.Events <https://developer.android.com/reference/android/provider/CalendarContract.Events>`_
in the Android calendar provider. These iCalendar properties are mapped to Android fields:

* ``SUMMARY`` ↔ title
* ``LOCATION`` ↔ event location
* ``DESCRIPTION`` ↔ description
* ``COLOR`` ↔ event color (only if enabled in DAVx⁵ account settings)
* ``DTSTART`` ↔ start date/time, event timezone / all-day event
* ``DTEND``, ``DURATION`` ↔ end date/time, event end timezone / all-day event
* ``CLASS`` ↔ access level
* ``TRANSP`` ↔ availability

All-day events
^^^^^^^^^^^^^^

Events are considered to be all-day events when ``DTSTART`` is a date (and not a time). All-day events

* without end date or
* with an end date that is not after the start date

are stored with a duration of one day for Android compatibility.

Reminders
^^^^^^^^^

``VALARM`` components are mapped to `CalendarContract.Reminders <https://developer.android.com/reference/android/provider/CalendarContract.Reminders>`_ records and vice versa.

Reminder methods (``ACTION``) are mapped to `Android values <https://developer.android.com/reference/android/provider/CalendarContract.RemindersColumns#METHOD>`_ as good as possible.

Recurring events
^^^^^^^^^^^^^^^^

``RRULE``, ``RDATE``, ``EXRULE`` and ``EXDATE`` values are stored in the respective Android event fields. The Android calendar provider uses these fields to calculcate the instances of a recurring event, which are then saved as CalendarContract.Instances so that calendar apps can access them.

Exceptions of recurring events are identified by ``RECURRENCE-ID``. DAVx⁵ inserts exceptions as separate event records with ``ORIGINAL_SYNC_ID`` set to the ``SYNC_ID`` of the recurring event and ``ORIGINAL_TIME`` set to the ``RECURRENCE-ID`` value.

.. note::

   DAVx⁵ is not responsible for calculating the instances of a recurring event.
   It only provides ``RRULE``, ``RDATE``, ``EXRULE``, ``EXDATE`` and a list of exceptions to the Android calendar provider.


Group-scheduled events
^^^^^^^^^^^^^^^^^^^^^^

``ATTENDEE`` properties are mapped to `CalendarContract.AttendeesColumns <https://developer.android.com/reference/android/provider/CalendarContract.AttendeesColumns>`_ records and vice versa.

Events with at least one attendee are considered to be group-scheduled events. Only for group-scheduled events, the ``ORGANIZER`` property

* is imported from iCalendars to the Android event so that only the organizer can edit a group-scheduled event,
* is exported from the Android event to the iCalendar.

When you add attendees to an event, DAVx⁵ sets the ``RSVP=TRUE`` property for the attendees, which means that a
response is expected. If supported by the server, the server sends invitations to the attendees (for instance, by email).
DAVx⁵ doesn't send invitation emails on its own.

.. note:: DAVx⁵ doesn't implement :term:`CalDAV Scheduling` because the Android calendar provider doesn't have fields for it.
   Very basic operations like managing attendees (when being organizer of an event) or setting your own availability should
   work.

Time zones
^^^^^^^^^^

Thanks to `ical4j <https://github.com/ical4j/ical4j>`_, DAVx⁵ is able to really process time zone definitions of events
(``VTIMEZONE``). If a certain time zone is referenced by identifier but ``VTIMEZONE`` component is provided,
DAVx⁵ uses the `default time zone definitions from ical4j (Olson DB) <https://github.com/ical4j/ical4j/wiki/Timezones>`_.

When an iCalendar references a time zone which is not available in Android, DAVx⁵ tries to find an available time zone
with (partially) matching name. If no such time zone is found, the system default time zone is used. The original value will
be shifted to the available time zone.

For instance, if an event has a start time of *10:00 Custom Time Zone*, DAVx⁵ will
use the *Custom Time Zone* ``VTIMEZONE`` to calculate the corresponding time in the system default time zone,
let's say 12:00 *Europe/Vienna*, and then save the event as 12:00 *Europe/Vienna*.

.. warning::

   Because the Android calendar provider can only process events with time zones which are available in Android, recurring events in time zones which are not available in Android and their exceptions may not be expanded correctly.

Event classification
^^^^^^^^^^^^^^^^^^^^

iCalendar `event classification <https://tools.ietf.org/html/rfc5545#section-3.8.1.3>`_ is mapped to
`Android's ACCESS_LEVEL <https://developer.android.com/reference/android/provider/CalendarContract.EventsColumns#ACCESS_LEVEL>`_ like that:

* no ``CLASS`` → ``ACCESS_LEVEL`` = ``ACCESS_DEFAULT`` ("server default")
* ``CLASS:PUBLIC`` → ``ACCESS_LEVEL`` = ``ACCESS_PUBLIC`` ("public")
* ``CLASS:PRIVATE`` → ``ACCESS_LEVEL`` = ``ACCESS_PRIVATE`` ("private")
* ``CLASS:CONFIDENTIAL`` → ``ACCESS_LEVEL`` = ``ACCESS_CONFIDENTIAL`` (currently not supported by many calendar apps, which will reset the access level to ``ACCESS_DEFAULT`` or ``ACCESS_PRIVATE`` when the event is edited); additionally, ``CONFIDENTIAL`` is stored as *original value*
* other ``CLASS`` value (x-name or iana-token) → ``ACCESS_LEVEL`` = ``ACCESS_PRIVATE``; additionally, the value is stored as *original value*

In the other direction, the locally stored access level is mapped to ``CLASS`` like that:

* ``ACCESS_LEVEL`` = ``ACCESS_PUBLIC`` ("public") → ``CLASS:PUBLIC``
* ``ACCESS_LEVEL`` = ``ACCESS_PRIVATE`` ("private") → ``CLASS:PRIVATE``
* ``ACCESS_LEVEL`` = ``ACCESS_CONFIDENTIAL`` ("confidential", if available in calendar app) → ``CLASS:CONFIDENTIAL``
* ``ACCESS_LEVEL`` = ``ACCESS_DEFAULT`` ("server default") →

  - if there is an *original value*: use that value
  - no ``CLASS`` otherwise (same as ``PUBLIC``)

Categories
^^^^^^^^^^

.. versionadded:: 2.6.2
   In earlier versions, event categories were treated as unknown properties (see below).

iCalendar ``CATEGORIES`` are mapped from/to `exended properties <https://developer.android.com/reference/kotlin/android/provider/CalendarContract.ExtendedProperties>`_
with these fields:

* ``name`` = ``categories``
* ``value`` = list of category names, separated by backslash (``\``), for example: ``Cat A\Cat B\Cat C``. If a
  category name contains a backslash, the backslash will be dropped silenty.

This is the same format as it `is used by the AOSP ActiveSync Exchange sync adapter <https://android.googlesource.com/platform/packages/apps/Exchange/+/refs/tags/android-6.0.1_r31/src/com/android/exchange/eas/EasSyncCalendar.java#107>`_.

Unknown properties
^^^^^^^^^^^^^^^^^^

.. _event-unknown-properties:

iCalendar properties which are not processed by DAVx⁵ (like ``X-`` properties) are retained (unless they're larger than ≈ 25 kB).
When importing an iCalendar, DAVx⁵ saves all unknown event properties as extended property rows.
When the respective event is modified and DAVx⁵ generates the iCalendar again, it will include all unknown properties.

Protected properties
^^^^^^^^^^^^^^^^^^^^

These iCalendar properties are processed/generated by DAVx⁵ and cannot be changed by users:

* ``PRODID`` is set to the DAVx⁵ identifier
* ``UID`` is used to identify an iCalendar (for new iCalendars, a random UUID will be generated)
* ``RECURRENCE-ID`` is used to identify certain instances of recurring events
* ``SEQUENCE`` is increased when an iCalendar is modified
* ``DTSTAMP`` is set to the current time when generating an iCalendar


Supported task fields
---------------------

DAVx⁵ synchronizes ``VTODO`` components (= tasks) with the OpenTasks provider ``org.dmfs.tasks``, so
`OpenTasks <https://play.google.com/store/apps/details?id=org.dmfs.tasks>`_ must be installed for task synchronization.

To use some features (for instance, to see subtasks as indented task) in the UI, you may need
another tasks app that is able to access the OpenTasks provider, like `aCalendar+ <https://play.google.com/store/apps/details?id=org.withouthat.acalendarplus>`_.

These properties are synchronized by DAVx⁵:

* ``UID``
* ``SUMMARY``, ``DESCRIPTION``
* ``LOCATION``
* ``GEO``
* ``URL``
* ``ORGANIZER``
* ``PRIORITY``
* ``COMPLETED``, ``PERCENT-COMPLETE``
* ``STATUS``
* ``CREATED``, ``LAST-MODIFIED``
* ``DTSTART``, ``DUE``, ``DURATION``
* ``RDATE``, ``EXDATE``, ``RRULE``
* ``CATEGORIES``
* ``RELATED-TO`` (used for subtasks)

Unknown properties
^^^^^^^^^^^^^^^^^^

See :ref:`unknown properties of events <event-unknown-properties>`.



TLS stack (protocol versions, ciphers)
======================================

.. versionadded:: 2.5
  DAVx⁵ uses `Conscrypt <https://github.com/google/conscrypt/blob/master/CAPABILITIES.md>`_ to support modern TLS protocol versions and ciphers
  even on older devices. Both your client (DAVx⁵) and the CalDAV/CardDAV server must share at least one cipher, otherwise a ``SSLProtocolException`` will occur.


API / integration
=================

Launching the DAVx⁵ login screen
--------------------------------

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

Intent data (URI): login flow entry point (``<server>/index.php/login/flow``). Intent extras:

+------------+--------+---------------------------------------------------------------------------+
| extra name | type   | description                                                               |
+============+========+===========================================================================+
| loginFlow  | Int    | set to 1 to indicate Login Flow                                           |
+------------+--------+---------------------------------------------------------------------------+
| davPath    | String | CalDAV/CardDAV base URL; will be appended to server URL returned by Login |
|            |        | Flow without further processing (e.g. ``/remote.php/dav``)                |
+------------+--------+---------------------------------------------------------------------------+

For compatibility with old DAVx⁵ versions, you can use both methods at the same time. Old DAVx⁵ versions will
use the ``url``, ``username``, ``password`` extras, while new versions will see the ``loginFlow``
extra and switch to the Login Flow method.
