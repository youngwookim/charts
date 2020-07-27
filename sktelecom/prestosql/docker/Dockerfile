FROM openjdk:11

ARG PRESTO_VERSION=339
ENV PRESTO_HOME=/usr/lib/presto
ENV PRESTO_DATA_DIR=/presto/data
ENV BASE_URL=https://repo1.maven.org/maven2

# extra dependency for running launcher
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    vim wget curl git && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd -g 999 presto && \
    useradd -r -u 999 -g presto --create-home --shell /bin/bash presto

WORKDIR /home/presto
RUN wget -q ${BASE_URL}/io/prestosql/presto-server/${PRESTO_VERSION}/presto-server-${PRESTO_VERSION}.tar.gz && \
  tar xvzf presto-server-${PRESTO_VERSION}.tar.gz -C /usr/lib/ && \
  ln -s /usr/lib/presto-server-${PRESTO_VERSION} $PRESTO_HOME && \
  rm -rf /home/presto/presto-server-${PRESTO_VERSION}.tar.gz

RUN mkdir -p ${PRESTO_DATA_DIR}

COPY etc ${PRESTO_HOME}/etc

EXPOSE 8080

USER presto:presto

VOLUME ["${PRESTO_HOME}/etc", "${PRESTO_DATA_DIR}"]

WORKDIR ${PRESTO_HOME}

ENTRYPOINT ["./bin/launcher"]

CMD ["run"]
