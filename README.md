# nwn-devbase
This repository is meant to function as boilerplate for anyone who wants to version control their module development for the game Neverwinter Nights (NWN), including NWN:EE, using git. It contains a skeleton with the necessary tools and usage documentation, including instructions for setting up a local test environment using Docker.

In addition, the texts here are meant to function as a reference for users unfamiliar with git and Docker, as is the case with some of my team members. [INTRODUCTION](https://github.com/jakkn/nwn-devbase/blob/master/INTRODUCTION.md) introduces the problem areas git and Docker solve, and attempts to explain how the development process is supposed to work. It also presents an overview of how the systems are wired together.


## But seriously, what's the point?
Can't people just version control their sources without this? Of course they can. However, it is not a straight forward process. The Aurora Toolset stores all module content in file archives with the *.mod* extension. git does not handle *.mod* archives, so for git to be of any use the archive must first be extracted. The process of extracting and packing module content may be cumbersome to some, which is why I have created this repository. It is an attempt at sharing the work I have done so that anyone who may want to do the same can do so with minimal effort. The basis for this work is what I have already done on an existing server; [Bastion of Peace](https://www.facebook.com/nwnbastionofpeace/).


## Intended audience
- **Admin** - Please see [SETUP](https://github.com/jakkn/nwn-devbase/blob/master/SETUP.md). It contains instructions on how to initialize and customize the repository to your server.
- **Developers** - Continue reading to find instructions on how to install and use the tools.


## Dependencies
*Note to windows users: chocolatey is a package manager for Windows that empowers you to install and upgrade software from the command line. For those not using chocolatey the direct download links follow after the choco command.* 

You will need to install:

- git, the version control software
  - Arch: `pacman -S git`
  - Ubuntu: `apt install git`
  - Windows: `choco install git` [git-scm.com](https://git-scm.com/download/win)

- Ruby, to run the build script and nwn-lib to convert gff to yml
  - Arch: `pacman -S ruby`
  - Ubuntu: `apt install ruby`
  - Windows: `choco install ruby` [rubyinstaller.org](https://rubyinstaller.org/downloads/)

- nwnsc, the nwscript compiler
  - All platforms: [https://neverwintervault.org/project/nwn1/other/tool/nwnsc-nwn-enhanced-edition-script-compiler](https://neverwintervault.org/project/nwn1/other/tool/nwnsc-nwn-enhanced-edition-script-compiler)

- nim, to use neverwinter_utils.nim
  - Arch: `pacman -S nim`
  - Ubuntu: [choosenim](https://github.com/dom96/choosenim)
  - Windows: [choosenim](https://github.com/dom96/choosenim)

- neverwinter_utils.nim for module packing and extracting
  - All platforms: [https://github.com/niv/neverwinter_utils.nim](https://github.com/niv/neverwinter_utils.nim)

- (Optional) Docker, to run nwserver
  - Arch: `pacman -S docker`
  - Ubuntu: See [https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
  - Windows: Depends on Hyper-V support (Windows Pro and above), please refer to [https://forums.docker.com/t/linux-container-on-windows-docker-host/25884/2](https://forums.docker.com/t/linux-container-on-windows-docker-host/25884/2) for details.
    - No Hyper-V: `choco install virtualbox docker-toolbox`
    - With Hyper-V: `choco install docker-for-windows`

- (Optional) docker-compose, for simplified docker configuration
  - Arch: `pacman -S docker-compose`
  - Ubuntu: See [https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)
  - Windows: 
    - If you installed docker-toolbox in the previous step you already have it
    -  `choco install docker-compose`


## Initialize

### Get the sources
Your module admin should have provided you with a link to your repository (NOT [nwn-devbase](https://github.com/jakkn/nwn-devbase)!). Cloning can be done via a gui client, or by running `git clone <repository-url>` from the command line.

Using a git client like [SourceTree](https://www.sourcetreeapp.com/) or [another](https://git-scm.com/download/gui/linux) is nice if you prefer a gui, but you can also do everything from the command line. Some git basics and best practices are covered and referenced in [INTRODUCTION](https://github.com/jakkn/nwn-devbase/blob/master/INTRODUCTION.md).

### Install ruby gems
Open a console, navigate to the repository, and type
```
gem install bundler
bundle install
```

If there are errors it is most likely due to improper Ruby configurations or missing PATH entries. See [troubleshooting](https://github.com/jakkn/nwn-devbase#troubleshooting).

### Symlinks
A symbolic link is used to reveal the *.mod* file to the aurora toolset. This is necessary because the module builds to a directory within the repository, while the toolset looks for the file in *NWN_INSTALLDIR/modules/*.

Windows users may use [Link Shell Extension](http://schinagl.priv.at/nt/hardlinkshellext/linkshellextension.html) to create symbolic links instead of running the shell commands.

Replace *NWN_INSTALLDIR* with the path to the install dir of your local NWN installation, and *PATH_TO_REPO* with the path to the repository for the below commands.

- Linux: `ln -s "PATH_TO_REPO"/server/modules/my-module.mod "NWN_INSTALLDIR"/modules/`
- Windows: `MKLINK "NWN_INSTALLDIR\modules\my-module.mod" "PATH_TO_REPO\server\modules\module.mod"`

### Paths
For nss compilation to work, it may be necessary to set some PATHs if the script defaults do not match with your system environment. Either specify the paths at run time (where $HOME tanslates to the home/user directory and the remaining paths must be changed to match your environment) with
```
NWN_INSTALLDIR="$HOME/Beamdog Library/00829" NSS_COMPILER="$HOME/bin/nwnsc" ruby build.rb compile
```
or set them permantently in system environment variables.

If you need more information what an environment variable is, you can find more information in the [wikipedia article](https://en.wikipedia.org/wiki/Environment_variable)

#### NSS compiler
`build.rb` looks for the *NSS_COMPILER* environment variable, and defaults to `nwnsc` if that does not exist. Either add the compiler to your PATH, or create the NSS_COMPILER environment variable that points to the nss compiler of your choice.

#### NWN install dir
The compiler run arguments specify game resources located in *NWN_INSTALLDIR* environment variable. This is needed to locate `nwscript.nss` and base game includes.

## Use
All use should be done through `build.rb` and not the rake files, because `build.rb` will update the cache properly. Run it from the command line by navigating to the repository root folder, and issue `ruby build.rb`. No arguments prints the usage information.

Example use:
```
cd /home/user/my-module-repository
ruby ./build.rb extract
```

To version control your changes to the sources use the git commands `git pull`, `git add`, `git commit`, `git push` accordingly.

For Docker usage, please refer to [DOCKERGUIDE](https://github.com/jakkn/nwn-devbase/blob/master/DOCKERGUIDE.md).

#### Hints
##### Scripting in other editors
Some might find other editors to be faster, easier to navigate, and to provide better syntax highlighting than the Aurora toolset.
###### Visual Studio Code
VSCode is an excellent choice for scripting outside of the toolset. The editor will prompt you to add the nwscript extension when you open a .nss file. 

###### Sublime Text
*TODO: OUTDATED INSTRUCTIONS*
Setting up Sublime Text for scripting requires a few steps to set up the custom NWN script compiler (NWNScriptCompiler).

- Install [Sublime Text 3](http://www.sublimetext.com/3)
- Install [Package Control](https://packagecontrol.io/installation)
- Install [STNeverwinterScript](https://github.com/CromFr/STNeverwinterScript) plugin
- Open the Sublime project described by the file nwn-devbase.sublime-project located in the root directory of this repository
- Tools->Build System->NWN compile
- Hit ctrl+b to compile open nss files or ctrl+shift+b for all the other build options
 
##### Windows PowerShell
Windows users may find this blog post titled [make powershell and git suck less on windows](http://learnaholic.me/2012/10/12/make-powershell-and-git-suck-less-on-windows/) useful.

## Troubleshooting
"Too many files open": nwn-lib opens all files for reading at the same time when packing the module. This can lead to an error stating too many files are open.
Fix:

- *Linux* `ulimit -n 4096` (or any other number higher than the number of files in the module)
- *Windows* the Java library modpacker is used instead. If modpacker cannot be found build.rb will print out instructions.

I have installed Ruby but it does not work: This is most likely due the Ruby executable missing from your PATH environment variable. If this is new to you and you're on Windows, [please ask google first](https://www.google.com/search?q=windows+path&oq=windows+path&aqs=chrome.0.0l6.1280j0j1&sourceid=chrome&ie=UTF-8#q=windows+10+change+path). Linux users should not have this issue.


## build.rb schematics

Schematics of how the build script operates available at https://drive.google.com/file/d/156hELaw_3fwGeCWexYFmJDJiBrJu23-x/view?usp=sharing


## Feedback
Feedback is greatly appreciated. If you have any thoughts, suggestions or ideas about how to improve this project, please feel free to raise issues, create pull requests, or look me up on email or on discord.
