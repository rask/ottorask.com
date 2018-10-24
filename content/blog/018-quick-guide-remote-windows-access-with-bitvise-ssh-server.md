+++
title = "Quick guide: remote Windows access with Bitvise SSH Server"
date = 2015-06-24T22:18:00Z
slug = "quick-guide-remote-windows-access-with-bitvise-ssh-server"
+++

Forget Windows Remote Desktop (or whatever it was) and TeamViewer. Let’s see how
to get a baseline remote access to your Windows installation using SSH.

Note: I’m using Windows 7. Also, SSH access usually means **no GUI**: if you’re
not used to working with terminals or command lines you should probably stick
with TeamViewer or something.

## 1. Open your ports

Open up your SSH port (`22` by default) on your router. You can also use a
different port if you wish. Route the traffic to your Windows’ IP address.

## 2. Install Bitvise SSH Server

Download and install the
{{ link(label="Bitvise SSH Server", url="http://www.bitvise.com/ssh-server", external=true) }}
software. The software has a free personal-use plan available.

The installation should be pretty straight-forward. Pick installation
directories and so on as you like. After installation the Bitvise SSH Server
service should keep the server running between reboots and such.

## 3. Configure the SSH server

Set the base configurations: set the user that can be logged in with. If you
have a password for the Windows user account you can use password
authentication. Additionally you can use or force SSH public key authentication.
You’ll need the public keys of each computer you need to allow to access your
Windows.

Set the firewall to allow all traffic. Bitvise complained that starting the
server would only allow local (LAN) connections if the firewall setting would
not be changed.

I did no other configuration than set the SSH session to allow only public key
access (make sure you import them to the application properly) and set login
attempts made against `root`, admin and Administrator to get IP-banned
instantly. Additionally you can whitelist or blacklist IP ranges.

Additionally I set my session shell to GitBash using
`C:\Path\To\Git\bin\bash.exe --login -i`. This way the remote environment works
just like I would use it at home. You can use the default Windows `cmd.exe` if
you want to, or perhaps PowerShell.

## 4. Test it out

I needed to reboot my PC before testing the connection.

By opening an SSH session with

    $ ssh User@127.0.0.1 # Replace `User` with your Windows username

you should be able to access your computer using SSH (either by giving your
password or auto-login using public keys). I also tested with my VPS by first
SSHing into it first, then SSHing back to my PC’s (house’s) IP address. If that
works, you’re good to go.

## 5. Work

If the tests worked, you can now access the command line of your Windows
installation using SSH (yay!). What you can do with the SSH connection depends
on Bitvise SSH Server’s settings and the command login shell you chose. E.g. I
use GitBash, which is different and a bit more stripped down from the Windows 7
`cmd.exe` or PowerShell. But it allows me to operate with my Git projects and
move files around.

Additionally I’ve got Vim installed for Windows in case I need to do some file
editing directly over SSH (still need to learn to use the damn thing though).

## 6. Remember

This quick guide contains a speedy setup with not that much security in mind.
Always make sure you are the only person that can access your PC remotely. Any
harm done to you or your belongings after using this guide is not my fault.

Also, you need to keep your PC turned on to access it remotely.