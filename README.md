## TS3Web - dockerized TeamSpeak 3 Webinterface by Psychokiller
This is a GitHub repository for a Docker image of the [TeamSpeak 3 Psychokiller Webinterface](https://www.teamspeak-info.de/ts_server_webinterface_von_psychokiller.htm).

The web interface is no longer in development by [Psychokiller](https://forum.teamspeak.com/threads/49547-DEV-Ts3-Webinterface)!

You should whitelist the IP address of the server where the interface is hosted.
If both (web interface and TeamSpeak server) are on the same server you can simply whitelist the whole Docker network using the ip range (e.g. `172.18.0.0/24`)

### Environment Variables
* ALIAS: Name
* HOST: hostname or IP address of the server
* PORT: server query port  (default: 10011)
* LANG: Language of the webinterface (de, en, nl)

### Build the Image
```bash
sudo docker build -t ts3web -f Dockerfile .
```

### Example setup: docker-compose.yml
```
version: '3'
services:
  db:
    image: mariadb
    container_name: root_db_1
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: MyS3Cr3tP4sSw0rD
      MYSQL_DATABASE: teamspeak
    volumes:
      - /srv/mariadb/data:/var/lib/mysql

  ts3:
    image: teamspeak
    container_name: root_ts3_1
    restart: always
    links:
      - db
    environment:
      TS3SERVER_DB_PLUGIN: ts3db_mariadb
      TS3SERVER_DB_SQLCREATEPATH: create_mariadb
      TS3SERVER_DB_HOST: db
      TS3SERVER_DB_USER: root
      TS3SERVER_DB_PASSWORD: MyS3Cr3tP4sSw0rD
      TS3SERVER_DB_NAME: teamspeak
      TS3SERVER_DB_WAITUNTILREADY: 30
      TS3SERVER_LICENSE: accept
      TS3SERVER_IP_WHITELIST: /whitelist.txt
    volumes:
      - /srv/ts3/whitelist.txt:/whitelist.txt
    ports:
      - '2008:2008'      # accounting port
      - '2010:2010'      # weblist port
      - '9987:9987'      # default port (voice)
      - '30033:30033'    # filetransfer port
      - '41144:41144'    # tsdns port

  ts3web:
    image: ts3web
    container_name: root_ts3web_1
    restart: always
    environment:
      ALIAS: 'My Testserver'
      HOST: 'ts3'
      PORT: '10011'
      LANG: 'en'
    ports:
      - '8080:80'        # webinterface port (host:container)
```

### Example usage
You can remove the `-f` parameter, if the file is in the working directory and named `docker-compose.yml`

Start the services:
```bash
$ sudo docker-compose -f docker-compose.yml up -d
Creating network "root_default" with the default driver
Creating ts3wi_db_1 ... done
Creating ts3web     ... done
Creating ts3        ... done
```

Check which services are running:
```bash
$ sudo docker-compose -f docker-compose.yml ps
   Name                 Command               State               Ports
------------------------------------------------------------------------------------
ts3          entrypoint.sh ts3server          Up      10011/tcp, 30033/tcp, 9987/udp
ts3web       /usr/sbin/apache2ctl -D FO ...   Up      0.0.0.0:8888->80/tcp
ts3wi_db_1   docker-entrypoint.sh mysqld      Up      3306/tcp
```

Check the logs of the teamspeak container to get the query credentials and admin token
```bash
$ sudo docker logs root_ts3_1
2018-07-02 23:03:49.450691|INFO    |ServerLibPriv |   |TeamSpeak 3 Server 3.2.0 (2018-05-08 06:11:20)
2018-07-02 23:03:49.450803|INFO    |ServerLibPriv |   |SystemInformation: Linux 4.9.0-6-amd64 #1 SMP Debian 4.9.88-1+deb9u1 (2018-05-07) x86_64 Binary: 64bit
2018-07-02 23:03:49.450843|INFO    |ServerLibPriv |   |Using hardware aes
2018-07-02 23:03:49.451150|INFO    |DatabaseQuery |   |dbPlugin name:    MariaDB plugin, version 3, (c)TeamSpeak Systems GmbH
2018-07-02 23:03:49.451200|INFO    |DatabaseQuery |   |dbPlugin version: 2
2018-07-02 23:04:13.595299|INFO    |SQL           |   |db_CreateTables() tables created

------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
               Server Query Admin Account created                 
         loginname= "serveradmin", password= "LXc09B35"
------------------------------------------------------------------

2018-07-02 23:04:14.681673|WARNING |Accounting    |   |Unable to open licensekey.dat, falling back to limited functionality
2018-07-02 23:04:14.695926|INFO    |Accounting    |   |Licensing Information
2018-07-02 23:04:14.695980|INFO    |Accounting    |   |licensed to       : Anonymous
2018-07-02 23:04:14.696013|INFO    |Accounting    |   |type              : No License
2018-07-02 23:04:14.696051|INFO    |Accounting    |   |starting date     : Wed May 31 22:00:00 2017
2018-07-02 23:04:14.696086|INFO    |Accounting    |   |ending date       : Fri Aug 31 22:00:00 2018
2018-07-02 23:04:14.696120|INFO    |Accounting    |   |max virtualservers: 1
2018-07-02 23:04:14.696152|INFO    |Accounting    |   |max slots         : 32
2018-07-02 23:04:15.240024|INFO    |              |   |Puzzle precompute time: 511
2018-07-02 23:04:15.240990|INFO    |FileManager   |   |listening on 0.0.0.0:30033, [::]:30033
2018-07-02 23:04:15.355223|INFO    |VirtualSvrMgr |   |executing monthly interval
2018-07-02 23:04:15.355830|INFO    |VirtualSvrMgr |   |reset virtualserver traffic statistics
2018-07-02 23:04:16.817572|INFO    |VirtualServer |1  |listening on 0.0.0.0:9987, [::]:9987
2018-07-02 23:04:16.909194|WARNING |VirtualServer |1  |--------------------------------------------------------
2018-07-02 23:04:16.909325|WARNING |VirtualServer |1  |ServerAdmin privilege key created, please use the line below
2018-07-02 23:04:16.909349|WARNING |VirtualServer |1  |token=ZBnJVnhEOULGt4uX9XDMZR3fd1TKxZhBM6lv4sxf
2018-07-02 23:04:16.909397|WARNING |VirtualServer |1  |--------------------------------------------------------

------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
      ServerAdmin privilege key created, please use it to gain
      serveradmin rights for your virtualserver. please
      also check the doc/privilegekey_guide.txt for details.

       token=ZBnJVnhEOULGt4uX9XDMZR3fd1TKxZhBM6lv4sxf
------------------------------------------------------------------

2018-07-02 23:04:16.909896|INFO    |CIDRManager   |   |updated query_ip_whitelist ips: 172.18.0.0/24,
2018-07-02 23:04:16.910864|INFO    |Query         |   |listening on 0.0.0.0:10011, [::]:10011
```
