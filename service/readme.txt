#copy the dog service script in /etc/init.d and give run permissions

cp ./dog /etc/init.d
chmod ug+rwx /etc/init.d/dog
chown root:root /etc/init.d/dog

#copy the dog default configuration in /etc/default

cp ./default/dog /etc/default
chmod ug+rwx /etc/default/dog
chown root:root /etc/default/dog

#unzip the Dog2.3 zip file in usr/local/lib
cd /usr/local/lib
mkdir Dog2.3
chown dog:users Dog2.3
cd Dog2.3
unzip Dog2.3.zip
chown -R dog:users *
chmod -R ug+rwx *

#copy the dog start script in /usr/local/bin
cp ./start_dog /usr/local/bin
chown dog:users /usr/dog/bin/start_dog
chmod ug+x /usr/local/bin/start_dog

#prepare the dog log directory
cd /var/log
mkdir dog
chown dog:users dog

#insert dog as a service
insserv dog

#to start the dog service type
service dog start

#to stop the dog service type
service dog stop

#to connect to the dog OSGi console type
telnet localhost 7777

#to disconnect from the dog OSGi console type
disconnect
