FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y openssh-server sudo python3 iproute2 && \
    mkdir /var/run/sshd

ARG DEPLOY_PASSWORD
RUN useradd -m -s /bin/bash deploy && \
    echo "deploy:${DEPLOY_PASSWORD}" | chpasswd && \
    usermod -aG sudo deploy

RUN mkdir -p /home/deploy/.ssh && \
    chmod 700 /home/deploy/.ssh && \
    chown deploy:deploy /home/deploy/.ssh

COPY ansible_lab_key.pub /home/deploy/.ssh/authorized_keys
RUN chmod 600 /home/deploy/.ssh/authorized_keys && \
    chown deploy:deploy /home/deploy/.ssh/authorized_keys

RUN sed -i 's/#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#\?PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
