# PowerBI-to-Postgresql-over-ssl
Microsoft PowerBI to Postgresql over ssl

Bundle Postgresql server 9.6 and above connect with PowerBI over a "On-premises data gateway" connection

At begin we configure postgresql server 

Edit file **pg_hba.conf**
```bash
# TYPE    DATABASE        USER            ADDRESS                        METHOD
# IPv4 local connections:
hostssl   powdb      powerbi_usr    192.168.0.10/32               trust clientcert=1
```

Edit file **postgresql.conf**
```bash
ssl = on                                                        # (change requires restart)
ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL'                        # allowed SSL ciphers
ssl_prefer_server_ciphers = on                                  # (change requires restart)
ssl_cert_file = '/etc/ssl/postgresql/postgresql.crt'            # (change requires restart)
ssl_key_file = '/etc/ssl/postgresql/postgresql.key'             # (change requires restart)
ssl_ca_file = '/etc/ssl/postgresql/root.crt'                    # (change requires restart)
```

Create directory in PGDATA like this /var/lib/pgsql/11/data if you installed Postgresql version 11
```bash
ls -l /var/lib/pgsql/11/data/   # you need know that directory cert not created before
mkdir /var/lib/pgsql/11/data/cert
cd /var/lib/pgsql/11/data/cert
```

Create certificates
First we need create root.crt file valid 2 years where "PBIEgwService" - username of PowerBI gateway
```bash
openssl req -new -text -nodes -keyout server.key -out server.csr -subj '/C=RU/ST=Russia/L=Moscow/O=Mycompany/OU=Mycompany/CN=PBIEgwService'
openssl req -x509  -days 750 -text -in server.csr -extensions v3_ca -key server.key -out server.crt
cp server.crt root.crt
rm -f server.csr
chmod og-rwx server.key
```

Second we need create client certificate file valid 2 years where db.mycompeny.local your postgresql server name with IP 192.168.0.10
```bash
openssl req -new -nodes -keyout client.key -out client.csr -subj '/C=RU/ST=Russia/L=Moscow/O=Mycompany/OU=Mycompany/CN=db.mycompeny.local'
openssl x509 -req -days 750 -CAcreateserial -in client.csr -CA root.crt -CAkey server.key -out client.crt
rm -f client.csr
```

