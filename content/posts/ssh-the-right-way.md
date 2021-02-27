---
title: "SSH, the right way"
date: 2020-07-13T17:38:53+01:00
draft: true
---

When I first learned what SSH was, I thought I had it all figured out. I
mean, how complicated could it be? It's just the command line, with
remote access, right?

Until I was faced with the dreaded error message:

    Permission denied (publickey).

I wrestled with this error message for about half an hour every time I
tried to set up keys on a new server or from a new client. I would think
to myself, "why does SSH have to be so complicated? Why can't I just
create keypairs and have them work automatically?"

Eventually I figured out that was the issue: **I was generating too many
SSH keys.**

## Mistakes

It turns out that **you don't need a separate keypair for every possible
client-server combination.** I would have a keypair named
`mail_root_arch` for my connection to my mail server, as user `root`,
from my home desktop, which was running Arch Linux at the time. I would
then have a different key on that machine which I used to log into my
Raspberry Pi, this time as a `sudo` user instead of `root`. Right off
the bat, there is a lot wrong with this.

Firstly, I should have taken the time to create a new user with `sudo`
permissions, and disable remote `root` login. It's a bad idea to do
anything as `root`, including remote access, even if
`PasswordAuthentication` is disabled and you have a keypair for
accessing it. It is instead much better to create a `sudo` user and
authorize your keys to that.

Secondly, having a keypair for every connection means that you must have
more than one keypair per client. This means that you need to set the
`IdentityFile` for each of your private keys in `~/.ssh/config`, since
when the client cannot find one of five keyfile names (one of them being
`id_rsa`) and there is no custom filename specified in the config, it
will fail to connect. On top of this, setting your `IdentityFile`s must
be done on every client that you use SSH on. This is a *wildly*
overcomplicated way of doing things.

Deviating from the default options also has some silly gotchas, one of
them being that you cannot use the tilde to reference your home folder
when creating a custom keyfile:

    $ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/aaron/.ssh/id_rsa):
    ~/.ssh/some-key
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Saving key "~/.ssh/some-key" failed: No such file or directory

> "What the hell? What do you mean? You're supposed to be creating it!"

Creating the keyfile by just specifying the name of the file without the
path, with the innocent assumption that `ssh-keygen` will just
automatically save the file in `$HOME/.ssh`, reveals the second
gotcha: this actually saves the keys in the current directory.

Ok, but you can work around this easily enough. One can simply move the
keyfiles into `~/.ssh`. This reveals the third gotcha: because the files
have been created outside of the SSH folder, they have taken on
permissions which SSH will not allow you to have. SSH keyfiles must not
be readable by anyone other than yourself, so you must change them as
follows:

    $ chmod 600 ~/.ssh/*

Yikes. Who knew that SSH would be this complicated?

I also went through a phase of having only one SSH keypair and using
that wherever I wanted to connect with SSH. This is arguably worse
because if the key is comprimised, finding out which machine was
involved may be more difficult, and revoking it would also revoke access
for every client that used it. It also involves storing the private key
in more than one location, making it more vulnerable.

## The solution

There is, in fact, a much simpler way. **Generate one keypair per
machine, and leave it with the default name** (usually `id_rsa`). Now
there is no need to set the `IdentityFile` anywhere, keeping your
`~/.ssh/config` tiny. Clients can then be authenticated on the desired
servers with `ssh-copy-id`. If `PasswordAuthentication` is enabled on
the server, this is very easy, as the clients can authorize themselves
then and there, as follows:

    $ ssh-copy-id -i id_rsa.pub user@server-name

If it is disabled, it can be temporarily enabled in
`/etc/ssh/sshd_config` for this purpose, or if this isn't possible, the
public key of the client can be copied over to a machine that already
has key access. The above command can then be executed. No sharing of
private keys between machines.

In the event that you run into an error which tells you that a private
key cannot be found, this is apparently a bug in SSH which can be
avoided by following the error message's suggestion to run the same
command with the `-f` flag added.

## Final thoughts

This is one of the broader issues concerning the usage of technologies
that you might not yet be familiar with: an inexperienced user such as
myself may incorrectly assume that some action must be done in a certain
way, then proceed to search for how to accomplish this on Google, and
come across a result that tells me how to do it. It is often the case
with using these tools that you are not prevented from doing things in
an incorrect way.

It is much the same with real life hardware tools. For example, many
people who service their own bicycles will own a pair of cable cutters,
made for cutting brake cables and housing. They are made for that
purpose only, but to a person who doesn't know beforehand, they look
like they would be useful for cutting other things, which they would be,
at least initially. Over time this actually damages them to the point
where they can no longer be used for their intended purpose, as they
have been prematurely blunted and cannot be easily sharpened.

I don't think a tool would be much of a tool if it prevented you from
doing things that might not be the best things to do.  Because
occasionally, things do need to be bodged!
