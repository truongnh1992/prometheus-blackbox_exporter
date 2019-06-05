FROM centos

ADD blackbox_exporter-0.14.0.linux-amd64.tar.gz /

RUN cd /blackbox_exporter-* && \
    mv blackbox_exporter /bin/

COPY blackbox.yml       /etc/blackbox_exporter/config.yml

EXPOSE      9115
ENTRYPOINT  [ "/bin/blackbox_exporter" ]
CMD         [ "--config.file=/etc/blackbox_exporter/config.yml" ]
