Building windows openconnect-gui distro that can make use of .stokenrc

# Usage

* Make sure you have a valid `.stokenrc` file with pin set up
* Download installer && run
* Configure openconnect profile and use STOKEN for token mode and '--' as token string

# Building

1. Install msys2
    * http://www.msys2.org/
    * `pacman -Syuu` *
    * `pacman -S git p7zip base-devel autoreconf gcc mingw-w64-x86_64-cmake`


2. Install deps:

```
pacman --needed -S \
  mingw-w64-x86_64-gnutls \
  mingw-w64-x86_64-libidn2 \
  mingw-w64-x86_64-libunistring \
  mingw-w64-x86_64-nettle \
  mingw-w64-x86_64-gmp \
  mingw-w64-x86_64-p11-kit \
  mingw-w64-x86_64-zlib \
  mingw-w64-x86_64-libxml2 \
  mingw-w64-x86_64-lz4 \
  mingw-w64-x86_64-libproxy \
  mingw-w64-x86_64-toolchain \
  mingw-w64-x86_64-gtk3 \
  mingw-w64-x86_64-nsis
   ```

3. Prepare env vars

```
export ST_TAG=v0.92
export OC_TAG=v7.08
export OC_GUI_TAG=v1.5.3
# compile gtk apps without console
export GTK_CFLAGS="-mwindows $(pkg-config --cflags 'gtk+-3.0' 2>/dev/null)"
```

4. Build stoken

```
[ -d stoken ] ||  git clone https://github.com/cernekee/stoken
cd stoken
git checkout ${ST_TAG}
./autogen.sh
[ -d build ] || mkdir build
cd build
git clean -fdx && \
  ../configure --disable-dependency-tracking --without-tomcrypt --with-gtk && \
  make -j4 && \
  make install
cd ../..
```

5. Build openconnect

```
[ -d openconnect ] || git clone https://gitlab.com/openconnect/openconnect.git
cd openconnect
git checkout ${OC_TAG}
curl https://raw.githubusercontent.com/atanasenko/openconnect_rsa/master/openconnect.patch | git apply -v --index
./autogen.sh
[ -d build ] || mkdir build
cd build
git clean -fdx && \
  ../configure --disable-dependency-tracking --with-gnutls --without-openssl --without-libpskc --with-vpnc-script=vpnc-script-win.js && \
  make -j4

rm -rf pkg
mkdir pkg && cd pkg

mkdir nsis && cd nsis
cp ${MINGW_PREFIX}/bin/libffi-6.dll .
cp ${MINGW_PREFIX}/bin/libgcc_*-1.dll .
cp ${MINGW_PREFIX}/bin/libgmp-10.dll .
cp ${MINGW_PREFIX}/bin/libgnutls-30.dll .
cp ${MINGW_PREFIX}/bin/libhogweed-4.dll .
cp ${MINGW_PREFIX}/bin/libintl-8.dll .
cp ${MINGW_PREFIX}/bin/libnettle-6.dll .
cp ${MINGW_PREFIX}/bin/libp11-kit-0.dll .
cp ${MINGW_PREFIX}/bin/libtasn1-6.dll .
cp ${MINGW_PREFIX}/bin/libwinpthread-1.dll .
cp ${MINGW_PREFIX}/bin/libxml2-2.dll .
cp ${MINGW_PREFIX}/bin/zlib1.dll .
cp ${MINGW_PREFIX}/bin/libproxy-1.dll .
cp ${MINGW_PREFIX}/bin/liblz4.dll .
cp ${MINGW_PREFIX}/bin/libiconv-2.dll .
cp ${MINGW_PREFIX}/bin/libunistring-2.dll .
cp ${MINGW_PREFIX}/bin/libidn2-0.dll .
cp ${MINGW_PREFIX}/bin/libstdc++-6.dll .
cp ${MINGW_PREFIX}/bin/liblzma-5.dll .
cp ../../.libs/libopenconnect-5.dll .
cp ../../.libs/openconnect.exe .

cp ${MINGW_PREFIX}/bin/libatk-1.0-0.dll .
cp ${MINGW_PREFIX}/bin/libcairo-2.dll .
cp ${MINGW_PREFIX}/bin/libcairo-gobject-2.dll .
cp ${MINGW_PREFIX}/bin/libdatrie-1.dll .
cp ${MINGW_PREFIX}/bin/libepoxy-0.dll .
cp ${MINGW_PREFIX}/bin/libexpat-1.dll .
cp ${MINGW_PREFIX}/bin/libfontconfig-1.dll .
cp ${MINGW_PREFIX}/bin/libfribidi-0.dll .
cp ${MINGW_PREFIX}/bin/libgdk-3-0.dll .
cp ${MINGW_PREFIX}/bin/libgdk_pixbuf-2.0-0.dll .
cp ${MINGW_PREFIX}/bin/libgio-2.0-0.dll .
cp ${MINGW_PREFIX}/bin/libglib-2.0-0.dll .
cp ${MINGW_PREFIX}/bin/libgmodule-2.0-0.dll .
cp ${MINGW_PREFIX}/bin/libgobject-2.0-0.dll .
cp ${MINGW_PREFIX}/bin/libgtk-3-0.dll .
cp ${MINGW_PREFIX}/bin/libpango-1.0-0.dll .
cp ${MINGW_PREFIX}/bin/libpangocairo-1.0-0.dll .
cp ${MINGW_PREFIX}/bin/libpangoft2-1.0-0.dll .
cp ${MINGW_PREFIX}/bin/libpangowin32-1.0-0.dll .
cp ${MINGW_PREFIX}/bin/libpixman-1-0.dll .
cp ../../../../stoken/build/.libs/libstoken-1.dll .
cp ../../../../stoken/build/.libs/stoken.exe .
cp ../../../../stoken/build/.libs/stoken-gui.exe .
cp ../../../../stoken/gui/*.ui .
cp ../../../../stoken/gui/*.png .


curl -v -o vpnc-script-win.js http://git.infradead.org/users/dwmw2/vpnc-scripts.git/blob_plain/HEAD:/vpnc-script-win.js
7za a -tzip -mx=9 ../../openconnect-${OC_TAG}_mingw64.zip *
cd ..

mkdir devel && cd devel

mkdir lib && cd lib
cp ${MINGW_PREFIX}/lib/libgmp.dll.a .
cp ${MINGW_PREFIX}/lib/libgnutls.dll.a .
cp ${MINGW_PREFIX}/lib/libhogweed.dll.a .
cp ${MINGW_PREFIX}/lib/libnettle.dll.a .
cp ${MINGW_PREFIX}/lib/libp11-kit.dll.a .
cp ${MINGW_PREFIX}/lib/libxml2.dll.a .
cp ${MINGW_PREFIX}/lib/libz.dll.a .
cp ${MINGW_PREFIX}/lib/libstoken.dll.a .
cp ${MINGW_PREFIX}/lib/libproxy.dll.a .
cp ${MINGW_PREFIX}/lib/liblz4.dll.a .
cp ${MINGW_PREFIX}/lib/libiconv.dll.a .
cp ${MINGW_PREFIX}/lib/libunistring.dll.a .
cp ${MINGW_PREFIX}/lib/libidn2.dll.a .
cp ${MINGW_PREFIX}/lib/liblzma.dll.a .
cp ../../../.libs/libopenconnect.dll.a .

mkdir pkgconfig && cd pkgconfig
cp ${MINGW_PREFIX}/lib/pkgconfig/gnutls.pc .
cp ${MINGW_PREFIX}/lib/pkgconfig/hogweed.pc .
cp ${MINGW_PREFIX}/lib/pkgconfig/libxml-2.0.pc .
cp ${MINGW_PREFIX}/lib/pkgconfig/nettle.pc .
cp ${MINGW_PREFIX}/lib/pkgconfig/zlib.pc .
cp ${MINGW_PREFIX}/lib/pkgconfig/stoken.pc .
cp ../../../../openconnect.pc .
cd ../..

mkdir include && cd include
cp -R ${MINGW_PREFIX}/include/gnutls/ .
cp -R ${MINGW_PREFIX}/include/libxml2/ .
cp -R ${MINGW_PREFIX}/include/nettle/ .
cp -R ${MINGW_PREFIX}/include/p11-kit-1/p11-kit/ .
cp ${MINGW_PREFIX}/include/gmp.h .
cp ${MINGW_PREFIX}/include/zconf.h .
cp ${MINGW_PREFIX}/include/zlib.h .
cp ${MINGW_PREFIX}/include/stoken.h .
cp ../../../../openconnect.h .
cd ..

7za a -tzip -mx=9 ../../openconnect-devel-${OC_TAG}_mingw64.zip *
cd ../../../..
```

6. Build openconnect-gui

```
[ -d openconnect-gui ] || git clone https://github.com/openconnect/openconnect-gui.git
cd openconnect-gui
git checkout ${OC_GUI_TAG}
curl https://raw.githubusercontent.com/atanasenko/openconnect_rsa/master/openconnect-gui.patch | git apply -v --index

cp ../openconnect/build/openconnect-${OC_TAG}_mingw64.zip external/
cp ../openconnect/build/openconnect-devel-${OC_TAG}_mingw64.zip external/

[ -d build ] || mkdir build
cd build
git clean -fdx && \
  cmake -G "MSYS Makefiles" -DCMAKE_BUILD_TYPE=Release .. && \
  make && \
  make package

cd ../..
```

References:
* https://github.com/openconnect/openconnect-gui/tree/master/contrib
