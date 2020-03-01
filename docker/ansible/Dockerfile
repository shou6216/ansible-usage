FROM ubuntu

ENV ANSIBLE_HOST_KEY_CHECKING False

RUN apt-get update && \
    apt-get install -y software-properties-common && \
    apt-add-repository ppa:ansible/ansible && \
    apt-get update && \
    apt-get install -y ansible vim

CMD /bin/bash

