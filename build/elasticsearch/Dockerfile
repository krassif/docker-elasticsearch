# This Dockerfile was generated from templates/Dockerfile.j2

FROM armv7/armhf-java7
LABEL maintainer "Elastic Docker Team <docker@elastic.co>"

ENV ELASTIC_CONTAINER true
ENV PATH /usr/share/elasticsearch/bin:$PATH

RUN groupadd --gid 1000 elasticsearch && \
    adduser -uid 1000 --gid 1000 --home /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

# Download/extract defined ES version. busybox tar can't strip leading dir.
RUN wget --progress=bar:force https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-0.90.8.tar.gz && \
    EXPECTED_SHA=$(wget -O - https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-0.90.8.tar.gz.sha1.txt) && \
    tar zxf elasticsearch-0.90.8.tar.gz && \
    chown -R elasticsearch:elasticsearch elasticsearch-0.90.8 && \
    mv elasticsearch-0.90.8/* . && \
    rmdir elasticsearch-0.90.8 && \
    rm elasticsearch-0.90.8.tar.gz

##    test $EXPECTED_SHA == $(sha1sum elasticsearch-0.90.8.tar.gz | awk '{print $1}') &&

RUN set -ex && for esdirs in config data logs plugins; do \
        mkdir -p "var/$esdirs"; \
        chown -R elasticsearch:elasticsearch "var/$esdirs"; \
        ln -s "var/$esdirs" "$esdirs"; \
    done

USER elasticsearch

# Install x-pack and also the ingest-{agent,geoip} modules required for Filebeat
## RUN for PLUGIN_TO_INST in x-pack ingest-user-agent ingest-geoip; do elasticsearch-plugin install --batch "$PLUGIN_TO_INST"; done
COPY elasticsearch.yml config/
COPY log4j2.properties config/
COPY bin/es-docker bin/es-docker

USER root
RUN apt-get update && \
    apt-get -y --force-yes install curl jq lsof && \
    chown elasticsearch:elasticsearch config/elasticsearch.yml config/log4j2.properties bin/es-docker && \
    chmod 0750 bin/es-docker

USER elasticsearch

# Define overridable health check

CMD ["/bin/bash", "bin/es-docker"]

EXPOSE 9200 9300
