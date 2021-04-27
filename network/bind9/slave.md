# Configurando o servidor DNS Slave

##  Configurar o DNS master na interface de rede

   * O primeiro passo é usar o DNS Master para fazer o ns2 acessar a Internet. Para isso configure a interface de rede com o netplan

```base
$ sudo nano /etc/netplan/50-cloud-init.yaml 
```
```
network:
    ethernets:
        enp0s3:
            addresses: [10.0.0.11/24]
            gateway4: 10.0.0.1    
            dhcp4: false
            nameservers: 
                addresses:
                - 10.0.0.10
                - 10.0.0.11
                search: [labredes.ifalarapiraca.local]
    version: 2
```

   * Aplique as configurações
```bash
$ sudo netplan apply
``` 
   * veja se funcionou
```bash
$ ifconfig
```


### Configurar e instalar servidor DNS secundário (slave)
```bash
$ sudo apt-get install bind9 dnsutils bind9-doc -y
```


   * Verifique o status do serviço:
```bash
$ sudo systemctl status bind9
```
   * Se não estiver rodando:
```bash
$ sudo systemctl enable bind9
```

### configuração de zonas

```bash
$ sudo nano db.labredes.ifalarapiraca.local
```
```
zone "labredes.ifalarapiraca.local" {
  type slave;
  file "/etc/bind/zones/db.labredes.ifalarapiraca.local";
  masters { 10.0.0.10; };
};

zone "0.0.10.in-addr.arpa" IN {
  type slave;
  file "/etc/bind/zones/db.10.0.0.rev";
  masters { 10.0.0.10; };
};
```

### Checagem de sintaxe

```bash
$ sudo named-checkconf
```

### Testes com dig
   * utilize o o comando **dig @server hostname** para resolver um hostname em um server específico.
   * Veja a resposta em **ANSWER SECTION**.

```bash
$ dig @10.0.0.11 ns1.labredes.ifalarapiraca.local
```

```
; <<>> DiG 9.11.3-1ubuntu1.9-Ubuntu <<>> @10.0.0.11 ns1.labredes.ifalarapiraca.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15875
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 2ada115f11a1f1503fc446975da5644227f17494e73108e6 (good)
;; QUESTION SECTION:
;ns1.labredes.ifalarapiraca.local. IN A

;; ANSWER SECTION:
ns1.labredes.ifalarapiraca.local. 604800 IN A 10.0.0.10

;; AUTHORITY SECTION:
labredes.ifalarapiraca.local. 604800 IN NS  ns2.labredes.ifalarapiraca.local.
labredes.ifalarapiraca.local. 604800 IN NS  ns1.labredes.ifalarapiraca.local.

;; ADDITIONAL SECTION:
ns2.labredes.ifalarapiraca.local. 604800 IN A 10.0.0.11

;; Query time: 0 msec
;; SERVER: 10.0.0.11#53(10.0.0.11)
;; WHEN: Tue Oct 15 06:16:34 UTC 2019
;; MSG SIZE  rcvd: 153
```
