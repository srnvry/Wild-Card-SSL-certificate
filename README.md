# Wild-Card-SSL-certificate
Create wild card ssl certificate for OpenStack IDM Satellite Quay 

**Step 1: Set Up Your Root CA**
-------------------------------

### **1.1 Create Required Directories**

```
mkdir -p ~/myCA/{certs,crl,newcerts,private,csr}
chmod 700 ~/myCA/private
touch ~/myCA/index.txt
echo 1000 > ~/myCA/serial
```

### **1.2 Create the OpenSSL Configuration**

Create `~/myCA/openssl.cnf` with the following content

```
[ ca ]
default_ca = CA_default

[ CA_default ]
dir              = ~/myCA
database         = $dir/index.txt
new_certs_dir    = $dir/newcerts
certificate      = $dir/certs/ca.crt
serial           = $dir/serial
private_key      = $dir/private/ca.key
default_md       = sha256
policy           = policy_any
default_days     = 7300

[ policy_any ]
countryName             = supplied
stateOrProvinceName     = supplied
localityName            = supplied
organizationName        = supplied
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
prompt = no

[ req_distinguished_name ]
C  = IN
ST = New Delhi
L  = New Delhi
O  = NIC
OU = SCI
CN = SCI Root CA

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical,CA:true
keyUsage = critical,keyCertSign,cRLSign
```

### **1.3 Generate Root CA Key and Certificate**

  
```
openssl genrsa -out ~/myCA/private/ca.key 4096
```
```
openssl req -x509 -new -nodes -key ~/myCA/private/ca.key -sha256 -days 7300 -out ~/myCA/certs/ca.crt -config ~/myCA/openssl.cnf
```
* * *

**Step 2: Generate SSL Certificates for Applications**
------------------------------------------------------

### **2.1 Create a Configuration for Application Certificates**

Create `~/myCA/app.cnf`:

  
```
[ req ]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[ req_distinguished_name ]
C  = IN
ST = New Delhi
L  = New Delhi
O  = <org>
OU = <OU>
CN = <domain>

[ v3_req ]
subjectAltName = @alt_names
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth

[ alt_names ]
DNS.1 = <domain>
DNS.2 = <sub-domain>
DNS.3 = *.<sub-domain/domain>
IP.1 = <ip1>
IP.2 = <ip2>
```

### **2.2 Generate Certificate for Applications**

#### **Generate Key and CSR**

  
```
openssl genrsa -out ~/myCA/private/app.key 2048

openssl req -new -key ~/myCA/private/app.key -out ~/myCA/csr/app.csr -config ~/myCA/app.cnf
```

#### **Sign the Certificate with Root CA**

  
```
openssl x509 -req -in ~/myCA/csr/app.csr -CA ~/myCA/certs/ca.crt -CAkey ~/myCA/private/ca.key -CAcreateserial -out ~/myCA/certs/app.crt -days 7300 -sha256 -extfile ~/myCA/app.cnf -extensions v3_req
```

* * *

**Step 3: Deploy the Certificates**
-----------------------------------

*   **Root CA (`ca.crt`)** → Install on all servers and machines to establish trust.
*   **Application Certificate (`app.crt`)** → Use for Quay Registry, Satellite, OpenStack, and FreeIPA.
*   **Private Key (`app.key`)** → Securely store and use in server configurations
