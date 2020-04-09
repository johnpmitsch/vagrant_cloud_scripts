# Vagrant cloud upload box script for Katello

## Description
This allows for stable Katello box images to automatically be created and uploaded to vagrant cloud. This can be used in a cron job.

## Usage
See the comments in `./upload_to_vagrant_cloud` for requirements and installation details

See `./upload_to_vagrant_cloud --help` for CLI arguments

Typical usage:
```
VAGRANT_CLOUD_API_KEY=mykey ./upload_to_vagrant_cloud -b katello/katello-nightly -f centos7-katello-nightly-stable.box -t nightly -e centos7-katello-nightly-stable.json -d /home/jomitsch/forklift/
```

A cron job can be created with this script or:

### Using systemd timers

A systemd timer allows for logging to journalctl with default settings, which makes for easy debugging.

1. Create a timer file and update `onCalender` to your preferred time

`/etc/systemd/system/katello-nightly-stable-box.timer`
```
[Unit]
Description=Stable Katello box timer

[Timer]
OnCalendar=*-*-* 00:15:00

[Install]
WantedBy=timers.target
```

2. Create a service file, updating the user and api key environment variable

`/etc/systemd/system/katello-nightly-stable-box.service`
```
[Unit]
Description=Builds a katello nightly box and uploads to vagrant cloud

[Service]
Type=oneshot
User=jomitsch
Environment=VAGRANT_CLOUD_API_KEY=replacemewithkey
ExecStart=/home/jomitsch/katello-packer/upload_to_vagrant_cloud -b katello/katello-nightly -f centos7-katello-nightly-stable.box -t nightly -e centos7-katello-nightly-stable.json -d /home/jomitsch/forklift/
```
3. Run `systemctl daemon-reload`
4. Enable the timer: `systemctl enable katello-nightly-stable-box.timer`, don't enable the service unless you want the script to run on boot.
5. Start the timer or reboot: `systemctl start katello-nightly-stable-box.timer`

The service will execute the script when specified by the timer. This service will log to journalctl `journalctl -u katello-nightly-stable-box` when running.

Note: You probably will run into SELinux issues if you have it enabled on your system. You can check this by starting the service (but don't enable), which will execute the script. Check journalctl for logs and use audit2allow or similar to give proper access. 

## Box cleanup

The `./clean_old_boxes` script can be used to delete boxes older than a month from Vagrant cloud. See the comments in the script for setup and usage.
