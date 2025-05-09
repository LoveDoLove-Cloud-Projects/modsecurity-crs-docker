ARG HTTPD_VERSION="n/a"

FROM image AS build

ARG MODSEC2_VERSION="n/a"
ARG MODSEC2_FLAGS="n/a"
ARG LUA_VERSION="n/a"

RUN set -eux; \
    echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections; \
    apt-get update -qq; \
    apt-get install -y -qq --no-install-recommends --no-install-suggests \
        automake \
        ca-certificates \
        g++ \
        git \
        libapr1-dev \
        libaprutil1-dev \
        libcurl4-gnutls-dev \
        libfuzzy-dev \
        libpcre2-dev \
        libtool \
        libxml2-dev \
        libyajl-dev \
        lua${LUA_VERSION}-dev \
        make \
        pkgconf \
        wget

RUN set -eux; \
    wget --quiet https://github.com/owasp-modsecurity/ModSecurity/archive/refs/tags/v${MODSEC2_VERSION}.tar.gz; \
    tar -zxvf v${MODSEC2_VERSION}.tar.gz; \
    cd ModSecurity-${MODSEC2_VERSION}; \
    ./autogen.sh; \
    ./configure ${MODSEC2_FLAGS}; \
    make; \
    make install; \
    make clean

FROM image AS crs_release

ARG CRS_RELEASE

# hadolint ignore=DL3008,SC2016
RUN set -eux; \
    apt-get update; \
    apt-get -y install --no-install-recommends \
      ca-certificates \
      curl \
      gnupg; \
    mkdir /opt/owasp-crs; \
    curl -sSL https://github.com/coreruleset/coreruleset/releases/download/v${CRS_RELEASE}/coreruleset-${CRS_RELEASE}-minimal.tar.gz -o v${CRS_RELEASE}-minimal.tar.gz; \
    curl -sSL https://github.com/coreruleset/coreruleset/releases/download/v${CRS_RELEASE}/coreruleset-${CRS_RELEASE}-minimal.tar.gz.asc -o coreruleset-${CRS_RELEASE}-minimal.tar.gz.asc; \
    gpg --fetch-key https://coreruleset.org/security.asc; \
    gpg --verify coreruleset-${CRS_RELEASE}-minimal.tar.gz.asc v${CRS_RELEASE}-minimal.tar.gz; \
    tar -zxf v${CRS_RELEASE}-minimal.tar.gz --strip-components=1 -C /opt/owasp-crs; \
    rm -f v${CRS_RELEASE}-minimal.tar.gz coreruleset-${CRS_RELEASE}-minimal.tar.gz.asc; \
    mv -v /opt/owasp-crs/crs-setup.conf.example /opt/owasp-crs/crs-setup.conf

FROM image

ARG MODSEC2_VERSION
ARG LUA_VERSION
ARG LUA_MODULES

LABEL maintainer="Felipe Zipitria <felipe.zipitria@owasp.org>"

ENV \
    ACCESSLOG=/var/log/apache2/access.log \
    APACHE_ALWAYS_TLS_REDIRECT=off \
    APACHE_ERRORLOG_FORMAT='"[%{u}t] [%-m:%l] [pid %P:tid %T] %7F: %E: [client\ %a] %M% ,\ referer\ %{Referer}i"' \
    APACHE_LOGFORMAT='"%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\""' \
    APACHE_METRICS_LOGFORMAT='"%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-agent}i\""' \
    BACKEND=http://localhost:8080 \
    BACKEND_WS=ws://localhost:8081 \
    ERRORLOG='/proc/self/fd/2' \
    H2_PROTOCOLS='h2 http/1.1' \
    LOGLEVEL=warn \
    METRICS_ALLOW_FROM='127.0.0.0/255.0.0.0 ::1/128' \
    METRICS_DENY_FROM='All' \
    METRICSLOG='/dev/null' \
    MODSEC_ARGUMENT_SEPARATOR="&" \
    MODSEC_ARGUMENTS_LIMIT=1000 \
    MODSEC_AUDIT_ENGINE="RelevantOnly" \
    MODSEC_AUDIT_LOG=/dev/stdout \
    MODSEC_AUDIT_LOG_FORMAT=JSON \
    MODSEC_AUDIT_LOG_PARTS='ABIJDEFHZ' \
    MODSEC_AUDIT_LOG_RELEVANT_STATUS="^(?:5|4(?!04))" \
    MODSEC_AUDIT_LOG_TYPE=Serial \
    MODSEC_COOKIE_FORMAT=0 \
    MODSEC_AUDIT_STORAGE_DIR=/var/log/modsecurity/audit/ \
    MODSEC_DATA_DIR=/tmp/modsecurity/data \
    MODSEC_DEBUG_LOG=/dev/null \
    MODSEC_DEBUG_LOGLEVEL=0 \
    MODSEC_DEFAULT_PHASE1_ACTION="phase:1,pass,log,tag:'\${MODSEC_TAG}'" \
    MODSEC_DEFAULT_PHASE2_ACTION="phase:2,pass,log,tag:'\${MODSEC_TAG}'" \
    MODSEC_DISABLE_BACKEND_COMPRESSION="On" \
    MODSEC_PCRE_MATCH_LIMIT=100000 \
    MODSEC_PCRE_MATCH_LIMIT_RECURSION=100000 \
    MODSEC_REQ_BODY_ACCESS=on \
    MODSEC_REQ_BODY_JSON_DEPTH_LIMIT=512 \
    MODSEC_REQ_BODY_LIMIT=13107200 \
    MODSEC_REQ_BODY_LIMIT_ACTION="Reject" \
    MODSEC_REQ_BODY_NOFILES_LIMIT=131072 \
    MODSEC_RESP_BODY_ACCESS=on \
    MODSEC_RESP_BODY_LIMIT=1048576 \
    MODSEC_RESP_BODY_LIMIT_ACTION="ProcessPartial" \
    MODSEC_RESP_BODY_MIMETYPE="text/plain text/html text/xml" \
    MODSEC_RULE_ENGINE=on \
    MODSEC_SERVER_SIGNATURE="Apache" \
    MODSEC_STATUS_ENGINE="Off" \
    MODSEC_TAG=modsecurity \
    MODSEC_TMP_DIR=/tmp/modsecurity/tmp \
    MODSEC_TMP_SAVE_UPLOADED_FILES="on" \
    MODSEC_UNICODE_MAPPING=20127 \
    MODSEC_UPLOAD_DIR=/tmp/modsecurity/upload \
    MODSEC_UPLOAD_FILE_MODE=0600 \
    MODSEC_UPLOAD_KEEP_FILES=RelevantOnly \
    MUTEX='default' \
    PORT=8080 \
    PROXY_ERROR_OVERRIDE=on \
    PROXY_PRESERVE_HOST=on \
    PROXY_SSL_CA_CERT=/etc/ssl/certs/ca-certificates.crt \
    PROXY_SSL_CERT_KEY=/usr/local/apache2/conf/proxy.key \
    PROXY_SSL_CERT=/usr/local/apache2/conf/proxy.crt \
    PROXY_SSL_CHECK_PEER_NAME=off \
    PROXY_SSL_CIPHERS="ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384" \
    PROXY_SSL=off \
    PROXY_SSL_PROTOCOLS="all -SSLv3 -TLSv1 -TLSv1.1" \
    PROXY_SSL_VERIFY=none \
    PROXY_TIMEOUT=60 \
    REMOTEIP_INT_PROXY='10.1.0.0/16' \
    REQ_HEADER_FORWARDED_PROTO='https' \
    SERVER_ADMIN=root@localhost \
    SERVER_NAME=localhost \
    SERVER_SIGNATURE=Off \
    SERVER_TOKENS=Full \
    SSL_CERT_KEY=/usr/local/apache2/conf/server.key \
    SSL_CERT=/usr/local/apache2/conf/server.crt \
    SSL_CIPHERS="ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384" \
    SSL_ENGINE=on \
    SSL_HONOR_CIPHER_ORDER=off \
    SSL_OCSP_STAPLING=On \
    SSL_PORT=8443 \
    SSL_PROTOCOLS="all -SSLv3 -TLSv1 -TLSv1.1" \
    SSL_SESSION_TICKETS=off \
    TIMEOUT=60 \
    WORKER_CONNECTIONS=400 \
    # CRS specific variables
    ANOMALY_INBOUND=5 \
    ANOMALY_OUTBOUND=4 \
    BLOCKING_PARANOIA=1

COPY --from=build /usr/local/apache2/modules/mod_security2.so /usr/local/apache2/modules/mod_security2.so
COPY --from=build /usr/local/apache2/ModSecurity-${MODSEC2_VERSION}/unicode.mapping /etc/modsecurity.d/unicode.mapping
COPY --from=crs_release /opt/owasp-crs /opt/owasp-crs
COPY src/etc/modsecurity.d/*.conf /etc/modsecurity.d/
COPY src/bin/* /usr/local/bin/
COPY apache/conf/extra/*.conf /usr/local/apache2/conf/extra/
COPY src/etc/modsecurity.d/*.conf /etc/modsecurity.d/
COPY src/opt/modsecurity/* /opt/modsecurity/
COPY apache/docker-entrypoint.sh /

RUN set -eux; \
    echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections; \
    apt-get update -qq; \
    apt-get install -qq -y --no-install-recommends --no-install-suggests \
        ca-certificates \
        curl \
        ed \
        gnupg \
        iproute2 \
        libcurl3-gnutls \
        libfuzzy2 \
        liblua${LUA_VERSION} \
        ${LUA_MODULES} \
        libpcre2-8-0 \
        libxml2 \
        libyajl2; \
    update-ca-certificates -f; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

RUN useradd --system httpd

RUN set -eux; \
    mkdir -p /etc/modsecurity.d/; \
    mkdir -p /tmp/modsecurity/data; \
    mkdir -p /tmp/modsecurity/upload; \
    mkdir -p /tmp/modsecurity/tmp; \
    mkdir -p /var/log/apache2/; \
    ln -s /opt/owasp-crs /etc/modsecurity.d/; \
    sed -i -E 's|(Listen) [0-9]+|\1 ${PORT}|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|(ServerTokens) Full|\1 ${SERVER_TOKENS}|' /usr/local/apache2/conf/extra/httpd-default.conf; \
    sed -i -E 's|(ServerSignature) Off|\1 ${SERVER_SIGNATURE}|' /usr/local/apache2/conf/extra/httpd-default.conf; \
    sed -i -E 's|#(ServerName) www.example.com:80|\1 ${SERVER_NAME}|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|(ServerAdmin) you@example.com|\1 ${SERVER_ADMIN}|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|^(\s*CustomLog)(\s+\S+)+|\1 ${ACCESSLOG} modsec "env=!nologging"|g' /usr/local/apache2/conf/httpd.conf; \
	sed -i -E 's|^(\s*ErrorLog)\s+\S+|\1 ${ERRORLOG}|g' /usr/local/apache2/conf/httpd.conf; \
	sed -i -E 's|^(\s*TransferLog)\s+\S+|\1 ${ACCESSLOG}|g' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule unique_id_module)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule rewrite_module modules/mod_rewrite.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule proxy_module modules/mod_proxy.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule proxy_http_module modules/mod_proxy_http.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule remoteip_module modules/mod_remoteip.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule socache_shmcb_module modules/mod_socache_shmcb.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule ssl_module modules/mod_ssl.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(LoadModule http2_module modules/mod_http2.so)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(Include conf/extra/httpd-default.conf)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(Include conf/extra/httpd-proxy.conf)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(Include conf/extra/httpd-ssl.conf)|\1|' /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|#(Include conf/extra/httpd-vhosts.conf)|\1|' /usr/local/apache2/conf/httpd.conf; \
    echo 'Include conf/extra/httpd-locations.conf' >> /usr/local/apache2/conf/httpd.conf; \
    echo 'Include conf/extra/httpd-modsecurity.conf' >> /usr/local/apache2/conf/httpd.conf; \
    sed -i -E 's|(MaxRequestWorkers[ ]*)[0-9]*|\1${WORKER_CONNECTIONS}|' /usr/local/apache2/conf/extra/httpd-mpm.conf; \
    chown -R httpd:httpd \
        /var/log/ \
        /usr/local/apache2/ \
        /etc/modsecurity.d \
        /tmp/modsecurity \
        /opt/owasp-crs

USER httpd

ENTRYPOINT ["/docker-entrypoint.sh"]

HEALTHCHECK CMD /usr/local/bin/healthcheck

# Use httpd-foreground from upstream
CMD ["httpd-foreground"]
