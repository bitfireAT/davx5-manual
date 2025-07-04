========
Settings
========

System-wide settings
====================

DAVx⁵ makes use of system proxy settings, unless you use app-wide settings to override them. Only HTTP(S) proxies will be used. System proxy settings can most importantly be found in

* Android APN settings (for mobile network connections; defined by network provider),
* Android WiFi settings (for WiFi connections; defined when adding a WiFi network).

Your network provider may have defined a system proxy in APN settings that you're not aware of. Check in case of networking problems.


App-wide settings
=================

There are some settings that apply to the whole DAVx⁵ app. You can access them in the DAVx⁵ navigation drawer / Settings.

Show debug info
   Shows important information about your device and the current system state, DAVx⁵ accounts (including collection URLs) etc. Use this information to narrow down sync problems systematically. Can be shared to email or other apps.

Verbose logging
   If this option is enabled, DAVx⁵ will create a verbose log file which contain various status messages and the whole HTTP traffic. A permanent notification will appear which you can use to send this log to any app, including email, `Share via HTTP <https://github.com/marcosdiez/shareviahttp>`_, or `Amaze <https://github.com/TeamAmaze/AmazeFileManager>`_'s "Save to file". Use this log file to narrow down sync problems systematically. (You may have to expand the notification to see the "Send" action.) As soon as you turn off verbose logging, the log will be deleted and the notification cancelled.

Override proxy settings
   Allows to override the system proxy settings (see above). If this option is enabled, you have to specify a HTTP(S) proxy which is used instead of the system proxy. Can be used to route all DAVx⁵ traffic through a certain proxy, for instance a Tor proxy.

Distrust system certificates
   If this option is enabled, system-wide CA certificates will not be trusted automatically. In this case, every certificate has to be verified and accepted explicitly.

Reset (un)trusted certificates
   Clears the app key stores where both previously trusted and rejected certificates are saved. If those certificates are encountered again, they will have to be verified again.

App permissions
   Opens the dialog where you can manage DAVx⁵ permissions.

Notification settings
   Allows you to change appearance and priority of various DAVx⁵ notifications.

Reset hints
   When DAVx⁵ is started the first time, there are some setup hints. Use this option to make all hints that have been previously dismissed re-appear (if applicable).

Tasks app
   Select your preferred tasks app. Tasks are only synchronized with the selected app.

UnifiedPush (experimental)
   Select the preferred UnifiedPush `distributor <https://unifiedpush.org/users/distributors/>`_. You can also select
   *FCM (over Google)* (recommended for most users; requires Google Play services).


Account settings
================

.. _account-settings-sync:

Synchronization
---------------

Sync intervals
   Here you can choose the preferred sync interval for address books, calendars and task lists. If you select "only manually", synchronization is only run on explicit user request. In the other cases, the sync interval is saved in Android settings. Android will run synchronization when local data has been changed (normally very quickly, but in some cases, it may take a few seconds or even minutes) and in regular intervals (given that system-wide automatic synchronization is enabled, permissions are granted, battery saving is disabled for DAVx⁵ and there is a network connection).

   .. note:: DAVx⁵ uses a combination of Android's `WorkManager <https://developer.android.com/topic/libraries/architecture/workmanager/>`_ and `sync framework <https://developer.android.com/training/sync-adapters/creating-sync-adapter>`_ to run synchronization. WorkManager is used for scheduled (automatic) and manual synchronization. The sync framework is used when other apps (like calendar apps) initiate synchronization, and to notify DAVx⁵ about local changes (for instance, when you edit an event in your calendar app).

   Keep in mind that low sync intervals need more battery (although only changed records are transmitted). Sync intervals less than 15 minutes are not allowed by Android 7 and later, because it would use too much battery. If you need up-to-date information in your calendar app, use a moderate sync interval (like one hour) and the "Synchronize now" function of your calendar app when you need it.

Sync over WiFi only
   If this option is enabled, DAVx⁵ skips synchronization unless the device is connected to a WiFi network. Android Settings / Accounts will still show a successful sync even when the synchronization has been skipped. Manually forced synchronization will ignore this setting! It is not intended as a security function, but to avoid network connection error notifications.

   Use **WiFi SSID restriction** to restrict synchronization to one or more specific WiFi networks (requires location permission, including background location). List all allowed SSIDs as a comma-separated list, for instance SSID1,ssid2. Hidden networks and SSIDs with commas are not supported.

VPN connectivity
   This setting controls when DAVx⁵ assumes to have a network connection that is suitable to run synchronization. **It does not control which connection is used for actual data transfer** (it will use the default connection that is selected by Android).

   *Non-VPN connection required (recommended)*: DAVx⁵ will not synchronize if there's a VPN connection, but no underlying validated Internet connection. Useful for "always-on" VPNs like Ad blockers etc. which permanently pretend to provide a validated Internet connection, whether it's true or not.

   *VPN counts as Internet connection*: DAVx⁵ is content with a VPN connection that identifies itself as validated Internet connection. Useful if you use a real VPN whose connection status correctly indicates whether the connection can be used for synchronization.

   In this context, the definition of a "validated Internet" connection `is given by Android <https://developer.android.com/reference/android/net/NetworkCapabilities>`_ (``NET_CAPABILITY_INTERNET`` and ``NET_CAPABILITY_VALIDATED``).


Authentication
--------------

User name / password / client certificate
  You can change the credentials used for synchronization at any time here (for instance, when your password has been changed on the server).

  This option is not available when the account uses OAuth for authentication (provider-specific login).

  See :ref:`authentication_methods` for more information about supported authentication methods.

CalDAV
------

Past event time limit
  Number of days which events will be synchronized in the past. For instance, *90* (default setting) will synchronize events which are 90 days in the past and all newer events. Older events won't be synchronized and won't show up in the calendar anymore. **An empty value means that all events will be synchronized.**

Default reminder
  Number of minutes a default reminder will be created before the start of every event that:

    * is a date/time event (= not a full-day event)
    * doesn't have a reminder.

  An empty value means that no default reminders will be created. Default reminders are only set locally, but if an event is edited and uploaded, they will be uploaded like normal reminders. 

Manage calendar colors
  When turned on, DAVx⁵ will set the local calendar colors to the colors transmitted by the server (or default green, if not sent by the server). To fetch updated colors from the server, see :ref:`refresh-collections`.

Event color support
  Whether colors of single events are synchronized to the Android device. If you enable this option, only new events are affected, so you mave have to unselect the calendar, sync, select it again and sync again. This option is disabled by default because there are some big calendar apps which crash when processing colorized events.

CardDAV
-------

Contact group method
  Controls which contact group method is used for this account. For more information, see :ref:`contact-groups`.
