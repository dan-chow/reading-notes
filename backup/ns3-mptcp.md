Download 

https://wiki.fd.io/view/Libicnet

https://www.nsnam.org/wiki/Installation#Ubuntu.2FDebian

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

http://download.eclipse.org/technology/m2e/releases/
http://www.eclipse.org/m2e/m2e-downloads.html

http://m.blog.csdn.net/article/details?id=50879075


https://github.com/GENI-NSF/geni-tutorials/raw/master/OldTutorials/ContentCentricNetworking/ccnx-0.6.2.tar.gz

https://github.com/ProjectCCNx/ccnx/releases

https://github.com/libos-nuse/net-next-nuse/wiki/Linux-mptcp-with-libos

Hi,

What you are trying to do is completely wrong both in terms of C++/programing, as well as meaningfulness.

You created a new YansWifiPhy object and called a method named GetRxGain from that.
Problems with this approach: uninitialized object, you ask for signal strength however you call a method to get the antenna gain...

The signal strength is calculated on per packet basis. You can use the NS3 tracing system (http://www.nsnam.org/docs/release/3.21/manual/html/tracing.html) and hook to one of the trace sources that provide the singal strength (e.g. MonitorSnifferRx).
Otherwise, you can modify the code slightly and use the SNR tag as it is explained in other threads (just search for it in the mailing list).

Regards,
K.