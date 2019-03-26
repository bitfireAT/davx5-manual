
=====================
Technical information
=====================


Synchronization
===============

Synchronization logic
---------------------

These steps are performed when synchronizing a collection:

#. Prepare synchronization: prepare local collection, settings etc.
#. Query capabilities with HTTP :code:`PROPFIND`:

   * determine whether the server supports Collection Synchronization
   * CardDAV: determine whether the server supports vCard 4
   * fetch current :code:`CTag` and :code:`sync-token`

#. Process locally deleted resources: if a local resource is flagged as deleted,

   * delete it on the server (HTTP :code:`DELETE` with :code:`If-Match` set to last known :code:`ETag` to avoid deleting resources which have been changed on the server in the meanwhile) and
   * then remove it locally

#. Upload locally modified ("dirty") resources:

   * Assign a random UID and resource name to new resources; prepare contact group and recurring events, if necessary
   * If no previous :code:`ETag` of the resource is known (i.e. the resource has not been uploaded yet), use HTTP :code:`PUT` with :code:`If-None-Match: *` to avoid overwriting a possibly existing resource with the same name
   * If a previous :code:`ETag` of the resource is known, use HTTP :code:`PUT` with :code:`If-Match` set to last known :code:`ETag` to avoid overwriting changes which happend on the server in the meanwhile
   * remember returned :code:`ETag` as last known :code:`ETag`; otherwise reset last known :code:`ETag`

#. Choose sync algorithm (:code:`PROPFIND`/:code:`REPORT` vs. Collection Synchronization):

   * CardDAV: use Collection Synchronization if supported by server, :code:`PROPFIND` otherwise
   * CalDAV events: use Collection Synchronization if supported by server and past time event limit is disabled, :code:`REPORT calendar-query` otherwise
   * CalDAV tasks: use :code:`REPORT calendar-query`

#. Check whether further synchronization is needed. Only continue when:

   * modifications (uploads/deletions) have been sent to the server, or
   * the sync state (:code:`CTag`/:code:`sync-token`) of the collection has changed since last sync, or
   * the :code:`PROPFIND`/:code:`REPORT` algorithm shall be used and the sync has been initiated manually

#. Continue with chosen sync algorithm (see below).


Sync algorithm: PROPFIND/REPORT
-------------------------------

#. Unset *present remotely* flag for all resources
#. List and process remote resources (only names and :code:`ETag`) using :code:`PROPFIND` or :code:`REPORT` (see above)

   * Download resources which have been added/modififed remotely in bunches using :code:`GET` or :code:`REPORT addressbook-multiget/calendar-multiget` into the local storage.
   * Set *present remotely* flag for all received resources.

#. Locally delete all resources which are not flagged as *present remotely*
#. Post-processing: clean up empty contact groups etc.
#. Save sync state (:code:`CTag`/:code:`sync-token`)


Sync algorithm: Collection Synchronization
------------------------------------------

#. Was a previous *initial sync* aborted and is now being continued? → If yes, set *initial sync*.
#. Do we have a previous :code:`sync-token`? → If no, set *initial sync*.
#. List and process changes since last :code:`sync-token` (or all records if no previous :code:`sync-token` is known) using :code:`REPORT sync-collection`.

   * Download resources which have been added/modififed remotely in bunches using :code:`GET` or :code:`REPORT addressbook-multiget/calendar-multiget` into the local storage.
   * Set *present remotely* flag for all received resources.

#. If the requested :code:`sync-token` was invalid:

   * forget the :code:`sync-token`
   * reset *present remotely* flags of all local resources
   * set *initial sync* and continue with 3.

#. Save :code:`sync-token`.
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

* DAVx⁵ relies on HTTP :code:`ETag` to determine whether a resource has been changed on the server.
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

   * :code:`FN` ↔ display name
   * :code:`N` ↔ prefix, given name, middle name, family name, suffix
   * :code:`X-PHONETIC-FIRST-NAME` ↔ phonetic given name
   * :code:`X-PHONETIC-MIDDLE-NAME` ↔ phonetic middle name
   * :code:`X-PHONETIC-LAST-NAME` ↔ phonetic first name

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Nickname <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Nickname>`_ records:

   * :code:`NICKNAME` ↔ nick name (types are mapped as :code:`TYPE` x-values)

Phone number
^^^^^^^^^^^^

vCard :code:`TEL` properties are mapped to `ContactsContract.CommonDataKinds.Phone <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Phone>`_ records (phone number).

|vCard_types_mapped|

Email address
^^^^^^^^^^^^^

vCard :code:`EMAIL` properties are mapped to `ContactsContract.CommonDataKinds.Email <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Email>`_ records (email address).

|vCard_types_mapped|

Photo
^^^^^

vCard :code:`PHOTO` properties are mapped to `ContactsContract.CommonDataKinds.Photo <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Photo>`_ records.

Because of `Android limitations <https://code.google.com/p/android/issues/detail?id=226875>`_, contact photos with more than 1 MB can't be
stored in the Android contacts provider, so DAVx⁵ has to resize large vCard photos to the values given by
`CONTENT_MAX_DIMENSIONS_URI <https://developer.android.com/reference/android/provider/ContactsContract.DisplayPhoto#CONTENT_MAX_DIMENSIONS_URI>`_.
This limit does not apply in the other direction (Android → vCard).

Organization
^^^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Organization <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Organization>`_ records:

* :code:`ORG` ↔ company, department
* :code:`TITLE` ↔ (job) title
* :code:`ROLE` ↔ job description

Messenger / SIP address
^^^^^^^^^^^^^^^^^^^^^^^

vCard :code:`IMPP` properties are mapped to `ContactsContract.CommonDataKinds.Im <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Im>`_
(messenger account) and – if the URI scheme is :code:`sip:` – `ContactsContract.CommonDataKinds.SipAddress <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.SipAddress>`_ (SIP address) records.

|vCard_types_mapped|

When importing a vCard, :code:`X-SIP` values are treated like :code:`IMPP:sip:...` and stored as SIP address.

Note
^^^^

vCard :code:`NOTE` properties are mapped to `ContactsContract.CommonDataKinds.Note <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Note>`_ records (note).

Postal address
^^^^^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.StructuredPostal <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.StructuredPostal>`_ records:

* :code:`ADR` ↔ street address, p/o box, extended address, locality, region, postal code, country, vCard 4: formatted address
* :code:`LABEL` ↔ vCard3: formatted address

|vCard_types_mapped|

If a vCard doesn't contain a formatted address, it will be generated by DAVx⁵ in this format:

.. code-block:: none

   street po.box (extended)
   postcode city
   region
   COUNTRY

Web site
^^^^^^^^

vCard :code:`URL` properties are mapped to `ContactsContract.CommonDataKinds.Website <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Website>`_ records (Web site).

|vCard_types_mapped|

Event/date
^^^^^^^^^^

These vCard properties are mapped to `ContactsContract.CommonDataKinds.Event <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Event>`_ records:

* :code:`BDAY` ↔ birthday
* :code:`ANNIVERSARY` ↔ anniversary

Partial dates without year are supported.

Relation
^^^^^^^^

vCard :code:`RELATED` properties are mapped to `ContactsContract.CommonDataKinds.Relation <https://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.Relation>`_ records (relation).

Not all vCard values have a corresponding Android value and vice versa. Custom relation names are supported.

.. _contact-groups:

Contact groups
^^^^^^^^^^^^^^

If the *Groups are per-contact categories* method is set in the account settings, DAVx⁵ will match contact groups
to :code:`CATEGORIES`. For instance, when a contact is in the groups "Friends" and "Family", this property will be added: :code:`CATEGORIES:Friends,Family`.

If the *Groups are separate vCards* method is set in the account settings, DAVx⁵ will use

* :code:`KIND` (or :code:`X-ADDRESSBOOKSERVER-KIND` if the server doesn't support VCard 4) to distinguish between contacts and contact groups, and
* :code:`MEMBER` (or :code:`X-ADDRESSBOOKSERVER-MEMBER` if the server doesn't support VCard 4) to store contact group members.


.. _custom-labels:

Custom labels
^^^^^^^^^^^^^

For some properties, custom labels are supported by vCard property groups. For custom labels, the :code:`X-ABLABEL` property is used like that:

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

Contact properties which are not processed by DAVx⁵ (like :code:`X-` properties) are retained. When importing a vCard, DAVx⁵ saves all unknown properties.
When the respective contact is modified and DAVx⁵ generates the vCard again, it starts with all unknown properties and then adds the known ones.

Protected properties
^^^^^^^^^^^^^^^^^^^^

These vCard properties are processed/generated by DAVx⁵ and cannot be changed by users:

* :code:`PRODID` is set to the DAVx⁵ identifier
* :code:`UID` is used to identify a vCard (for new vCards, a random UUID will be generated)
* :code:`REV` is set to the current time when generating a vCard
* :code:`SOURCE` is removed because it doesn't apply anymore as soon as DAVx⁵ generates the vCard
* :code:`LOGO`, :code:`SOUND` are removed because retaining them might cause out-of-memory errors



Supported event fields
----------------------

Events are stored as `CalendarContract.Events <https://developer.android.com/reference/android/provider/CalendarContract.Events>`_
in the Android calendar provider. These iCalendar properties are mapped to Android fields:

* :code:`SUMMARY` ↔ title
* :code:`LOCATION` ↔ event location
* :code:`DESCRIPTION` ↔ description
* :code:`COLOR` ↔ event color (only if enabled in DAVx⁵ account settings)
* :code:`DTSTART` ↔ start date/time, event timezone / all-day event
* :code:`DTEND`, :code:`DURATION` ↔ end date/time, event end timezone / all-day event
* :code:`CLASS` ↔ access level
* :code:`TRANSP` ↔ availability

All-day events
^^^^^^^^^^^^^^

Events are considered to be all-day events when :code:`DTSTART` is a date (and not a time). All-day events

* without end date or
* with an end date that is not after the start date

are stored with a duration of one day for Android compatibility.

Reminders
^^^^^^^^^

:code:`VALARM` components are mapped to `CalendarContract.Reminders <https://developer.android.com/reference/android/provider/CalendarContract.Reminders>`_ records and vice versa.

Reminder methods (:code:`ACTION`) are mapped to `Android values <https://developer.android.com/reference/android/provider/CalendarContract.RemindersColumns#METHOD>`_ as good as possible.

Recurring events
^^^^^^^^^^^^^^^^

:code:`RRULE`, :code:`RDATE`, :code:`EXRULE` and :code:`EXDATE` values are stored in the respective Android event fields. The Android calendar provider uses these fields to calculcate the instances of a recurring event, which are then saved as CalendarContract.Instances so that calendar apps can access them.

Exceptions of recurring events are identified by :code:`RECURRENCE-ID`. DAVx⁵ inserts exceptions as separate event records with :code:`ORIGINAL_SYNC_ID` set to the :code:`SYNC_ID` of the recurring event and :code:`ORIGINAL_TIME` set to the :code:`RECURRENCE-ID` value.

.. note::

   DAVx⁵ is not responsible for calculating the instances of a recurring event.
   It only provides :code:`RRULE`, :code:`RDATE`, :code:`EXRULE`, :code:`EXDATE` and a list of exceptions to the Android calendar provider.


Group-scheduled events
^^^^^^^^^^^^^^^^^^^^^^

:code:`ATTENDEE` properties are mapped to `CalendarContract.AttendeesColumns <https://developer.android.com/reference/android/provider/CalendarContract.AttendeesColumns>`_ records and vice versa.

Events with at least one attendee are considered to be group-scheduled events. Only for group-scheduled events, the :code:`ORGANIZER` property

* is imported from iCalendars to the Android event so that only the organizer can edit a group-scheduled event,
* is exported from the Android event to the iCalendar.

When you add attendees to an event, DAVx⁵ sets the :code:`RSVP=TRUE` property for the attendees, which means that a
response is expected. If supported by the server, the server sends invitations to the attendees (for instance, by email).
DAVx⁵ doesn't send invitation emails on its own.

.. note:: DAVx⁵ doesn't implement :term:`CalDAV Scheduling` because the Android calendar provider doesn't have fields for it.
   Very basic operations like managing attendees (when being organizer of an event) or setting your own availability should
   work.

Time zones
^^^^^^^^^^

Thanks to `ical4j <https://github.com/ical4j/ical4j>`_, DAVx⁵ is able to really process time zone definitions of events
(:code:`VTIMEZONE`). If a certain time zone is referenced by identifier but :code:`VTIMEZONE` component is provided,
DAVx⁵ uses the `default time zone definitions from ical4j (Olson DB) <https://github.com/ical4j/ical4j/wiki/Timezones>`_.

When an iCalendar references a time zone which is not available in Android, DAVx⁵ tries to find an available time zone
with (partially) matching name. If no such time zone is found, the system default time zone is used. The original value will
be shifted to the available time zone.

For instance, if an event has a start time of *10:00 Custom Time Zone*, DAVx⁵ will
use the *Custom Time Zone* :code:`VTIMEZONE` to calculate the corresponding time in the system default time zone,
let's say 12:00 *Europe/Vienna*, and then save the event as 12:00 *Europe/Vienna*.

.. warning::

   Because the Android calendar provider can only process events with time zones which are available in Android, recurring events in time zones which are not available in Android and their exceptions may not be expanded correctly.

Event classification
^^^^^^^^^^^^^^^^^^^^

CalDAV `event classification <https://tools.ietf.org/html/rfc5545#section-3.8.1.3>`_ is mapped to
`Android's ACCESS_LEVEL <https://developer.android.com/reference/android/provider/CalendarContract.EventsColumns#ACCESS_LEVEL>`_ like that:

* no :code:`CLASS` → :code:`ACCESS_LEVEL` = :code:`ACCESS_DEFAULT` ("server default")
* :code:`CLASS:PUBLIC` → :code:`ACCESS_LEVEL` = :code:`ACCESS_PUBLIC` ("public")
* :code:`CLASS:PRIVATE` → :code:`ACCESS_LEVEL` = :code:`ACCESS_PRIVATE` ("private")
* :code:`CLASS:CONFIDENTIAL` → :code:`ACCESS_LEVEL` = :code:`ACCESS_CONFIDENTIAL` (currently not supported by many calendar apps, which will reset the access level to :code:`ACCESS_DEFAULT` or :code:`ACCESS_PRIVATE` when the event is edited); additionally, :code:`CONFIDENTIAL` is stored as *original value*
* other :code:`CLASS` value (x-name or iana-token) → :code:`ACCESS_LEVEL` = :code:`ACCESS_PRIVATE`; additionally, the value is stored as *original value*

In the other direction, the locally stored access level is mapped to :code:`CLASS` like that:

* :code:`ACCESS_LEVEL` = :code:`ACCESS_PUBLIC` ("public") → :code:`CLASS:PUBLIC`
* :code:`ACCESS_LEVEL` = :code:`ACCESS_PRIVATE` ("private") → :code:`CLASS:PRIVATE`
* :code:`ACCESS_LEVEL` = :code:`ACCESS_CONFIDENTIAL` ("confidential", if available in calendar app) → :code:`CLASS:CONFIDENTIAL`
* :code:`ACCESS_LEVEL` = :code:`ACCESS_DEFAULT` ("server default") →

  - if there is an *original value*: use that value
  - no :code:`CLASS` otherwise (same as :code:`PUBLIC`)

Unknown properties
^^^^^^^^^^^^^^^^^^

iCalendar properties which are not processed by DAVx⁵ (like :code:`X-` properties) are retained (unless they're larger than ≈ 25 kB).
When importing an iCalendar, DAVx⁵ saves all unknown properties as
`CalendarContract.ExtendedPropertiesColumns <https://developer.android.com/reference/android/provider/CalendarContract.ExtendedPropertiesColumns>`_ records. 
When the respective event is modified and DAVx⁵ generates the iCalendar again, it will include all unknown properties.

Protected properties
^^^^^^^^^^^^^^^^^^^^

These iCalendar properties are processed/generated by DAVx⁵ and cannot be changed by users:

* :code:`PRODID` is set to the DAVx⁵ identifier
* :code:`UID` is used to identify an iCalendar (for new iCalendars, a random UUID will be generated)
* :code:`RECURRENCE-ID` is used to identify certain instances of recurring events
* :code:`SEQUENCE` is increased when an iCalendar is modified
* :code:`DTSTAMP` is set to the current time when generating an iCalendar


Supported task fields
---------------------

DAVx⁵ synchronizes :code:`VTODO` tasks with the OpenTasks provider :code:`org.dmfs.tasks`, so
`OpenTasks <https://play.google.com/store/apps/details?id=org.dmfs.tasks>`_ must be installed for task synchronization.

These properties are synchronized by DAVx⁵:

* :code:`UID`
* :code:`SUMMARY`, :code:`DESCRIPTION`
* :code:`LOCATION`
* :code:`GEO`
* :code:`URL`
* :code:`ORGANIZER`
* :code:`PRIORITY`
* :code:`COMPLETED`, :code:`PERCENT-COMPLETE`
* :code:`STATUS`
* :code:`CREATED`, :code:`LAST-MODIFIED`
* :code:`DTSTART`, :code:`DUE`, :code:`DURATION`
* :code:`RDATE`, :code:`EXDATE`, :code:`RRULE`

At the moment, other properties of tasks are not retained.



TLS stack (protocol versions, ciphers)
======================================

To establish secure connections with TLS, DAVx⁵ makes use of the Android system TLS stack.
Supported protocol versions (TLS 1.1, 1.2 etc.) and ciphers (for key exchange and encryption, e.g. :code:`TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA`) depend on the used Android version.

Both your client (DAVx⁵ / Android device) and the CalDAV/CardDAV server must share at least one cipher, otherwise a :code:`SSLProtocolException` will occur. For example, if your server requires the most recent ciphers, connecting with older Android versions may not work.

.. seealso::

   See the `Android documentation for a list of supported protocols and ciphers <https://developer.android.com/reference/javax/net/ssl/SSLEngine>`_
   for various Android versions (scroll down to "Default configuration for different Android versions").

**Android versions below 6.0 only:** Not all protocols and ciphers supported by a device are automatically enabled for apps by default. DAVx⁵

* enables SNI,
* disables SSL 3 and enables all supported TLS versions (like TLS 1.2), and
* enables some ciphers considered to be secure (see source code of class :code:`SSLSocketFactoryCompat` for details).

.. note:: 

   Some major browsers bring their own TLS stack and don't rely on the system TLS stack.
   So when testing TLS ciphers, it may work to access an URL with your Android browser, while
   other apps like DAVx⁵ still can't connect to the server.

