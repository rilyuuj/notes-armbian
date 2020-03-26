# Armbian something
note sth with Armbian

## Confirguration
system configuration



## set apt source

#### run with sudo
```bash
cat << EOF >> /etc/apt/sources.list
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
EOF
```

---
## set docker source

#### run with sudo
```
echo "deb [arch=armhf] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian buster stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian/gpg | sudo apt-key add -
```
setup
```
export DOWNLOAD_URL=https://mirrors.tuna.tsinghua.edu.cn/docker-ce
sh -c "$(curl -fsSL https://get.docker.com)"
```
docker mirror
```
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn/"
  ]
}
EOF
sudo systemctl restart docker
```

---
## install Docker GUI
```bash
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer:linux-arm
```

---
## install trojan-client
<details>
  <summary>test</summary>    

```
    mkdir /etc/trojan
    cd /etc/trojan/
    get https://raw.githubusercontent.com/trojan-gfw/trojan/master/examples/client.json-example
    mv client.json-example config.json
    vi config.json
    docker pull teddysun/trojan
    docker run -d --name trojan --restart always --net host -v /etc/trojan/:/etc/trojan teddysun/trojan
```
</details>

---
## import openwrt docker image
```
modprobe pppoe
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macvlan_lan
docker import openwrt-armv7-bpi-m2u-rootfs.tar.gz lean-openwrt
docker run --restart always -d --network macvlan_lan --privileged --name openwrt lean-openwrt /sbin/init
docker exec -it 'container id' sh
vi /etc/config/network
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.254'
        option netmask '255.255.255.0'
        option gateway '192.168.1.1'
/etc/init.d/network restart
```
## to be continued...
