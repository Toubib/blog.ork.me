https://www.scaleway.com
C1 serveur (armv7l)

We have to compile nearly everything:

* phantomjs
casperjs need version < 2.0 at this time. We will pick 1.9.8.

apt-get install g++ flex bison-doc bison gperf ruby ruby-dev perl libsqlite3-doc sqlite3-dev libfontconfig1-dev icu-doc libicu-dev libfreetype6 libssl-dev libpng12-dev libjpeg8-dev ttf-mscorefonts-installer fontconfig build-essential chrpath git-core openssl
git clone git://github.com/ariya/phantomjs.git
cd phantomjs
git checkout 1.9.8
./build.sh

http://phantomjs.org/build.html
https://www.bitpi.co/2015/02/11/compiling-phantomjs-on-raspberry-pi/

Didn't work:

...
~/phantomjs/src/qt
Building main PhantomJS application. Please wait...

cd src/ && make -f Makefile.phantomjs
make[1]: Entering directory '/root/phantomjs/src'
/root/phantomjs/src/qt/qtbase/bin/qmake -o Makefile.phantomjs phantomjs.pro
Could not find qmake configuration file linux-g++.
Error processing project file: phantomjs.pro
Makefile.phantomjs:361: recipe for target 'Makefile.phantomjs' failed
make[1]: *** [Makefile.phantomjs] Error 3
make[1]: Leaving directory '/root/phantomjs/src'
Makefile:39: recipe for target 'sub-src-phantomjs-pro-make_default-ordered' failed
make: *** [sub-src-phantomjs-pro-make_default-ordered] Error 2

Plan B is to dl non-official binaries here: https://github.com/piksel/phantomjs-raspberrypi
git clone git@github.com:piksel/phantomjs-raspberrypi.git
cp ~/phantomjs-raspberrypi/bin/phantomjs /usr/local/bin/
chmod +x /usr/local/bin/phantomjs

* casperjs
https://github.com/n1k0/casperjs/zipball/1.1-beta3

* nodejs

* go

* influxdb
