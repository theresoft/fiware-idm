FROM ubuntu:14.04

MAINTAINER FIWARE Indentity Manager Team. DIT-UPM

ENV HOME=/

# Install Ubuntu dependencies
RUN sudo apt-get update && \
    sudo apt-get install -y wget python git vim expect && \
    wget https://bootstrap.pypa.io/get-pip.py && \
    sudo python get-pip.py

# Install Ubuntu project dependencies
RUN sudo apt-get install -y python-dev python-virtualenv libssl-dev libffi-dev libjpeg8-dev libxml2-dev libxslt1-dev libsasl2-dev libssl-dev libldap2-dev libffi-dev libsqlite3-dev libmysqlclient-dev python-mysqldb

# Download latest version of the code 
RUN git clone https://github.com/theresoft/keystone.git
#RUN git clone https://github.com/ging/keystone && \
#    cd keystone && \
#    git checkout tags/keyrock-5.1.0

RUN git clone https://github.com/theresoft/horizon.git    
#RUN git clone https://github.com/ging/horizon && \
#    cd horizon && \
#    git checkout tags/keyrock-5.1.0

# Configuring settings files
RUN cp keystone/etc/keystone.conf.sample keystone/etc/keystone.conf && \
    cp horizon/openstack_dashboard/local/local_settings.py.example horizon/openstack_dashboard/local/local_settings.py && \
    sed -i s/\$\$IDM_PASS/idm/g horizon/openstack_dashboard/local/local_settings.py

# Install python dependecies
RUN sudo python keystone/tools/install_venv.py && \
    sudo python horizon/tools/install_venv.py

# Sync database
RUN sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync && \
    sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync --extension=endpoint_filter && \
    sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync --extension=oauth2 && \
    sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync --extension=roles && \
    sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync --extension=user_registration && \
    sudo keystone/tools/with_venv.sh keystone/bin/keystone-manage db_sync --extension=two_factor_auth

# Prepare to set up idm password
COPY expect_idm_password .
RUN chmod +x expect_idm_password && \
    sudo ./expect_idm_password

# Mount volumes to grant access from host
VOLUME /keystone/
VOLUME /horizon/

### Exposed ports
# - Keystone
EXPOSE 5000
# - Horizon (HTTP)
EXPOSE 8000

CMD sudo keystone/tools/with_venv.sh keystone/bin/keystone-all -v & sudo horizon/tools/with_venv.sh python horizon/manage.py runserver 0.0.0.0:8000
