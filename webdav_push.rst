
===========
WebDAV-Push
===========

By default, DAVx⁵ has to query the server in fixed intervals to see whether there is new data to be loaded. This is inconvenient, as it can take a while before new data is available on the device.

.. versionadded:: 4.4.10
   To solve this, we have developed `WebDAV-Push <https://github.com/bitfireAT/webdav-push/>`_ so that DAVx⁵ can receive a notification from the server when new data is available.

.. note::
   WebDAV-Push support is still experimental and subject of continuous improvement.

Push messages are `always end-to-end encrypted <https://unifiedpush.org/users/faq/#q-is-unifiedpush-secure>`_
so that push transports can't read their content.


Server-side requirements
========================

The server needs to support and be configured for WebDAV-Push.

Currently, there's only an experimental implementation of WebDAV-Push for Nextcloud:
`nc_ext_dav_push <https://github.com/bitfireAT/nc_ext_dav_push/>`_. If you know of any
other CalDAV/CardDAV server that supports WebDAV-Push or if you're interested
in implementing it for some server, please `let us know <https://www.davx5.com/support>`_.

You just need to install the `dav_push extension <https://apps.nextcloud.com/apps/dav_push>`_
on the Nextcloud server. However, due to the experimental nature of the app, it's possible
that the version in the app store is outdated or not working. In that case, you can
fetch the latest source code directly from Github into the respective app directory.

Currently, the dav_push app doesn't need any configuration. It automatically creates a
VAPID server key.


DAVx⁵ requirements
==================

You need DAVx⁵ version 4.4.10 or higher to use WebDAV-Push.

A `UnifiedPush distributor <https://unifiedpush.org/users/distributors/>`_ or Google FCM
(Google Play services) must be available on the device.

You need to choose the distributor you like to use from DAVx⁵ settings. Scroll to the bottom, and choose your distributor.


How to use
==========

#. Make sure your server supports WebDAV-Push (= install the ``dav_push`` extension on Nextcloud;
   fetch latest version from Github if Push doesn't work).
#. Select a distributor in DAVx⁵ settings.
#. Refresh the collection list in the respective DAVx⁵ account (to fetch the respective WebDAV properties).
   To do so, open the account in DAVx⁵, select a tab (for instance, CalDAV or CardDAV) and then
   use the respective action menu (⋮) entry to choose "Refresh collection list".
#. Tap on a collection to view its details. It should say *Server offers Push support* when the
   collection is not selected for synchronization, or *Push support: subscribed* (at a date/time that is always
   more recent than about a day) when the collection is selected for synchronization.
#. Make some change on a subscribed collection, for instance change/add an event. It doesn't matter
   which client you use to do this. However for testing, we recommend to do it directy in the
   Nextcloud Web interface.
#. Wait some seconds and watch your calendar on the DAVx⁵ device. The push notification should
   immediately arrive and the calendar should be updated within a few seconds. In case that DAVx⁵
   doesn't have the quota to synchronize in the background, it will show a notification.


Troubleshooting
===============

If DAVx⁵ doesn't subscribe or subscriptions became stale, because for instance the server has
switched to a new VAPID key, you can manually reset the subscriptions:

#. Open the DAVx⁵ app settings.
#. Disable UnifiedPush support (deletes all subscriptions).
#. Enable UnifiedPush support again. DAVx⁵ will create and register subscriptions again.
