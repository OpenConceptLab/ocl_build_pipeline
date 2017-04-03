FROM gocd/gocd-agent:17.2.0

MAINTAINER hevalberknevruz@gmail.com

RUN \
   apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927 && \
   echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list && \
   apt update && \
   apt install -y mongodb-org=3.2.11 \
   build-essential \
   python-dev \
   python-pip \
   git \
   libssl-dev \
   libffi-dev \
   wget \
   libpq-dev \
   postgresql \
   postgresql-contrib \
   redis-server && \
   wget -qO- https://deb.nodesource.com/setup_7.x | sudo bash - && \
   apt install -y nodejs  \
   mongodb && \
   pip install virtualenv && \
   /etc/init.d/postgresql start && \
   mkdir -p /data/db && \
   sudo -u postgres createuser go && \
   sudo -u postgres psql -c 'alter user go with createdb' postgres && \
   echo "go ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
