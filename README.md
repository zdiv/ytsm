#ytsm

Dependencies: umph, youtube-dl, youtube-viewer (3.0.8+ recommended), sed, awk, GNU date, bash, a cron daemon of your choice, and realpath.

ytsm is a command line YouTube subscription manager composed of two separate scripts: a script that fetches updates to feeds, and a client to view said updates.

##Why?

Traditionally, the best way of keeping up with YouTube uploads is through the subscriptions feature of a YouTube account. I don't particularly trust Google, and I would prefer to view my subscriptions as anonymously as is conveniently possible. Additionally, the YouTube website has a few major issues: it is big, slow, and doesn't integrate particularly well with my dynamic window manager. The YouTube website also requires the execution of non-free JavaScript, which I would like to avoid if possible. Tools like youtube-viewer are nice, but they require logging into an account to track subscriptions.

This is where ytsm comes in. It keeps track of subscriptions locally, in a reasonably fast and semi-anonymous manner. Google still sees your IP address, but neither ytsm nor its dependencies will transmit your identity or store cookies unless you explicitly log in to youtube through youtube-viewer. ytsm and its dependencies are all free software as defined by the Free Software Foundation.

##How it works

The fetch script, ytsm-cronjob, reads subscriptions from a file on the computer, uses umph to find the URLs of the most recent videos of each and youtube-dl to find the name of each video. The results are stored along with the fetch date in a TSV file called the feed file.

When the client starts ("ytsm" at a terminal prompt), it compares the current date to a file on the hard drive to find out when the client was last run. If the user asks for new videos, it shows only the videos in the feed file that are newer than the lastrun date. The mechanic used to find new videos is actually quite simple: it loops over the feed file searching for either certain dates or line numbers, formats any matches with a combination of sed and echo, and adds anything it finds to the $OUTPUT variable. The variable is then split into pages of 25 items with sed and echoed to stdout.

The user can then use n or p to scroll pages (next or previous, respectively), reso to set the preferred resolution (the default is autodetected from youtube-viewer.conf), q to return to the main menu, a number to play the video with youtube-viewer, or a few other minor options.

##Installation

Before setting up ytsm, set up youtube-viewer. Make sure it points to your player of choice, etc.

Add a few subscriptions at a time to your subscriptions file and run the fetch script. I'd recommend you wait at least a few minutes before adding more, because if you process too many videos all at once, Google may want you to type in a captcha, which will prevent the script from fetching the names of the blocked videos. After you are done adding subscriptions, run the script one last time to update your feeds. Then, add "@daily /usr/bin/ytsm-cronjob" to your crontab, adjusting the path accordingly.

##FAQ

<b>What does this run on/what versions of dependencies?</b>
This was created and tested on a system running Gentoo. I used whatever the latest youtube-dl is, umph 0.2.5, youtube-viewer 3.0.8, GNU awk 4.1.0, and GNU sed 4.2.2.

<b>POSIX compliance and portability between shells?</b>
The script works with #!/bin/bash --posix or #!/bin/sh, but not with #!/bin/dash, or #!/bin/zsh. The major blockers are that dash's echo handles color codes differently from bash's, and zsh's read works differently than bash's. dash and zsh are technically more correct than bash, but everybody runs bash, so that's what I'm targeting for now. I'll try and get it to work with both bash and dash eventually, because bash is a slow and quirky shell.

<b>How and where are the data files organized?</b>
ytsm puts the files in $XDG_CONFIG_HOME/ytsm by default, or ~/.config/ytsm if that variable is not set. That directory contains three more directories: client, data, and feed as well as the subscriptions file. The client dir contains nothing more than the lastrun file, which is a plain text file containing a date in the format YYYYMMDD. The data dir contains several files, each named after a subscription. These files are simple tab separated files where each line contains a date stamp, URL, title, and the uploader's username. The cronjob combines these into a feed file titled "feed-$DATE", $DATE replaced with a datestamp as needed. It then links the latest feed file to "feed", so that the client can easily find it. These files can be trivially parsed with sed or cut.

<b>I have a ton of old feed files in my feed dir and my current feed file is getting huge.</b>
Go ahead and trim the files manually. The only reason I still keep them around is because keeping a few copies of the file was convenient during development. I intend to have ytsm-cronjob do it automatically, and ytsm to do it on demand as soon as I implement a config file for user settings. Right now, everything is hardwired into the script because there really isn't much to adjust.

<b>Why call it ytsm?</b>
It's short for YouTube Subscription Manager, an entirely unimaginative name for a simple utility. ytsm is also easy to type on both the Qwerty and Dvorak keyboard layouts.

##Shortcomings

ytsm currently depends on GNU userland. This means that ytsm probably won't run on the BSDs or OS X out of the box. I intend to fix this in a future revision.

youtube-dl doesn't seem to make it easy to find the upload date, so this script uses the fetch date instead. If I find a way to work around this, I will update the script.

The fetch script may require a captcha if it processes too many new videos at once. During testing with an old version of ytsm that used quvi 0.4, I used 75 videos from three uploaders without incident, but when I added all 62 of my subscriptions (about 1300 videos in total), at least 100 of them were nameless because of the captcha. Youtube-viewer and quvi were both unable to view anything for about four hours.

On low-end hardware, these scripts are probably not incredibly fast. On my Intel i7-3720QM, an Ivy Bridge CPU with a maximum frequency of 3.6 GHz, it can complete the built-in benchmark in about 170ms. With high end AMD hardware circa 2013, I would expect it to complete in about 300ms. With a higher end ARM processor (A9/A15 at > 1 GHz, or just about any ARMv8 CPU), I expect a score of 2000ms (two seconds) at the very worst.

The user-facing documentation isn't excellent. There are passable help screens at the prompts, but both ytsm and ytsm-cronjob really could use better '--help' screens.

The week command probably depends on GNU date. If you are running another implementation of date, it may be possible to tweak the script to get it to work. I may add support for BSD-style date in a future revision.

youtube-viewer can't play any videos that YouTube requires the user to log in for (though youtube-viewer _does_ have a login command that would possibly work). This limitation mostly applies to videos intended for mature audiences.

The fetch script has no error checking whatsoever. However, as of this writing in October 2013, my copy of ytsm has only failed to download a name for one video of the 564 that have been processed in the last three months, and that was a video that simply wouldn't work except from the youtube website for reasons still unknown. If the script doesn't work as well for you as it does for me, I would greatly appreciate your input on how it can be improved.

ytsm-cronjob should probably optionally become a standalone daemon for users that can't or won't run a cron daemon on their system.
