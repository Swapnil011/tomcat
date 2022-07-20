#    	<u>HOW TO SETUP TOMCAT ON CENTOS 7</u>



##  **What is tomcat ?**
It is an open-source Java servlet container that implements many Java Enterprise Specs such as the Websites API, Java-Server Pages and last but not least, the Java Servlet.

For tomcat setup we need java installed on our server.



## **1. Install java JRE ** 

#### **1.1 To Check latest java version**
```
sudo yum search openjdk
```
#### **1.2 To install latest java version 8**
```
sudo yum install java-1.8.0-openjdk*
```
#### **1.3 After installation check java version**

```
java -version
```

## **2. Download tomcat for centos 7**

#### **2.1 Check tomcat latest version on given site **

https://downloads.apache.org/tomcat/tomcat-9/

Go on above website check latest version of tomcat 9 and find core tar.gz right click on it and copy link address like follows and download it by commands
```
cd /opt

wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.64/bin/apache-tomcat-9.0.64.tar.gz
```
#### **2.2 Extract tar file**

```
tar -xf apache-tomcat-9.0.64.tar.gz    
```

#### **2.3 Rename file name apache-tomcat-9.0.27 to tomcat**
```
mv apache-tomcat-9.0.64 tomcat
```

## 3. Change webapps folder location 

By default `webapps` folder is in `/opt/tomcat/` . We move it to `/var/www/tomcat/`

```
mkdir /var/www/tomcat

cd /opt/tomcat

mv webapps/ /var/www/tomcat/
```

## 4. Softlink to logs folder 

By default the `logs` folder is in `/opt/tomcat/`.  We softlink it to `/var/logs/`.

```
mkdir /var/log/tomcat

cd /opt/tomcat

rm -rf logs/

ln -s /var/log/tomcat logs
```

## 5. Change port of tomcat 

By default the port is `8080`. We need to make changes in the `server.xml` file to change tomcat port. 
```
cd /opt/tomcat/conf/

vi server.xml
```
We have to edit the following section. This is original code in `server.xml` file with port  `8080` and redirecting port `8443`
```
<Connector port="8080" protocol="HTTP/1.1"
	              connectionTimeout="20000"
	              redirectPort="8443" />
```
This is edited code in `server.xml` file with change port `443` and redirecting port `8787`

```
<Connector port="443" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8787" />
```

## 6. Generate SSL Certificates using openSSL

Following command will generate `key` and `crt` this 2 files
```
mkdir /opt/tomcat/keys

cd /opt/tomcat/keys

openssl req -newkey 2048 -nodes -keyout tomcat.key -x509 -days 365 -out tomcat.crt
```
We have to export those 2 file in `p12` file using following command
```
openssl pkcs12 -inkey tomcat.key -in tomcat.crt -export -out tomcat.p12

set password = pass@123
```
Now add details of certification  in `server.xml` 

## 7. Changes in `server.xml` 

#### **7.1 Change webapps folder path **

By default `webapps` folder path is `/opt/tomcat` but we move it to `/var/www/tomcat`

```
cd /opt/tomcat/conf/

vi server.xml
```

We have to edit the following section. This is original code in `server.xml` file with appBase  `webapps` 
```
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
```
his is edited code in `server.xml` file with change appBase `/var/www/tomcat`
```
<Host name="localhost"  appBase="/var/www/tomcat/webapps/"
            unpackWARs="true" autoDeploy="true">
```
####    **7.2 Add Certification details **

Now we have 3 files `tomcat.key , tomcat.crt , tomcat.p12` we need to add their entry in `server.xml`

We have to edit the following section. This is original code in `server.xml` file
```
<Connector port="443" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8787" />
```
This is edited code in `server.xml` file
```
<Connector
           protocol="org.apache.coyote.http11.Http11AprProtocol"
           port="443" maxThreads="200" redirectPort="8787"
           scheme="https" secure="true" SSLEnabled="true"
           SSLCertificateFile="/opt/tomcat/keys/tomcat.crt"
           SSLCertificateKeyFile="/opt/tomcat/keys/tomcat.key"
           SSLVerifyClient="optional" SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
           certificateKeystoreFile="/opt/tomcat/keys/tomcat.p12"
      	   certificateKeystorePassword="pass@123"	/>
```

## 8. Create tomcat user 
```
useradd tomcat
```

## 9. Create service file

#### **9.1 Create service file **
To add service file we have to create a file in `/etc/init.d` file name as `tomcat` 
```
vi /etc/init.d/tomcat
```
#### **9.2 Add following code tomcat service file **

```
#export CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9000 -Dcom.sun.management.jmxremote.rmi.port=9000 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.local.only=false -Djava.rmi.server.hostname=172.27.1.51"
#!/bin/bash
#
# description: Apache Tomcat init script
# processname: tomcat
# chkconfig: 2345 80 20


export JAVA_HOME=/usr/java/jdk1.8.0_131/jre
#export JAVA_OPTS="-server -Xms2g -Xmx22g -XX:+UseG1GC -XX:+UseStringDeduplication"
export JAVA_OPTS="-server -Xms500m -Xmx1024m -XX:+UseG1GC"

export PATH=$JAVA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$JAVA_HOME/lib/amd64/jli:$JAVA_HOME/jre/lib/amd64/jli

export CATALINA_HOME=/opt/tomcat
export CATALINA_BASE=/opt/tomcat
#export CATALINA_OPTS="-Djava.security.debug=all"
export TOMCAT_USER=tomcat

TOMCAT_USAGE="Usage: $0 {\e[00;32mstart\e[00m|\e[00;31mstop\e[00m|\e[00;31mkill\e[00m|\e[00;32mstatus\e[00m|\e[00;31mrestart\e[00m}"

SHUTDOWN_WAIT=20

tomcat_pid() {
        echo `ps -fe | grep $CATALINA_BASE | grep -v grep | tr -s " "|cut -d" " -f2`
}

start() {
  pid=$(tomcat_pid)
  if [ -n "$pid" ]
  then
    echo -e "\e[00;31mTomcat is already running (pid: $pid)\e[00m"
  else
    echo -e "\e[00;32mStarting tomcat\e[00m"
    #ulimit -n 100000
    #umask 007
    #/bin/su -p -s /bin/sh $TOMCAT_USER
        if [ `user_exists $TOMCAT_USER` = "1" ]
        then
                /bin/su $TOMCAT_USER -c "cd $CATALINA_HOME/bin; ./catalina.sh start "
        else
                echo -e "\e[00;31mTomcat user $TOMCAT_USER does not exists. Starting with $(id)\e[00m"
                sh $CATALINA_HOME/bin/catalina.sh start
        fi
        status
  fi
  return 0
}

status(){
          pid=$(tomcat_pid)
          if [ -n "$pid" ]
            then echo -e "\e[00;32mTomcat is running with pid: $pid\e[00m"
          else
            echo -e "\e[00;31mTomcat is not running\e[00m"
            return 3
          fi
}

terminate() {
        echo -e "\e[00;31mTerminating Tomcat\e[00m"
        kill -9 $(tomcat_pid)
}

stop() {
  pid=$(tomcat_pid)
  if [ -n "$pid" ]
  then
    echo -e "\e[00;31mStoping Tomcat\e[00m"
    /bin/su $TOMCAT_USER -c "sh $CATALINA_HOME/bin/shutdown.sh"

    let kwait=$SHUTDOWN_WAIT
    count=0;
    until [ `ps -p $pid | grep -c $pid` = '0' ] || [ $count -gt $kwait ]
    do
      echo -n -e "\n\e[00;31mwaiting for processes to exit\e[00m";
      sleep 1
      let count=$count+1;
    done

    if [ $count -gt $kwait ]; then
      echo -n -e "\n\e[00;31mkilling processes didn't stop after $SHUTDOWN_WAIT seconds\e[00m"
      terminate
    fi
  else
    echo -e "\e[00;31mTomcat is not running\e[00m"
  fi

  return 0
}

user_exists(){
        if id -u $1 >/dev/null 2>&1; then
        echo "1"
        else
                echo "0"
        fi
}

case $1 in
        start)
          start
         # /sbin/MongosClient.sh &
        ;;
        stop)
          stop
         #kill -9 `pidof mongos`
        ;;
        restart)
          stop
          start
        ;;
        status)
                status
                exit $?
        ;;
        kill)
                terminate
        ;;
        *)
                echo -e $TOMCAT_USAGE
        ;;
esac
exit 0
```


## 10. Changes in `/etc/init.d/tomcat` file

#### **10.1 Check java path **

To check path of java use following command

```
readlink -f $(which java)
```
#### **10.2 Change  JAVA_HOME path**

Copy java path till jre and add in following section of `/etc/init.d/tomcat` file
```
export JAVA_HOME=/usr/java/jdk1.8.0_131/jre
```
#### **10.2 Change  JAVA_HOME path & JAVA_BASE path**

Add tomcat folder path in following section in code in `/etc/init.d/tomcat` file
```
export CATALINA_HOME=/opt/tomcat

export CATALINA_BASE=/opt/tomcat
```

## 11. Change ownership root to tomcat and permission 
```
chown -R tomcat:tomcat /opt/tomcat/

chown -R tomcat:tomcat /var/www/tomcat/

chown -R tomcat:tomcat /var/log/tomcat/

chown tomcat:tomcat /etc/init.d/tomcat

chmod 755 /etc/init.d/tomcat
```

## 12. Start & Stop tomcat using service file

#### **12.1 Start tomcat using service file**

We can start tomcat by `service` command and also by `tomcat` file

```
service tomcat start

/etc/init.d/tomcat start
```
#### **12.2 Check tomcat status using service file**

We can check tomcat status by `service` command and also by `tomcat` file

```
service tomcat status

/etc/init.d/tomcat status
```
#### **12.3 Stop tomcat using service file**

We can start tomcat by `service` command and also by `tomcat` file
```
service tomcat stop

/etc/init.d/tomcat stop
```
## 13. Check ports

To check ports we can check it by using the `netstat` command. We can find port for tomcat process using process id in output of netstat command in last entry there is process for tomcat
```
netstat -tlupn | grep tomcat
```

if above command does not work then use following command to check it 
```
netstat -tlupn | grep java
```
## 14. Check Certificate is valid or not
```
curl -k -v https://localhost/
```

## 14. Check Logs 

To check logs we can check it in `/var/www/tomcat/logs` folder using `catalina.out` file
```
cd /var/www/tomcat/logs
```
