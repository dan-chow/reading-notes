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
