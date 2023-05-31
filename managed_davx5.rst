=============
Managed DAVx⁵
=============

Managed DAVx⁵ is a version of DAVx⁵ that contains mass-deployment and configuration features
for organizations.


Managed configuration
=====================

Managed DAVx⁵ allows you to manage DAVx⁵ clients centrally by using these configuration methods (in order of precedence):

* Android Enterprise (recommended)
* network configuration: fixed URL (QR code)
* network configuration: unicast DNS
* network configuration: Zeroconf (DNS-SD)

This configuration is used in the Managed DAVx⁵ UI and for new Managed DAVx⁵ accounts.

.. note:: Existing Managed DAVx⁵ accounts on your devices won't be modified when the managed configuration is changed.


Configuration by Android Enterprise
===================================

Android Enterprise is the recommended method to configure Managed DAVx⁵. With Android Enterprise, IT departments can deploy apps to
**managed devices** and configure them in a standardized way using MDM software. Apps are configured by **managed configurations**
(sometimes called restrictions) which can be set in the MDM software for every deployed app.

.. figure:: images/android_enterprise_configuration.png
   :alt: Screenshot of Managed DAVx⁵ configuration over an MDM
   :target: _images/android_enterprise_configuration.png

   You can configure Managed DAVx⁵ using MDM/EMM software.


Network configuration
=====================

If Android Enterprise is not an option for you, you can choose between three network configuration methods:

* fixed URL (entered directly or via QR code)
* unicast DNS
* Zeroconf (DNS-SD)

Network configuration requires your Android devices to be connected to the network where the configuration file can be found.

Certificates
------------

When accessing the configuration file, PKI is used to verify the TLS certificate, so a self-signed certificate won't work without adding it to the Android device first. We recommend to put the configuration file to a location which is accessible over a trusted certificate. You can then define custom trusted certificates in the configuration file.

Caching
-------

Two types of caching are used to cache Managed DAVx⁵ configuration when it's taken from the network:

#. configuration cache and
#. HTTP cache.

Configuration cache:
   Managed DAVx⁵ caches the configuration file which is fetched from the network so that Managed DAVx⁵ configuration is available when there is no network access (and for the time when Managed DAVx⁵ has been started, but the new network configuration is not ready yet). The cache will be overwritten when a new configuration file is downloaded. To reset the cache without a new configuration file, use: Managed DAVx⁵ / About/License / Managed configuration / Reload configuration.

HTTP cache:
   The configuration file is cached when it has been downloaded from the network according to the rules of the HTTP protocol. For instance, if the Web server which hosts the configuration file returns a freshness period of one hour, Managed DAVx⁵ will always use the cached version for one hour. However, the configuration file will be downloaded at least once a day (``max-age: 1 day``) to avoid problems caused by obsolete configuration files. If there is no ``Expires``, the cache will use ``If-Match`` and ``If-Unmodified-Since``.

It's advisable to set an expiration time for the configuration file on the Web server (for instance, one hour) explicitly to avoid unnecessary network traffic every time Managed DAVx⁵ is started on a device.

Configuration by fixed URL
--------------------------

The simplest method to configure Managed DAVx⁵ over the network is to use a fixed configuration file URL, which can for example be provided as a QR code. This method can be used if you don't want to use automatic discovery of the Managed DAVx⁵ configuration file:

#. **Upload the Managed DAVx⁵ configuration file to a HTTPS server** in the network (file name: ``davdroid-config.json``)
#. Managed DAVx⁵ / Managed configuration / Scan the QR code of the configuration file URL or enter the URL.

Configuration by unicast DNS
----------------------------

Managed DAVx⁵ tries to resolve the ``SRV`` and ``TXT`` path records of ``davdroid-configs.local`` in the local network. In case of success, the resulting URL (``https`` scheme, domain and host taken from ``SRV``, path taken from ``TXT path``, or ``/`` else) is used to fetch Managed DAVx⁵ configuration.

An example DNS configuration could look like this:

.. code-block:: none

   davdroid-configs.local   IN SRV 1 0 443 internal.example.com
   davdroid-configs.local   IN TXT "path=/davdroid/davdroid-config.json"

In this case, Managed DAVx⁵ would try to access the configuration file at ``https://internal.example.com:443/davdroid/davdroid-config.json``.

.. warning::

   Do not join unsafe WiFi networks when you use DNS configuration. Other networks might offer other
   Managed DAVx⁵ configuration files, which could lead to confusion. To avoid this problem, only
   join well-defined WiFi networks (or use Android Enterprise or fixed URL configuration).

Configuration by Zeroconf (DNS-SD)
----------------------------------

Managed DAVx⁵ can discover a service called ``davdroid-configs._tcp`` using `DNS-SD <http://www.dns-sd.org/>`_. The network configuration file URL (``https`` scheme) will be built from the host and path parts of ``TXT`` records (the ``SRV`` record is not used because the discovery service is not the same as the referenced configuration). If no host is specified, the host name of the host running the avahi service is used. If no path is specified, ``/`` will be used.

You can use any DNS-SD server. If you use `avahi <https://avahi.org/>`_, the configuration file could be put into ``/etc/avahi/services`` and look like this:

.. code-block:: none

   <?xml version="1.0" standalone='no'?>
   <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
   <service-group>
     <name>Managed DAVx⁵ configuration</name>
     <service protocol="ipv4">
       <type>_davdroid-configs._tcp</type>
       <port>443</port>
       <txt-record>host=internal.example.com</txt-record>
       <txt-record>path=/public/davdroid-config.json</txt-record>
     </service>
   </service-group>

In this case, Managed DAVx⁵ would try to download the configuration file from ``https://internal.example.com/public/davdroid-config.json``.

.. warning::

   Do not join unsafe WiFi networks when you use DNS configuration. Other networks might offer other
   Managed DAVx⁵ configuration files, which could lead to confusion. To avoid this problem, only
   join well-defined WiFi networks (or use Android Enterprise or fixed URL configuration).

Configuration variables
=======================

These variables can be used for Managed DAVx⁵ configuration:

.. list-table:: Configuration variables
   :header-rows: 1
   
   * - Name
     - Type
     - Description
   * - license
     - text*
     - license data (JSON without surrounding curly brackets)
   * - license_signature
     - text*
     - license signature (Base64)
   * - organization
     - text
     - organization display name; shown in app drawer and login activity
   * - logo_url
     - text (URL)
     - organization logo; shown in login activity; must be publicly accessible without authentication
   * - support_homepage_url
     - text (URL)
     - URL of intranet page with details on how to use Managed DAVx⁵ in this organization and how to get internal support; shown in app drawer
   * - support_email_address
     - text (email address)
     - internal support email address – shown in app drawer and some notifications
   * - support_phone_number
     - text (phone number)
     - internal support phone number – shown in app drawer and some notifications
   * - login_introduction
     - text (simple HTML)
     - message that will be shown when the user adds an account; may contain simple HTML like paragrahps, bold text and links
   * - login_base_url
     - text (URL)*
     - base URL for CalDAV/CardDAV service discovery when an account is added;
       example: ``https://server.example.com/dav/``
   * - login_user_name
     - text
     - pre-filled user name when an account is added
   * - login_password
     - text
     - pre-filled password when an account is added; see security note below
   * - login_lock_credentials
     - boolean
     - whether user name and password are locked (= can't be edited by the user) in case they are provided by managed configuration
   * - login_certificate_alias
     - text
     - if provided, client certificates will be used for authentication (instead of user name/password); value of this field will be pre-selected (if available)
   * - preselect_collections
     - integer
     - whether collections are automatically selected for synchronization after their initial detection. |br| |br|
       0 = none (don't preselect) |br|
       1 = all (preselect if not blacklisted) |br|
       2 = personal only (preselect if personal and not blacklisted)
   * - preselect_collections_blacklist
     - text (Regexp)
     - regular expression whose matches with collection URLs will be excluded from preselection;
       example: ``z-app-generated--contactsinteraction--recent`` (Nextcloud's "Recently Contacted" addressbook)
   *  - force_read_only_addressbooks
      - boolean
      - *true* = DAVx⁵ will set all address books to read-only. This will only prevent *client side* editing of contacts from DAVx⁵. If any changes are made they will be reverted to the version present on the server. Keep in mind that this is not preventing changes to the address book in general. For instance other apps can still change the address book on the server. |br|
        *false* = (default) DAVx⁵ won't change standard read-only setting.
   * - max_accounts
     - integer
     - maximum number of accounts – no new accounts can be created when this number of accounts is reached
   * - proxy_type
     - integer
     - Sets the proxy type for all HTTP(S) connections. Uses ``override_proxy_host`` and ``override_proxy_port``, if applicable. |br| |br|
       -1 = system default |br|
       0 = none |br|
       1 = HTTP |br|
       2 = SOCKS
   * - override_proxy_host
     - text (host name)
     - HTTP proxy host name
   * - override_proxy_port
     - integer (port number)
     - HTTP proxy port number
   * - default_sync_interval
     - integer (number of seconds)
     - initial sync interval at account creation (contacts/calendars/tasks); default value: 14400 seconds (4 hours). Only these values are eligible: 900 (15 min), 1800 (30 min), 3600 (1 h), 7200 (2 h), 14400 (4 h), 86400 (1 day). |br|
       Can always be overwritten by users. Changing this value will only affect newly created accounts.
   * - wifi_only
     - boolean
     - *true* = DAVx⁵ will only sync when a WiFi connection is active (doesn't apply to manually forced synchronization) |br|
       *false* = DAVx⁵ will sync regardless of the connection type
   * - wifi_only_ssids
     - text (comma-separated list)
     - when set, DAVx⁵ will only sync when device is connected to one of these WiFis;
       only used when wifi_only is true;
       example: ``wifi1,wifi2,wifi3``
   * - contact_group_method
     - text: ``CATEGORIES`` or ``GROUP_VCARDS``
     - ``CATEGORIES`` = contact groups are stored as per-contact category tags |br|
       ``GROUP_VCARDS`` = contact groups are separate VCards
   * - manage_calendar_colors
     - boolean
     - *true* = DAVx⁵ will overwrite local calendar colors with the server colors at every sync |br|
       *false* = DAVx⁵ won't change local calendar colors at every sync
   * - event_colors
     - boolean
     - *true* = DAVx⁵ will synchronize event colors |br|
       *false* = DAVx⁵ won't synchronize event colors |br| |br|
       Setting to *true* causes some default calendar apps to crash → make sure that your preferred calendar app is working with this setting
   * - default_alarm
     - integer (number of minutes)
     - number of minutes a default reminder will be created before the start of every non-full-day event without reminder; no value (null) or value -1: no default reminders |br|
       Can always be overwritten by users. Changing this value will only affect newly downloaded events.
   * - show_only_personal
     - integer
     - -1 (default value) = user can choose |br|
       0 = show all collections |br|
       1 = show only collections in the user's own home-sets

      

\*... required

.. warning::
   Using ``login_password`` is only recommended with app-specific per-user passwords. Keep in mind that the user
   may be able to retrieve the password even if ``login_lock_credentials`` is set.


Configuration file syntax
=========================

For the network or local file configuration method, a Managed DAVx⁵ configuration file is required.
It contains configuration variables in JSON format, like this:

.. code-block:: json

   {
     "license": "<escaped JSON, don't change this>",
     "license_signature": "<don't change this>",
     "organization": "bitfire",
     "logo_url": "https://intranet.example.com/your-logo.png",
     "support_homepage_url": "https://intranet.example.com/how-to-use-davdroid",
     "support_email_address": "it-support@example.com",
     "support_phone_number": "+1 234 56789",
     "login_base_url": "https://caldav+carddav.example.com/",
     "max_accounts": 1,
     "override_proxy": false,
     "wifi_only": true,
     "wifi_only_ssids": "wifi1,wifi2",
     "contact_group_method": "GROUP_VCARDS",
     "manage_calendar_colors": true,
     "default_sync_interval": 3600,
     "event_colors": false
   }
