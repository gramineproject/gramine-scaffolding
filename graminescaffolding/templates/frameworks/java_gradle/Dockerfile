{% extends 'Dockerfile' %}

{% block install %}
{{ super() }}
{{ apt_install(
    'gradle',
    'openjdk-17-jdk',
    'openjdk-17-jre-headless',
) }}
{% endblock %}

{% block build %}
RUN gradle build && \
    rm -Rf /app/src
{% endblock %}

{#- vim: set ft=jinja : #}
