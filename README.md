# Hands-on Labs: prometheus-blackbox_exporter

`Prometheus` is a tool monitoring the microservices and applications metrics using the pull mechanism and `blackbox_exporter` allows blackbox probing of endpoints over HTTP, HTTPS, DNS, TCP and ICMP.

This document will show you the way to monitor the microservices enpoints using a `prometheus` server and `blackbox_exporter`.

## Prometheus

**Prometheus Dockerfile**
```dockerfile
FROM centos

ADD prometheus-2.10.0.linux-amd64.tar.gz /

#Download prometheus and configure

RUN cd /prometheus-* && \
    mv prometheus /bin/ && \
    mv promtool /bin/ && \
    mkdir /usr/share/prometheus/  && \
    mv console_libraries /usr/share/prometheus/console_libraries/ && \
    mv consoles/ /usr/share/prometheus/consoles/ && \
    mkdir /etc/prometheus && \
    cd  && \
    rm -rf /prometheus-*

COPY prometheus.yml /etc/prometheus/prometheus.yml

RUN ln -s /usr/share/prometheus/console_libraries /usr/share/prometheus/consoles/ /etc/prometheus/

WORKDIR /usr/share/prometheus

EXPOSE 9090

ENTRYPOINT ["/bin/prometheus"]
CMD  ["--config.file=/etc/prometheus/prometheus.yml", \
      "--storage.tsdb.path=/prometheus-storage", \
"--web.external-url=http://localhost/"]
```

**Prometheus config file** (prometheus.yml)
```yml
global:
 scrape_interval: 10s
scrape_configs:
 - job_name: blackbox
   metrics_path: /probe
   params:
     module: [http_2xx]
   static_configs:
    - targets:
       - https://www.robustperception.io/
       - http://prometheus.io/blog
       - https://google.com
   relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
replacement: localhost:9115 # The blackbox exporter.
```
**Building prometheus image and running its container**
```console
sudo docker build -t prometheus .
sudo docker run -d -p 9090:9090 --name prometheus prometheus
```
**Checking logs**
```console
sudo docker logs prometheus
```
```log
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:286 msg="no time or size retention was set so using the default time retention" duration=15d
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:322 msg="Starting Prometheus" version="(version=2.10.0, branch=HEAD, revision=d20e84d0fb64aff2f62a977adc8cfb656da4e286)"
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:323 build_context="(go=go1.12.5, user=root@a49185acd9b0, date=20190525-12:28:13)"
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:324 host_details="(Linux 4.4.0-131-generic #157-Ubuntu SMP Thu Jul 12 15:51:36 UTC 2018 x86_64 314dd2a449c1 (none))"
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:325 fd_limits="(soft=1048576, hard=1048576)"
level=info ts=2019-06-05T08:25:55.723Z caller=main.go:326 vm_limits="(soft=unlimited, hard=unlimited)"
level=info ts=2019-06-05T08:25:55.724Z caller=main.go:645 msg="Starting TSDB ..."
level=info ts=2019-06-05T08:25:55.724Z caller=web.go:417 component=web msg="Start listening for connections" address=0.0.0.0:9090
level=info ts=2019-06-05T08:25:55.736Z caller=main.go:660 fs_type=794c7630
level=info ts=2019-06-05T08:25:55.736Z caller=main.go:661 msg="TSDB started"
level=info ts=2019-06-05T08:25:55.736Z caller=main.go:730 msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2019-06-05T08:25:55.745Z caller=main.go:758 msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml
level=info ts=2019-06-05T08:25:55.745Z caller=main.go:614 msg="Server is ready to receive web requests."
```

**Accessing prometheus server at `http://localhost:9090`**

## blackbox_exporter
**blackbox Dockerfile**
```dockerfile
FROM centos

ADD blackbox_exporter-0.14.0.linux-amd64.tar.gz /

RUN cd /blackbox_exporter-* && \
    mv blackbox_exporter /bin/

COPY blackbox.yml       /etc/blackbox_exporter/config.yml

EXPOSE      9115
ENTRYPOINT  [ "/bin/blackbox_exporter" ]
CMD [ "--config.file=/etc/blackbox_exporter/config.yml" ]
```

**blackbox config file** (blackbox.yml)
```yml
modules:
  http_2xx:
    prober: http
    http:
  http_post_2xx:
    prober: http
    http:
method: POST
```
**Building blackbox image and running its container**
```console
sudo docker build -t blackbox .
sudo docker run -d -p 9115:9115 --name blackbox blackbox
```
