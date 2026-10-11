# ABB robot backup automation

Every Sunday, two of our programmers spent about an hour each backing up around 40 ABB robots by hand. I wrote this to do it for them.

Each controller takes its own backup at a set time. This script then runs from Windows Task Scheduler on the HMI PC and, for every robot in `robots.csv`:

1. Pings it. If it's offline, it logs that and moves on to the next one.
2. Connects over FTP (WinSCP) and finds the newest backup folder.
3. Copies it to the network share, filed by year and month. If it's already there, it skips the download.
4. Only after the backup is safe on the network does it delete old backups off the controller, keeping the 3 newest.

At the end it logs a summary of what downloaded, which robots were offline, and what failed.

## The robot side: `AutoBackup.mod`

The robots trigger their own backups with a small RAPID module. This version is the one from our offline cell, where it runs on Saturdays.

- Once it's Saturday, 7 AM or later, and the robot hasn't backed up yet that day, it starts watching for the line to be idle.
- It waits until there's no job running **and** no job queued for 5 straight minutes. Our normal cycle is 60 seconds, so a 5 minute gap only happens when the line is really stopped, not during a short gap between jobs.
- Then it pulses `doBackup`, which is set up on the controller to kick off a system backup, and saves the date so it only runs once that day.

It needs a `clock1` clock and the `JobInProgress`, `JobPending` and `doBackup` signals set up on the controller.

## Setup

- Install WinSCP. The script loads `WinSCPnet.dll` from `C:\Program Files (x86)\WinSCP\`.
- Copy `robots.example.csv` to `robots.csv` and fill in your robots. Keep the real file out of git, since it holds the FTP logins.
- Set `$NetworkBackupRoot` at the top of the script to your share.
- Schedule it weekly in Task Scheduler, after the controllers take their own backups.

Paths, robot names and logins here are made up. The real ones stay at work.

More about this and my other projects at [markmatheson.dev](https://markmatheson.dev/#case-backups).
