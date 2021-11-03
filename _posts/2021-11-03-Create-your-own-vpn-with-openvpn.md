---
layout: post
hero_title: Create Your Own VPN Server with OpenVPN
description: In this post I will walk you through how to create a free VPN server using an opensource software called OpenVPN community edition
date: 2021-11-03
permalink: /:title/
redirect_from: /2021/11/03/create-your-own-vpn-server-with-openvpn.html
hero_image: /img/OpenVPN-Protocol-Logo-1536x864.webp
#hero_height: is-large
image: /img/OpenVPN-Protocol-Logo-1536x864.webp
tags: vpn openvpn TLS_VPN secure_connection free_vpn
---

If you are looking to build your own VPN server for personal use or as a business, this setup below will work just fine. All you need is a machine with good badnwidth and speed connection, the machine can be Dedicated, Virtual, or cloud server. 

In the below setup I used CentOS 8 as an operating system, same setps will works on RedHat, CentOS, Fedora of the same version. For Ubuntu and Debian, the repository will be different and there are some minor changes such as the path of the config files, easyrsa, and so on. If you need a separate post for Ubuntu/Debian OpenVPN setup, please leave me a comment below.

# What is OpenVPN?

OpenVPN is a full-featured SSL VPN that implements OSI layer 2 or 3 secure network extension using the industry-standard SSL/TLS protocol, supports flexible client authentication methods based on certificates, smart cards, and/or username/password credentials, and allows user or group-specific access control policies using firewall rules applied to the VPN virtual interface. And the best part is, it free and open source!


# Prepare the Server and the Client

For this demo, I am going to prepare two virtual machines with RHEL 8 as the operating system, we will use epel repoistory which will install the OpenVPN package community edition. I should mention here that the OpenVPN package is the same for the server and the client, the only difference will be with the configuration file that will be passed to the service, we are going to discuss that later on. 

The machines will look like the follows:

| Machine No. | Type | Hostname | IP Address |
|-----------|--------|--------|----------|
| 1        | VPN Server | openvpn.test | 192.168.56.103 |
| 2        | VPN Client | client1.test | 192.168.56.110 |

# Install OpenVPN & Easy RSA

First, we start with installing the OpenVPN package:

```bash
yum install epel-release -y
yum install openvpn easy-rsa -y
```

it's best to copy `easy-rsa` directory to a location such as ``/etc/openvpn``, before any edits, so that future easy-rsa package upgrades won't overwrite your modifications:

```bash
cp /usr/share/easy-rsa/3.0.8 /etc/openvpn/easy-rsa
```
The first step is to create CA root, server, client key-pairs so that will be used for the communication between the server and the client. Before that, we need to edit vars file with necessary values that will be used by the init-pki and ca-build scripts

we can find this file in the docs folder of easy-rsa, in RHEL 8 it is in `/usr/share/doc/easy-rsa/`, so we copy it to openvpn directory:

```bash
cp /usr/share/doc/easy-rsa/vars.example /etc/openvpn/easy-rsa/vars 
```

Next edit vars and uncomment/edit the required variables as mentioned below:

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
```


Now initialize the PKI:

`./easyrsa init-pki`

Then build the CA Root
```bash
./easyrsa clean-all
./easyrsa build-ca
```

> Note: Make sure when asked for CA Key Passphrase to choose a word with 8 character or more with including letters and numbers. Remember this password as you will need it when generaing the key-pair of the server in the next section.

The output should similar to this:

```bash
Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 4096 bit long modulus (2 primes)
........................................................................++++
.....................................++++
e is 65537 (0x010001)
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
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:openvpn.test
Email Address [tariqhawis @ gmail.com]:

CA creation cis omplete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easyrsa/pki/ca.crt
```

If you received the same results then everything is alright so far.


# Generate Certificate & Key for The Server

Next, we will generate a certificate and private key for the server with the below command:

`./easyrsa build-server-full openvpn.test`

openvpn.test is the name of my server, change it to the name of your server and make it the same as the Common Name that you specified earlier  

> Note that during the build you will be asked to set a passphrase that will protect your private key, make sure it's 8 characters long with letters and numbers. Remember this password as you will need it when you start the VPN server in the coming section.

The results should be similar to this:

```bash
Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating a RSA private key
.....................................................................................................++++
.................................++++
writing new private key to '/etc/openvpn/easyrsa/pki/easy-rsa-5610.1QZe2i/tmp.myCfTr'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from /etc/openvpn/easyrsa/pki/easy-rsa-5610.1QZe2i/tmp.6aH8h1
Enter pass phrase for /etc/openvpn/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'San Francisco'
organizationName      :ASN.1 12:'Tariqhawis.com'
organizationalUnitName:ASN.1 12:'TariqHawis CA'
commonName            :ASN.1 12:'openvpn.test'
emailAddress          :IA5STRING:'tariqhawis @ gmail.com'
Certificate is to be certified until Oct 29 11:40:07 2023 GMT (730 days)

Write out database with 1 new entries
Data Base Updated
```


# Generate Certificate & Key for the Client

Generating client certificates is very similar to the server step.

`./easyrsa build-client-full client1`

> Same note of the server key build. You also need to remember the private key password next as you will need it when you start your VPN client in the coming section.

```bash
Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating a RSA private key
.............................++++
....................................................................................++++
writing new private key to '/etc/openvpn/easyrsa/pki/easy-rsa-5733.z66hOr/tmp.suVvIi'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from /etc/openvpn/easyrsa/pki/easy-rsa-5733.z66hOr/tmp.5usucr
Enter pass phrase for /etc/openvpn/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'San Francisco'
organizationName      :ASN.1 12:'Tariqhawis.com'
organizationalUnitName:ASN.1 12:'TariqHawis CA'
commonName            :ASN.1 12:'client1'
emailAddress          :IA5STRING:'tariqhawis @ gmail.com'
Certificate is to be certified until Oct 29 11:45:39 2023 GMT (730 days)

Write out database with 1 new entries
Data Base Updated
```

I have defined the Common Name of the client as "client1", you may choose whatever name suits you, However, *make sure to always use a unique common name for each client you generate.*

In the production environment, the scenario of generating client key/certificate may deffer as the client may generate its own key locally and then submit the generated Certificate Signing Request CSR file to the PKI server "the one that has CA root certificate/key pair". In turn, the PKI server will process the CSR and return a signed certificate to the client. This way, the secret .key file will never need to leave the hard drive of the client machine on which it was generated. 

> For the above scenario, instead of build-client-full parameter, you need to pass gen-req, then submit the generated csr file to the server where the ca is installed, and from there run ./easyrsa sign-req [csr file] to sign the file you have received from the client.


# Generate Diffie Hellman Key

Diffie Hellman key must be generated for the OpenVPN server in order for the keys to be exchanged securely between the server and the client. Run the following command:

`./easyrsa gen-dh`

Now you may wait for several minutes before it finished

The output will be similar to this:

```bash
Note: using Easy-RSA configuration from: /etc/openvpn/easyrsa/vars
Using SSL: openssl OpenSSL 1.1.1g FIPS  21 Apr 2020
Generating DH parameters, 4096 bit long safe prime, generator 2
This is going to take a long time
................................................................................................................................+.++*++*++*

DH parameters of size 4096 created at /etc/openvpn/easyrsa/pki/dh.pem
```

# Key Files

Now we will find our newly-generated keys and certificate in the pki subdirectory. The tree of easy-rsa directory should look like this:

```
pki
|-- ca.crt
...
|-- dh.pem
...
|-- issued
|   |-- client1.crt
|   `-- openvpn.test.crt
|-- openssl-easyrsa.cnf
|-- private
|   |-- ca.key
|   |-- client1.key
|   `-- openvpn.test.key
...
```

Here is an explanation of the relevant files: 

| Filename    | Needed By                | Purpose                   | Secret |  
|-------------|--------------------------|---------------------------|--------|
| ca.crt      | server + all clients     | Root CA certificate       | NO     | 
| ca.key      | key signing machine only | Root CA key               | YES    | 
| dh.pem   | server only              | Diffie Hellman parameters | NO     | 
| openvpn.test.crt  | server only              | Server Certificate        | NO     | 
| openvpn.test.key  | server only              | Server Key                | YES    | 
| client1.crt | client1 only             | Client1 Certificate       | NO     | 
| client1.key | client1 only             | Client1 Key               | YES    | 


The final step is to copy the certificate/key of the client to his machine. In case the machine we have configured and built the pki is different than the OpenVPN server, you need to copy certificate/key pair to the server's machine as well.

>Note: in the production environment the scenario of the key generating process may differ slightly from the steps above, as you may have pki server and OpenVPN server separated from each other. Therefore, you will generate the private key and Certificate Signing Request (CSR) on the server machine and submit the CSR file to pki server which will return a signed certificate back to the server. The same steps go for the clients as they generate the key and the CSR file and send the CSR to the pki server to sign and return back the certificate. This scenario will ensure that the secret .key file never leaves the hard drive of the machine on which it was generated. 


# Configure OpenVPN Server & Client

At this stage, we need first to create server and client configuration. we can use the sample conf files from the OpenVPN docs folder since they are ideal as starting points for an OpenVPN server configuration. With this sample server configuration, the OpenVPN server will create a VPN using a virtual TUN network interface (for routing), will listen for client connections on UDP port 1194 (OpenVPN's official port number), and distribute virtual addresses to connecting clients from the 10.8.0.0/24 subnet. Let's copy them to OpenVPN folder:

**At the Server Machine:**

Copy `server.conf` from the docs folder to our OpenVPN folder

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/server/
```

Next, edit the copied server.conf and find the lines that start with: ca, cert, key, dh; and add the path of the corresponding key that we have generated before, as follows:

```bash
ca /etc/openvpn/easyrsa/pki/ca.crt

cert /etc/openvpn/easyrsa/pki/issued/openvpn.test.crt

key /etc/openvpn/easyrsa/pki/private/openvpn.test.key

dh /etc/openvpn/easyrsa/pki/dh.pem
```

Also, search for the line `tls-auth ta.key` and comment it out, unless you want to do extra security beyond that provided by SSL/TLS by creating an "HMAC firewall". This will help to block DoS attacks and UDP port flooding. If you are facing such an attack it's good to have it, otherwise, you can disable it.


```bash #tls-auth ta.key 0 # This file is secret ```


**At the Client Machine:**

Similarly, copy the `client.conf` from docs:

```bash
cp /usr/share/doc/openvpn/sample/sample-config-files/client.conf $HOME/
```

Also, edit the `client.conf` and make the necessary changes with the key paths:

```bash
ca /etc/openvpn/easyrsa/pki/ca.crt

cert /etc/openvpn/easyrsa/pki/issued/client1.crt

key /etc/openvpn/easyrsa/pki/private/client1.key
```

Simlarily, if you don't want "HMAC Firewall", just comment out the line `tls-auth ta.key`

```bash 
#tls-auth ta.key 1
```

Finally, search for the line remote my-server-1 1194 and replace it with the server hostname or the IP:

```bash
remote openvpn.test 1194
```

> Note: make sure your client can resolve the server's hostname, if there is no common DNS in the network, add this line in the client's hosts file:

```bash
echo "192.168.56.103 openvpn.test" >> /etc/hosts
```

# Start OpenVPN Server & Client

Finally, we have everything set and ready to run. To run OpenVPN at the server and the client you can either use the OpenVPN command and pass the configuration file `server.conf` and `client.conf` or run it as a daemon. It's recommended that for the server you run OpenVPN as a daemon and use the OpenVPN command for the client. Here is how we do it.

**At the Server Machine:**

1. Run systemd-tty-ask-password-agent to pass the password securely to the systemd daemon:

```bash
systemd-tty-ask-password-agent
```

you will receive this line to enter the password of your private key that you set earlier:

```bash
Enter Private Key Password: ********
```

2. Enable OpenVPN daemon to run the service on system startup. Note that the name `@server` is pointing to the name of the configuration file `server.conf`, so if you renamed the conf file then make the same change of '@server'.

```bash
systemctl enable openvpn-server@server
```

3. Start the daemon:

```bash
systemctl start openvpn-server@server
```

To check that the server is running smoothly, run `systemctl status openvpn-server@server, you should see something like below:

```bash
● openvpn-server@server.service - OpenVPN service for server
   Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: dis>
   Active: active (running) since Wed 2021-11-03 15:01:18 EDT; 23s ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
 Main PID: 2216 (openvpn)
   Status: "Initialization Sequence Completed"
    Tasks: 1 (limit: 4940)
   Memory: 2.2M
   CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@server.service
           └─2216 /usr/sbin/openvpn --status /run/openvpn-server/status-server.log --status-ver>

Nov 03 15:01:29 rhel1.lab openvpn[2216]: /sbin/ip addr add dev tun0 local 10.8.0.1 peer 10.8.0.2
Nov 03 15:01:29 rhel1.lab openvpn[2216]: /sbin/ip route add 10.8.0.0/24 via 10.8.0.2
Nov 03 15:01:29 rhel1.lab openvpn[2216]: Could not determine IPv4/IPv6 protocol. Using AF_INET
Nov 03 15:01:29 rhel1.lab openvpn[2216]: Socket Buffers: R=[212992->212992] S=[212992->212992]
Nov 03 15:01:29 rhel1.lab openvpn[2216]: UDPv4 link local (bound): [AF_INET][undef]:1194
Nov 03 15:01:29 rhel1.lab openvpn[2216]: UDPv4 link remote: [AF_UNSPEC]
Nov 03 15:01:29 rhel1.lab openvpn[2216]: MULTI: multi_init called, r=256 v=256
Nov 03 15:01:29 rhel1.lab openvpn[2216]: IFCONFIG POOL: base=10.8.0.4 size=62, ipv6=0
Nov 03 15:01:29 rhel1.lab openvpn[2216]: IFCONFIG POOL LIST
Nov 03 15:01:29 rhel1.lab openvpn[2216]: Initialization Sequence Completed
```
As you can see from the systemd journal that openvpn has created a virtual TUN network interface: `10.8.0.0` and assigned itself with the first ip: `10.8.0.1`. And it's listening for client connections on UDP port `1194`.

Confirm that using ip:

```bash
ip addr show
```

You should see the tun0 interface listed:

```bash
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::72b6:ab19:ffa5:c66/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever

```

**At the Client Machine:**

Finally, let's connect to the VPN server from our client machine, from your home directory run:

```bash
openvpn client.conf
```
As with the server, you will be asked for your private key password:

```bash
Enter Private Key Password: ********
```
After that, the connection will start initiating, the output will be similar to this:

```bash
Wed Nov  3 16:14:55 2021 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Wed Nov  3 16:14:55 2021 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.56.103:1194
Wed Nov  3 16:14:55 2021 Socket Buffers: R=[212992->212992] S=[212992->212992]
Wed Nov  3 16:14:55 2021 UDP link local: (not bound)
Wed Nov  3 16:14:55 2021 UDP link remote: [AF_INET]192.168.56.103:1194
Wed Nov  3 16:14:55 2021 TLS: Initial packet from [AF_INET]192.168.56.103:1194, sid=4cf51b64 792890f6
Wed Nov  3 16:14:55 2021 VERIFY OK: depth=1, C=US, ST=California, L=San Francisco, O=Tariqhawis.com, OU=TariqHawis CA, CN=openvpn.test, emailAddress=tariqhawis @ gmail.com
Wed Nov  3 16:14:55 2021 VERIFY KU OK
Wed Nov  3 16:14:55 2021 Validating certificate extended key usage
Wed Nov  3 16:14:55 2021 ++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
Wed Nov  3 16:14:55 2021 VERIFY EKU OK
Wed Nov  3 16:14:55 2021 VERIFY OK: depth=0, C=US, ST=California, L=San Francisco, O=Tariqhawis.com, OU=TariqHawis CA, CN=openvpn.test, emailAddress=tariqhawis @ gmail.com
Wed Nov  3 16:14:55 2021 Control Channel: TLSv1.3, cipher TLSv1.3 TLS_AES_256_GCM_SHA384, 4096 bit RSA
Wed Nov  3 16:14:55 2021 [openvpn.test] Peer Connection Initiated with [AF_INET]192.168.56.103:1194
Wed Nov  3 16:14:56 2021 SENT CONTROL [openvpn.test]: 'PUSH_REQUEST' (status=1)
Wed Nov  3 16:14:56 2021 PUSH: Received control message: 'PUSH_REPLY,route 10.8.0.1,topology net30,ping 10,ping-restart 120,ifconfig 10.8.0.6 10.8.0.5,peer-id 1,cipher AES-256-GCM'
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: timers and/or timeouts modified
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: --ifconfig/up options modified
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: route options modified
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: peer-id set
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: adjusting link_mtu to 1624
Wed Nov  3 16:14:56 2021 OPTIONS IMPORT: data channel crypto options modified
Wed Nov  3 16:14:56 2021 Data Channel: using negotiated cipher 'AES-256-GCM'
Wed Nov  3 16:14:56 2021 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Nov  3 16:14:56 2021 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Nov  3 16:14:56 2021 ROUTE_GATEWAY 10.0.3.2/255.255.255.0 IFACE=enp0s8 HWADDR=08:00:27:5d:d1:cc
Wed Nov  3 16:14:56 2021 TUN/TAP device tun0 opened
Wed Nov  3 16:14:56 2021 TUN/TAP TX queue length set to 100
Wed Nov  3 16:14:56 2021 /sbin/ip link set dev tun0 up mtu 1500
Wed Nov  3 16:14:56 2021 /sbin/ip addr add dev tun0 local 10.8.0.6 peer 10.8.0.5
Wed Nov  3 16:14:56 2021 /sbin/ip route add 10.8.0.1/32 via 10.8.0.5
Wed Nov  3 16:14:56 2021 Initialization Sequence Completed
```

Confirm your new tunnel interface with `ip addr show`:

```bash
5: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 100
    link/none 
    inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::3751:62b9:ded4:f112/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

Let's double-check and ping to the server IP: 10.8.0.1

```bash
ping -c 1 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=1.90 ms
```

If the ping succeeds, congratulations! You now have a functioning VPN. 


If you faced any issue or received any error here or there, leave me a comment below.

