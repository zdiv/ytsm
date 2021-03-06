ytsm
====

Dependencies: umph, youtube-dl, youtube-viewer (3.0.8+ recommended), sed, awk,
GNU date, bash, a cron daemon of your choice, and realpath.

ytsm is a command line YouTube subscription manager composed of two separate
scripts: a script that fetches updates to feeds, and a client to view said
updates.

Why?
----

Traditionally, the best way of keeping up with YouTube uploads is through the
subscriptions feature of a YouTube account. I don't particularly trust Google,
and I would prefer to view my subscriptions as anonymously as is conveniently
possible. Additionally, the YouTube website has a few major issues: it is big,
slow, and doesn't integrate particularly well with my dynamic window manager.
The YouTube website also requires the execution of non-free JavaScript, which I
would like to avoid if possible. Tools like youtube-viewer are nice, but they
require logging into an account to track subscriptions.

This is where ytsm comes in. It keeps track of subscriptions locally, in a
reasonably fast and semi-anonymous manner. Google still sees your IP address,
but neither ytsm nor its dependencies will transmit your identity or store
cookies unless you explicitly log in to youtube through youtube-viewer. ytsm and
its dependencies are all free software as defined by the Free Software
Foundation.

How it works
------------

The fetch script, ytsm-cronjob, reads subscriptions from a file on the computer,
uses umph to find the URLs of the most recent videos of each and youtube-dl to
find the name of each video. The results are stored along with the fetch date in
a TSV file called the feed file.

When the client starts ("ytsm" at a terminal prompt), it compares the current
date to a file on the hard drive to find out when the client was last run. If
the user asks for new videos, it shows only the videos in the feed file that are
newer than the lastrun date. The mechanic used to find new videos is actually
quite simple: it loops over the feed file searching for either certain dates or
line numbers, formats any matches with a combination of sed and echo, and adds
anything it finds to the $OUTPUT variable. The variable is then split into pages
of 25 items with sed and echoed to stdout.

The user can then use n or p to scroll pages (next or previous, respectively),
reso to set the preferred resolution (the default is autodetected from
youtube-viewer.conf), q to return to the main menu, a number to play the video
with youtube-viewer, or a few other minor options.

Installation
------------

Before setting up ytsm, set up youtube-viewer. Make sure it points to your
player of choice, etc.

Add a few subscriptions at a time to your subscriptions file and run the fetch
script. I'd recommend you wait at least a few minutes before adding more,
because if you process too many videos all at once, Google may want you to type
in a captcha, which will prevent the script from fetching the names of the
blocked videos. After you are done adding subscriptions, run the script one last
time to update your feeds. Then, add "0 0 * * * /usr/bin/ytsm-cronjob" to your
crontab, adjusting the path as needed.

FAQ
---

###What does this run on?
ytsm was created and tested on a system running Gentoo. I used a recent version
of youtube-dl, umph 0.2.5, youtube-viewer 3.0.9, GNU awk 4.0.2, and GNU sed
4.2.1.

###Tell me about releases
Releases are made whenever I feel the current git version of the script is good
enough for regular use. For the user, this means that the releases are probably
less likely to have bugs than a random git version would, but it also means that
the user might have to deal with bugs specific to the older version of the
script. The choice between git and releases basically comes down to which bugs
the user prefers: bigger bugs that appear and disappear at random, or smaller
bugs that they might need to live with until the next release.

###POSIX compliance and portability between shells?
As of 16 Feb 2014, ytsm will run in most if not all of the popular shells,
because I am making an effort to avoid shell-specific extensions to standard
utilities. To the best of my knowledge, any POSIX-compliant shell should work,
but the script defaults to bash because almost everybody has it and I know it
works. ytsm-cronjob works with dash and bash. I know it currently has problems
with ksh, and other shells are untested. I recommend using dash for both scripts
not only because it is the fastest shell, but also because most of ytsm's
testing is done with dash.

###How fast is each shell?
I benchmarked each of the following shells by running the built-in benchmark
three times, then taking the average of the three scores. The test system had an
Intel i7-3720QM processor.

 * dash 0.5.7.3: 176 ms
 * ksh 2012-02-29: 176 ms
 * zsh 5.0.2: 207 ms
 * mksh R48b: 215 ms
 * bash 4.2: 263 ms

###How and where are the data files organized?
ytsm puts the files in $XDG_CONFIG_HOME/ytsm, or ~/.config/ytsm if that variable
is not set. That directory contains three more directories: client, data, and
feed as well as the subscriptions file. The client dir contains nothing more
than the lastrun file, which is a plain text file containing a date in the
format YYYYMMDD. The data dir contains several files, each named after a
subscription. These files are simple tab separated files where each line
contains a date stamp, URL, title, and the uploader's username. The cronjob
combines these into a feed file titled "feed-$DATE", $DATE replaced with a
datestamp as needed. It then links the latest feed file to "feed", so that the
client can easily find it. These files can be trivially parsed with sed or cut.

###I have a ton of old feed files in my feed dir and my current feed file is getting huge.
Go ahead and remove the old feed files manually, but leave the current one
alone. The only reason I still keep the old ones around is because keeping a few
copies of the file was convenient during early development. I intend to have
ytsm-cronjob do it automatically, and ytsm to do it on demand as soon as I
implement a config file for user settings. Right now, almost everything is
hardwired into the script because there really isn't much to adjust.

###Why call it ytsm?
It's short for YouTube Subscription Manager, an entirely unimaginative name for
a simple utility. ytsm is also easy to type on both the Qwerty and Dvorak
keyboard layouts.

Shortcomings
------------

 * ytsm currently depends on GNU userland. This means that ytsm probably won't
   run on the BSDs or OS X out of the box. I intend to fix this in a future
   revision.

 * The fetch script may require a captcha if it processes too many new videos at
   once. During testing with an old version of ytsm that used quvi 0.4, I used
   75 videos from three uploaders without incident, but when I added all 62 of
   my subscriptions (about 1300 videos in total), at least 100 of them were
   nameless because of the captcha. Youtube-viewer and quvi were both unable to
   view anything for about four hours.

 * The user-facing documentation isn't excellent. There are passable help
   screens at the prompts, but both ytsm and ytsm-cronjob really could use
   better '--help' screens.

 * The week command probably depends on GNU date. If you are running another
   implementation of date, it may be possible to tweak the script to get it to
   work. I may add support for BSD-style date in a future revision.

 * youtube-viewer can't play any videos that YouTube requires the user to log in
   for (though youtube-viewer _does_ have a login command that would possibly
   work). This limitation mostly applies to videos intended for mature
   audiences.

 * The cronjob has no error checking whatsoever. However, as of this writing in
   October 2013, my copy of ytsm has only failed to download a name for one
   video of the 564 that have been processed in the last three months, and that
   was a video that simply wouldn't work except from the youtube website for
   reasons still unknown. If the script doesn't work as well for you as it does
   for me, I would greatly appreciate your input on how it can be improved.

 * ytsm-cronjob should probably optionally become a standalone daemon for users
   that can't or won't run a cron daemon on their system.
