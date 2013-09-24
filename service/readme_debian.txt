#--------------- Creating the system user with which dog will be executed ------------------------------#

useradd dog

# Dog files copy
# Move Dog standalone folder in /usr/local/lib/ 

mv Dog2.x  /usr/local/lib/Dog2.x

# check for the folder to have right permissions

chown -R dog: /usr/local/lib/Dog2.2

#--------------- Creation of files needed to start dog as a service --------------------------------#
#
#  Examples in the service folder...
#
# Create /usr/local/bin/start_dog with the following content
#
#----------------------------------------------------------------------------------------------------

    #!/bin/bash

    JDK_DIRS="/usr/lib/jvm/java-6-openjdk /usr/lib/jvm/java-6-sun /usr/lib/jvm/java-1.5.0-sun /usr/lib/j2sdk1.5-sun /usr/lib/j2sdk1.5-ibm"

    # Look for the right JVM to use
    for jdir in $JDK_DIRS; do
        if [ -r "$jdir/bin/java" -a -z "${JAVA_HOME}" ]; then
            JAVA_HOME="$jdir"
        fi
    done
    export JAVA_HOME

    OSGI_CONSOLE_PORT="$1"
    OSGI_JAR_LOCATION="$2"
    DOG_CONFIG_FOLDER="$3"
    LOG_FILE_LOCATION="$4"

    JAVA_OPTS="-DforceSimpleModel=true -DconfigFolder=$DOG_CONFIG_FOLDER"

    #$DOG_HOME/org.eclipse.osgi_3.6.2.R36x_v20110210.jar
    exec $JAVA_HOME/bin/java $JAVA_OPTS -jar $OSGI_JAR_LOCATION -console $OSGI_CONSOLE_PORT -clean -Xmx512M &>> $4

#-------------------------------------------------------------------------------

# Give execution privileges to the file

chmod +x /usr/local/bin/start_dog



#Create /etc/init.d/dog with the following content

#-------------------------------------------------------------------------------

    #!/bin/sh
    #
    # /etc/init.d/dog -- startup script for the Domotic OSGI Gateway
    #
    ### BEGIN INIT INFO
    # Provides:          dog
    # Required-Start:    $local_fs $remote_fs $network
    # Required-Stop:     $local_fs $remote_fs $network
    # Should-Start:      $named
    # Should-Stop:       $named
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Start dog.
    # Description:       Start Domotic OSGI Gateway.
    ### END INIT INFO

    set -e

    if [ `id -u` -ne 0 ]; then
            echo "You need root privileges to run this script"
            exit 1
    fi

    NAME=dog
    DESC="Domotic OSGI Gateway"
    DAEMON=/usr/local/bin/start_dog
    DOG_PID=/var/run/$NAME.pid
    DEFAULT=/etc/default/$NAME

    # The following variables can be overwritten in $DEFAULT
    DOG_USER=dog
    DOG_GROUP=dog
    OSGI_CONSOLE_PORT=7777
    DOG_HOME=/usr/local/lib/Dog2.2/
    DOG_CONFIG_FOLDER="$DOG_HOME/configuration/KnxIPDemoBox"
    OSGI_JAR_LOCATION="$DOG_HOME/org.eclipse.osgi_3.6.2.R36x_v20110210.jar"
    LOG_FILE_LOCATION="/var/log/dog/dog.log"

    # overwrite settings from default file
    if [ -f "$DEFAULT" ]; then
            . "$DEFAULT"
    fi


    . /lib/lsb/init-functions


    dog_sh() {
            start-stop-daemon -b --start -c $DOG_USER -u $DOG_USER -g $DOG_GROUP -m --pidfile /var/run/$NAME.pid -x /bin/bash -- -c \
                    "$DAEMON $OSGI_CONSOLE_PORT $OSGI_JAR_LOCATION $DOG_CONFIG_FOLDER $LOG_FILE_LOCATION" > /dev/null
            log_end_msg $?
    }

    case "$1" in
      start)
            log_daemon_msg "Starting $DESC" "$NAME"

            if start-stop-daemon --test --start --pidfile "$DOG_PID" -x /usr/lib/jvm/java-6-sun/bin/java >/dev/null; then
                    dog_sh
                    sleep 5
                    if start-stop-daemon --test --start --pidfile "$DOG_PID" -x /usr/lib/jvm/java-6-sun/bin/java >/dev/null; then
                            if [ -f "$DOG_PID" ]; then
                                    rm -f "$DOG_PID"
                            fi
                            log_end_msg 1
                    else
                            log_end_msg 0
                    fi
            else
                    log_progress_msg "(already running)"
                    log_end_msg 0
            fi

            ;;
      stop)
            log_daemon_msg "Stopping $DESC" "$NAME"

            set +e
            if [ -f "$DOG_PID" ]; then
                    start-stop-daemon --stop --pidfile "$DOG_PID" --user "$DOG_USER" --retry=TERM/20/KILL/5 >/dev/null
                    if [ $? -eq 1 ]; then
                            log_progress_msg "$DESC is not running but pid file exists, cleaning up"
                    elif [ $? -eq 3 ]; then
                            PID="`cat $DOG_PID`"
                            log_failure_msg "Failed to stop $NAME (pid $PID)"
                            exit 1
                    fi
                    rm -f "$DOG_PID"
            else
                    log_progress_msg "(not running)"
            fi
            log_end_msg 0
            set -e

            ;;
      status)
            set +e
            start-stop-daemon --test --start --pidfile "$DOG_PID" -x /usr/lib/jvm/java-6-sun/bin/java >/dev/null;
            if [ "$?" = "0" ]; then

                    if [ -f "$DOG_PID" ]; then
                        log_success_msg "$DESC is not running, but pid file exists."
                            exit 1
                    else
                        log_success_msg "$DESC is not running."
                            exit 3
                    fi
            else
                    log_success_msg "$DESC is running with pid `cat $DOG_PID`"
            fi
            set -e
            ;;


      restart)
            if [ -f "$DOG_PID" ]; then
                    $0 stop
                    sleep 1
            fi
            $0 start
            ;;
      *)
            log_success_msg "Usage: $0 {start|stop|restart|status}"
            exit 1
            ;;
    esac

    exit 0
#---------------------------------------------------------------------------------------------
# Give execution privileges to the file

chmod +x /etc/init.d/dog

# Create /etc/default/dog  with the following content

---------------------------------------------------------------------------------------------

# Dog configuration, these variables overrides declarations in /etc/init.d/dog

DOG_USER=dog
DOG_GROUP=dog
#leave this empty if you don't want osgi console to listen to localhost:$OSGI_CONSOLE_PORT
OSGI_CONSOLE_PORT=7777
DOG_HOME=/usr/local/lib/Dog2.2/
DOG_CONFIG_FOLDER="$DOG_HOME/configuration/KnxIPDemoBox"
OSGI_JAR_LOCATION="$DOG_HOME/org.eclipse.osgi_3.6.2.R36x_v20110210.jar"
LOG_FILE_LOCATION="/var/log/dog/dog.log"

#----------------------------------------------------------------------------------------------

#This last file is the one to change when modifying the Dog configuration.

#Particular attention shall be devoted to the OSGI_JAR_LOCATION and DOG_CONFIG_FOLDER values.


#------------------------------ Create the log destination folder ------------------------------------#

    mkdir /var/log/dog
    chown -R dog: /var/log/dog


#------------------------------ Installation check ---------------------------------------------------#

#service start

/etc/init.d/dog start

#service status

/etc/init.d/dog status

#service stop

/etc/init.d/dog stop

#service restart

/etc/init.d/dog restart

# The OSGi console is exported to the 7777 telnet port

telnet localhost 7777

# The 7777 port number shall be chenged according to the one configured in /etc/default/dog

# Dog logs will be available at /var/log/dog/dog.log (or in the place configured through /etc/default/dog)

# to exit from the OSGi console type

disconnect

#if you type

exit

#the OSGi console will be stopped and the same will happen for dog

#Startup at boot time

# type
insserv dog

