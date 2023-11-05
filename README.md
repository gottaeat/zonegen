# zonegen
zonegen creates a local zone for the subnet of the LAN interface and populates
it with PTR and A records by parsing the DHCP lease information obtained from
`odhcpd`, while also accounting for the static leases stated in
`/etc/config/dhcp`.

## installation
1. establish a working internet connection, make sure that we are not resolving
with our own `named` yet. it is safe to set the following for the WAN interface:
```
option peerdns '0'
list dns '1.1.1.1'
```

2. set `odhcpd` as the main DHCP provider with `zonegen` to be triggered each
time a lease action happens:
```
uci set dhcp.odhcpd.maindhcp='1'
uci set dhcp.odhcpd.leasetrigger='/bin/zonegen'
uci commit
```

3. nuke `dnsmasq`, remove the v6 client and allow `odhcpd` to be used on v4 too:
```
opkg remove --force-removal-of-dependent-packages --autoremove \
    dnsmasq \
    odhcp6c \
    odhcpd-ipv6only
```

4. install the bits:
```
opkg update
opkg_extra="luci luci-app-vnstat2 luci-app-wireguard tcpdump openssh-sftp-server"

opkg install \
    bind-server-filter-aaaa \
    curl \
    ${opkg_extra}
```

5. download and install `zonegen` (OR build yours as stated below):
```
cd /tmp

curl -L -O https://github.com/gottaeat/zonegen/releases/download/2-23.05.0/\
zonegen_c9f9a69d-2_mipsel_24kc.ipk

opkg install ./zonegen_c9f9a69d-2_mipsel_24kc.ipk
```

(if necessary, edit the domain name for the local zone in `/etc/zonegen.conf`)

6. set up `named`:
(the acl lists in `named.conf` might require updating)
```
cd /etc/bind

rm -rfv /etc/bind/*

baseurl="https://raw.githubusercontent.com/gottaeat/zonegen/main/files/"
curl -L -O "${baseurl}"/named.conf

mkdir zone/
cd    zone/

filelist="0 0.0.127 255 empty localhost root-nov6"
for i in ${filelist}; do curl -L -O "${baseurl}"/zone/"${i}"; done

mkdir chaos/
cd    chaos/

chaosfiles="bind server"
for i in ${chaosfiles}; do curl -L -O "${baseurl}"/zone/chaos/"${i}"; done
```

7. restart `bind` and set zones:
```
/etc/init.d/named restart
/bin/zonegen
```

## build instructions
### 1. local
to locally build by running the openwrt SDK on your machine, run the following.
(you might want to update the toolchain specified within)
```
./build-openwrt.sh
```

### 2. docker
if the compilation is noticably slower than the local build with the shell
script up above, that mostly likely is due to the `fakeroot` process within the
container choking up, you might want to add `--ulimit "nofile=1024:524288"` to
your `docker` run args, or change the line `LimitNOFILE=infinity` to
`LimitNOFILE=1024:524288` for `docker` and `containerd` systemd service files.
to learn more, refer to: https://github.com/moby/moby/issues/38814

(please note that the Dockerfile assumes the `ramips-mt7621` for the target)
```
docker build -t mss-owrt-ramips-mt7621 .

docker run --rm \
    -e MAKEFLAGS="-j$(nproc)"                     \
    -v "${PWD}"/openwrt-feed/:/build/openwrt-feed \
    -v "${PWD}"/bin:/build/bin                    \
    mss-owrt-ramips-mt7621                        \
    bash -c \
        "echo src-link local /build/openwrt-feed >> feeds.conf.default && \
         ./scripts/feeds update packages base local                    && \
         ./scripts/feeds install zonegen                               && \
         make defconfig                                                && \
         make package/zonegen/compile"
```
