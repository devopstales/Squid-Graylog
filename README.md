# Squid-Graylog
# Graylog Server.

Graylog version: 3.1

Elasticsearch version: 6

Create indice for Squid. In System / Indices. The index prefix must be squid as the image show.
This is important for the proper functioning of the streams.
![Indices](https://devopstales.github.io/img/include/squid_pfsense1.png)

Content Pack

![alt text](https://devopstales.github.io/img/include/squid_pfsense2.png)

Import de file in forder Content Pack and upload it.

![alt text](https://devopstales.github.io/img/include/squid_pfsense3.png)

Select squid from the list

![alt text](https://devopstales.github.io/img/include/squid_pfsense4.png)

![alt text](https://devopstales.github.io/img/include/squid_pfsense5.png)

And apply the content

![alt text](https://devopstales.github.io/img/include/squid_pfsense6.png)

Edit squid stream and select the index previusly created.

![alt text](https://devopstales.github.io/img/include/squid_pfsense7.png)

# Cerebro

As previously explained, by default graylog for each index that is created generates its own template and applies it every time the index rotates. If we want our own templates w$

To import personalized template open cerebro and will go to more/index template

![Content Pack](https://devopstales.github.io/img/include/More-Cerebro.png)

We create a new template

![Content Pack](https://devopstales.github.io/img/include/cerebro_template.png)

In the name we fill it with pfsense-custom and open the git file that has the template and paste its content here.

![Content Pack](https://devopstales.github.io/img/include/Pfsense_Custom_template.png)

And then we press the create button.

Now we will stop the graylog service to proceed to eliminate the index through Cerebro.

`#systemctl stop graylog-server.service`

In Cerebro we stand on top of the index and unfold the options and select delete index.

![Content Pack](https://devopstales.github.io/img/include/Delete-index-pfsense.png)

We start the graylog service again and this will create the index with this template.

`#systemctl start graylog-server.service`

# Pipelines

We need to edit the pipeline of pfsense then in System/Pipelines

Source of the rule that makes the adjustment of the timestamp that we are going to use in grafana:

```
    rule "timestamp_pfsense_for_grafana"
     when
     has_field("timestamp")
    then
    // the following date format assumes there's no time zone in the string
     let source_timestamp = parse_date(substring(to_string(now("America/Habana")),0,23), "yyyy-MM-dd'T'HH:mm:ss.SSS");
     let dest_timestamp = format_date(source_timestamp,"yyyy-MM-dd HH:mm:ss");
     set_field("real_timestamp", dest_timestamp);
    end
```

Add the existing pipeline to the squid stream by clicking the Edit connections at Pipeline connections.

![alt text](https://devopstales.github.io/img/include/squid_pfsense8.png)

# Confifure pfsense
```
# http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/
pkg add http://pkg.freebsd.org/FreeBSD:11:amd64/latest/All/beats-6.7.1.txz

nano /usr/local/etc/filebeat.yml
filebeat.prospectors:
- input_type: log
  document_type: squid3
  paths:
    - /var/squid/logs/access.log

output.logstash:
  # The Logstash hosts
  hosts: ["192.168.0.112:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  bulk_max_size: 2048
 #ssl.certificate_authorities: ["/etc/filebeat/logstash.crt"]
  template.name: "filebeat"
  template.path: "filebeat.template.json"
  template.overwrite: false
  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

/usr/local/sbin/filebeat -c /usr/local/etc/filebeat.yml test config
cp /usr/local/etc/rc.d/filebeat /usr/local/etc/rc.d/filebeat.sh
echo "filebeat_enable=yes" >> /etc/rc.conf.local
echo "filebeat_conf=/usr/local/etc/filebeat.yml" >> /etc/rc.conf.local

/usr/local/etc/rc.d/filebeat.sh start
ps aux | grep beat
```
