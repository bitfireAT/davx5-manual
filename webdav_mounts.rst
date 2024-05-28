
=============
WebDAV Mounts
=============


What is a WebDAV mount?
=======================

A WebDAV mount allows you to work with remote folders and files of a WebDAV server. You can "mount" the WebDAV server in DAVx⁵ and then use

* a file manager app to manage your files,
* any app that supports remote files to access your files (like sending a document directly from your WebDAV server over email,
  or watch a video from your WebDAV server).

.. youtube:: FwfwNXFtvYE


Server requirements
===================

WebDAV mounts should work with any compliant WebDAV server.

When using HTTP/1.1, DAVx⁵ uses chunked transfer encoding for PUT requests because it doesn't know the
file size in advance. This requires some servers to be configured for request buffering. Otherwise resulting
files may always have 0 bytes or a 411 Length Required error is returned. Using HTTP/2 may avoid this.
`sabre/dav recommends <https://sabre.io/dav/0bytes/>`_ to use Apache with mod_php (not FastCGI) or a
recent version of nginx and to avoid Lighttpd.


Restrictions
============

.. note:: You can only use DAVx⁵ WebDAV mounts with file managers that support Android's Storage Access Framework (SAF).

At the moment, these file managers are known to work well:

* AOSP File Manager (default in many custom ROMs like LineageOS)
* `Material Files <https://github.com/zhanghai/MaterialFiles>`_
* Total Commander (choose "Add Custom  Location" → open side drawer → Choose the DAVx5 mount). Hint: you need to reload / swipe down and refresh the folder if it appears to be empty.
* fx File Exporer (top right menu: "Connect to Storage" → open side drawer → Choose the DAVx5 mount)

Also, not all apps support content from remote files (:abbr:`SAF (Storage Access Framework)`). If you can't open a WebDAV file with a specific app, chances are high that

1. the app doesn't support remote files at all,
2. or the app opens the file in read/write mode (which is currently not supported by WebDAV/DAVx⁵).

.. warning:: If you're transferring or accessing big files (like streaming a video), Android may kill DAVx⁵ because it uses the CPU in the background. In this case, enable the DAVx⁵ setting *Keep in foreground* to avoid DAVx⁵ to be killed.


Managing WebDAV mounts
======================

.. versionadded:: 4.0

1. Open DAVx⁵ and select *Tools / WebDAV mounts* in the navigation drawer.
2. The *WebDAV mounts* activity appears.
3. In this activity, you can

  * add new mounts,
  * view your existing mounts together with some information (URL and quota, if available),
  * share content from your mounts (see below),
  * unmount existing mounts.

To add a mount, enter a display name (the name for the mount that will appear in Android UI),
the WebDAV URL and optionally user name and password. Then select *Mount*. DAVx⁵ will check
whether there is a WebDAV service at the given URL and then mount it.


How to access files with a file manager
=======================================

The workflow is different between file managers that are a system package (like AOSP File Manager) and
file managers that are installed as normal apps.


Pre-installed file managers
---------------------------

File managers that are pre-installed system packages have direct access to :abbr:`SAF (Storage Access Framework)`.
When the file manager supports :abbr:`SAF (Storage Access Framework)`, you can directly browse into DAVx⁵ WebDAV
mounts (usually in the navigation drawer).


Manually installed file managers
--------------------------------

Manually installed file managers (like Material Files) don't have the permission
to access :abbr:`SAF (Storage Access Framework)` mounts automatically. So you have to

1. add an external mount,
2. then select the respective DAVx⁵ mount in the directory chooser (usually in the navigation drawer), and
3. confirm the location ("Use this directory as root directory").
4. Then you can use the DAVx⁵ mount in the file manager.


How to access files from DAVx⁵
==============================

If you can't install a file manager or if you only want to share files from your WebDAV mount, you can use the
*Share content* button in the *DAVx⁵ WebDAV mounts*:

1. Use the *Share content* button.
2. Now there is a little workaround to view files without installing a file manager: next to each file that
   can be opened there's a small *open* symbol (a bit like this: ✢). Hit this symbol to view the file directly.
3. If you select a file, it can be shared with apps directly.


How to create backups with Seedvault
====================================

You can use `Seedvault <https://github.com/seedvault-app/seedvault>`_ to create system backups with
WebDAV storage over DAVx⁵:

1. Add a WebDAV mount in DAVx⁵.
2. Launch Seedvault, and press the "Choose where to store backups" option.
3. Your WebDAV mount from DAVx⁵ will be shown automatically. Select it.
4. Wait until the configuration process completed.
5. Now you can configure your backup, which will be created at the root of your WebDAV mount.

