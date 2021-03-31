# Backup to Google Drive

![Logo][logo]

## Overview

This add-on will upload files from your hassio backup folder (typically .tar
files created by the hassio SnapShot feature) to your Google Drive. A few key
things to note:

1. This add-on asks only for permission to add new files to your Google Drive,
and to manage the files that it adds. It will not have permission to view any
files on your Google Drive that it did not create itself. This is to protect
the contents of your Google Drive.
2. This add-on exposes a Web User Interface for obtaining your authorization to
upload files to your Google Drive. Once that authorization is established, the
Web User Interface is no longer needed - it exists only for this initial setup.
3. Backups are performed by executing a REST service exposed on the same port
as the Web User Interface. Find [more information on the doBackup service
operation below](#Calling-the-`doBackup`-Operation). You can use
[Home Assistant's RESTful
Command](<https://www.home-assistant.io/components/rest_command/>) to integrate this
add-on's REST service opertion into your own scripts and automations. You may
want to use a REST testing tool like [Postman](https://www.getpostman.com/) to
perform initial testing.
4. Optionally, you may configure this add-on to purge older files from your
hassio backup folder.
5. Optionally, you may also configure this add-on to purge older files from the
Google Drive folder that your are archiving your snapshots to.

## Installation

You can install this hassio add-on using [my add-on
repository](<https://github.com/samccauley/hassio-repository>) following these
[Home Assistant Add-on Instructions](https://www.home-assistant.io/hassio/installing_third_party_addons/).

## Configuration Options

Example configuration:

```json
{"fromPattern" : "/backup/*.tar",
 "backupDirID" : "1CvPDzNz1v-OuOUqKq3jjoKQt020hKK7R",
 "purge" : {"enabled" : true, "preserve" : 3},
 "purge_google" : {"enabled" : true, "preserve" : 12},
 "debug" : false}
```

### `fromPattern`

Use this to identify the files on your hassio host that you wish to backup.

This pattern is used to identify a list of files to backup. That list is then
pared down by checking Google Drive to see if any files in the list have
already been backed up by this add-on. This check is performed to avoid backing
up the same file twice (Google Drive allows duplicate files with the same name).

Note that this add-on can only see files on your Google Drive that it created
itself. Therefore, if you have backed up some of your snapshots on your own to
Google Drive, this add-on will not be aware of those and it will back them up anyway.

### `backupDirID`

This identifies the Google Drive folder in which you want to place your backed
up files. Because this add-on does not have permission to browse any files or
directories on your Google Drive that it does not itself create, you cannot
simply provide the folder name. You must instead provide Google Drive's unique
opaque ID of the folder. Google Drive doesn't make it easy to get this value.
But, here's how you can get it:

1. In your favorite web browser, navigate to the Google Drive folder to which
you plan to upload your files (create a new folder in Google Drive if you wish).
Be sure that you have the perferred folder open so that its contents (even if
it's empty) are displayed.
2. From the address bar of your browser, copy the last portion of the URL. That
value is Google Drive's unique opaque ID for the folder. Paste that value in for
the `backupDirID` value in the configuration.

### `purge`

This configures the option to purge (delete) older files from your source
location (e.g. your /backup folder on hassio). There two sub-elements:

- `enabled` Set this boolean to true if you wish to take advantage of this purge
feature.
- `preserve` Set this integer value to the number of files that you wish to
preserve (to keep) in your source location. If enabled, this purge feature will
delete the oldest files (by date modified) in your source location, preserving
the number of more recent files that you specify with this value.

### `purge_google`

Contrary to its ominous sounding name, this does not purge every file from your
Google Drive ;-). This add-on can see **only files that it creates** on your
Google Drive. It can also see the folder that you identify for it to place
backup files into, but it cannot see files inside that folder unless this add-on
created them itself. This option configures the feature to purge (delete) older
files from your Google Drive folder (the one you identify in the `backupDirID`
option). There two sub-elements:

- `enabled` Set this boolean to true if you wish to take advantage of this feature.
- `preserve` Set this integer value to the number of files that you wish to
preserve (to keep) in your Google Drive folder. If enabled, this feature will
delete the oldest files (by date modified) in your Google Drive folder,
preserving the number of more recent files that you specify with this value.

**Important notes about this option:**

1. This **permanently deletes** the selected files from your Google Drive,
bypassing your Google Drive Trash. The idea is to free up available storage on
your Google Drive.
2. It only considers files in the currently configured `backupDirID` folder on
Google Drive.
3. It only considers files created by this add-on (because those are the only
files it can see).
4. It does **not** consider the `fromPattern` setting at all.

### `debug`

Defaults to `false` if not present. Set this to `true` to enable debug-level logging.

## Authorizing this Add-On to Upload to Google Drive

This add-on requires you to authorize it to a limited scope of access to your
Google Drive. The specific scope it requires is
`https://www.googleapis.com/auth/drive.file`. You can read information about
what that scope entails in
[Google's Guide to OAuth 2.0
Scopes](https://developers.google.com/identity/protocols/googlescopes).
Essentially, it allows this add-on to view and manage Google Drive files and
folders that you have opened or created with this add-on.

Before using the
[`doBackup` service operation](Calling-the-`doBackup`-Operation), you need to
follow these steps to grant this add-on the required authorization:

1. Start this add-on and make sure that it is set to `Start on boot`.
2. Open the Web User Interface for this add-on using the `Open Web UI` link on
the add-on details page. Alternatively, you can open your own browser window and
navigate to: `http://<Your_Hassio_Host>:<Host_Port>/gb`, substituting your
hassio host name and the Host Port number you've configured for this add-on.
3. Following the instructions in the Web User Interface, do the following:
    1. Click on the `AUTHORIZE` button to launch a separate browser tab to the
    Google Authorization Server.
    2. If required, login to Google and confirm your Google user ID.
    3. Google will then tell you what application (this add-on) is requesting
    authority and what scope of authority is being requested. Click Google's
    `Allow` button.
    4. Google will then show you an authorization code. Copy this code so that
    you can paste into this add-on's Web User Interface (next step).
    5. Return to the browser tab containing this add-on's Web User Interface and
    paste the copied value (from the previous step) into the provided
    `Authorization Code' field.
    6. Click the `INGEST CODE` button.
    7. You should be presented with a message indicating **authorization received**.
4. This completes the authorization step! You're now ready to begin using the
`doBackup` Service as described below.

## Calling the `doBackup` Operation

Backups are performed by calling the `doBackup` service operation exposed by
this add-on. When you start this add-on, the service becomes available on the
Host Port that you've configured this add-on to use.

The `doBackup` service operation does not require any arguments. It gets the
information it needs from the [Configuration Options](#Configuration-Options)
and from your having completed the [authorization process described above](#Authorizing-this-Add-On-to-Upload-to-Google-Drive).

You call the doBackup service operation by simply performing a GET against this
URI (in fact, you can just click this link):

```HTML
    http://<YOUR_HASSIO_HOST>:<HOST_PORT>/gb/doBackup
```

Substitute in your hassio host name (usually `hassio.local`) and the Host Port
number you've configured for this add-on.

The `doBackup` service operation will respond with JSON reminding you of the
configuration settings that it used and indicating:

- the time the backup operation began, in ISO 8601 format.
- how many files were found using the `fromPattern`
- how many of those files had alredy been backed up to your Google Drive and
were therfore skipped this time
- how many files the `doBackup` service operation actually backed up during this
run
- how many old files were purged (deleted) from the source location
- how many old files were purged (deleted) from the target Google Drive folder.

### Sample JSON Response

```json
{
    "backupTimestamp": "2018-11-26T09:57:36.206259",
    "fromPattern": "/backup/*.tar",
    "backupDirID": "1CvPDzNz1v-OuOUqKq3jjoKQt020hKK7R",
    "fileCount": 5,
    "alreadyCount": 2,
    "backedUpCount": 3,
    "deletedCount": 2,
    "deletedFromGoogle": 1
}
```

Unexpected errors will return an HTTP Status Code of some value other than the
normal `200` Success Code.

## Calling the `adhocBackup` Operation

As a completely separate feature, adhoc backups may be performed. These backups
do not use the configured options `fromPattern`, `backupDirID`, `purge` and
`purge_google`. Instead, an adhoc backup request identifies which files are to
be backed up and which Google Drive folder is to be targeted each time it is
placed (hence the name, "adhoc"). The concept of "purging" of older files while
preserving more recent files does not apply at all to adhoc backups. Adhoc
backups simply copy each identified file from your hassio host to your Google
Drive, **replacing the target file on Google Drive if it already exists**.
Adhoc backups are performed by calling the `adhocBackup` service operation
exposed by this add-on. When you start this add-on, the service becomes
available on the Host Port that you've configured this add-on to use.

The `adhocBackup` service operation has two required arguments that must be
provided in the form of a JSON body on the HTTP POST.

- `fromPatterns` This is an array of file name patterns to be used to identify
which files to copy from your hassio host (only the following folders are
accessible for this add-on to copy from: `config`, `ssl`, `addons`, `backup`,
and `share`).
- `backupDirID` This is identifies the particular folder on your Google Drive
where you want the files to be copied to. See the description of the
[backupDirID configuration operation](#`backupDirID`), above, for details on
how to obtain the identifier of a Google Drive folder. Keep in mind that you
must have completed the one-time [authorization process described above](#Authorizing-this-Add-On-to-Upload-to-Google-Drive)
in order for either of these service operations to work.

**WARNING:** It is recommended that you do NOT specify the same target Google
Drive folder for adhoc backups that you have specified in the add-on options
configuration for the normal backup operation, especially if you are using the
`purge_google` feature.

You call the adhocBackup service operation by performing a POST against this URI:

```HTML
    http://<YOUR_HASSIO_HOST>:<HOST_PORT>/gb/adhocBackup
```

Substitute in your hassio host name (usually `hassio.local`) and the Host Port
number you've configured for this add-on.

### Sample JSON Request

```json
{"fromPatterns" : ["/config/configuration.yaml", "/config/automations.yaml"],
 "backupDirID" : "4FtLMzNz1v-OuOUqKq3jjoKQt020hLL9P"}
```

The `adhocBackup` service operation will respond with JSON indicating:

- a timestamp of when the operation was performed
- the list of fromPatterns that were provided on the call to the service operation
- the identifier of the Google Drive folder provided on the call to the service operation
- how many files were copied
- how many of those files replaced existing files on Google Drive
- how many files were added as new files in the target Google Drive folder

### Sample JSON Response

```json
{
    "adhocBackupTimestamp": "2019-03-15T07:59:56.205540",
    "fromPatterns": [
        "/config/configuration.yaml",
        "/config/automations.yaml"
    ],
    "backupDirID": "4FtLMzNz1v-OuOUqKq3jjoKQt020hLL9P",
    "copyCount": 2,
    "newCount": 0,
    "replacedCount": 2
}
```

Unexpected errors will return an HTTP Status Code of some value other than the
normal `200` Success Code.

## Integrating into Home Assistant

### Automating Backups with Home Assistant

You can easily integrate this add-on's `doBackup` REST service operation into
Home Assistant using [Home Assistant's RESTful
Command](https://www.home-assistant.io/components/rest_command/). You'll
probably need to use `localhost` instead of `hassio.local` in this
configuration. You'll also want to specify an adequate timeout value. Here's
how I setup mine:

```YAML
rest_command:
  google_backup:
    url: 'http://localhost:8055/gb/doBackup'
    timeout: '300'
```

With this REST Command created, you'll see your Google Backup Service available
as `rest_command.google_backup` in
[Home Assistant's Services Development
Tool](https://www.home-assistant.io/docs/tools/dev-tools/), and you'll also be
able to call it as part of [Home Assistant Automations](https://www.home-assistant.io/components/automation/).

Because the `adhocBackup` operation requires a JSON array, I've not been able
to figure out how to configure a REST Command for it in Home Assistant. If you
figure it out, let me know. For now, you can call the `adhocBackup` operation
from tools like [Postman](https://www.getpostman.com/). You'll want to setup
your tool do a POST method, passing in JSON (Content-Type of appliction/json).
Here's an example JSON request: `{"fromPatterns" :
["/config/configuration.yaml", "/config/automations.yaml"], "backupDirID" : "4FtLMzNz1v-OuOUqKq3jjoKQt020hLL9P"}`.

### MQTT Event

Everytime the doBackup operation, or the adhocBackup operation, is executed, an
event is published to the
[MQTT broker](https://www.home-assistant.io/components/mqtt/) that you have
configured in Home Assistant. The payload of the event is a copy of the JSON
response described above. The event is published to either
the `googlebackup/result` or the `googlebackup/adhocresult` topic, depending on
which operation was called.

Here is an example Home Assistant Sensor configuration that monitors for these events:

```YAML
  - platform: mqtt
    name: "Google Backup Results"
    state_topic: "googlebackup/result"
    json_attributes_topic: "googlebackup/result"
  - platform: mqtt
    name: "Google Adhoc Backup Results"
    state_topic: "googlebackup/adhocresult"
    json_attributes_topic: "googlebackup/adhocresult"
```

[logo]: https://github.com/samccauley/addon-hassiogooglebackup/raw/master/googlebackup/logo.png