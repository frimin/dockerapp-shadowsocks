# dockerapp-shadowsocks

Deploy shadowsocks with docker-compose.

Has three config file in this deploy case:

## Config Description

### ssserver
  
 Launch shadowsocks server and kcptun & udp2raw tunnel server.

> shadowsocks server (tcp:4000) -> kcptun server (udp:6500) -> udp2raw server (udp:4000) -> docker public port

### ssproxy

 Launch kcptun & udp2raw tunnel client.

 > (ssserver:port) -> udp2raw client (udp:4000) -> kcptun client (tcp:8000) -> docker public port 

### sslocal

`Optional`

Launch shadowsocks client for http proxy on CLI. 

> (ssproxy:port) -> sslocal (tcp:8118) -> docker public port

You can use desktop application to connect `ssproxy:port` address.

## Config Environment Parameters

### ssserver

Parameter | Function 
:-: |  :-:
`SSSERVER_KCP_KEY` | Key for kcp tunnel server
`SSSERVER_UDP2RAW_KEY` | Key for udp2raw tunnel server
`SSSERVER_PASSWORD` | Password for shadowsocks server
`SSSERVER_LISTEN` | Udp2raw server listen port (this port need to public)

### ssproxy

Parameter | Function 
:-: |  :-:
`SSPROXY_KCP_KEY` | Key for kcp tunnel client
`SSRPOXY_UDP2RAW_KEY` | Key for udp2raw tunnel client
`SSPROXY_LISTEN` | Kcp client listen port  (this port need to public)
`SSPROXY_SSSERVER_ADDR` | Udp2raw client connect **ssserver** address.

### sslocal

Parameter | Function 
:-: |  :-:
`SSLOCAL_SSPROXY_HOST` | Hostname of **ssproxy** address.
`SSLOCAL_SSPROXY_PORT` | Port of **ssproxy** address.
`SSLOCAL_PASSWORD` | Remote shadowsocks server password for client.
`SSLOCAL_BIND` | Addrees of sslocal bind for http proxy. default value **127.0.0.1:4002**

## Usage 

### Start ssserver

Can start **ssserver** containers directly in `host1`:

    # in host1
    cd ssserver
    sudo docker-compose up -d


### Start ssproxy

In `host2`. Edit env file **ssproxy/.env**. Change env option **SSPROXY_SSSERVER_ADDR** as `host1` public address.

Now you can start **ssproxy** containers in `host2`:

    # in host2
    cd ssproxy
    sudo docker-compose up -d

### Start sslocal

If want start **sslocal** in `host2`. Change env option **SSLOCAL_SSPROXY_HOST** as without loopback address value, like **192.168.1.2** or **10.0.0.1**. recommand use this host real address.

The default **127.0.0.1** is not working, because **127.0.0.1** will connect address inside container.

If start **sslocal** in other device, change env option **SSLOCAL_SSPROXY_HOST** as `host2` public address.

Now you can start **sslocal** containers in `host2`:

    # in host2
    cd sslocal
    sudo docker-compose up -d

After **sslocal** started, setup http proxy to ~/.bashrc:
    
    echo "export http_proxy=http://127.0.0.1:4002" >> ~/.bashrc
    echo "export https_proxy=http://127.0.0.1:4002" >> ~/.bashrc
    source ~/.bashrc

Testing:

    curl -v https://www.google.com/

### Desktop client

In desktop application client. Use proxy server address: **host2:4001**, default password: **sspassword**, encrypt method: **aes-256-cfb**.

Now TRE FREE.

## LICENSE

MIT License