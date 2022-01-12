# How to automate setup with bash

The beauty about Linux is that you can automate just about anything if you know
a little bit of bash scripting.  Anything, without extra software or any of
that.  There is software to help you, especially if you run Ops in a company.
But if you are not automatically deploying dozens of Linux instances on some
kind of Docker setup you'll probably do fine with just bash.

So since I am about to be setting up more than one new Linux machine for myself
(that's a different story) I figured I would use that to test my automation.

## GitHub

The whole point of this automation is not just that you don't have to deal with
it again every time you setup a new machine (or setup the same machine again
because you broke your Arch installation), but also that you can make changes
to it easily.  The best way is to version control it and by default that
usually means GitHub.

This way you can download your raw installation scripts using `curl`.  As an
example, my installation script is located at
[https://github.com/SonkeWohler/.vim/blob/master/setup/install.bash](https://github.com/SonkeWohler/.vim/blob/master/setup/install.bash)
and I download it with `curl
https://raw.githubusercontent.com/SonkeWohler/.vim/master/setup/install.bash`
from any new machine before I do pretty much anything else.

## Some formatting Notes

Since this script will have a lot of output from other programs I like to mark
any output from this script itself with `   +++   ` at the start, so I know
what is output I put there and what is output from other tools.

```
print() {
  echo "   +++   $1"
}
```

## Step zero, Permissions

Probably one of the first things you will have to do is install all the
software you are about to use and configure to your liking.  And that means
package manager.  Sure, you can install Flatpacks now or Snaps if you like
them, you can use AppImages, but none of those are quite as convenient to
automate out of the box.  I don't need Flatpack yet, so I don't bother adding
it as a dependency and an extra step.  Same with AppImages, I don't need them
so why add the hassle.  What I will definitely have to interact with, however,
is the package manager as a given.

So if you know you will only ever use one specific distribution you can go
ahead and just slap a list of packages in a file and do something like this:

```
curl link.to/your/file >> /home/user/file
file=/home/user/file

for package in $file; do
  sudo apt install $package
done
```

And this is where we get our first problem, you can't just use `sudo` in a
script unless you run it while having root privileges.  Try it, it won't even
ask, it'll usually just be stuck until you kill the script.

What you will need to do is `sudo /home/user/script`.  As a bonus you can leave
out any `sudo` from the script, since this is now a no-op.

We'll get back to this `sudo` problem later when we are setting up our configs,
so keep it in the back of your head.

## Step one, Package Managers

As I said, if you know you will always be using a debian based system you can
go ahead and hard-code `apt` into your script (or `apt-get` if you prefer). But
I am usually using [Garuda](https://garudalinux.org/) which uses `pacman`, I
may want to try fedora, I definitely will have a debian based machine, perhaps
I will even use [Alpine Linux](https://www.alpinelinux.org/) for some things
([on my phone](https://www.makeuseof.com/tag/how-to-linux-on-android/), for
example).  So I need to make my script robust against most package managers out
there and select the right one at runtime.

Now I didn't come up with this myself, I found it [on
Stackoverflow](https://unix.stackexchange.com/questions/46081/identifying-the-system-package-manager),
but it works great.  We identify the package manager by the distribution
specified in the `/etc/*-release` file.  Using a bash array is probably the
easiest way to do that.

```
declare -A packageManager;
packageManager[/etc/arch-release]=pacman
packageManager[/etc/debian_version]=apt
packageManager[/etc/alpine-release]=apk
packageManager[/etc/SuSE-release]=zypp
packageManager[/etc/redhat-release]=yum
packageManager[/etc/gentoo-release]=emerge

for f in ${!manager[@]}
do
    if [[ -f $f ]];then
        manager=${packageManager[$f]}
    fi
done
```

There are a few differences between the package managers.  First of all the
name (`apt`, `apk`, `pacman`, `yum`) but also the commands (`pacman -S` vs `apt
install`) and then there are update commands (`pacman -Syu` vs `apt update &&
apt update`).  Since we know the package manager we can use other arrays to
remember the commands for each manager.  Lets start with the updates.

```
declare -A packageUpdates;
packageUpdates[pacman]="pacman -Syu"
packageUpdates[apk]="apk update && apk upgrade"
packageUpdates[apt]="apt update && apt upgrade"
packageUpdates[yum]="yum update"

print "updating the system"
eval ${packageUpdates[$manager]}
print "system update done"
```

Notice I use `eval` to evaluate the string that is stored inside the array.
Feel free to see what happens if you leave this out.

### Bonus for Package Manager

I like to ensure the script made the correct choice with the dialogue below
before doing any updates.

```
print "The package manager appears to be: $manager"
print "is this correct [Y/n]?"
read -n 1 yesNo
echo "$yesNo"
yesNo="${yesNo,,}"
if [ $yesNo = "n" ]; then
  print "No?"
  print "this script doesn't handle that yet"
  print "aborting script..."
  # according to man exit 127 may be the correct code, as a utility
  # to be executed (package manager) wasn't found
  exit 127
elif [ $yesNo = "y" ]; then
  print "Good..."
else
  print "invalid answer"
  print "this script doesn't handle that yet"
  print "aborting script..."

  exit 1
fi
```

## Step two, The Packages

So far so good.  We have the package manager and we have a list of packages to
install.  Sounds simple right?

```
for package in /home/user/package_list; do
  eval ${packageInstall[manager]} $package
done
```

For many packages this will work - try it with `git`, `neovim`, ` tmux` for
example - but not with all.  For example in `pacman` I install java with
`jdk-openjdk` and things mostly work out fine, while for `apt` I install it
with `openjdk-<version>-jdk`.  So we need to have a file for each package
manager and need another array referencing the correct file.

```
declare -A packageList;
packageList[pacman]=raw.githubusercontent.com/link/to/file/for/pacman
packageList[apt]=raw.githubusercontent.com/link/to/file/for/apt
packageList[apk]=raw.githubusercontent.com/link/to/file/for/apk
   # etc ...

curl ${packageList[manager]} >> package_list
```

Now we can sort things out using our loop from above.

```
declare -A packageInstall;
packageInstall[pacman]="pacman -S"
packageInstall[apt]="apt install"
packageInstall[apk]="apk add"
packageInstall[yum]="yum install"

for package in /home/user/package_list; do
  eval "${packageInstall[manager]} $package"
done
```

### Package Installation Bonus: FZF

I have found that installing `fzf` via the package manager can be incomplete on
some systems (this is the case on Ubuntu, but I guess on anything `apt` and
possibly more).  In order to use autocompletion you need to have some files
somewhere inside `/usr/share/**/fzf` and the only package manager I know that
places them there automatically is `pacman`.  Since `/usr/share` requires root
privileges to write to I like to finish the installation script with these
files.

First we check whether they already exist (in case the package manager did
their job completely).

```
if ! [ -f /usr/share/fzf/completion.bash ]; then
  mkdir --parents --verbose /usr/share/fzf
  cd /usr/share/fzf
  wget https://raw.githubusercontent.com/junegunn/fzf/master/shell/completion.bash
  wget https://raw.githubusercontent.com/junegunn/fzf/master/shell/key-bindings.bash
fi
```

This code snippet won't bother to check any of the other locations these files
might have been installed to, but that is ok.  As far as I know fzf will also
work if the files are in `/usr/share/doc/fzf/examples` or any of these
directories, but it will also work if the files are in more than one of these
locations.  Issues only arise if you start editing them, but I trust you'll be
able to sort things out by yourself in that case.

## Step three, Git

At this point we are done with the root user privileges.  But that is not the
end of a new machine, I don't have a GitHub identity yet, and without that I
can't do much git related stuff.  Which basically means I can't do a single
useful thing.

Before anything else `git` will usually want you to have a user name and email.
You can keep a default `.gitconfig` in your setup on GitHub and `wget` it now
to your home directory.

```
cd ~
wget raw.githubusercontent.com/user/link/to/file
```

Later you can overwrite it as part of your version control, but for now this
will allow things to just work at first.

And this is where we run into the `sudo` problem again.  Try this inside your
script and there won't be any `.gitconfig` file inside your home directory
under `/home/user`.  Try it, I dare you. 

## The real step three, no more Root

Remember how we were running our
script with `sudo` earlier?  That means we ran it as `root` and not as us
(`user`).  Which in turn means `cd ~` took us to `/root` instead of
`/home/user`.  Up to this point this didn't matter since we were just
installing packages to the machine for all users.  But user specific tasks are
a problem.

The solution, I find, is to finish off your script by downloading another
script and running that without root privileges.  Something along the lines of
this.

```
curl raw.githubusercontent.com/link/to/new/file >> /home/user/script
print "setup with root prviliges finished, please continue with non-root setup"
cd /home/user
print "run it with 'source script'"
```

And now we can move on with what we were doing before.

## Step four, Git again

We start out new non-root script with the `.gitconfig` file from above.

You will also want to connect your new machine to your GitHub account so you
can push as well as pull from private repositories.  GitHub has
[insttructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
on how to do this, and most of it can be automated.

Since things revolve around ad ssh-key-file at `~/.ssh` you might want to check
if it exists already.  If not we generate it in the default file location.

```
if [ -f ~/.ssh/id_ed25519.pub ]; then
  print "ssh key already exists"
else
  print "generating ssh key for GitHub"
  mkdir --parents --verbose ~/.ssh
  ssh-keygen -t ed25519 -C "user@email.com" -f "~/.ssh/id_ed25519"
fi
```

This way we are still prompted for a passphrase at runtime, which is ok.  I
don't use one for most of my machines, but it can be useful.

Now the ssh-agent needs to know this key.

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

And here comes the part you can't automate.  You'll need to add this key to
your GitHub as a trusted public key.  All you can do in your script is to
prompt the user, copy the key to clipboard and refer them to the correct url.

```
xclip ~/.ssh/id_ed25519.pub -select clipboard
print "please go to https://github.com/settings/keys to add this ssh key to your trusted keys"
print "the key should be in your clipboard, but is printed below"
cat ~/.ssh/id_ed25519.pub
print "more information at https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account"
read -p "press any key to continue..." yesNo
```

## Step five, Dotfiles and other Clones

Finally, you can clone your dotfiles and anything else you want to clone. These
files should contain all your customization that you can do on command line
that you haven't done already.  In my case that most is importantly my
`.bashrc`, `.vimrc`, `.tmux.config` and a few others, but also some config
files for my sound plugins and hopefully soon settings for my desktop
environment.

Either way, you first have to clone the repo.

```
cd ~
git clone github.com/user/dotfiles
```

And then you will want to symlink those config files something like this.

```
ln --symbolic --verbose ~/dotfiles/vimrc ~/.vimrc
ln --symbolic --verbose ~/dotfiles/bashrc ~/.bashrc
...
```

The rest is up to you.  These are your configuration files so it depends on
what you want to keep configured.

## Step six, Enjoy!

Enjoy and improve.  Remember that life is a constant work in progress anyway.
