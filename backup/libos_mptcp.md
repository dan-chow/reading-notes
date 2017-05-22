mptcp@mptcp-VirtualBox:~/net-next-nuse$ git checkout --track origin/mptcp_trunk_libos
Branch mptcp_trunk_libos set up to track remote branch mptcp_trunk_libos from origin.
Switched to a new branch 'mptcp_trunk_libos'
mptcp@mptcp-VirtualBox:~/net-next-nuse$ git branch
  master
* mptcp_trunk_libos

mptcp@mptcp-VirtualBox:~/net-next-nuse$ make defconfig ARCH=lib
  HOSTCC  scripts/basic/fixdep
make[1]: *** No rule to make target `.config'.  Stop.
  HOSTCC  scripts/kconfig/conf.o
  SHIPPED scripts/kconfig/zconf.tab.c
  SHIPPED scripts/kconfig/zconf.lex.c
  SHIPPED scripts/kconfig/zconf.hash.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf  --defconfig arch/lib/Kconfig
#
# configuration written to .config
#

mptcp@mptcp-VirtualBox:~/net-next-nuse$ make menuconfig ARCH=lib
make[1]: Nothing to be done for `.config'.
scripts/kconfig/mconf  arch/lib/Kconfig
configuration written to .config

*** End of the configuration.
*** Execute 'make' to start the build or try 'make help'.

mptcp@mptcp-VirtualBox:~/net-next-nuse$ make library ARCH=lib

https://github.com/libos-nuse/net-next-nuse/wiki/Quick-Start
https://github.com/libos-nuse/net-next-nuse/wiki/Linux-mptcp-with-libos

https://www.nsnam.org/wiki/Installation
https://askubuntu.com/questions/265471/autoreconf-not-found-error-during-making-qemu-1-4-0/490839

https://github.com/libos-nuse/net-next-nuse/blob/nuse/Documentation/virtual/libos-howto.txt
https://stackoverflow.com/questions/36252495/unable-to-locate-package-python-gnomedesktop-installing-pyviz-in-ns3
