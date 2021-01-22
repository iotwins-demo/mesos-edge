FROM httpd:2.4-alpine

LABEL maintainer="marica.antonacci@ba.infn.it"

RUN apk update && \
    apk add --no-cache -X http://dl-cdn.alpinelinux.org/alpine/edge/main hiredis && \
    apk add --no-cache -X http://dl-cdn.alpinelinux.org/alpine/edge/testing apache-mod-auth-openidc

RUN cp /usr/lib/apache2/mod_auth_openidc.so /usr/local/apache2/modules/ && \
    sed -i -e '$aInclude conf/extra/httpd-openidc.conf' /usr/local/apache2/conf/httpd.conf

