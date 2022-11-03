# Dockerfile for bitnami/fluentd sidecar image with all the necessary plugins for our log analytic providers
FROM bitnami/fluentd:latest
LABEL maintainer="Partner Engineering <partner_support@jfrog.com>"

## Build time Arguments, short circuit them to ENV Variables so they are available at run time also
ARG SOURCE=JFRT
ARG TARGET=NEWRELIC

## Environment Variables set by this docker file, there will be seperate env params set by a env file while running the containers
## For better maintainability always depend dockerfile code on the environment variables declared in this file to add more platforms
ENV SRC_PLATFORM=$SOURCE
ENV TGT_PLATFORM=$TARGET

USER root

## Install JFrog Plugins
RUN fluent-gem install fluent-plugin-jfrog-siem
RUN fluent-gem install fluent-plugin-jfrog-metrics
RUN fluent-gem install fluent-plugin-jfrog-send-metrics

# Install prerequisites
RUN apt-get update && apt-get install -y \
curl
CMD /bin/bash


## Install custom Fluentd plugins
RUN if [ "$TGT_PLATFORM" = "SPLUNK" ] ; then echo "Downloading the fluentd plugin for $TGT_PLATFORM "; fluent-gem install fluent-plugin-splunk-hec; else echo "Not Downloading"; fi
RUN if [ "$TGT_PLATFORM" = "DATADOG" ] ; then echo "Downloading the fluentd plugin for $TGT_PLATFORM "; fluent-gem install fluent-plugin-datadog; else echo "Not Downloading"; fi
RUN if [ "$TGT_PLATFORM" = "ELASTIC" ] ; then echo "Downloading the fluentd plugin for $TGT_PLATFORM "; fluent-gem install fluent-plugin-elasticsearch; else echo "Not Downloading"; fi
RUN if [ "$TGT_PLATFORM" = "NEWRELIC" ] ; then echo "Downloading the fluentd plugin for $TGT_PLATFORM "; fluent-gem install fluent-plugin-newrelic; else echo "Not Downloading"; fi

## Download Config Files
RUN if [ "$SRC_PLATFORM" = "JFRT" ] ; then echo "Downloading the fluentd config file for $SRC_PLATFORM and $TGT_PLATFORM "; curl https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/InitialSetup/fluent.conf.rt -o /opt/bitnami/fluentd/conf/fluentd.conf; else echo "Not Downloading"; fi
RUN if [ "$SRC_PLATFORM" = "JFXRAY" ] ; then echo "Downloading the fluentd config file for $SRC_PLATFORM and $TGT_PLATFORM "; curl https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/InitialSetup/fluent.conf.xray -o /opt/bitnami/fluentd/conf/fluentd.conf; else echo "Not Downloading"; fi

ENTRYPOINT if [ "$TGT_PLATFORM" = "SPLUNK" ] ; then  sed -i -e "s/HEC_HOST/$HEC_HOST/g" \
                                             -e "s/HEC_PORT/$HEC_PORT/g" \
                                             -e "s/METRICS_HEC_TOKEN/$METRICS_HEC_TOKEN/" \
                                             -e "s/HEC_TOKEN/$HEC_TOKEN/" \
                                             -e "s/COM_PROTOCOL/$COM_PROTOCOL/g" \
                                             -e "s/INSECURE_SSL/$INSECURE_SSL/g" \
                                             -e "s/JPD_URL/$JPD_URL/" \
                                             -e "s/ADMIN_USERNAME/$JPD_USER_NAME/" \
                                             -e "s/API_KEY/$JPD_API_KEY/" /opt/bitnami/fluentd/conf/fluentd.conf && fluentd -v -c /opt/bitnami/fluentd/conf/fluentd.conf; fi

ENTRYPOINT if [ "$TGT_PLATFORM" = "NEWRELIC" ] ; then  sed -i -e "s/JPD_URL/$JPD_URL/g" \
                                             -e "s/ADMIN_USERNAME/$ADMIN_USERNAME/g" \
                                             -e "s/JFROG_API_KEY/$JFROG_API_KEY/" \
                                             -e "s/JFROG_ACCESS_TOKEN/$JFROG_ACCESS_TOKEN/" \
                                             -e "s/COMMON_JPD/$COMMON_JPD/g" \
                                             -e "s/LICENSE_KEY/$LICENSE_KEY/g" \
                                              /opt/bitnami/fluentd/conf/fluentd.conf && cat /opt/bitnami/fluentd/conf/fluentd.conf; fluentd -v -c /opt/bitnami/fluentd/conf/fluentd.conf; fi


USER 1001

STOPSIGNAL SIGTERM