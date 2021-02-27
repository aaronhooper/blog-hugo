---
title: "Downloading SoundCloud tracks"
date: 2020-06-11
draft: true
---

The following details my process of acquiring local copies of my entire
library of liked tracks on SoundCloud.

Recently I've been thinking about how much of my music &mdash; and, by
extension, much of my life &mdash; is not in my control. I have nearly
1,500 liked tracks on my account going back 4 years. What if SoundCloud
goes under?  The music is worth more to me than just owning the raw data
for its own sake; it has sentimental value that cannot be replaced if it
is lost.  Buying a subscription will not solve this problem, and many
tracks on SoundCloud are not available anywhere else.  I must have the
data.

I will leverage the power of cloud computing to quickly download,
compress and transfer all this data to my local computer, and verify
that it all made it in one piece (well actually several, but I elaborate
on this further on in the article!)

Part of the reason I am writing this in the first place is because I
have pretty terrible broadband, and as I am downloading these files
right now, I cannot do many other things on this connection
simultaneously. It also brought together a lot of UNIX skills I have
gradually learned over the years. Some commands shown here I have used
for as long as I have been a Linux user, such as `apt` and `wget`, and
others I only got round to learning about while doing this, like
`split`. I hope you get something out of this, regardless!

## Setting up the server

I'm going to demonstrate this with Linode. There are many cloud
platforms out there and they all work much the same. I just use Linode
because they were the first one I found that supported reverse DNS
lookup without much hassle (I'm looking at *YOU*, AWS! 😠)

From the dashboard, we will provision a new Debian server with enough
storage to contain all the music. For reference, my 1,500 tracks ended
up taking around 14 GB in total. We wait for it to finish booting, copy
the IP address into the terminal and open an SSH connection to it,
giving it the root password we set it up with:

    $ ssh root@255.255.255.255

Boom, we're in! Now, some housekeeping:

    $ apt update && apt upgrade -y

The tool that will be used to download the actual files is called
`youtube-dl`. It's a python command-line program and it is remarkable,
but don't be fooled by its name: it'll work on just about any site that
can stream non-DRM video or audio.

Another tool that makes terminal screen management in SSH more
effortless is `tmux`, or the *terminal multiplexer*. Its main function
is to be able to manage multiple terminal screens in one shell, and it
runs as a server so you can attach and detach from running processes,
meaning you can disconnect and reconnect from the SSH session without
interrupting your processes. Truly a must-have if you work a lot in
remote shells.

    $ apt install python3-pip tmux
    $ pip3 install youtube-dl --upgrade

## Getting the tracks

Starting the download is really as simple as opening `tmux` and running
`youtube-dl` on the URL on the likes playlist of the SoundCloud profile:

    $ tmux
    $ youtube-dl -i https://www.soundcloud.com/the_cool_guy_username/likes

The `-i` option is important, it tells `youtube-dl` to skip any tracks
that return an error. This is usually geolocation-related, but without
specifying this option the program just quits upon receiving this error.

You'll notice that a green bar appears at the bottom of the shell after
running `tmux`. This is its status bar. You can do a few things here,
like create a new pane (another shell instance) with `Ctrl-b c` and
cycle the panes with `Ctrl-b n` and `Ctrl-b p`. There are [many more
things][2] that can be done with this program, but these might be useful
to check, for example, the current size of the directory in another pane
(assuming you're downloading to a folder called `music`):

    $ du -sh music
    7.2G    music

## Post-processing

Now that all the files are downloaded, we need to do some things to them
before we put them in object storage.

It turns out that a directory with potentially thousands of files in
them is quite cumbersome to manipulate, especially if you're looking to
transfer them across network connections.

The first thing we need to do is get them into one contiguous file, so
that it is easier to work with. This would also be a good place to apply
some compression too, just to shave some extra bytes off so we can save
some time downloading it.

    $ tar czvf music.tgz music

I use the `gzip` algorithm here because it's ubiquitous and fast. I
could have used `bzip2` which takes longer but compresses to a smaller
size, but I figured it wouldn't be worth the extra time spent
compressing it.

Once this is done, we are left with a 13.5 GB file, saving ~500 MB of
space. It's still very large, and so there are more things we can do to
it to avoid potentially tripping up later on.

We can split the file up into multiple chunks. This is good for multiple
reasons: some filesystems have a limit on how big each file stored on it
can be, and when we get to downloading it and are not able to get the
whole thing in one day *and* the object storage server doesn't support
resuming downloads &mdash; if we were to be so unlucky &mdash; we will
lose at most 1 GB of data upon killing the download. We can therefore
mitigate both the risks of not being able to hold the entire file on
disk, and not being able to download the entire file in one single
session, with only a bit of extra work.

    $ split -b 1G music.tgz 'music.tgz.part.'

The files will be appended with a string of letters in alphabetical
order by the program to ensure they are in the correct order when we
join them back up on the other side.

> **Q:** Why can't you just leave your computer on so that you can
> download it all in one go?
>
> **A:** I could do this, but I don't like leaving my computer on
> overnight, partly for security reasons (it's very difficult to break
> into a computer that is turned off) but also because I just don't like
> hearing the sound of my fans spinning when I'm trying to sleep,
> especially in the summer when it's hot already.

Finally, we can compute checksums for these files so that we can verify
that they all arrived on the other side with no errors. I'm using MD5
here because security isn't a priority. If you haven't heard, [MD5 is
broken for security-related purposes][3], but it's fine for what we're
doing here.

    $ md5sum -b music.tgz.part.* > md5sums.txt

## Transferring to object storage

Now we will set up *Linode Object Storage*, so that we can expose these
files with a bucket via HTTP so we can download them easily. If you
haven't used it before, know that it costs $5/month for [as long as it
is enabled on your account][1]. When you are finished using the bucket,
remember to cancel Object Storage in your settings panel if you are no
longer using it. The link to enabling it is available on the dashboard.

Next, we will set up `linode-cli` on the server so that we can create
our bucket and transfer these files to it.

    $ pip3 install linode-cli --upgrade
    $ pip3 install boto
    $ linode-cli configure

Next, we create a new bucket to hold our files.

    $ linode-cli obj mb music-dl-1

Now we can upload the compressed files and the MD5 sums to the bucket.

    $ linode-cli obj put --acl-public music.tgz.part.* music-dl-1
    $ linode-cli obj put --acl-public md5sums.txt music-dl-1

You may also need to add `--cluster` and follow it with a particular
region different to the one you set up with `linode-cli configure` as
[not all regions support object storage yet][5]. The nearest one for me
is `eu-central-1` which is located in Frankfurt, DE.

## Downloading

Now that the files have been transferred to object storage, we need to
do one final thing before we delete the server.

The Linode documentation says that links to files hosted on a public
bucket take the form
`http://<bucket-name>.<region>.linodeobjects.com/<filename>`. This is
very handy because it means we can programmatically create a list of
links to these files for easy downloading.

We first need to create a text file with links to all the files in the
bucket, because we will be using a tool called `wget` to download all of
the files to our local computer using this text file.

We can create a list of all of the files we care about by running `ls`
and specifying the files in question:

    $ ls music.tgz.part.* md5sums.txt > music-urls.txt

This makes the rest very easy, because we can just add our bucket URL to
the beginning of each line in the file. It can be done in many ways:
copying and pasting, some other text editor magic, or &mdash; my
favourite &mdash; using the good old UNIX command, `sed`:

    $ sed -i 's/^/http:\/\/music-dl-1\.eu-central-1\.linodeobjects.com\//' music-urls.txt

A downside is that we have to escape `/` and `.` in the URL so that
`sed` doesn't interpret them as expressions. On the upside, it makes for
easy inclusion into an article like this one!

Now we need to transfer this list of URLs to our local machine. We could
just upload it to the bucket and download it through the web interface,
but we can do it more easily with one command called `scp`. Running it
on our local machine in a similar way we connected to the server:

    $ scp scp://root@255.255.255.255/music-urls.txt .

Now it exists in our local directory! We're safe to shut down and remove
the server now, to avoid additional costs.

Finally, we will use `wget` to reliably download these files to our
local computer. Using it is pretty simple:

    $ wget --input-file=music-urls.txt --continue

The `--continue` flag is used to restart the download at the exact point
it stopped, if it did not download the file completely. I believe the
server that is offering the download also needs to support this kind of
download-resuming for it to work, which is part of the reason we set up
split files earlier, just so there is some redundancy there.

This can optionally be used as part of a `cron` job, so that downloading
can occur at a specific time.

    $ crontab -e

At the bottom I can add the following, assuming that the downloads will
be held in a directory called `soundcloud` in the user's home folder:

    0 1 * * * cd /home/the_cool_user/soundcloud && wget --append-output=wget.log --input-file=music-urls.txt --continue

The preceding characters tell `cron` to execute the command at 1am every
morning. This is useful for me because it means I don't disrupt the
broadband for anyone else, and it stops when I turn my computer off, so
it can be resumed the next morning.

I have also set up logging here, so that if something went wrong I can
figure out why. It's pretty good practice to make sure there is a log
for whatever is scheduled with `cron`, because it might fail, and it
could be quite difficult to figure out why without one.

## Putting it all back together

Verifying that we got all the data with no errors is as simple as
running the same program we used to compute the checksums:

    $ md5sum -c md5sums.txt

All the files should return `OK`. If not, then hopefully it was only one
file that borked, and it can be re-downloaded without much hassle.

Now we can join the files back up and extract them with a single
command:

    $ cat music.tgz.part.* | tar xzv

## Good links

- [How to Use Linode Object Storage | Linode][1]
- [tmux/tmux Wiki][2]
- [MD5 - Wikipedia][3]
- [How to Access Objects with Linode Object Storage | Linode][4]


[1]: https://www.linode.com/docs/platform/object-storage/how-to-use-object-storage/#enable-object-storage
[2]: https://github.com/tmux/tmux/wiki
[3]: https://en.wikipedia.org/wiki/MD5
[4]: https://www.linode.com/docs/platform/object-storage/how-to-access-objects-with-linode-object-storage/
[5]: https://www.linode.com/community/questions/19567/changing-regions-when-deploying-an-object-storage-bucket-through-the-linode-cli

