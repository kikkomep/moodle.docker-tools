FROM ubuntu:14.04

# Let the conatiner know that there is no tty
ENV DEBIAN_FRONTEND noninteractive

# Update
RUN apt-get update

# Install Apache/MySQL/PHP and additional Software
RUN apt-get update \
    && apt-get -y --force-yes install \
       apache2 mysql-client php5 git-core nodejs-legacy npm \
       graphviz aspell php5-pspell nano \
       php5-curl php5-gd php5-intl php5-mysql php5-xmlrpc php5-ldap clamav \
    && apt-get clean
	
# Add git info of the current Git Repository
COPY gitinfo.sh /usr/local/bin
RUN cat /usr/local/bin/gitinfo.sh >> ~/.bashrc

# update apache configuration
RUN echo "ServerName localhost" >>  /etc/apache2/apache2.conf
RUN echo "<Location \"/server-status\">\n\tSetHandler server-status\n\t#Require host example.com\n\tRequire all granted\n</Location>" >>  /etc/apache2/apache2.conf

# Install GRUNT plugins
RUN npm config set strict-ssl false && npm install -g grunt-cli

# Update the PHP.ini file, enable <? ?> tags and quieten logging.
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php5/apache2/php.ini

# Path of the apache WWW root
ENV WWW_ROOT /var/www/html

# enable HTTPS
RUN a2enmod ssl
RUN a2ensite default-ssl.conf

# copy certificates
COPY certs/apache.key /etc/apache2/ssl/apache.key
COPY certs/apache.crt /etc/apache2/ssl/apache.crt

### Moodle Download and Setting ###

# Moodle Env
ENV MOODLE_VERSION MOODLE_29_STABLE
ENV MOODLE_SRC "/opt/moodle"
ENV MOODLE_WWW_ROOT "${WWW_ROOT}/moodle"
ENV MOODLE_DATA_DIR "/var/www/moodledata"

# Copy Moodle source
COPY moodle ${MOODLE_SRC}

### Add scripts ###
ADD scripts/start-services.sh /usr/local/bin/start-services
ADD scripts/install-omero-modules.sh ${MOODLE_SRC}/install-omero-modules.sh
RUN chmod +x /usr/local/bin/start-services ${MOODLE_SRC}/install-omero-modules.sh

# Checkout the proper Moodle version and install omero plugins
RUN cd ${MOODLE_SRC} \
    && git checkout ${MOODLE_VERSION} \        
    && npm install grunt --save-dev \
    && npm install grunt-contrib-uglify --save-dev \
    && npm install grunt-contrib-jshint --save-dev \
    && npm install grunt-contrib-less --save-dev \
    && npm install grunt-contrib-watch --save-dev \
    && npm install shifter --save-dev \
    && ${MOODLE_SRC}/install-omero-modules.sh

# set user permissions
RUN usermod -u 1000 www-data

# Init Moodle dirs
RUN mkdir -p ${MOODLE_DATA_DIR}
RUN chown -R www-data ${MOODLE_DATA_DIR}
RUN chmod -R 777 ${MOODLE_DATA_DIR}


# set timezone
RUN echo Europe/Rome > /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata

# Set VOLUMEs to persist MOODLE instance and its data
VOLUME ${WWW_ROOT}
VOLUME ${MOODLE_DATA_DIR}

### PORTS to expose ###
EXPOSE 80 443

### ENTRYPOINT ##
CMD ["start-services"]
