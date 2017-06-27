NetworkManager-l2tp

sudo apt-add-repository ppa:seriy-pr/network-manager-l2tp
sudo apt-get update
sudo apt-get install network-manager-l2tp-gnome

sudo service xl2tpd stop                                   
sudo update-rc.d xl2tpd disable


[1] http://mr21.cc/codes/l2tp-plugins-for-ubuntu-network-manager.html
[2] https://github.com/nm-l2tp/network-manager-l2tp
