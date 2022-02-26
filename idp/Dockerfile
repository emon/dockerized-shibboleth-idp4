FROM alpine:latest as tmp

ENV JETTY_VERSION=9.4.45.v20220203
ENV JETTY_HASH=97fb0b0456d21ba18df4b89707157e02ebbc1a60
ENV JETTY_HOME=/opt/jetty-home
ENV JETTY_BASE=/opt/jetty-base
ENV JAVA_HOME=/usr/lib/jvm/default-jvm
ENV PATH=$PATH:$JAVA_HOME/bin

LABEL maintainer="Noriyuki TAKEI"

RUN apk --no-cache add curl tar openjdk11-jre-headless

# Jettyをダウンロードする。
RUN curl -O https://repo1.maven.org/maven2/org/eclipse/jetty/jetty-distribution/$JETTY_VERSION/jetty-distribution-$JETTY_VERSION.tar.gz \
    && echo "$JETTY_HASH  jetty-distribution-$JETTY_VERSION.tar.gz" | sha1sum -c - \
    && tar -zxvf jetty-distribution-$JETTY_VERSION.tar.gz -C /opt \
    && ln -s /opt/jetty-distribution-$JETTY_VERSION/ $JETTY_HOME \
    && rm jetty-distribution-$JETTY_VERSION.tar.gz

# Jettyの初期設定を行う。
RUN mkdir -p $JETTY_BASE/modules $JETTY_BASE/lib/ext $JETTY_BASE/lib/logging $JETTY_BASE/resources \
    && cd $JETTY_BASE \
    && $JAVA_HOME/bin/java -jar $JETTY_HOME/start.jar --create-startd --add-to-start=http2,http2c,deploy,ext,annotations,jstl,rewrite,ssl,http-forwarded

COPY opt/jetty-base/etc/tweak-ssl.xml $JETTY_BASE/etc/tweak-ssl.xml
COPY opt/jetty-base/webapps/idp.xml $JETTY_BASE/webapps/idp.xml

FROM alpine:latest

ENV JETTY_HOME=/opt/jetty-home
ENV JETTY_BASE=/opt/jetty-base
ENV JAVA_HOME=/usr/lib/jvm/default-jvm

ENV IDP_VERSION=4.1.5
ENV IDP_HASH=8173a2f5e05ec75e7b6ccbbab45639decb3286ecd705f1350a75a2b551096596

ENV JETTY_KEYSTORE_PASSWORD=storepwd
ENV JETTY_KEYSTORE_PATH=etc/keystore
ENV IDP_HOME=/opt/shibboleth-idp
ENV IDP_SRC=/opt/shibboleth-identity-provider-$IDP_VERSION
ENV IDP_SCOPE=example.org
ENV IDP_HOST_NAME=idp.example.org
ENV IDP_ENTITY_ID=https://idp.example.org/idp/shibboleth
ENV IDP_KEYSTORE_PASSWORD=password
ENV IDP_SEALER_PASSWORD=password
ENV JETTY_JAVA_ARGS="jetty.home=$JETTY_HOME \
    jetty.base=$JETTY_BASE \
    -Djetty.sslContext.keyStorePassword=$JETTY_KEYSTORE_PASSWORD \
    -Djetty.sslContext.keyStorePath=$JETTY_KEYSTORE_PATH"

RUN apk --no-cache add patch openjdk11-jre-headless

COPY bin/ /usr/local/bin/

RUN chmod +x /usr/local/bin/gen-idp-conf.sh

COPY --from=tmp /opt/ /opt/
COPY /opt/ /opt/

ADD shibboleth-idp/ /opt/shibboleth-idp/

# Shibbolethの設定ファイル(attribute-resolverなど)にパッチを当てるスクリプトをコンテナにコピーする。
COPY bin/mod-idp-conf.sh /usr/local/bin/

# Shibbolethの設定ファイル(attribute-resolverなど)に当てるパッチをコンテナにコピーする。
RUN mkdir /tmp/patch
COPY patch/* /tmp/patch/

# Shibbolethの設定ファイル(attribute-resolverなど)にパッチを当てるスクリプトを実行する。
RUN /bin/sh /usr/local/bin/mod-idp-conf.sh

CMD $JAVA_HOME/bin/java -jar $JETTY_HOME/start.jar $JETTY_JAVA_ARGS

EXPOSE 8080 8443 2222
