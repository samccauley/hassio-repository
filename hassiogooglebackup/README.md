# Backup to Google Drive
![Logo][logo]
## Overview
This add-on will upload files from your Hass[]().io backup folder (typically .tar files created by the Hass[]().io SnapShot feature) to your Google Drive. A few key things to note:
1. This add-on asks only for permission to add new files to your Google Drive, and to manage the files that it adds. It will not have permission to view any files on your Google Drive that it did not create itself. This is to protect the contents of your Google Drive.
2. This add-on exposes a Web User Interface for obtaining your authorization to upload files to your Google Drive. Once that authorization is established, the Web User Interface is no longer needed - it exists only for this initial setup.
3. Backups are performed by executing a REST service exposed on the same port as this add-on's Web User Interface. You can use [Home Assistant's RESTful Command](https://www.home-assistant.io/components/rest_command/) to integrate this add-on's REST service into your own scripts and automations. You may want to use a REST testing tool like [Postman](https://www.getpostman.com/) to perform initial testing.
4. Optionally, you may configure this add-on to purge older files from your hass[]().io backup folder.

You can find all the [details about this add-on here](https://github.com/samccauley/addon-hassiogooglebackup).

[logo]: https://github.com/samccauley/hassio-repository/raw/master/hassiogooglebackup/logo.png