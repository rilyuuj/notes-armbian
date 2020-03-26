# Armbian something
note sth with Armbian

## Confirguration
system configuration



### set apt source

run with sudo
```bash
cat << EOF >> /etc/apt/sources.list
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
EOF
```

---
### set docker source

run with sudo
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
### install Docker GUI
```
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer:linux-arm
```

---
* ### install trojan-client   
```
mkdir /etc/trojan
cd /etc/trojan/
get https://raw.githubusercontent.com/trojan-gfw/trojan/master/examples/client.json-example
mv client.json-example config.json
vi config.json
```
<details>
  <summary>expand for <q>config.json</q> content</summary>
	
```
      	{
		"run_type": "client",
		"local_addr": "127.0.0.1",
		"local_port": 1080,
		"remote_addr": "www.cclimber.tk",
		"remote_port": 443,
		"password": [
			"password1"
		],
		"log_level": 1,
		"ssl": {
			"verify": true,
			"verify_hostname": true,
			"cert": "",
			"cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:AES128-SHA:AES256-SHA:DES-CBC3-SHA",
			"cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
			"sni": "",
			"alpn": [
				"h2",
				"http/1.1"
			],
			"reuse_session": true,
			"session_ticket": false,
			"curves": ""
		},
		"tcp": {
			"no_delay": true,
			"keep_alive": true,
			"reuse_port": false,
			"fast_open": false,
			"fast_open_qlen": 20
		}
	}
```
</details>

```
docker pull teddysun/trojan
docker run -d --name trojan --restart always --net host -v /etc/trojan/:/etc/trojan teddysun/trojan
```


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
