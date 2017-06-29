# iCloud Photo Station

* A command-line tool to download all your iCloud photos.
* Works on Mac, Linux, Windows, and Synology DSM.
* Run it multiple times to download any new photos.
* Store photos either locally or [Synology NAS running Photo Station](https://www.synology.com/en-global/dsm/6.1/packages/PhotoStation).

### Why?

* I use the Photos app on my MacBook, set to "Optimize Mac Storage". It stores full-resolution images in iCloud, and only stores thumbnails on my computer until they are requested.
* I want to download a copy of all my photos onto my Linux PC, because:
  * I use Plex instead of an Apple TV. Now I can display photo slideshows with Plex.
  * I like having a backup of all my photos on my own hard-drive.
* I only want to download the "medium" size of all my photos. They save some space while still being big enough for slideshows. If "medium" is not available, then fall back to downloading the original size.


### Installation

    # Clone the repo somewhere
    git clone https://github.com/skarppi/icloud_photo_station.git
    cd icloud_photo_station

    # Install dependencies
    sudo pip install -r requirements.txt

(Please note that `requirements.txt` references a patched version of the
[pyicloud](https://github.com/picklepete/pyicloud) library. We are using the [#photos-update branch](https://github.com/picklepete/pyicloud/pull/100),
which has a fix for updating recent photos.)

### Authentication

If your account has two-factor authentication enabled,
you will be prompted for a code when you run the script.

Two-factor authentication will expire after an interval set by Apple,
at which point you will have to re-authenticate.
This interval is currently two months.

NOTE: Using the [system keyring to store your iCloud password](https://github.com/picklepete/pyicloud#authentication) is not supported, because this is an automated script. Just provide your iCloud password by using the `--password` option.


### Usage

    $ ./download_photos.py <download_directory or photostation root album>
                           --username=<username>
                           --password=<password>
                           [--size=(original|medium|thumb)]
                           [--recent <integer>]
                           [--until-found <integer>]
                           [--auto-delete]
                           [--photostation https://username:password@to.photo.station/photo/webapi/]

    Options:
      --username <username>           Your iCloud username or email address
      --password <password>           Your iCloud password
      --size [original|medium|thumb]  Image size to download (default: original)
      --recent INTEGER                Number of recent photos to download (default: download all photos)
      --until-found INTEGER RANGE     Download most recently added photos until we
                                      find x number of previously downloaded
                                      consecutive photos (default: download all photos)
      --download-videos               Download both videos and photos (default: only download photos)
      --force-size                    Only download the requested size
                                      (default: download original if size is not available)
      --auto-delete                   Scans the "Recently Deleted" folder and deletes any files
                                      found in there. (If you restore the photo in iCloud,
                                      it will be downloaded again.)
      --photostation                  Instead of local file system store images to Synology Photo Station
      -h, --help                      Show this message and exit.


Example:

    $ ./download_photos.py ./Photos \
        --username=testuser@example.com \
        --password=pass1234 \
        --size=original \
        --recent 500 \
        --auto-delete


### Error on first run

The first time you run the script, you will probably see an error message like this:

```
Bad Request (400)
```

This error usually means that Apple's servers are getting ready to send you data about your photos.
This process can take around 5-10 minutes, so please wait a few minutes, then try again.

(If you are still seeing this message after 30 minutes, then please open an issue on GitHub.)

### Synology DSM installation and synching photos to Photo Station

    # Create a SPK installation package containing virtualenv, python scripts and all necessary dependencies.

    cd spk
    sh build.sh

Manually install resulting `icloud_photo_station-0.1.0.spk` in your DSM `Package Station`. Now you can set up `User-defined script` into `Task Scheduler` and set up scheduling and notification emails for script output.

    source /volume1/@appstore/icloud_photo_station/env/bin/activate
    python /volume1/@appstore/icloud_photo_station/app/download_photos.py \
        --username '<YOUR ICLOUD USERNAME>' \
        --password '<YOUR ICLOUD PASSWORD>' \
        --download-videos \
        --recent 1000 \
        --photostation 'http://<YOUR PHOTOSTATION USERNAME>:<YOUR PHOTOSTATION PASSWORD>@localhost/photo/webapi/' root-album

If your iCloud account has two-factor authentication enabled, SSH to Synology box and run the script manually first time in order to input the verification code.

### Run once every 3 hours using Cron

    cp cron_script.sh.example cron_script.sh

* Edit cron_script.sh with your username, password, and other options

* Run `crontab -e`, and add the following line:

```
0 */3 * * * /path/to/icloud_photos_downloader/cron_script.sh
```
