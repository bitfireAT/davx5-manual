
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


Account settings
================

Synchronization
---------------

Sync intervals
   Here you can choose the preferred sync interval for address books, calendars and task lists. If you select "only manually", synchronization is only run on explicit user request. In the other cases, the sync interval is saved in Android settings. Android will run synchronization when local data has been changed (normally very quickly, but in some cases, it may take a few seconds or even minutes) and in regular intervals (given that system-wide automatic synchronization is enabled, permissions are granted, battery saving is disabled for DAVx⁵ and there is a network connection).

   .. note:: Synchronization is not initiated by DAVx⁵ itself. DAVx⁵ relies on the Android content provider framework, which manages synchronization and calls sync adapters like DAVx⁵ according to its settings (like sync interval). If :faq:`Android does not call DAVx⁵ for some reason <synchronization-is-not-run-as-expected>`, there is no way that DAVx⁵ can synchronize automatically.

   Keep in mind that low sync intervals need more battery (although only changed records are transmitted). Sync intervals less than 15 minutes are not allowed by Android 7 and later, because it would use too much battery. If you need up-to-date information in your calendar app, use a moderate sync interval (like one hour) and the "Synchronize now" function of your calendar app when you need it.

Sync over WiFi only
   If this option is enabled, DAVx⁵ skips synchronization unless the device is connected to a WiFi network. Android Settings / Accounts will still show a successful sync even when the synchronization has been skipped. Manually forced synchronization will ignore this setting! It is not intended as a security function, but to avoid network connection error notifications.

   Use **WiFi SSID restriction** to restrict synchronization to one or more specific WiFi networks (requires location permission). List all allowed SSIDs as a comma-separated list, for instance SSID1,ssid2. Hidden networks and SSIDs with commas are not supported.

.. todo::

   Other settings (CalDAV/CardDAV, UI, etc.)
