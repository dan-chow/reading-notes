Download 

bindings/python/wscript
REQUIRED_PYBINDGEN_VERSION = 'xxxxxx'

https://askubuntu.com/questions/265471/autoreconf-not-found-error-during-making-qemu-1-4-0/490839

https://stackoverflow.com/questions/36252495/unable-to-locate-package-python-gnomedesktop-installing-pyviz-in-ns3

https://github.com/gjcarneiro/pybindgen

https://plus.google.com/+HajimeTazaki/posts/aYuboVFcQeF

https://wiki.fd.io/view/Libicnet

- ERROR 1: #error This file requires compiler and library support for the ISO C++ 2011 standard
	- Refer to [this link][cxx11], you have to configure the ns3 project using the following command:  
`CXXFLAGS="-std=c++0x" ./waf -d debug --enable-examples --enable-tests configure`

- ERROR 2: openssl/sha.h: No such file or directory
	- As explained in [this link][openssl], opssl/sha.h is included in libssl-dev, so you have to install it:  
`sudo apt-get install libssl-dev`

- ERROR 3: /usr/bin/ld: Cannot find -lgcrypt
	- Refer to [this link][libgcrypt], you have to install libgcrypt:  
`sudo apt-get install libgcrypt-dev`

[cxx11]: http://stackoverflow.com/questions/36379632/error-in-program-saying-this-file-requires-compiler-and-library-support-for-the
[openssl]: https://unix.stackexchange.com/questions/87458/make-fatal-error-openssl-sha-h-no-such-file-or-directory
[libgcrypt]: http://stackoverflow.com/questions/7150323/configure-unable-to-find-libgcrypt

http://m.blog.csdn.net/article/details?id=50879075


https://github.com/GENI-NSF/geni-tutorials/raw/master/OldTutorials/ContentCentricNetworking/ccnx-0.6.2.tar.gz

https://github.com/ProjectCCNx/ccnx/releases



Hi,

What you are trying to do is completely wrong both in terms of C++/programing, as well as meaningfulness.

You created a new YansWifiPhy object and called a method named GetRxGain from that.
Problems with this approach: uninitialized object, you ask for signal strength however you call a method to get the antenna gain...

The signal strength is calculated on per packet basis. You can use the NS3 tracing system (http://www.nsnam.org/docs/release/3.21/manual/html/tracing.html) and hook to one of the trace sources that provide the singal strength (e.g. MonitorSnifferRx).
Otherwise, you can modify the code slightly and use the SNR tag as it is explained in other threads (just search for it in the mailing list).

Regards,
K.