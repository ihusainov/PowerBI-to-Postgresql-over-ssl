# PowerBI-to-Postgresql-over-ssl
Microsoft PowerBI to Postgresql over ssl

Bundle Postgresql server 9.6 and above connect with PowerBI over a "On-premises data gateway" connection

At begin we configure postgresql server 

Edit file **pg_hba.conf**
```bash
# TYPE    DATABASE        USER            ADDRESS                        METHOD
# IPv4 local connections:
hostssl   powdb      powerbi_sync    192.168.0.10/32               trust clientcert=1
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

Next we need copy crtificate to the config directory
```bash
rm -f /etc/ssl/postgresql/*
cp root.crt /etc/ssl/postgresql/root.crt
cp client.crt /etc/ssl/postgresql/postgresql.crt
cp client.key /etc/ssl/postgresql/postgresql.key
chmod -R 700 /etc/ssl/postgresql
chown -R postgres.postgres /etc/ssl/postgresql
systemctl restart postgresql-11
```

And the last we need copy cert files to the server with PowerBI 
```
scp /etc/ssl/postgresql/postgresql.crt /etc/ssl/postgresql/postgresql.key root.crt   user@powerBI.mycompeny.local:/home
```

Connect over RDP to the Windows Server and copy cert files to postgresql directory and create test ODBC connector
```cmd
Put certificate files to directory  C:\Windows\ServiceProfiles\PBIEgwService\AppData\Roaming\postgresql
Put certificate files to directory  C:\Users\user\AppData\Roaming\postgresql
```

![image](https://user-images.githubusercontent.com/62062799/120436610-ce0ea680-c387-11eb-96df-363fdfdac6c9.png)


Start - Control pannel - Administrative tools - ODBC Data Sources (64-bit) - Add - Test

![image](https://user-images.githubusercontent.com/62062799/120437678-0c589580-c389-11eb-8bcf-734ab21b7fc8.png)

![image](https://user-images.githubusercontent.com/62062799/120436689-e8488480-c387-11eb-97ba-f3e41488d18d.png)

Next go to the PowerBI site https://app.powerbi.com/

![image](https://user-images.githubusercontent.com/62062799/120436758-031af900-c388-11eb-9b10-854470121aa7.png)

Add new Gateway then apply and Test all connections

```bash
database=jira_db;driver={PostgreSQL Unicode(x64)};port=5432;server=postgre-serv.local;sslmode=verify-full
```

![image](https://user-images.githubusercontent.com/62062799/120438579-3a8aa500-c38a-11eb-9873-29046ffeb649.png)

All done




