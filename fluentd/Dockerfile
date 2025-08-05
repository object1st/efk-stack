FROM fluent/fluentd:v1.16-debian-1

USER root

# Install plugins
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "5.3.0"]
RUN ["gem", "install", "fluent-plugin-record-modifier", "--no-document"]

USER fluent
