FROM jenkinsdev

# root user for Jenkins, need to get access to /var/run/docker.sock (fix this in the future!)
USER root

# Environment
ENV HOME /root
ENV JENKINS_HOME /root/jenkins
#ENV JENKINS_VERSION 2.174

# GitHub repository to store Jenkins configuration
ENV GITHUB_USERNAME iteabit
ENV GITHUB_CONFIG_REPOSITORY demo1-jenkins-config

# Make Jenkins home directory
RUN mkdir -p $JENKINS_HOME

# Install Jenkins plugins
RUN /usr/local/bin/install-plugins.sh \
    scm-sync-configuration:0.0.10 \
    workflow-aggregator:2.6 \
    docker-workflow:1.18

# Set timezone
RUN echo "America/Los_Angeles" > /etc/timezone &&\
    dpkg-reconfigure --frontend noninteractive tzdata &&\
    date

# Copy RSA keys for Jenkins config repository (default keys).
# This public key should be added to:
# https://github.com/iteabit/demo1-jenkins-config/settings/keys
COPY keys/jenkins.config.id_rsa     $HOME/.ssh/id_rsa
COPY keys/jenkins.config.id_rsa.pub $HOME/.ssh/id_rsa.pub
RUN chmod 600 $HOME/.ssh/id_rsa &&\
    chmod 600 $HOME/.ssh/id_rsa.pub
RUN echo "    IdentityFile $HOME/.ssh/id_rsa" >> /etc/ssh/ssh_config &&\
    echo "    StrictHostKeyChecking no      " >> /etc/ssh/ssh_config
RUN /bin/bash -c "eval '$(ssh-agent -s)'; ssh-add $HOME/.ssh/id_rsa;"

# Copy RSA keys for your application repository and add
# host 'github.com-application-jenkins' for application code pulls.
# This public key should be added to
# https://github.com/iteabit/demo-go-app/settings/keys
COPY keys/jenkins.application.id_rsa     $HOME/.ssh/jenkins.application.id_rsa
COPY keys/jenkins.application.id_rsa.pub $HOME/.ssh/jenkins.application.id_rsa.pub
RUN chmod 600 $HOME/.ssh/jenkins.application.id_rsa &&\
    chmod 600 $HOME/.ssh/jenkins.application.id_rsa.pub
RUN touch $HOME/.ssh/config &&\
    echo "Host github.com-application-jenkins                     " >> $HOME/.ssh/config &&\
    echo "    HostName       github.com                           " >> $HOME/.ssh/config &&\
    echo "    User           git                                  " >> $HOME/.ssh/config &&\
    echo "    IdentityFile   $HOME/.ssh/jenkins.application.id_rsa" >> $HOME/.ssh/config &&\
    echo "    IdentitiesOnly yes                                  " >> $HOME/.ssh/config

# Configure git
RUN git config --global user.email "jenkins@container" &&\
    git config --global user.name  "jenkins"

# Clone Jenkins config
RUN cd /tmp &&\
    git clone git@github.com:$GITHUB_USERNAME/$GITHUB_CONFIG_REPOSITORY.git &&\
    cp -r $GITHUB_CONFIG_REPOSITORY/. $JENKINS_HOME &&\
    rm -r /tmp/$GITHUB_CONFIG_REPOSITORY

# Jenkins workspace for sharing between containers
VOLUME $JENKINS_HOME/workspace

# Run init.sh script after container start
COPY src/init.sh /usr/local/bin/init.sh
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/init.sh"]
