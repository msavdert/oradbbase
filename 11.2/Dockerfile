FROM melihsavdert/oel7-systemd

MAINTAINER Melih Savdert

# Passwords for grid and oracle users
ENV ["GRID_PASSWORD", "oracle"]
ENV ["ORACLE_PASSWORD", "oracle"]
ENV ["ROOT_PASSWORD", "oracle"]

ENV ORA_MOUNT_PATH /u01
ENV ORA_ORACLE_BASE /u01/app/oracle
ENV ORA_ORACLE_HOME /u01/app/oracle/product/11.2.0.1/dbhome_1
ENV GRID_ORACLE_BASE /u01/app/grid
ENV GRID_ORACLE_HOME /u01/app/11.2.0.1/grid
ENV ORAINVENTORY /u01/app/oraInventory
ENV TZ=Europe/Istanbul

# Update the operating system
RUN ["yum", "-y", "update"]

RUN yum-config-manager \
    --add-repo \
    http://public-yum.oracle.com/public-yum-ol7.repo

# Install necessary packages
RUN ["yum", "-y", "install", \
       "oracle-rdbms-server-11gR2-preinstall", \
       "vim", \
       "net-tools", \
       "wget", \
       "ntp"]

# Clean the yum cache
RUN ["yum", "clean", "all"]

##  Users and Groups
RUN userdel -r oracle
RUN groupdel oinstall
RUN groupdel dba
RUN groupadd -g 54421 oinstall
RUN groupadd -g 54329 asmadmin
RUN groupadd -g 54327 asmdba
RUN groupadd -g 54328 asmoper
RUN groupadd -g 54322 dba
RUN groupadd -g 54323 oper
RUN groupadd -g 54324 backupdba
RUN groupadd -g 54325 dgdba
RUN groupadd -g 54326 kmdba
RUN groupadd -g 54330 racdba
RUN useradd -u 54321 -g oinstall -G dba,asmdba,backupdba,dgdba,kmdba,racdba,oper oracle
RUN useradd -u 54322 -g oinstall -G asmadmin,asmdba,racdba,asmoper,dba grid

# Give root, grid and oracle users passwords
RUN echo "grid:${GRID_PASSWORD}" | chpasswd
RUN echo "oracle:${ORACLE_PASSWORD}" | chpasswd
RUN echo "oracle:${ROOT_PASSWORD}" | chpasswd

# Add ulimits configuration file for grid user
# oracle user ulimits configuration file already added by oracle-rdbms-server-11gR2-preinstall
RUN echo -e "grid soft nproc 131072\n\
grid hard nproc 131072\n\
grid soft nofile 131072\n\
grid hard nofile 131072\n\
grid soft stack 10240\n\
grid soft core unlimited" >> /etc/security/limits.conf

RUN echo -e "if [ \$USER = \"oracle\" ] || [ \$USER = \"grid\" ]; then\n\
    if [ \$SHELL = \"/bin/ksh\" ]; then\n\
        ulimit -p 16384\n\
        ulimit -n 65536\n\
    else\n\
        ulimit -u 16384 -n 65536\n\
    fi\n\
    umask 022\n\
fi" >> /etc/profile

##  SSH Shared Keys
# Create SSH shared key directory for the oracle user
RUN ["mkdir", "-p", "-m", "0700", "/home/oracle/.ssh/"]

# Generate SSH shared keys for the oracle user
RUN ssh-keygen -q -C '' -N '' -f /home/oracle/.ssh/id_rsa

# Create the authorized_keys file for the oracle user
RUN cat /home/oracle/.ssh/id_rsa.pub > /home/oracle/.ssh/authorized_keys

# Change ownership of the SSH shared key files for the oracle user
RUN chown -R oracle:oinstall /home/oracle/.ssh

# Change permissions of the authorized_keys file for the oracle user
RUN ["chmod", "0640", "/home/oracle/.ssh/authorized_keys"]

# Create SSH shared key directory for the grid user
RUN ["mkdir", "-p", "-m", "0700", "/home/grid/.ssh/"]

# Generate SSH shared keys for the grid user
RUN ssh-keygen -q -C '' -N '' -f /home/grid/.ssh/id_rsa

# Create the authorized_keys file for the grid user
RUN cat /home/grid/.ssh/id_rsa.pub > /home/grid/.ssh/authorized_keys

# Change ownership of the SSH shared key files for the grid user
RUN chown -R grid:oinstall /home/grid/.ssh

# Change permissions of the authorized_keys file for the grid user
RUN ["chmod", "0640", "/home/grid/.ssh/authorized_keys"]

# Generate SSH host ECDSA shared keys
RUN ssh-keygen -q -C '' -N '' -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key

# Create the ssh_known_hosts file
#RUN for NODE in rac1 rac2; do (echo -n "$NODE " && cat /etc/ssh/ssh_host_ecdsa_key.pub) >> /etc/ssh/ssh_known_hosts; done

## .bash_profile ###
RUN echo -e "#this is for oracle install#\n\
if [ -t 0 ]; then\n\
        stty intr ^C\n\
fi" >> /home/oracle/.bashrc && \
echo -e "#this is for oracle install#\n\
if [ -t 0 ]; then\n\
        stty intr ^C\n\
fi" >> /home/grid/.bashrc
## .bash_profile ###
RUN echo -e "### for oracle install ####\n\
export ORACLE_BASE=${ORA_ORACLE_BASE}\n\
export GRID_BASE=${GRID_ORACLE_BASE}\n\
export ORACLE_HOME=${ORA_ORACLE_HOME}\n\
export ORACLE_HOSTNAME=\`hostname\`\n\
" >> /home/oracle/.bash_profile
RUN echo -e "export TMPDIR=/tmp\n\
export TEMP=/tmp\n\
export PATH=\$ORACLE_HOME/bin:\$ORACLE_HOME/jdk/bin:\${PATH}\n\
export LD_LIBRARY_PATH=\$ORACLE_HOME/lib\n\
alias cdob='cd \$ORACLE_BASE'\n\
alias cdoh='cd \$ORACLE_HOME'\n\
alias tns='cd \$ORACLE_HOME/network/admin'\n\
alias envo='env | grep ORACLE'\n\
alias sqla='sqlplus / as sysdba'\n\
alias psf='ps -fe | grep pmon'\n\
alias taila='tail -200f $ORACLE_BASE/diag/rdbms/\`hostname\`/\${ORACLE_SID}/trace/alert_\${ORACLE_SID}.log'\n\
" >> /home/oracle/.bash_profile
RUN echo -e "### for oracle install ####\n\
export ORACLE_BASE=${GRID_ORACLE_BASE}\n\
export ORACLE_HOME=${GRID_ORACLE_HOME}\n\
export ORACLE_HOSTNAME=\`hostname\`\n\
" >> /home/grid/.bash_profile
RUN echo -e "export TMPDIR=/tmp\n\
export TEMP=/tmp\n\
export PATH=\$ORACLE_HOME/bin:\$ORACLE_HOME/jdk/bin:\${PATH}\n\
export LD_LIBRARY_PATH=\$ORACLE_HOME/lib\n\
alias cdob='cd \$ORACLE_BASE'\n\
alias cdoh='cd \$ORACLE_HOME'\n\
alias tns='cd \$ORACLE_HOME/network/admin'\n\
alias envo='env | grep ORACLE'\n\
alias sqla='sqlplus / as sysdba'\n\
alias psf='ps -fe | grep pmon'\n\
alias taila='tail -200f /u01/app/oracle/diag/rdbms/\`hostname\`/\${ORACLE_SID}/trace/alert_\${ORACLE_SID}.log'\n\
" >> /home/grid/.bash_profile

## create oraclehome
RUN  mkdir ${ORA_MOUNT_PATH} && \
     mkdir -p ${GRID_ORACLE_BASE} && \
     mkdir -p ${GRID_ORACLE_HOME} && \
     chown -R grid:oinstall ${ORA_MOUNT_PATH} && \
     mkdir -p ${ORA_ORACLE_BASE} && \
     chown oracle:oinstall ${ORA_ORACLE_BASE} && \
     chmod -R 775 ${ORA_MOUNT_PATH}

RUN mkdir -p /u01/software
RUN chown -R oracle:oinstall /u01/software
RUN chmod -R g+w /u01/software/

# oraInst.loc
RUN echo -e "inventory_loc=/u01/app/oraInventory\n\
inst_group=oinstall" >> /etc/oraInst.loc
RUN chown oracle:oinstall /etc/oraInst.loc

# Add the following line to the "/etc/pam.d/login"
RUN echo -e "session    required     pam_limits.so" >> /etc/pam.d/login

# Allow non-privileged users the ability to execute the ping command
RUN ["chmod", "4755", "/bin/ping"]

# SELinux bug fix
RUN mkdir -p /etc/selinux/targeted/contexts/
RUN echo -e "<busconfig><selinux></selinux></busconfig>" >> /etc/selinux/targeted/contexts/dbus_contexts

# Hide/disable the ttyS0 serial console service
RUN ["systemctl", "mask", "serial-getty@ttyS0.service"]

RUN systemctl disable firewalld
RUN systemctl enable sshd
RUN systemctl enable ntpd
RUN rm -rf /run/nologin

EXPOSE 1521 1158 5500 7803 7102 9803

CMD ["/usr/sbin/init"]
