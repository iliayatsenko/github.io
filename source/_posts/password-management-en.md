---
title: Programmerish password management
date: 2023-07-17
tags:
    - password-management
    - manual
categories:
    - EN
---

Modern life without password manager is hard. In this article I will tell how I made it easier.

<!-- more -->

## The problem

You know all this stuff: endless repetitive restoring of passwords, having one password for all the cases, and getting annoyed when sites refuse to accept it again and again, then adding random symbols to the end, forgetting about them, and the same story again… I suffered for a long time because I was afraid of wrapping my head around new concepts and setting up new tools. But one day I decided to try one of the popular password managers. How did it end, read next.

At first, I installed OnePassword and LastPass browser extensions and mobile (iOS) apps. I tried to store-retrieve several passwords and ran into complexity and strange behavior. For example, the extension’s popup suggesting passwords on inputs was constantly overlapped with a similar Chrome default popup, which I didn’t find a way to turn off.

Management panels of these services were crowded with irrelevant details and many redundant features for me. Also, these services were too “engaging”, asking for a lot of attention all the time: popups, notifications, “upgrade to pro”, “free trial”, adorable gamified collecting of personal data, etc. Also, I didn’t understand how they work: there was no explanation or simplified diagram, only endless marketing proclamations about how secure they are.

I can’t say that these tools are bad in any way, they are great and very popular. That was just not what I wanted. I wanted a minimalistic tool, understandable for me (at least superficially, I am not a security expert in any way). So I decided to return to my previous management style (or, rather, its absence).

## Solution

But after some time I encountered a linux ‘pass’ utility. “This is salvation” - thought I after reading the short and clear [description](https://www.passwordstore.org/). And it really was: now I am enjoying using it on all my devices.

It is the simple console app to manage encrypted notes. Except basic operations like create, read, update, delete, it's possible to sync your notes with remote git repository. Encryption is made using GPG key. Also, clients exist for all popular platforms.

Think about GPG keys as an analog of SSH keys, but with some subtle differences. Anyway, they serve a similar task - securely sharing data between peers, and both are based on asymmetric encryption (public and private keys).

But setup was not so easy, despite it absolutely was worth it. Here I describe what I've done, sequentially. If some steps are not relevant for you, just skip them.

#### Setting up `pass` on Ubuntu
```bash
sudo apt-get install pass
```

Then, it is needed to initialize it by passing the ID of GPG key. Here is how to [find GPG key ID](#setting-up-gpg-keys). After initialization, we can start [using `pass`](#pass-usage) locally, or set up [synchronization with remote repository](#synchronization-with-remote-git-repository).

#### Setting up GPG keys

Usually, one pair of GPG keys is already present in the system, but I preferred to generate a new one. Here are good manuals from Github about how to [find](https://docs.github.com/en/authentication/managing-commit-signature-verification/checking-for-existing-gpg-keys) or [generate new](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key) GPG keys.

By the way, some people recommend to [make QR-codes from your GPG keys](#gpg-keys-to-qr-codes), print these codes on paper and store them securely. I've done it, because in case of keys loss, it will be impossible to recover them, and all encrypted data will be lost. Here is how to [make QR-codes from your GPG keys](#gpg-keys-to-qr-codes).

When we have GPG key ID in place, run:
```bash
pass init <your GPG key ID>
```

This setup is enough to [start using `pass`](#pass-usage) locally.

#### Synchronization with remote git repository

But we would like to sync passwords across devices, so we need to share our GPG keys to all of them and to set up git integration.

Create a private git repository, name it somehow like "password-store". Then run:
```bash
pass git init \
    && pass git remote add origin git@github.com:yourgithubname/password-store.git \
    && pass git push -u --all
```

Now we can setup `pass` on other devices, like [iOS](#set-up-for-ios) or [Mac](#setup-for-mac), and sync connect them to this repository and synchronize passwords.

#### Set up for iOS

Install application [Pass for iOS](https://mssun.github.io/passforios/). And open it.

Go to `Settings` → `PGP Key`. Here we can choose how to import PGP keys on your iPhone. PGP stands for “Pretty-Good-Privacy” and GPG stands for “GNU Privacy Guard”. As GPG is simply an open-source implementation of PGP, here we can use them interchangeably. I selected “ASCII-Armor Key” and it allowed me to paste the key as text or to scan QR-codes. I chose QR-codes, because it looks more secure than sending them as text via internet.  And here is [how to create QR-codes](#gpg-keys-to-qr-codes) from your GPG keys. After creating QR-codes, open and scan them with your iPhone camera one by one.

When both private and public GPG keys are imported on iPhone, we need to connect our Github repo. Go to Settings → Password Repository and fill in the fields with the following values:

- Git repository URL: `ssh://git@github.com/yourgithubname/password-store.git` (NOTE replacement of “:” with “/”, original was `git@github.com:yourgithubname/password-store.git`),
- Username: `git`,
- Branch name: `<name of your main branch>`,
- Supported authentication Method: `SSH Key`.

Now select `ASCII-Armor Key`. Then we need to import on your iPhone a private SSH key, which we use to access Github. I’ve chosen to scan them as QR-codes. Here are good manuals from Github about how to [find](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys) or [generate new](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) SSH keys, and how to [add them to your Github account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account). 

After your key (new or existing one) is added to your Github, generate QR-code from your private SSH key:
```bash
cd ~/.ssh && \
    qrencode -r id_rsa -o id_rsa.png
```

Open id_rsa.png image and scan it with your iPhone camera. SSH key is imported. Do not forget to delete sensitive files safely:
```bash
shred id_rsa.png \
    && rm id_rsa.png
```

Now we can switch to Passwords pane, swipe down, and all your passwords will be synced with the remote repository. Congratulations!

#### Setup for Mac

On macbook to install `pass`, run this:
```bash
brew install pass
```

Then we need to [setup GPG keys](#setting-up-gpg-keys). After that we can start [using `pass` locally](#pass-usage), or [sync with remote git repository](#synchronization-with-remote-git-repository).

But I've started on Ubuntu, so I've got my GPG keys there. Now I need to import them on Mac somehow. If both your Ubuntu and Mac computers are connected to the same network, the easiest way to do this is [via SSH in local network](#sharing-gpg-keys-via-ssh-in-local-network).

#### Sharing GPG keys via SSH in the local network

 To share GPG keys between two computers inside the local network we need to set up SSH server on one computer and `scp` needed files on another computer. In this example we will share GPG keys from Ubuntu to Mac.
 
 Ubuntu side run this:
```bash
sudo apt install openssh-server \
    && sudo ufw allow ssh
```

Then to know your IP in local network:
```bash
ip address | grep wl
```

The command above will print something like `3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000 inet 192.168.0.122/24 brd 192.168.0.255 scope global dynamic noprefixroute wlp0s20f3`, where `192.168.0.122` is your IP in local network.

Now we need to dump our public and private GPG keys to files:
```bash
gpg --armor --export-secret-key --output secret.asc <your GPG key ID> \
    && gpg --armor --export --output public.asc <your GPG key ID>
```

Then, on Mac run these commands to copy GPG keys via `scp` utility:

```bash
scp <your Ubuntu user name>@<your Ubuntu IP in local network>:/absolute/path/to/public.asc public.asc
```
and for secret key:
```bash
scp <your Ubuntu user name>@<your Ubuntu IP in local network>:/absolute/path/to/secret.asc secret.asc
```

These commands possibly will ask for the password of your Ubuntu user, and then copy private key from Ubuntu to Mac in current directory.

After this we need to import GPG keys from the received files:
```bash
gpg --import secret.key \
    && gpg --import public.key
```

Now to check if keys imported:
```bash
gpg --list-keys
```

The command above should list all GPG keys, where the one with the same ID as on Ubuntu should be present.

Now init `pass` and `git`:
```bash
pass init <your GPG key ID from Ubuntu system> \
    && pass git init
```

Now add your repository:
```bash
pass git remote add origin git@github.com:yourgithubname/password-store.git
```

If your Mac can access your Github, now you can run
```bash
pass git pull
``` 
And all the data will be pulled from the repository.

But to make your Mac able to access your Github, possibly you will need add it’s SSH key to the account.

Do not forget to safely delete sensitive files both on Ubuntu and on Mac:
```bash
shred secret.asc public.asc \
    && rm secret.asc public.asc
```

And maybe deny further SSH connections on Ubuntu:
```bash
sudo ufw deny ssh
```

#### GPG keys to QR-codes

As the whole key don’t fit into the amount of data which QR-code can transmit, we will need to split it on parts, and generate a sequence of QR-codes. Firstly, let's generate QR-codes for the private key.

We need to dump the key into the temporary file.
```bash
gpg --armor --export-secret-key --output secret.asc <your GPG key ID>
```
We will use `qrencode` utility which is already available on Ubuntu (if not, install it with `sudo apt install qrencode`).

We could try to run simply:
```bash
qrencode -o secret.png < secret.asc
```

But probably we will get this error “Failed to encode the input data: Input data too large”.

So, we need to split the file into several parts, and generate QR-code from each of them.

```bash
split -C 1000 secret.asc secret.asc-
```

Then we can generate QR-codes from each part with:
```bash
for i in secret.asc-*; do qrencode -o "${i}.png" < "${i}"; done
```

This will produce several PNG images, named somehow like this: `secret.asc-aa.png`, `secret.asc-ab.png`, `secret. asc-ac.png`, etc. Now we can print them (do not mix them up, order matters!) and store securely. After printing or scanning them, do not forget to delete them
```bash
shred secret.asc secret.asc-* \
    && rm secret.asc secret.asc-*
```

And for public one the process is similar:

```bash
gpg --armor --export --output public.asc <your GPG key ID> \
    && split -C 1000 public.asc public.asc- \
    && for i in public.asc-*; do qrencode -o "${i}.png" < "${i}"; done
```

Scan or store QR-codes and delete files and generated images after that:
```bash
shred public.asc public.asc-* && rm public.asc public.asc-*
```

#### Pass usage

Pass usage is pretty simple. 
For example, to generate a new password, store it and copy to the clipboard, simply run
```bash
pass generate -c <password name>
```
and newly generated password will be copied to the clipboard. It will be automatically removed from there after 45 seconds.

To copy password to the clipboard, run
```bash
pass -c <password name>
```
and password will be copied to the clipboard and automatically removed from there after 45 seconds.

After setting up the [synchronization with a remote repository](#synchronization-with-remote-git-repository), in order to sync your passwords, run
```bash
pass git push
```
or
```bash
pass git pull
```

More examples about how to use `pass` can be found on [man page](https://git.zx2c4.com/password-store/about/).