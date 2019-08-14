FROM registry.access.redhat.com/rhel:7.4

ARG ERLANG_VERSION=20.1.1
ARG RABBITMQ_AUTOCLUSTER_VERSION=0.10.0
ARG RABBITMQ_VERSION=3.6.15

LABEL io.k8s.description="Lightweight open source message broker" \
    io.k8s.display-name="RabbitMQ" \
    io.openshift.expose-services="4369:epmd, 5671:amqp, 5672:amqp, 25672:http" \
    io.openshift.tags="rabbitmq"
    
ENV HOME=/var/lib/rabbitmq \
    RABBITMQ_HOME=/opt/rabbitmq \
    RABBITMQ_LOGS=- \
    RABBITMQ_SASL_LOGS=-

RUN set -xe
#RUN curl -LO https://github.com/rabbitmq/erlang-rpm/releases/download/v${ERLANG_VERSION}/erlang-${ERLANG_VERSION}-1.el7.centos.x86_64.rpm
#RUN rpm -Uvh ./erlang-${ERLANG_VERSION}-1.el7.centos.x86_64.rpm
#RUN rm *.rpm

COPY erlang-20.3.8.14-1.el7.centos.x86_64.rpm erlang-20.3.8.14-1.el7.centos.x86_64.rpm
RUN rpm -Uvh ./erlang-20.3.8.14-1.el7.centos.x86_64.rpm
RUN rm *.rpm

COPY rabbitmq-server-3.7.14-1.el7.noarch.rpm rabbitmq-server-3.7.14-1.el7.noarch.rpm
RUN rpm -Uvh ./rabbitmq-server-3.7.14-1.el7.noarch.rpm
RUN rm *.rpm

COPY hostname-3.13-3.el7.x86_64.rpm hostname-3.13-3.el7.x86_64.rpm
RUN rpm -Uvh ./hostname-3.13-3.el7.x86_64.rpm
RUN rm *.rpm

# RUN curl -Lo rabbitmq-server.tar.xz https://www.rabbitmq.com/releases/rabbitmq-server/v${RABBITMQ_VERSION}/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz
# RUN curl -Lo rabbitmq-server.tar.xz.asc https://www.rabbitmq.com/releases/rabbitmq-server/v${RABBITMQ_VERSION}/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz.asc
COPY rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz rabbitmq-server.tar.xz
COPY rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz.asc rabbitmq-server.tar.xz.asc
RUN curl -Lo autocluster.ez https://github.com/rabbitmq/rabbitmq-autocluster/releases/download/${RABBITMQ_AUTOCLUSTER_VERSION}/autocluster-${RABBITMQ_AUTOCLUSTER_VERSION}.ez
RUN curl -Lo rabbitmq_aws.ez https://github.com/rabbitmq/rabbitmq-autocluster/releases/download/${RABBITMQ_AUTOCLUSTER_VERSION}/rabbitmq_aws-${RABBITMQ_AUTOCLUSTER_VERSION}.ez
RUN mkdir -p "$RABBITMQ_HOME"
RUN tar --extract --verbose --file rabbitmq-server.tar.xz --directory "$RABBITMQ_HOME" --strip-components 1
RUN rm rabbitmq-server.tar.xz*
RUN grep -qE '^SYS_PREFIX=\$\{RABBITMQ_HOME\}$' "$RABBITMQ_HOME/sbin/rabbitmq-defaults"
RUN sed -ri 's!^(SYS_PREFIX=).*$!\1!' "$RABBITMQ_HOME/sbin/rabbitmq-defaults"
RUN grep -qE '^SYS_PREFIX=$' "$RABBITMQ_HOME/sbin/rabbitmq-defaults"
RUN groupadd --system rabbitmq
RUN adduser -u 1001 -r -c "RabbitMQ User" -d /var/lib/rabbitmq -g rabbitmq rabbitmq
RUN mkdir -p /var/lib/rabbitmq /etc/rabbitmq
RUN mv *.ez ${RABBITMQ_HOME}/plugins
RUN ${RABBITMQ_HOME}/sbin/rabbitmq-plugins enable autocluster --offline
RUN ${RABBITMQ_HOME}/sbin/rabbitmq-plugins enable rabbitmq_management --offline
RUN chown -R 1001:0 /var/lib/rabbitmq /etc/rabbitmq ${RABBITMQ_HOME}/plugins
RUN chmod -R g=u /var/lib/rabbitmq /etc/rabbitmq
RUN rm -rf /var/lib/rabbitmq/.erlang.cookie
RUN ln -sf "$RABBITMQ_HOME/plugins" /plugins

COPY docker-entrypoint.sh /usr/local/bin/
USER 1001
ENV PATH=$RABBITMQ_HOME/sbin:$PATH
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 4369 5671 5672 25672 15672
CMD ["rabbitmq-server"]
