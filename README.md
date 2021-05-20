# PowerBI-to-Postgresql-over-ssl
Microsoft PowerBI to Postgresql over ssl

Bundle Postgresql server 9.6 and above connect with PowerBI over a "On-premises data gateway" connection

At begin we configure postgresql server 

Edit file pg_hba.conf
```bash
# TYPE    DATABASE        USER            ADDRESS                        METHOD
# IPv4 local connections:
hostssl   powdb      powerbi_usr    192.168.0.10/32               trust clientcert=1
```
