# CIMPL Setup and Installation

***

**Table of Contents**

[TOC]

***

# Background
CIMPL (and, by extension, the SHR-CLI tool) is a text-based tool that requires its users to understand and use your operating system's [command line](https://en.wikipedia.org/wiki/Command-line_interface).  If you are uncomfortable with using the command line, try [this short introduction](https://tutorial.djangogirls.org/en/intro_to_command_line/).

# Windows Installation Instructions
_Note: These installation instructions assume that you are installing applications using an account that has administrator-level access.  If you do not have administrator rights on your machine, some of the installations may fail or refuse to install._

For a successful installation, it is critical that you:

* Read the instructions **thoroughly**.
* Follow instructions **in order**, without skipping around.
* Pay attention to variables (like your organization's proxy) that you may need to replace.
* Allow enough time (30 min to 1 hour)

## Windows Proxy Setup
_Warning: Many professional organizations use proxy servers to filter incoming internet content (e.g., antivirus checks, adult material, etc.).  Many of the applications that support CIMPL require that these proxy servers be explicitly declared before they will function properly.  If your organization does not use a proxy server, you can skip ahead to installing the [Supporting Software](#supporting-software)._

If your organization uses a proxy server, you will need to set up environment variables before proceeding.

* Copy the following code block into a text editor and replace `your.proxy.org` with your organization's proxy server and `80` with the port used by your organization's proxy server:
```
setx HTTP_PROXY http://your.proxy.org:80
setx HTTPS_PROXY https://your.proxy.org:80
setx http_proxy http://your.proxy.org:80
setx https_proxy https://your.proxy.org:80
setx proxy http://your.proxy.org:80
setx JAVA_OPTS "-Dhttp.proxyHost=your.proxy.org -Dhttp.proxyPort=80 -Dhttps.proxyHost=your.proxy.org -Dhttps.proxyPort=80 -DsocksProxyHost=your.proxy.org -DsocksProxyPort=80"
```

* Open a windows command prompt
* Copy and paste the modified code block (with your organization's proxy server) into the command prompt; hit enter to execute.
* **Close and re-open your command prompt** to make sure those settings take effect

Some of the following installation steps will also include specific setups for running the CIMPL tooling and HL7 FHIR IG Publisher from behind a proxy server.

## Supporting Software
Unlike most applications that include a standalone installation file that you can click on to install, CIMPL is dependent on the installation of other applications, like [Node.js](#node-js-for-windows) and [Yarn](#yarn-for-windows), and optionally, [Git](#git-for-windows) and [Visual Studio Code](#visual-studio-code).

### Node.js for Windows
The CIMPL command-line utilities are built using the [Node.js runtime environment](https://nodejs.org/).  It is recommended that CIMPL developers [download and install the LTS (long-term support) version of Node.js](https://nodejs.org/).  Use the default options in the installer file (i.e., just keep clicking 'Next' until the installation is complete)

### Yarn for Windows
[Yarn](https://yarnpkg.com/lang/en/) is a dependency manager that ensures that all the prerequisite code libraries on which CIMPL depends are properly installed.  It is recommended that CIMPL developers [download and install the stable version of Yarn](https://yarnpkg.com/en/docs/install#windows-stable).  Use the default options in the installer file (i.e., just keep clicking 'Next' until the installation is complete)

#### _Yarn Proxy Setup_
_Note: If your organization uses a proxy server, replace `http://your.proxy.org` with your organization's proxy server in the following code block:_
```
yarn config set proxy http://your.proxy.org
yarn config set strict-ssl false
```
...then run the modified code block (with your organization's proxy server) in the command prompt.
**Close and re-open your command prompt** to make sure those settings take effect.

### Visual Studio Code
Visual Studio Code (occasionally referred to as 'VSCode', or simply 'Code') is not a strict dependency (i.e., you can author CIMPL files using any text editor), but it is recommended that you use it for CIMPL development.  An extension for VSCode has been developed that offers syntax highlighting, autocomplete, and other things that make life easier.

To get started, download and install the latest stable build from the [VSCode homepage](https://code.visualstudio.com).  You can use the default options in the installer file, but some developers find it helpful to check these checkboxes during the installation wizard:

* Add "Open with Code" action to Windows Explorer file context menu
* Add "Open with Code" action to Windows Explorer directory context menu
* Register Code as an editor for supported file types

#### Proxy Setup 
_Note: If your organization uses a proxy server, you'll need to set this up in the VSCode settings.  If your organization **does not** use a proxy server, you can skip ahead to the [CIMPL Extension Setup](#cimpl-extension-setup)._

To ensure that Visual Studio Code can download extensions from the internet,follow this steps:

* Open Visual Studio Code
* Open the User Settings menu (File -> Preferences -> Settings)
* Search the settings for "proxy", or open Application -> Proxy.
* Insert your proxy in the **Proxy** text box, in the format "http://your.proxy.org".
* Make sure "Proxy Strict SSL" is checked.
* Save, exit, and re-open Visual Studio Code to make sure the settings take effect

#### CIMPL Extension Setup
CIMPL authors have created tools for VSCode that offer syntax highlighting, autocomplete, and other things that make life easier. These tools are known as 'extensions' in VSCode (for more information, watch this [video about working with extensions](https://code.visualstudio.com/docs/introvideos/extend)).  To enable CIMPL syntax highlighting in VSCode:

* Open the extensions menu (Using View -> Extensions) or the icon on the bottom of the left toolbar that looks like boxes)
* In the search box, type "cimpl"
* Look for the `vscode-language-cimpl` extension and click the `Install` button
* When the installation has completed, click the `Reload` button

## SHR-CLI Tool
With the supporting software installed, you can move on to installing the actual SHR-CLI tool.

* Create the directory where you want to install CLI, for example, /Documents/cimpl
* Download the [SHR-CLI zip file](https://github.com/standardhealth/shr-cli/archive/master.zip)
* Unzip the file into the directory you created. This will create a new folder with a name like `shr-cli-x.y.z` where `x.y.z` is the version number of SHR-CLI.
* Open a command window, navigate into the directory where the new files have been unzipped, using the change directory (cd) command.
* Run the command: `yarn`

**Note**: Each time you download a new version of SHR-CLI, you must re-run `yarn` command.

For git mavens, there is an alternate approach. After [installing git](#git-for-windows), open **'Git Bash'** (not the windows command prompt) and enter the following commands, which grab the latest version of the SHR-CLI tool and places it in the `~/cimpl/shr-cli` directory, and installs it using Yarn:

```
cd ~
mkdir cimpl
cd cimpl
git clone https://github.com/standardhealth/shr-cli.git
cd shr-cli
yarn
```

## FHIR IG Generation Tool
The [FHIR implementation guide (IG) publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation) is a Java-based tool that is owned and operated by the HL7 FHIR team.  It requires that Java and Jekyll (a Ruby gem) be installed in order to run properly.

### Windows Java Installation
The IG publisher is primarily written in java and requires JRE version 8.  To install JRE:

* Go to the [Java SE Runtime Environment 8 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) page
* Click the 'Accept License Agreement' radio button
* Download the most recent JRE 8 installer for your operating system

**Note:** You may need to create an Oracle account to download JRE 8

### Windows Jekyll Installation
Jekyll is a `ruby` library that takes code and turns it into webpages.  That is especially helpful for generating static websites like the FHIR Implementation Guides.

Use the [Windows Installation Instructions for Jekyll](http://jekyllrb.com/docs/windows):

* [Download RubyInstaller](https://github.com/oneclick/rubyinstaller2/releases) and run it, using the default options in the installer file (i.e., just keep clicking 'Next' until the installation is complete)
* After clicking 'Finish', a command prompt should open asking about an `MSYS2 base installation`. Press `Enter` to continue.
* When that installation is complete, open a new command prompt from the Windows start menu and type:
```
gem install jekyll bundler
```
* Check to see that Jekyll is installed using:
```
jekyll -v
```
You should see the following:
```
jekyll X.X.X
```
...where `X.X.X` is the latest Jekyll version number.

If you're running the IG Generation tool behind a proxy, you may need to [explicitly declare the proxy at runtime](https://github.com/standardhealth/shr-cli#creating-the-fhir-implementation-guide-using-an-http-proxy), since the process tries to access external terminology servers.

That's it!  To test whether your CIMPL environment has been properly set up, try the [Hello World](https://github.com/standardhealth/shr-cli/wiki/Hello-World) example.

# macOS Installation Instructions
_Note: These installation instructions assume that you are installing applications using an account that has administrator-level access.  If you do not have administrator rights on your machine, some of the installations will fail or the applications will refuse to install._

For a successful installation, it is **critical** that you:

* Read the instructions **thoroughly**
* Follow instructions **in order**, without skipping around
* Pay attention to variables (like your organization's proxy) that you may need to replace

## Terminal Proxy Setup
_Warning: Many professional organizations use proxy servers to filter incoming internet content (e.g., antivirus checks, adult material, etc.).  Many of the applications that support CIMPL require that these proxy servers be explicitly declared before they will function properly.  If your organization does not use a proxy server, you can skip ahead to installing the [Supporting Software](#supporting-software)._

If your organization uses a proxy server and you use the standard [Terminal app for macOS](https://en.wikipedia.org/wiki/Terminal_(macOS)), you will need to set up environment variables before proceeding.

* Copy the following code block into a text editor and replace `your.proxy.org` with your organization's proxy server and `80` with the port used by your organization's proxy server:
```
echo "export http_proxy=http://your.proxy.org:80
export https_proxy=http://your.proxy.org:80
export HTTP_PROXY=http://your.proxy.org:80
export HTTPS_PROXY=http://your.proxy.org:80
export JAVA_OPTS='-Dhttp.proxyHost=your.proxy.org -Dhttp.proxyPort=80 \
-Dhttps.proxyHost=your.proxy.org -Dhttps.proxyPort=80 \
-DsocksProxyHost=your.proxy.org -DsocksProxyPort=80'" >> ~/.bash_profile;\
source ~/.bash_profile
```

* Open Terminal
* Copy and paste the modified code block (with your organization's proxy server) into the terminal

Some of the following installation steps will also include specific setups for running the CIMPL tooling from behind a proxy server.

## Supporting Software
Unlike most applications that include a standalone installation file that you can click on to install, CIMPL is dependent on the installation of other applications, like [Node.js](#node-js-for-macos), [Yarn](#yarn-for-macos), [Git](#git-for-macos), and [Visual Studio Code](#visual-studio-code-1).

### Node.js for macOS
The CIMPL command-line utilities are built using the [Node.js runtime environment](https://nodejs.org/).  It is recommended that CIMPL developers [download and install the LTS (long-term support) version of Node.js](https://nodejs.org/).  Use the default options in the installer file (i.e., just keep clicking 'Next' until the installation is complete)

### Homebrew
[Homebrew](https://brew.sh/) is a package manager for macOS which will allow us to install several other applications.

To install homebrew, enter the following in a terminal window:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
_Note: During the installation, you may be asked if you want to install 'OSX Developer Tools'.  If this occurs, answer 'yes'._

### Yarn for macOS
[Yarn](https://yarnpkg.com/lang/en/) is a dependency manager that ensures that all the prerequisite code libraries on which CIMPL depends are properly installed.

With [homebrew](#homebrew) installed, yarn can be installed by entering the following in a terminal window:
```
brew install yarn
```

### Visual Studio Code
Visual Studio Code (occasionally referred to as 'VSCode', or simply 'Code') is not a strict dependency (i.e., you can author CIMPL files using any text editor), but it is recommended that you use it for CIMPL development.  An extension for VSCode has been developed that offers syntax highlighting, autocomplete, and other things that make life easier.

To get started, download and install the latest stable build from the [VSCode homepage](https://code.visualstudio.com).

#### Proxy Setup 
_Note: If your organization uses a proxy server, you will need to add some proxy settings to Visual Studio Code.  If your organization **does not** use a proxy server, you can skip ahead to the [CIMPL Extension Setup](#cimpl-extension-setup-1)._

To ensure that Visual Studio Code can download extensions, a couple lines of code need to be added to the user settings file.  Code's user settings are a bit different than you may be used to - they are all stored in a JSON file that can be edited from within VSCode (for more information, check out the [user settings documentation](https://code.visualstudio.com/docs/getstarted/settings)). To add the proxy settings:

* Open Visual Studio Code
* Open the User Settings menu (Code -> Preferences -> Settings in the menu bar or `Command + ,`)
* Replace `http://your.proxy.org` with your actual proxy server in the following code block:
```
"http.proxy": "http://your.proxy.org",
"http.proxyStrictSSL": false
```

* Copy and paste those lines between the curly brackets in the panel on the right side
* Your User Settings should now look like this:

![macOS User Settings](https://github.com/standardhealth/cimpl-tutorial/blob/master/images/macos_proxy_settings.png?raw=true)

* Save the changes (Using File -> Save in the menu bar or `Command + S`)
* Exit and re-open Visual Studio Code to make sure the settings take effect

#### CIMPL Extension Setup
CIMPL authors have created tools for VSCode that offer syntax highlighting, autocomplete, and other things that make life easier. These tools are known as 'extensions' in VSCode (for more information, watch this [video about working with extensions](https://code.visualstudio.com/docs/introvideos/extend)).  To enable CIMPL syntax highlighting in VSCode:

* Open the extensions menu (Using View -> Extensions in the menu bar, `Command + Shift + X`, or the icon on the bottom of the left toolbar that looks like a box inside of a box)
* By default, the extensions are filtered by `@sort:installs`. Replace this with `cimpl`
* Look for the `vscode-language-cimpl` extension and click the `Install` button
* When the installation has completed, click the `Reload` button

## SHR-CLI Tool
With the supporting software installed, you can move on to installing the actual SHR-CLI tool.  Open a terminal window and enter the following commands:
```
cd ~
mkdir cimpl
cd cimpl
git clone https://github.com/standardhealth/shr-cli.git
```
This grabs the latest version of the SHR-CLI tool and places it in the `~/cimpl/shr-cli` directory.

To set up the SHR-CLI tool, run:
```
cd ~/cimpl/shr-cli
yarn
```

_Note: If you encounter `error unable to get local issuer certificate`, run the following command in terminal:_
```
yarn config set strict-ssl false
```
_...and then re-run yarn using:_
```
yarn
```

## FHIR IG Generation Tool
The [FHIR implementation guide (IG) publisher](http://wiki.hl7.org/index.php?title=IG_Publisher_Documentation) is a Java-based tool that is owned and operated by the HL7 FHIR team.  It requires that Java and Jekyll (a Ruby gem) be installed in order to run properly.

### MacOS Java Installation
The IG publisher is primarily written in java and requires JRE version 8.  To install JRE:

* Go to the [Java SE Runtime Environment 8 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) page
* Click the 'Accept License Agreement' radio button
* Download the most recent JRE 8 installer for your operating system

### MacOS Jekyll Installation
Jekyll is a `ruby` library that takes code and turns it into webpages.  That is especially helpful for generating static websites like the FHIR Implementation Guides.

If you don't know if the `xcode developer tools` are installed, run the following command:
```
xcode-select --install
```
Install the latest version of Ruby:
```
brew install ruby
```
When ruby is finished installing, run the following:
```
gem install bundler jekyll
```

_NOTE: If your system requires `sudo` to gem install bundler and jekyll, use the following command to preserve environment variables when using `sudo`:_
```
sudo -E gem install bundler jekyll
```

If you're running the IG Generation tool behind a proxy, you may need to [explicitly declare the proxy at runtime](https://github.com/standardhealth/shr-cli#creating-the-fhir-implementation-guide-using-an-http-proxy), since the process tries to access external terminology servers.

That's it!  To test whether your CIMPL environment has been properly set up, try the [Hello World](cimpl6Tutorial_helloWorld.md) example.

# Appendix: Installing Git for Team Projects


## Git for Windows
Git is a free and open source distributed version control system that allows developers working simultaneously on a single project to manage various updates to their code. 

Git is not a strict dependency (i.e., you can run the SHR-CLI tool without git), but for team projects, it is recommended for CIMPL model developers to use git, along with a hosted git solution like [GitHub](https://github.com/) or [GitLab](https://gitlab.com/explore).

Download the [installation package for Git for Windows](https://git-scm.com/download/win) and run it, using the default options in the installer file (i.e., just keep clicking 'Next' until the installation is complete).

This will install a few applications, but the most useful for CIMPL development will be the 'Git Bash' application.  This opens a command-line prompt which emulates the use of git on a Unix-based system (e.g., Linux or macOS).  Most CIMPL developers use Unix-based systems, so you may want to use 'Git Bash' for working with CIMPL instead of the standard Windows command line.  There are many tutorials online about the use of git.  Users that are new to Unix or bash may find this [video introduction to Git Bash](https://www.youtube.com/watch?v=oQc-2gsjgDg) helpful.

You should also set up git so that any changes you make to the codebase are properly identified (replacing the name and email placeholders with your name and email):
```
git config --global user.name "Your Name Here"
git config --global user.email "your_email@work.org"
```

#### Git Proxy Setup
_Note: If your organization uses a proxy server, you will need to add some proxy settings to git.  If your organization **does not** use a proxy server, you can skip ahead to installing [Visual Studio Code](#visual-studio-code)._

* Replace `your.proxy.org:80` with your actual proxy server and port in the following code block:
```
git config --global http.proxy http://your.proxy.org:80
git config --global url."https://".insteadOf git://
```

* Open a windows command prompt
* Copy and paste that code block (with your organization's proxy server) into the command prompt
* **Close and re-open your command prompt** to make sure those settings take effect.

#### Learning Git
If you're totally new to Git, github has a great introduction where you can learn buy reading or playing with some helpful visualizations: http://try.github.io/

## Git for macOS
Git is a free and open source distributed version control system that allows developers working simultaneously on a single project to manage various updates to their code.

Git is not a strict dependency (i.e., you can run the SHR-CLI tool without git), but for team projects, it is recommended for CIMPL model developers to use git, along with a hosted git solution like [GitHub](https://github.com/) or [GitLab](https://gitlab.com/explore).

With [homebrew](#homebrew) installed, git can be installed by entering the following in a terminal window:
```
brew install git
```

You should also set up git so that any changes you make to the codebase are properly identified.  **Replace the name and email placeholders with your name and email**:
```
git config --global user.name "Your Name Here"
git config --global user.email "your_email@work.org"
```

#### Git Proxy Setup
_Note: If your organization uses a proxy server, you will need to add some proxy settings to git.  If your organization **does not** use a proxy server, you can skip ahead to installing [Visual Studio Code](#visual-studio-code-2)._

* Replace `your.proxy.org:port` with your actual proxy server in the following code block:
```
git config --global http.proxy http://your.proxy.org:port
git config --global url."https://".insteadOf git://
```

* Open Terminal
* Copy and paste that code block (with your organization's proxy server) into the terminal

#### Learning Git
If you're totally new to Git, github has a great introduction where you can learn buy reading or playing with some helpful visualizations: http://try.github.io/