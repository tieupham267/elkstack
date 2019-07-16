# ELK Stack

Using ELK Stack

## I. Requirements

- CentOS 7
- Epel repo
- Java 1.8.0 Runtime (JRE)
- wget
- ElasticSearch 7.1.1
- Kibana 7.1.1
- LogStash 7.1.1
- FileBeat 7.1.1

## II. Model

- IP Server: 192.168.11.6
- Client

## III. Requirement

### CentOS 7 Update

```bash
yum -y update
```

### Install Epel repo

```bash
yum -y install epel-release
```

### Install Java

```bash
yum -y install java-1.8.0-openjdk
```

Ref: [How to install Java on CentOS 7
](https://linuxize.com/post/install-java-on-centos-7/)

### Download package

[ElasticSearch](https://www.elastic.co/downloads/elasticsearch)

[Kibana](https://www.elastic.co/downloads/kibana)

[LogStash](https://www.elastic.co/downloads/logstash)

[Filebeat](https://www.elastic.co/downloads/beats/filebeat)

## Setup ElasticSearch, Kibana, LogStash, File Beat

Ref: https://kifarunix.com/install-elastic-stack-7-on-ubuntu-18-04-debian-9-8/

### ElasticSearch

Install with command

`yum -y localinstall elasticsearch-7.1.0-x86_64.rpm`

Run ElasticSearch

`systemctl start elasticsearch`

Check status

`systemctl status elasticsearch`

Check port open

`ss -nao | grep 9200`

Check status with curl

`curl -X GET "localhost:9200/"`

Init start-up

`systemctl enable elasticsearch`

### Kibana

Install

`yum -y localinstall kibana-7.1.0-x86_64.rpm`

Configure

`vi /etc/kibana/kibana.yml`

`server.host: "0.0.0.0"`

Start Kibana

`systemctl start kibana`

Init start-up

`systemctl enable kibana`

Check status

`systemctl status kibana`

Check open port

`ss -nao | grep 5601`

Open firewall

`firewall-cmd --zone=public --add-port=5601/tcp --permanent`

`firewall-cmd --reload`

Open browser and acccess Kibana via url [http://192.168.11.6:5601](http://192.168.11.6:5601)

### LogStash

Install

`yum -y localinstall logstash-7.1.0.rpm`

Configure, more detail see on "snippet\logtash\conf.d"

`vi /etc/logstash/conf.d/02-syslog.conf`

```yml
input {
 beats {
   port => 5044

   # Set to False if you do not use SSL
   #ssl => true

   # Delete below linesif you do not use SSL
   #ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
   #ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
   }
}
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGLINE}" }
    }

    date {
      match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}
output {
  elasticsearch {
    hosts => localhost
    index => "syslog-%{+YYYY.MM.dd}"
  }

  stdout {
    codec => rubydebug
  }
}
```

Check configure test to ensure logstash work beforce apply to production

```bash'
/usr/share/logstash/bin/logstash --path.settings=/etc/logstash/ --config.test_and_exit -f /etc/logstash/conf.d/02-beat.conf
```

Ref: https://kifarunix.com/install-elasticsearch-7-x-on-centos-7-fedora-29/

## Configure

Ref:

### FileBeat

Install

`yum -y localinstall filebeat-7.1.0-x86_64.rpm`

Configure

`vi /etc/filebeat/filebeat.yml`

Comment

`#output.elasticsearch:`

`#hosts: ["localhost:9200"]`

Uncomment

`output.logstash:`

`hosts: ["localhost:5044"]`

Change

```yml
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/log/*
    #- c:\programdata\elasticsearch\logs\*
```

Verify config

```bash
filebeat test config
```

Assuming you're using filebeat 6.x (these tests were done with filebeat 6.5.0 in a CentOS 7.5 system)

To test your filebeat configuration (syntax), you can do:

```bash
[root@localhost ~]# filebeat test config
Config OK
```

If you just downloaded the tarball, it uses by default the filebeat.yml in the untared filebeat directory. If you installed the RPM, it uses /etc/filebeat/filebeat.yml.

If you want to define a different configuration file, you can do:

```bash
[root@localhost ~]# filebeat test config -c /etc/filebeat/filebeat2.yml
Config OK
```

To test the output block (i.e: if you have connectivity to elasticsearch instance or kafka broker), you can do:

```bash
[root@localhost ~]# filebeat test output
elasticsearch: http://localhost:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: ::1, 127.0.0.1
    dial up... ERROR dial tcp [::1]:9200: connect: connection refused
```

In this case my localhost elasticsearch is down, so filebeat throws an error saying it cannot connect to my output block.

The same way as syntax validation (test config), you can provide a different configuration file for output connection test:

```bash
[root@localhost ~]# filebeat test output -c /etc/filebeat/filebeat2.yml
logstash: localhost:5044...
  connection...
    parse host... OK
    dns lookup... OK
    addresses: ::1, 127.0.0.1
    dial up... ERROR dial tcp [::1]:5044: connect: connection refused
```

In this alternative configuration file, my output block also fails to connect to a logstash instance.

Run filebeat

```bash
systemctl start filebeat
```

Check status

```bash
systemctl status filebeat.service
```

Ref: https://stackoverflow.com/questions/51682649/filebeat-configuration-test-with-output

## Enable and Configure for Security Pack

TODO

# TODO

- Access metadata fields

  `index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"`
- filebeat command
- logstash advance configure
- Optimize JVM
- Classification log base on event (note base port)
