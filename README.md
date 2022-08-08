# nginx-mtls

### Nginx latest mtls build

```
docker run -v /tmp/certs/certs:/etc/ssl/selfsigned/  -v /tmp/certs/conf:/etc/nginx/conf.d/ -p 8443:443 nginx
```
![](https://github.com/x033f/nginx-mtls/blob/main/out2.gif)
#### build dirs and tmpl conf /tmp/certs/

```
cat<<EOF>ca.config
[ ca ]
default_ca = CA_CLIENT
[ CA_CLIENT ]
dir = ./db
certs = $dir/certs
new_certs_dir = $dir/newcerts
database = $dir/index.txt
serial = $dir/serial
certificate = ./ca.crt
private_key = ./ca.key
default_days = 365
default_crl_days = 7
default_md = md5
policy = policy_anything
[ policy_anything ]
countryName = optional
stateOrProvinceName = optional
localityName = optional
organizationName = optional
organizationalUnitName = optional
commonName = optional
emailAddress = optional
EOF

mkdir db
mkdir db/certs
mkdir db/newcerts
touch db/index.txt
echo "01" > db/serial
```

### Gen server certs 

```
openssl req -new -newkey rsa:2048 -nodes -keyout ca.key -x509 -days 500 -subj /C=RU/ST=Moscow/L=Moscow/O=Companyname/OU=Admin/CN=0.0.0.0/emailAddress=support@site.com -out ca.crt
openssl genrsa -des3 -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
openssl rsa -in server.key -out server.nopass.key
```

### Gen client certs 

```
openssl req -new -newkey rsa:2048 -nodes -keyout client2.key -subj /C=RU/ST=Moscow/L=Moscow/O=Companyname/OU=Client2/CN=172.17.0.1/emailAddress=client2@site.com -out client2.csr
openssl ca -config ca.config -in client2.csr -out client2.crt -batch
```

### Pack p12

```
openssl pkcs12 -export -in client2.crt -inkey client2.key -out cert.p12
````

