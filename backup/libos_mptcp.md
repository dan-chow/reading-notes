https://github.com/libos-nuse/net-next-nuse/wiki/Linux-mptcp-with-libos

		git clone https://github.com/libos-nuse/net-next-nuse.git
		git checkout mptcp_trunk_libos

generate default configuration (written to `.config`).

		make defconfig ARCH=lib


		make menuconfig ARCH=lib

Network support ---> Network options ---> MPTCP: advanced scheduler control ---> MPTCP Round-Robin
save and exit

		make library ARCH=lib

https://github.com/libos-nuse/net-next-nuse/blob/nuse/Documentation/virtual/libos-howto.txt

		make testbin -C tools/testing/libos

the url for ccnx is invalid

tools/testomg/libos/buildtop/bakefile.xml

xxx to xxx
xxx to xxx

tools/testomg/libos/buildtop/source
remove ccnx related files

if failed, execute bake.py download -vvv manually to see what happened

		make test ARCH=lib

bindings/python/wscript
REQUIRED_PYBINDGEN_VERSION = 'xxxxxx'


https://github.com/libos-nuse/net-next-nuse/wiki/Quick-Start


https://www.nsnam.org/wiki/Installation
https://askubuntu.com/questions/265471/autoreconf-not-found-error-during-making-qemu-1-4-0/490839

https://github.com/libos-nuse/net-next-nuse/blob/nuse/Documentation/virtual/libos-howto.txt
https://stackoverflow.com/questions/36252495/unable-to-locate-package-python-gnomedesktop-installing-pyviz-in-ns3

https://github.com/gjcarneiro/pybindgen

https://plus.google.com/+HajimeTazaki/posts/aYuboVFcQeF

https://plus.google.com/+HajimeTazaki/posts/aYuboVFcQeF
