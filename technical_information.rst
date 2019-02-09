
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
