FROM ubuntu:22.04

# Installer SSH et sudo (nécessaires pour qu'Ansible puisse se connecter et devenir root)
RUN apt-get update && \
    apt-get install -y openssh-server sudo python3 iproute2 && \
    mkdir /var/run/sshd

# Créer un utilisateur "deploy" utilisé par Ansible pour se connecter
RUN useradd -m -s /bin/bash deploy && \
    echo "deploy:deploy" | chpasswd && \
    usermod -aG sudo deploy && \
    echo "deploy ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Autoriser la connexion par mot de passe (simplicité pour le lab ; en pro on utilise des clés)
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
