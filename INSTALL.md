# Installation

Adaptive Bitrate Broadcaster can be installed in Linux or macOS.

The following softwares needs to be installed for Adaptive Bitrate Broadcaster to run properly
- Xcode (only for macOS users)
- V4L Utils and Libasound (only for Linux users)
- Python 2.7.x
- Python modules - mod_wsgi, webapp2, webob, psutil
- libx264
- ffmpeg 4.0 or above (configured with a x264 video encoder)
- A webserver with WSGI support(Apache2 preferred)

The below installation procedure is tested on macOS High Sierra, Ubuntu 14.04 and Ubuntu 18.04. Some these steps might need a little modification if they don't work or apply for your OS version.

# Quick Install

If you are using Ubuntu OS, then the `install_ubuntu.sh` script automates all the install steps. Just running that script will install all the required software and configure them.

```bash
git clone https://github.com/jkarthic-akamai/ABR-Broadcaster
cd ABR-Broadcaster
./install_ubuntu.sh
```

If the above scripts runs without any errors then it means ABR-Broadcaster is installed successfully in your system.

The basic installation will install only your webcam as input capture device. If you want to use a Decklink capture card for professional video capture then additionally follow the section [Decklink Capture Install](#decklink-capture-install).

# Detailed Step-by-Step Install

If you are some OS other than Ubuntu or if you need understand and control of the install steps then please follow the detailed but longer install procedure.

## Xcode (macOS only)

Full version of Xcode is required to be installed (not just the command line tools).
Go to Apple app store and install the Xcode app. After installation is complete run the following command.

```bash
$ xcode-select -p 
```

If the above command returns ``/Applications/Xcode.app/Contents/Developer`` then it means the Xcode installation in successful.

## V4L Utils and Libasound (Linux only)

Install the V4L Utils and Libasound with the following command.

```bash
sudo apt install v4l-utils libasound-dev
```

## Python

### macOS
macOS comes preinstalled with Python 2.7.x. Python version can be verified with the following command.

```bash
$ python --version
```

If the python version is something other than 2.7.x, it means some other version of python is also installed. Either uninstall the incompatible python version or modify the system's PATH settings so that the default python executable is version 2.7.x

Install pip for python

```bash
sudo easy_install pip
```

On older versions of macOS, pip installation might fail with an error `Download error on https://pypi.python.org/simple/pip/: [SSL: TLSV1_ALERT_PROTOCOL_VERSION] tlsv1 alert protocol version (_ssl.c:590) -- Some packages may not be found!`
In such cases, follow the below steps to install pip.

```bash
curl https://bootstrap.pypa.io/get-pip.py | sudo python
```

### Linux

On Ubuntu, install python and pip with the following command(if it is not installed already).

```bash
sudo apt update
sudo apt install python
sudo apt install python-pip
```

If installation of python-pip throws an error `Unable to locate package python-pip` then update `/etc/apt/sources.list` file to include "universe" category.

```bash
sudo vi /etc/apt/sources.list
```
then add "universe" at the end of each line, like this:

```bash
deb http://archive.ubuntu.com/ubuntu bionic main universe
deb http://archive.ubuntu.com/ubuntu bionic-security main universe
deb http://archive.ubuntu.com/ubuntu bionic-updates main universe
```

## Python modules (macOS and Linux)

Run the following commands to install the required python module dependencies

```bash
sudo pip install webapp2 webob psutil
```

## nasm (macOS only)

Install brew and nasm with the following commands.

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install nasm
```

## yasm (Linux only)

Install yasm with the following command.

```
sudo apt install yasm
```

## libx264 (macOS and Linux)
Run the following commands to download, build and install the latest version of [x264](http://www.videolan.org/developers/x264.html).

```bash
git clone http://git.videolan.org/git/x264.git
cd x264
./configure --enable-shared
make -j4
sudo make install
```

Alternatively on Ubuntu platforms x264 can be installed easily with the following command.

```bash
sudo apt install libx264-dev
```

## ffmpeg (macOS and Linux)

Run the following commands to download, build and install the latest version of [ffmpeg](http://ffmpeg.org/download.html).

```bash
git clone https://git.ffmpeg.org/ffmpeg.git
cd ffmpeg
./configure --enable-libx264 --enable-gpl
make -j4
sudo make install
```

## Adaptive Bitrate Broadcaster (macOS and Linux)

Download the latest version of Adaptive Bitrate Broadcaster

```bash
$ git clone https://github.com/jkarthic-akamai/ABR-Broadcaster
```

The present working directory where the ABR-Broadcaster is downloaded will be mentioned as ``<working directory>`` in this guide subsequently. All instances of ``<working directory>`` in this guide should be replaced with the correct path before using it. 

Say for the below example, ``<working directory>`` should be replaced by ``/home/akamai``

```bash
$ pwd
/home/akamai
```

## Apache2 (Linux only)

macOS comes preinstalled with Apache2. On Linux apache2 can be installed with the following command

```bash
sudo apt install apache2 libapache2-mod-wsgi
```

Add apache2's user `www-data` to video and audio group so that it has permissions to access AV capture devices. Run the following commands to achieve the same

```bash
sudo adduser www-data video
sudo adduser www-data audio
```

## Enable Virtual Hosts (macOS only)

Uncomment(or add) the following two lines in `/etc/apache2/httpd.conf` to enable Virtual Hosts

```
LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
Include /private/etc/apache2/extra/httpd-vhosts.conf
```

## Install and Enable WSGI module (macOS only)

Install WSGI module with the following command

```bash
sudo pip install mod_wsgi
```

and then run the following command

```bash
$ mod_wsgi-express module-config
```

It will output something like 

```
LoadModule wsgi_module "/Library/Python/2.7/site-packages/mod_wsgi/server/mod_wsgi-py27.so"
WSGIPythonHome "/System/Library/Frameworks/Python.framework/Versions/2.7"
```

Add the above two lines to ``/etc/apache2/httpd.conf``

## Add Directory (macOS and Linux)

The directory in which ABR-Broadcaster was downloaded needs to be added to Apache's configuration with relevant permissions and settings. Add the following lines in `/etc/apache2/httpd.conf`(macOS) or `/etc/apache2/apache2.conf`(Ubuntu) to do the same.

```
<Directory "<working directory>/ABR-Broadcaster">
  AllowOverride All
  Options MultiViews FollowSymLinks
  Require all granted
</Directory>
```

## Add Virtual Hosts (macOS and Linux)

Let us add a Virtual Host with the host name ``broadcaster.local`` for ABR-Broadcaster. Add the following lines to `/etc/apache2/extra/httpd-vhosts.conf`(macOS) or `/etc/apache2/sites-enabled/000-default.conf`(Ubuntu).

```
<VirtualHost *:80>
    DocumentRoot "<working directory>/ABR-Broadcaster/html"
    WSGIScriptAlias /broadcaster <working directory>/ABR-Broadcaster/wsgi-scripts/wc_config_handler.py
    ServerName broadcaster.local
    ErrorLog "/var/log/apache2/broadcaster-error_log"
    CustomLog "/var/log/apache2/broadcaster-access_log" common
</VirtualHost>
```

## Configuration Sub-Directory (macOS and Linux)

We need to create a sub-directory for broadcaster's configuration information to be stored and edited by the WSGI web application. Run the following commands to achieve the same.

```
cd <working directory>/ABR-Broadcaster
mkdir wsgi-scripts/db
chmod 777 wsgi-scripts/db
```

## Restart Apache2 (macOS and Linux)

In macOS run the following command to restart Apache2 server

```bash
sudo apachectl restart
```

In Ubuntu run the following command to restart Apache2 server

```bash
sudo service apache2 reload
```

# Decklink Capture Install

To use a Decklink SDI-input capture card for professional video capture, follow the below steps.

- The Decklink drivers and the SDK are two separate packages. Download it from the BlackMagicDesign's offical [site](https://www.blackmagicdesign.com/in/support/family/capture-and-playback). Download the latest versions for `Desktop Video` and `Desktop Video SDK`

- Uncompress the `Desktop Video` package. For example

```bash
tar -xvzf Blackmagic_Desktop_Video_Linux_10.11.2.tar.gz
```
The above command could change a little if you had downloaded a `Desktop Video` version different than above.

- Install the desktop video package relevant to your OS. For example if you are running Ubuntu 64-bit, run the below command to install it.

```bash
sudo dpkg -i Blackmagic_Desktop_Video_Linux_10.11.2/deb/x86_64/desktopvideo_10.11.2a3_amd64.deb
```

- Uncompress the `Desktop Video SDK` package. For example,

```bash
unzip Blackmagic_DeckLink_SDK_10.11.2.zip
```

- Copy the SDK header files to the system's include path so that ffmpeg can use it.

```
sudo cp Blackmagic\ DeckLink\ SDK\ 10.11.2/Linux/include/* /usr/local/include
```

- Rebuild and install ffmpeg with decklink plugin enabled.

```
install/ffmpeg.sh "--enable-decklink --enable-nonfree"
```

- In the Web UI interface of ABR-Broadcaster click on **"Refresh Input"** button to detect the presence of Decklink input device.