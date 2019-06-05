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
