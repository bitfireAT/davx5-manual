
========================
Accounts and Collections
========================


What is a DAVx⁵ account?
========================

A DAVx⁵ account represents a connection to a CalDAV/CardDAV service, which can contain address books, calendars and task lists. Most services provide both CalDAV and CardDAV together (technically, this is when both CalDAV and CardDAV can be detected using the same starting point). In this case, you need only one DAVx⁵ account. However, you can also create multiple DAVx⁵ accounts for separate CalDAV/CardDAV services.

When you add a DAVx⁵ account, you need either an email address or a base URL which is used as a starting point for service discovery. You can find the required configuration / base URL in your server manual or admin information. See our tested services for a list of servers/services and how they're used with DAVx⁵.


How does service discovery work?
================================

DAVx⁵ supports both `service location discovery by SRV/TXT records <https://tools.ietf.org/html/rfc6764>`_ and
`well-known URLs <https://tools.ietf.org/html/rfc5785>`_. To use CalDAV and CardDAV in one DAVx⁵ account, make sure these redirects are present on your server:

:code:`/.well-known/caldav` → CalDAV service path (302 Found), e.g. :code:`/remote.php/caldav/`
:code:`/.well-known/carddav` → CardDAV service path (302 Found), e.g. :code:`/remote.php/carddav/`

.. note::
   If these redirects are configured correctly, you can use the root URL :code:`http(s)://your.server.example/`
   without any additional paths as the base URL in DAVx⁵.


What is the Base URL?
=====================

When logging in by URL, DAVx⁵ asks for the *Base URL*. This can be:

* the root URL (/) of your server if well-known URLs are configured (recommended), or
* a valid CalDAV URL, i.e.

  - an URL that provides a CalDAV current-user-principal, or
  - an URL that provides calendar-home-set, or
  - a calendar URL (resourcetype: calendar), and/or

* a valid CardDAV URL, i.e.

  - an URL that provides a CardDAV current-user-principal, or
  - an URL that provides addressbook-home-set, or
  - an addressbook URL (resourcetype: addressbook).

So, DAVx⁵ will query the base URL for both CalDAV and CardDAV and use whatever it finds. If CalDAV and CardDAV are separated on your server and well-known URLs are not configured, you'll have to create two DAVx⁵ accounts: one for CalDAV (use the CalDAV URL as base URL) and one for CardDAV (use the CardDAV URL as base URL).


Address book accounts
=====================

In Android, contacts are assigned directly to accounts, without the possibility of having multiple address books (in contrast to events and tasks, where calendars and task lists are available as grouping entities).

So, DAVx⁵ defines a new account type "DAVx⁵ address book" beside the regular DAVx⁵ account type in order to provide support for multiple address books with one regular DAVx⁵ account.
**The address book account names will be shown in your Contacts app as possible destinations for contacts.**

.. figure:: images/manual_system_accounts_with_davx5_account.png
   :alt: Screenshot: Android Settings / Accounts with configured DAVx⁵ account
   :target: _images/manual_system_accounts_with_davx5_account.png

   Android Settings / Accounts with configured DAVx5 account

For every synchronized CardDAV address book, a DAVx⁵ address book account is created automatically. When synchronizing "address books" of the main DAVx⁵ account, DAVx⁵ starts synchronizing contacts for every DAVx⁵ address book account.

.. warning::
   Do not create, modify or delete DAVx⁵ address book accounts directly. Use the DAVx⁵ app to configure synchronized address books instead.

DAVx⁵ address books accounts are named after the CardDAV address book which they are connected to, plus a hash of the URL, for instance: *My addresses (UA)*
(in this case, the hash is *UA*). The hash is necessary when there are two address books with the same name.


Account names
=============

When creating a DAVx⁵ account, you have to enter the account name. This account name (which will be shown in the DAVx⁵ accounts list and in Android Settings / Accounts / DAVx⁵) is used as the
:code:`ORGANIZER` email address for group-scheduled events. So:

.. note:: If possible, always use your email address as DAVx⁵ account name.

During initial resource detection, DAVx⁵ queries the :code:`calendar-user-address-set` of the CalDAV principal URL and suggests the first email address as account name. If the property is not available, the user name/email address used for login is suggested as account name.

The account name must be unique, i.e. you can't have two DAVx⁵ accounts with the same account name. This could be relevant if you need separate accounts for CalDAV and CardDAV. In this case, use your email address as account name for the account to be used for CalDAV and another account name (like "My CardDAV Server") for the account to be used for CardDAV.


.. _refresh-collections:

Refreshing the collection list
==============================

**To detect new and changed address books/calendars, use the respective menu entry in the DAVx⁵ account activity.**
When you refresh the collection list, DAVx⁵ will search the home sets for new collections and check the already known collections (whether they are still there and whether properties like name and color have been changed). These functions will only be available if a principal and/or homeset URL can be found for the respective protocol.

The collections and their properties (name, color, read-only) are not synchronized to the Android system immediately, but as soon as synchronization is triggered.

For example, if a calendar's name and color have been changed on the server:

* Choose "Refresh calendar list" in the DAVx⁵ account. Now the new name and color will appear in the DAVx⁵ account, but not yet in the calendar app (because there was no synchronization yet).
* As soon as synchronization is started, the changed properties (name, color) are commited to the Android calendar provider. Calendar apps will now show the new name and color.


Read-only collections
=====================

There are two ways to restrict synchronization to one direction (only server to Android):

#. DAVx⁵ follows the WebDAV permissions from the server. If you don't have write permissions for a specific collection, it will be treated as read-only.
#. If you have write permissions for a specific collection, you can force read-only mode ("one-way sync") for this collection using the action overflow. (Note that you have to synchronize a collection before forced read-only takes effect.)

Regardless of why a collection is read-only, it will be shown as read-only (⛔) in the DAVx⁵ collection list.

.. note:: Android doesn't have native support for read-only address books. To emulate this feature, DAVx⁵ reverts local changes at every synchronization. You can still edit your contacts in the Contacts app, but all changes will be reverted when the next synchronization is run.

Read-only calendars will be marked as read-only in the Android calendar provider, so that calendar apps won't be able to create/modify/delete events in such calendars anymore. Currently, there's no read-only support for task lists.


Creating/deleting collections on/from the server
================================================

You can also manage collections with DAVx⁵.

To create a collection, use the respective menu entry in the DAVx⁵ account next to "CalDAV" or "CardDAV". For instance, choose "Create new address book" next to "CardDAV" to create a new address book on the server. (This will only work if it's supported by the server, which is not mandatory.) The same applies to calendars and task lists.

To delete a collection, choose "Delete collection" next to the respective collection in the DAVx⁵ account. After your confirmation, this will delete the collection and all its entries on the server, so be careful.


Webcal integration
==================

DAVx⁵ recognizes Webcal calendars in the calendar home set which are published with
:code:`resourcetype: subscribed` and shows them in the DAVx⁵ account activity. If you select such a Webcal collection for synchronization, DAVx⁵ passes
the URL to an installed Webcal-capable app like `ICSx⁵ <https://icsx5.bitfire.at>`_ so that this app can subscribe to the calendar.

If you're using ICSx⁵, DAVx⁵ can determine whether a Webcal collection is currently subscribed and can also remove the subscription again.

