---
layout: page
hero_title: Create Your Own VPN Server with OpenVPN
description: In this post I will walk you through how to create a free VPN server using OpenVPN community edition
date: 2021-11-03
permalink: /:title/
redirect_from: /2021/11/03/create-your-own-vpn-server-with-openvpn.html
toc: true
hero_image: /img/OpenVPN-Protocol-Logo-1536x864.webp
#hero_height: is-large
image: /img/OpenVPN-Protocol-Logo-1536x864.webp
tags: vpn openvpn TLS_VPN secure_connection free_vpn
---

If you are looking to build your own VPN server for personal use or as a business, this setup below will work just fine. All you need is a machine with good bandwidth and speed connection, the machine can be a Dedicated, Virtual, or cloud server. 

In the below setup I used CentOS 8 as an operating system, the same steps will work on all linux distros except for minor changes in Ubuntu/Debian such as the path of the config files, easyrsa's vars, and so on.

# What is OpenVPN?

OpenVPN is a full-featured SSL VPN that implements OSI layer 2 or 3 secure network extension using the industry-standard SSL/TLS protocol, supports flexible client authentication methods based on certificates, smart cards, and/or username/password credentials, and allows user or group-specific access control policies using firewall rules applied to the VPN virtual interface. And the best part is, it free and open source!


# Setup The VPN Server

For this demo, I am going to prepare two virtual machines with RHEL 8 as the operating system, we will use epel repoistory which will install the OpenVPN package community edition. I should mention here that the OpenVPN package is the same for the server and the client, the only difference will be with the configuration file that will be passed to the service, we are going to discuss that later on. 

The machines will look like the follows:

| Machine No. | Type | Hostname | IP Address |
|-----------|--------|--------|----------|
| 1        | VPN Server | openvpn.lab | 192.168.56.101 |
| 2        | VPN Client | client1.lab | 192.168.56.110 |


## Install OpenVPN & Easy RSA

In addition to OpenVPN, we will need to install Easy RSA which is a very good one tool for creating a simple pki where we can generate CA Root, sign certificates, and create key-pairs for servers and clients.

> Note that you need sudo/root priviliges in order to performs all instructions that follows, if you don't have root but sudo user, add sudo before all the commands that I mention in the article.

First, we are going to install the EPEL packages which has OpenVPN and Easy RSA with latest versions:

```bash
yum install epel-release -y
yum update
yum install openvpn easy-rsa -y
```

For Ubuntu/Debian:

apt update
apt install openvpn easy-rsa -y


Wait for the installation to complete without errors and then confirm you have got `/usr/share/easy-rsa` and `/etc/openvpn` at the the VPN server.

For Easy RSA, It's better idea to copy its folder `easy-rsa` to a different directory so that future easy-rsa package upgrades won't overwrite your modifications:

```bash
cp /usr/share/easy-rsa/3.0.8 /etc/easy-rsa
```
Now let's create our PKI as the first important step before moving forward with VPN configuration.


# Setup The PKI

In this stage we are going to initialize the PKI, build CA root, and generate server and client key-pairs so that will be used for the communication between the server and the client. 

## Initialize PKI

Before that, we need to edit vars file with necessary values that will be used by the init-pki and ca-build scripts.

We can find this file in the docs folder of easy-rsa, in RHEL 8 it is in `/usr/share/doc/easy-rsa/`, so we copy it to openvpn directory:

```bash
cp /usr/share/doc/easy-rsa/vars.example /etc/easy-rsa/vars 
```

Next edit vars file and uncomment/edit the required variables, your file should have the following variables uncommented:

```bash
set_var EASYRSA	"${0%/*}"
set_var EASYRSA_PKI		"$PWD/pki"
set_var EASYRSA_DN	"org"
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "California"
set_var EASYRSA_REQ_CITY       "San Francisco"
set_var EASYRSA_REQ_ORG		"Tariqhawis.com"
set_var EASYRSA_REQ_EMAIL	"tariqhawis @ gmail.com"
set_var EASYRSA_REQ_OU		"TariqHawis CA"
set_var EASYRSA_KEY_SIZE	4096
set_var EASYRSA_CA_EXPIRE	3650
set_var EASYRSA_CERT_EXPIRE	730
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

> Note that I used ec instead of RSA. Elliptic Curve Cryptography (ECC) is the modern algorithm to generate keys and secure signatures for your clients and OpenVPN server.


Now initialize the PKI:

```bash
cd /etc/easyrsa
./easyrsa init-pki
```

You will see a new folder called pki.


## Build the CA Root

In this step we run clean-all parameter to clean any previous CA certificates if any exist, then build the CA root with build-ca paramter. During the build process, you will be asked for the passphrase of your CA private key, please remmember it as you will need it everytime when you generate a key-pair or sign a CSR file.

Now run this:

```bash
./easyrsa clean-all
./easyrsa build-ca
```

The output should be similar to this:

```bash
Note: using Easy-RSA configuration from: /etc/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
read EC key
writing EC key
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:    
State or Province Name (full name) [California]:        
Locality Name (eg, city) [San Francisco]:
Organization Name (eg, company) [Tariqhawis.com]:
Organizational Unit Name (eg, section) [TariqHawis CA]:
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:openvpn.lab
Email Address [tariqhawis @ gmail.com]:

CA creation is complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/easyrsa/pki/ca.crt
```

If you received the same results then everything is alright so far.


## Generate The Certificate & Private Key For The Server

Next, we will generate a certificate and private key for the server with the below command:

```bash
./easyrsa build-server-full openvpn.lab nopass
```

Change `openvpn.lab` to the hostname of your VPN server.

> Note that I used nopass so that I will not be asked for the pass phrase each time the VPN server will started.

The results should be similar to this:

```bash
./easyrsa build-server-full openvpn.lab nopass

Note: using Easy-RSA configuration from: /etc/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating an EC private key
writing new private key to '/etc/easyrsa/pki/easy-rsa-7331.hGYu1P/tmp.XFBIJu'
-----
Using configuration from /etc/easyrsa/pki/easy-rsa-7331.hGYu1P/tmp.WgTHT0
Enter pass phrase for /etc/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'San Francisco'
organizationName      :ASN.1 12:'Tariqhawis.com'
organizationalUnitName:ASN.1 12:'TariqHawis CA'
commonName            :ASN.1 12:'openvpn.lab'
emailAddress          :IA5STRING:'tariqhawis @ gmail.com'
Certificate is to be certified until Oct 29 11:40:07 2023 GMT (730 days)

Write out database with 1 new entries
Data Base Updated
```

## Generate Shared Key for Extra protection

For extra security beyond that provided by SSL/TLS, we will create an "HMAC firewall". This will help to protect and tight the usage of your resources by blocking attacks like DoS and UDP port flooding.

Run the following commands:

```bash
openvpn --genkey --secret ta.key
cp ta.key /etc/easyrsa/pki/private/
```

## Examine our PKI folder

Let us check all our newly-generated keys and certificate in the pki subdirectory. The tree of pki should look like this:

```
pki
|-- ca.crt
...
|-- ecparams
|   |-- secp384r1.pem
...
|-- issued
|   |-- openvpn.lab.crt
|-- openssl-easyrsa.cnf
|-- private
|   |-- ca.key
|   |-- openvpn.lab.key
|   `--ta.key
...
```

Here is an explanation of the relevant files: 

| Filename    | Needed By                | Purpose                   | Secret |  
|-------------|--------------------------|---------------------------|--------|
| ca.crt      | server + all clients     | Root CA certificate       | NO     | 
| ca.key      | key signing machine only | Root CA key               | YES    | 
| secp384r1.pem    | server only         | ECC shared key            | YES     | 
| openvpn.lab.crt | server only          | Server Certificate        | NO     | 
| openvpn.lab.key | server only          | Server Key                | YES    | 
| ta.key      | server + all clients     | Shared Secret Key         | YES    | 


# Setup OpenVPN Service

After we have finished the PKI configuration, let's move on to OpenVPN configuration.

## Create VPN Server conf File

At this stage, we need first to create the server configuration that will determine all paramters needed by the VPN server, such as the network type to be used "TUN or TAP", listening port, and the network subnet. we can use the sample conf files from the OpenVPN docs folder since they are ideal as starting points for an OpenVPN server configuration. With this sample server configuration, the OpenVPN server will create a VPN using a virtual TUN network interface (for routing), will listen for client connections on UDP port 1194 (OpenVPN's official port number), and distribute virtual addresses to connecting clients from the 10.8.0.0/24 subnet. 

Let's copy the file to OpenVPN folder:

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/server/
```

Next, edit the copied server.conf and find the lines that start with: ca, cert, key, and add the path of the corresponding key that we have generated before, as follows:

```bash
ca /etc/easyrsa/pki/ca.crt

cert /etc/easyrsa/pki/issued/openvpn.lab.crt

key /etc/easyrsa/pki/private/openvpn.lab.key
```

Now comment out the line with `dh` as it's no more needed since we're using now ECC algorithm.

```bash
;dh dh2048.pem
dh none
```

Also, search and comment out the line of `tls-auth ta.key`, then add `tls-crypt` after it:

```bash 
;tls-auth ta.key 0 # This file is secret
tls-crypt ta.key
```

> tls-crypt is better that tls-auth because unlike the latter, it ensures that invalid certificates are reject before passing it, this also prevents flooding the port with invalid TLS connections. 

The last ont, search for cipher AES-256-CBC, comment it out, then add `cipher AES-256-GCM` instead:

```bash
;cipher AES-256-CBC
cipher AES-256-GCM
```

> AES-256-GCM cipher offers better security that AES-256-CBC and it's used by default by new versions of OpenVPN servers and clients

We also want to start OpenVPN with nobody prviliges for better security, so uncomment the following two lines:

```bash
user nobody
group nobody
```

The final server.conf will look like the following:
```bash
port 1194
;proto tcp
proto udp
;dev tap
dev tun
;dev-node MyTap
ca /etc/openvpn/easyrsa/pki/ca.crt
cert /etc/openvpn/easyrsa/pki/issued/openvpn.lab.crt
key /etc/openvpn/easyrsa/pki/private/openvpn.lab.key
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
keepalive 10 120
;tls-auth ta.key 0 # This file is secret
tls-crypt ta.key
;cipher AES-256-CBC
cipher AES-256-GCM
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
verb 3
explicit-exit-notify 1
```


## Firewall & Networking Configuration

This is very imprtant step to allow the clients to use our VPN server and to have internet access. In this section we will use IP Forwarding, a method that used to tell where the IP traffic should be routed, and Firewall rules that define how the clients' traffic should be handled.

### IP Forwarding

For this step we need to add in the file `/etc/sysctl.conf` the value `net.ipv4.ip_forward = 1`:

```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
```

confirm the changes:

```bash
sysctl -p
net.ipv4.ip_forward = 1
```


### Firewall configuration

In this section we are going to add out VPN interface "tun0" to our trusted zone, to do that, run the following commands:

```bash
firewall-cmd --add-interface=tun0 --zone=trusted --permanent
firewall-cmd --reload 
```

Confirm now the active zones:
```bash
firewall-cmd --get-active-zones 
home
  interfaces: enp0s3
internal
  sources: 192.168.56.1
trusted
  interfaces: tun0
```

> the output may differ from your network, but you should see the last two lines started with 'trusted' are added.

Next, we add the OpenVPN to the allowed services by the firewall, make the changes permanent, and apply the changes with reload:

```bash
firewall-cmd --add-service=openvpn --zone=trusted --permanent
firewall-cmd --reload
```

Confirm the changes:

```bash
firewall-cmd --list-services --zone=trusted
openvpn
```

Last change with the firewall is to enable NAT for the clients using linux network feature called [IP masquerading](https://www.linux.com/training-tutorials/what-ip-masquerading-and-when-it-use/)
With this feature, we can translate our VPN clients' addresses to the server's address and translate to the clients' addresses when they received the traffic back.

Run this command to enable this feature:

```bash
firewall-cmd --add-masquerade --permanent
firewall-cmd --reload
```

Now confirm that is enabled:

```bash
firewall-cmd --query-masquerade
yes
```

Finally we will create a routing rule to masquerade the traffic from OpenVPN network 10.8.0.1 to your default network interface. To do that, run the following commands:

DEV=$(ip route | awk '/^default/ {print $5}');
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o $DEV -j MASQUERADE
firewall-cmd --reload


## Start OpenVPN Service

Now it's time to start the VPN service. To run OpenVPN at the server as a daemon we will enable the service then start it

1. Enable the VPN Daemon:

```bash
systemctl enable openvpn-server@server
```

Note that the name `@server` is pointing to the name of the configuration file `server.conf` which you have already configured.

2. Start the Daemon:

```bash
systemctl start openvpn-server@server
```

To check that the server is running smoothly, run 

```bash
systemctl status openvpn-server@server
```

you should see something like below:

```bash
● openvpn-server@server.service - OpenVPN service for server
   Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: dis>
   Active: active (running) since Wed 2021-11-03 10:21:25 EDT; 1s ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
 Main PID: 10344 (openvpn)
   Status: "Initialization Sequence Completed"
    Tasks: 1 (limit: 4940)
   Memory: 1.0M
   CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
           └─10344 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-ve>

Nov 03 10:21:25 rhel1.lab openvpn[10344]: UDPv4 link remote: [AF_UNSPEC]
Nov 03 10:21:25 rhel1.lab openvpn[10344]: GID set to nobody
Nov 03 10:21:25 rhel1.lab openvpn[10344]: UID set to nobody
Nov 03 10:21:25 rhel1.lab openvpn[10344]: MULTI: multi_init called, r=256 v=256
Nov 03 10:21:25 rhel1.lab openvpn[10344]: IFCONFIG POOL: base=10.8.0.4 size=62, ipv6=0
Nov 03 10:21:25 rhel1.lab openvpn[10344]: ifconfig_pool_read(), in='client1,10.8.0.4', TODO: IP>
Nov 03 10:21:25 rhel1.lab openvpn[10344]: succeeded -> ifconfig_pool_set()
Nov 03 10:21:25 rhel1.lab openvpn[10344]: IFCONFIG POOL LIST
Nov 03 10:21:25 rhel1.lab openvpn[10344]: client1,10.8.0.4
Nov 03 10:21:25 rhel1.lab openvpn[10344]: Initialization Sequence Completed
```
As you can see from the systemd journal that openvpn has created a virtual TUN network interface: `10.8.0.0` and assigned itself with the first ip: `10.8.0.1` while listening for client connections on UDP port `1194`.

Confirm that using ip:

```bash
ip addr show
```

You should see the tun0 interface listed:

```bash
ip route
..
10.8.0.0/24 via 10.8.0.2 dev tun0 
10.8.0.2 dev tun0 proto kernel scope link src 10.8.0.1
```


# Setup The VPN Client

In my demo my client is running RHEL 8 so you can repeat the installation step I did for the server. In the next section we are going to discuss two kind of approachs that can be used when creating client's key-pair.


## Approach No.1 Generate The Client's Key-Pair At The VPN/CA Server

With this approach the step of generating client's key-pair is identical to the server's. At the VPN/CA server run:

`./easyrsa build-client-full client1.lab nopass`

> Note that nopass will not protect the client's private key with a pass phrase, however if you need the clients to enter their pass phrase each time they connect to the server, then rempve nopass.

The output: 

```bash
./easyrsa build-client-full client1.lab nopass

Note: using Easy-RSA configuration from: /etc/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating an EC private key
writing new private key to '/etc/easyrsa/pki/easy-rsa-11005.DFALPv/tmp.KcPqeF'
-----
Using configuration from /etc/easyrsa/pki/easy-rsa-11005.DFALPv/tmp.AKcbeB
Enter pass phrase for /etc/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'San Francisco'
organizationName      :ASN.1 12:'Tariqhawis.com'
organizationalUnitName:ASN.1 12:'TariqHawis CA'
commonName            :ASN.1 12:'client1.lab'
emailAddress          :IA5STRING:'tariqhawis @ gmail.com'
Certificate is to be certified until Oct 29 11:45:39 2023 GMT (730 days)

Write out database with 1 new entries
Data Base Updated
```

Then copy the key-pair to the client's machine:

```bash
scp pki/private/client1.lab.key root@192.168.56.110:/etc/openvpn/client/
scp pki/private/client1.lab.crt root@192.168.56.110:/etc/openvpn/client/
```

Next, proceed with the "Create Client VPN ovpn File" section down below.


## Approach No.2 Generate Key & CSR At The Client

You may require this approach if you don't want the clients' key to leave his hard drive. This is a better choice from security aspect. 

Before start with generate the certificate request, modify the vars file at the client machine and uncomment these two parameters to activate ECC algorithm:

```bash
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

Now run this command on the client machine:

```bash
cd /etc/easyrsa
./easyrsa gen-req client1.lab nopass
```

The output:

```bash
./easyrsa gen-req client1.lab nopass

Note: using Easy-RSA configuration from: /etc/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating an EC private key
writing new private key to '/etc/easyrsa/pki/easy-rsa-7729.fRFZPm/tmp.7IFW6h'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:
State or Province Name (full name) [California]:
Locality Name (eg, city) [San Francisco]:
Organization Name (eg, company) [Tariqhawis.com]:
Organizational Unit Name (eg, section) [TariqHawis CA]:
Common Name (eg: your user, host, or server name) [client1.lab]:
Email Address [tariqhawis @ gmail.com]:

Keypair and certificate request completed. Your files are:
req: /etc/easyrsa/pki/reqs/client1.lab.req
key: /etc/easyrsa/pki/private/client1.lab.key
```

Next, copy req file to the VPN/CA server in order to sign it:

```bash
scp pki/reqs/client1.lab.req root@192.168.56.101:/tmp
```

> Don't submit the key file to the server or to any other machine. This file supposed to remain secret at your machine.

At the server side, run this command to import the CSR file.

```bash
./easyrsa import-req /tmp/client1.lab.req client1.lab
```

The output:
```bash
Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020

The request has been successfully imported with a short name of: client1.lab
You may now use this name to perform signing operations on this request.
```

Then sign the request:

```bash
./easyrsa sign-req client client1.lab

Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 730 days:

subject=
    countryName               = US
    stateOrProvinceName       = California
    localityName              = San Francisco
    organizationName          = Tariqhawis.com
    organizationalUnitName    = TariqHawis CA
    commonName                = client1.lab
    emailAddress              = tariqhawis @ gmail.com


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /etc/easyrsa/pki/easy-rsa-14890.8Oz7Lu/tmp.dNU74L
Enter pass phrase for /etc/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'San Francisco'
organizationName      :ASN.1 12:'Tariqhawis.com'
organizationalUnitName:ASN.1 12:'TariqHawis CA'
commonName            :ASN.1 12:'client1.lab'
emailAddress          :IA5STRING:'tariqhawis @ gmail.com'
Certificate is to be certified until Nov  5 15:44:28 2023 GMT (730 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/easyrsa/pki/issued/client1.lab.crt
```
Finally, submit the signed certificate file back to the client:

```bash
scp pki/issued/client1.lab.crt root@192.168.56.110:/etc/openvpn/client/
root@192.168.56.110's password: 
client1.lab.crt                                               100% 3651   391.7KB/s   00:00    
```

In either approach, copy ca.crt and ta.key files to the client as well:

```bash
scp pki/ca.crt root@192.168.56.110:/etc/openvpn/client/
scp pki/private/ta.key root@192.168.56.110:/etc/openvpn/client/
```

## Create Client VPN ovpn File

Back to the client, just like the server, there is a sample configuration file for the client vpn at OpenVPN's docs, let's copy the file:

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/client.conf $HOME/client.ovpn
```

Now, edit the `client.ovpn` and make the necessary changes with the key paths:

```bash
ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/client1.lab.crt
key /etc/openvpn/client/client1.lab.key
```

Next, make the same changes of tls-auth 
```bash 
;tls-auth ta.key 1
tls-crypt /etc/openvpn/client/ta.key
```

Then change cipher to AES-256-GBC:

```bash
cipher AES-256-GBC
```

Next, search for the line remote my-server-1 1194 and replace it with the server IP `192.168.56.101`:

```bash
remote 192.168.56.101 1194 
```

Last thing, add this line to the end of the file and save the changes..

```bash
key-direction 1
```


## Connect to VPN Server

Finally, let's connect to the VPN server from our client machine, from your home directory run:

```bash
openvpn client.ovpn
```

You will receive output similar to this:

```bash
Nov  3 11:02:50 2021 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Nov  3 11:02:50 2021 ROUTE: default_gateway=UNDEF
Nov  3 11:02:50 2021 TUN/TAP device tun0 opened
Nov  3 11:02:50 2021 TUN/TAP TX queue length set to 100
Nov  3 11:02:50 2021 /sbin/ip link set dev tun0 up mtu 1500
Nov  3 11:02:50 2021 /sbin/ip addr add dev tun0 local 10.8.0.10 peer 10.8.0.9
Nov  3 11:02:50 2021 /sbin/ip route add 10.8.0.1/32 via 10.8.0.9
Nov  3 11:02:50 2021 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Nov  3 11:02:50 2021 Initialization Sequence Completed
```

Confirm your new tunnel interface with `ip addr show`:

```bash
5: tun0:  ...
    ... 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
...
```

Let's double-check and ping to the server IP: 10.8.0.1

```bash
ping -c 1 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=1.90 ms
```

If the ping succeeds, congratulations! You now have a functioning VPN. 


If you faced any issue or received any error here or there, leave me a comment below.

