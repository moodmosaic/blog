---
layout: basic
title: Setting up an Ubuntu environment
summary:
id-image: 10
tags:
  - Miscellaneous
---

Tried on Ubuntu 16.04.3 LTS (Xenial Xerus)

- Essentials
  * [Chrome](#chrome)
  * [Git](#git)
  * [Jekyll](#jekyll)
    + [Rouge](#rouge)
- Programming languages and tools
  * Haskell
    + [mafia](#mafia)
    + [stack](#the-haskell-tool-stack)
  * F#
    + [.NET Core](#net-core)
    + [Mono](#mono)
    + [Compiler and F# Interactive](#f)
  * Testing
    + [dieharder](#dieharder)
- Text editors
    * [Sublime Text](#sublime-text)
    * [Vim 8](#vim-8)
    * [Visual Studio Code](#visual-studio-code)
- Maintenance
  * [APT](#apt-maintenance)
- VMware Fusion
  * [Mount Shared Folders](#mount-shared-folders-vmware-fusion)

## Chrome

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

If you encounter any errors simply use

```bash
sudo apt-get -f install
```

To run it from terminal use `google-chrome` or `google-chrome --incognito`.

I usually run it with
- `google-chrome --disable-background-networking --incognito`

and in the settings I disable
- Continue running background apps when Google Chrome is closed
- Use hardware acceleration when available

See also:
- <https://askubuntu.com/a/510063>
- <https://askubuntu.com/a/894683/782554>

[Back to top](#)

## Sublime Text

Install the GPG key:

```bash
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```

Ensure APT is set up to work with https sources:

```bash
sudo apt-get install apt-transport-https
```

Use the stable channel:

```bash
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt-get update
sudo apt-get install sublime-text
```

Type `subl` to run it from terminal.

To find where Sublime Text ([or any other package](https://www.howtogeek.com/howto/ubuntu/see-where-a-package-is-installed-on-ubuntu/
)) is installed:

```bash
dpkg -L sublime-text
```

See also:
- <https://www.sublimetext.com/docs/3/linux_repositories.html>

[Back to top](#)

## Vim 8

There's an unofficial PPA that you can use to easily install Vim 8.0:

```bash
sudo add-apt-repository ppa:jonathonf/vim
sudo apt update
sudo apt install vim
```

That’s all. If you want to uninstall it, use the commands below:

```bash
sudo apt remove vim
sudo add-apt-repository --remove ppa:jonathonf/vim
```

You might also want to set Vim as the default editor for Git:

```bash
git config --global core.editor "vim"
```

See also:
- <https://itsfoss.com/vim-8-release-install/>
- <https://stackoverflow.com/a/2596835>

[Back to top](#)

## Git

```bash
sudo apt-get install git
```

If you're used to `git gui` on Windows, that's available too:

```bash
sudo apt-get install git-gui
```

You might also want to set Vim as the default editor for Git (make sure Vim is already [installed](#vim-8)):

```bash
git config --global core.editor "vim"
```

See also:
- <https://git-scm.com/download/linux>
- <https://stackoverflow.com/a/2596835>

[Back to top](#)

## Jekyll

```bash
sudo apt-get install ruby ruby-dev
sudo apt-get install build-essential
sudo gem install jekyll jekyll-feed jekyll-sitemap
```

Tested with [my Jekyll blog](http://github.com/moodmosaic/blog), which is extremely minimal. Perhaps you need to install more Jekyll extensions.

See also:
- <https://jekyllrb.com/docs/windows>

[Back to top](#)

## Rouge

```bash
sudo gem install kramdown rouge
```

Edit your `_config.yml` settings

```yaml
markdown: kramdown

kramdown:
  input: GFM
  syntax_highlighter: rouge
```

or, for Jekyll 3:

```
markdown: kramdown
highlighter: rouge
```

Create a css file for the highlighting style you want

```bash
rougify help style
rougify style igorpro > syntax.css
```

See also:
- <https://bnhr.xyz/2017/03/25/add-syntax-highlighting-to-your-jekyll-site-with-rouge.html>
- <https://github.com/jneen/rouge>

[Back to top](#)

## Mafia

Mafia expects both GHC and Cabal to be installed and on the `PATH`.

Follow the guides below to configure your system correctly:

* [Installing GHC](https://github.com/haskell-mafia/mafia/blob/master/docs/installing-ghc.md)
* [Installing Cabal](https://github.com/haskell-mafia/mafia/blob/master/docs/installing-cabal.md)

See also:
- <https://github.com/haskell-mafia/mafia>

[Back to top](#)

## The Haskell Tool Stack

```bash
curl -sSL https://get.haskellstack.org/ | sh
# or
wget -qO- https://get.haskellstack.org/ | sh
```

See also:
- <https://docs.haskellstack.org/en/stable/README>

[Back to top](#)

## Visual Studio Code

The repository and GPG key can be installed manually with the following script:

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
```

Then update the package cache and install the package using:

```bash
sudo apt-get update
sudo apt-get install code # or code-insiders
```

See also:
- <https://code.visualstudio.com/docs/setup/linux>

[Back to top](#)

## Mount Shared Folders (VMware Fusion)

```bash
cd ~
mkdir folder_name
vmhgfs-fuse .host:/$(vmware-hgfsclient) ~/folder_name
```

See also:
- <https://askubuntu.com/a/781314>

[Back to top](#)

## .NET Core

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg

sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update

sudo apt-get install dotnet-sdk-2.1.3
```

See also:
- <https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore2x?>

[Back to top](#)

## Mono

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/ubuntu xenial main" | sudo tee /etc/apt/sources.list.d/mono-official.list
sudo apt-get update
sudo apt-get install mono-devel
```

See also:
- <http://www.mono-project.com/download>

[Back to top](#)

## F#

```bash
sudo apt-get update
sudo apt-get install fsharp
```

See also:
- <http://fsharp.org/use/linux>

[Back to top](#)

## Dieharder

```bash
sudo apt-get install dieharder
```

See also:
- <http://manpages.ubuntu.com/manpages/xenial/man1/dieharder.1.html>

## Ration

Quickly resize and position your windows. Similar to [Divvy](http://mizage.com/divvy/), but for Linux.

```bash
sudo apt-get install python-gtk2 python-pip python-keybinder wmctrl
pip install ration
```

See also:
- <https://github.com/onyxfish/ration>

[Back to top](#)

## APT maintenance

```bash
sudo apt-get check
sudo apt-get -f install
sudo apt-get autoclean
sudo apt-get clean
sudo apt-get autoremove
```

See also:
- <https://help.ubuntu.com/community/AptGet/Howto>

```bash
sudo rm -rf /var/lib/apt/lists/*
sudo apt-get update
```

See also:
- <https://askubuntu.com/a/179976>

If you get this error while trying to install a package or tool:

```
E: Could not get lock /var/lib/dpkg/lock - open (11 Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/) is another process using it?
```

You can delete the lock file with the following command:

```bash
sudo rm /var/lib/apt/lists/lock
```

You may also need to delete the lock file in the cache directory:

```bash
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
```

See also:
- <https://askubuntu.com/a/102084>

[Back to top](#)
