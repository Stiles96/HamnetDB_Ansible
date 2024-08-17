# HamnetDB (Ansible)
Die HamnetDB als Webseite gibt es auf GitHub (https://github.com/hamnetdb/hamnetdb) direkt für den eigenen Webspace. Ebenfalls kann die SQL Datei mit den aktullen HamnetDB Daten von https://hamnetdb.net heruntergeladen werden.

## Installation

    # Install Packages
    sudo apt-get install apache2 perl libdbd-mysql-perl build-essential libx11-dev libXext-dev libjpeg-dev libpng-dev mysql-server mysql-client
    sudo a2enmod CGI

    # Configure MySQL
    sudo mysql -e "CREATE DATABASE IF NOT EXISTS hamnet;"
    sudo mysql -e "CREATE USER IF NOT EXISTS 'hamnet'@'localhost' IDENTIFIED BY 'securepassword234!';"
    sudo mysql -e "GRANT ALL ON hamnet.* TO 'hamnet'@'localhost';"
    sudo nano /etc/mysql/mysql.ini ??? => Change Listen Port to 3307
    sudo service mysql start

    # Create Directories
    sudo mkdir /var/www/hamnet
    sudo mkdir /var/www/hamnetressources
    sudo mkdir /var/www/hamnetressources/rftools
    sudo mkdir /var/www/hamnetressources/tile_output
    sudo chown www-data:www-data /var/www/hamnet -R
    sudo chown www-data:www-data /var/www/hamnetressources -R

    # Clone HamnetDB and Import Database
    sudo git clone https://github.com/hamndetdb/hamnetdb.git /var/www/hamnet
    wget -qO- 'https://hamnetdb.net/dump.cgi' | mysql --database hamnet

    # Clone Profile Calculator
    sudo git clone https://github.com/oe5hpm/dxlAPRS.git /var/www/hamnetressources/rftools
    sudo make /var/www/hamnetressources/rftools


## Cronjob für HamnetDB Database
Die Daten von der HamnetDB können mit einem Cronjob, solange eine Verbindung besteht, regelmäßig aktuallisiert werden. Die Monitoringdaten werden ebenfalls geladen. Sind die Daten älter als 2 Stunden, weden die Sites als offline bzw. unbekannt markiert. Der Aktuallisierungintervall sollte am besten innerhalb von 2h liegen. 
Cronjob:
    */60 * * * *    root    "wget -qO- 'https://hamnetdb.net/dump.cgi' | mysql --database hamnet"

## Config apache2
Beispielkonfiguration, /etc/apache2/sites-available/hamnetdb.conf:

    <VirtualHost *:88> # Port 88 (Port 8ß bereits von pbx belegt)

        ServerAdmin IT-Service@he-it.eu
        DocumentRoot /var/www/hamnet
        ServerName hamnetdb.he-it.eu
        ServerAlias pbx.Zuhause.local
        ServerAlias *
        ServerAlias 10.90.242.83 # AREDN Adresse

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        Protocols h2 h2c http/1.1
        H2Push on
        H2PushPriority * after
        H2PushPriority text/css before
        H2PushPriority image/jpg after 32
        H2PushPriority image/jpeg after 32
        H2PushPriority image/png after 32
        H2PushPriority application/javascript interleaved

        <Directory /var/www/hamnet/>
            AddHandler cgi-script .cgi .pl
            AddDefaultCharset UTF-8
            Options FollowSymlinks ExecCGI
            DirectoryIndex index.cgi index.html
            AllowOverride All
            Require all granted
        </Directory>

    </VirtualHost>


Konfiguration von Apache:
    sudo a2dissite 000-default
    sudo a2ensite hamnetdb
    sudo a2enmod headers
    sudo a2enmod env
    sudo a2enmod dir
    sudo a2enmod mime
    sudo a2enmod rewrite
    sudo a2enmod proxy_cfgi
    sudo a2enmod setenvif
    sudo a2enmod http2
    sudo a2enmod proxy
    sudo a2enmod proxy_http
    sudo a2enmod proxy_wstunnel
    sudo a2enmod mpm_event

    sudo service apache2 restart

## Konfiguration hamnetdb
Für die Profile Calculation sind die SRTM Files notwendig, diese sind öffentlich zugänglich und können heruntergeladen werden. Ein Download steht auch direkt für die HamnetDB hier zur Verfügung (~300GB): https://HEITServiceNAS01.myqnapcloud.com/share.cgi?ssid=869a5899f15f464c96c63560a184a669


/var/www/hamnetdb/config.cgi

    #!/usr/bin/perl
    # -------------------------------------------------------------------------
    # Hamnet IP database - Basic configuration
    #
    # Flori Radlherr, DL8MBT, http://www.radlherr.de
    #
    # Licensed with Creative Commons Attribution-NonCommercial-ShareAlike 3.0
    # http://creativecommons.org/licenses/by-nc-sa/3.0/
    # - you may change, distribute and use in non-commecial projects
    # - you must leave author and license conditions
    # -------------------------------------------------------------------------
    #
    $db_user= "hamnet";
    $db_name= "hamnet";
    $db_pass= "securepassword234!";
    $db_host= "localhost:3307"; # 3307, weil 3306 bereits von pbx belegt wird

    # -------------------------------------------------------------------------
    # Change here to integrate with your own web page layout
    sub htmlHead {
    my $element= shift;
    print qq(Content-Type: text/html\n\n<!DOCTYPE html>
        <html><head>
        <title>${element}Hamnet-DB</title>
        <link rel="shortcut icon" href="favicon.ico" />
        <script language="JavaScript" src="jquery.js"></script> 
        </head><body style='margin:10px;'>
    );
    }
    sub htmlFoot {
    print qq(
        </body></html>
    );
    }

    # From these agent systems the checks are performed.
    # Needs an authorized ssh key to get access without password
    #@fpingAgents= (
    #  "db0fhn"
    #  , "db0zm"
    #  , "ir3dv"
    #);

    #@monitorAgents= (["db0fhn", "http://monitoring"]);
    #@routingAgents= (["db0fhn", "http://monitoring"]);
    #@tracerouteAgents= (["db0fhn", "http://monitoring"]);
    #$monitoring_API= "http://monitoring";

    # uncomment and fill in information to activate profile-calculation
    $profile_path_program= "/var/www/hamnetressources/rftools/out-x86_64/profile";
    $profile_path_srtm= "/mnt/data/coverage"; # srtm Files Location
    $profile_path_errorlog= "/var/www/hamnetressources/logfile";
    #$profile_errorimg= "./error.png"
    $profile_path_color= "/var/www/hamnet/cols.txt";
    $profile_path_local_api= "http://localhost/rftools/calc_profile.cgi";

    $visibility_path_program= "/var/www/hamnetressources/rftools/out-x86_64/visibility";
    $visibility_path_srtm= "/mnt/data/coverage";
    $visibility_path_errorlog= "/var/www/hamnetressources/logfile";
    $visibility_path_out= "/var/www/hamnetressources/tile_output";

    $panorama_path_program= "/var/www/hamnetressources/rftools/out-x86_64/panorama";
    $panorama_path_srtm= "/mnt/data/coverage";
    $panorama_path_errorlog= "/var/www/hamnetressources/logfile";
    $panorama_path_web= "/var/www/hamnet";

    1;

Für die lokale OSM Server Verwendung muss der Datei "osm/hamnetdb-lf.js" folgendes ergänzt werden:

    }
    else if (source == "local") //local
    {
        var mapnikUrl = 'http://' + window.location.hostname + ':8080/tile/{z}/{x}/{y}.png';
        var mapnikUrl1 = 'http://' + window.location.hostname + ':8080/tile/{z}/{x}/{y}.png';
        var landscapeUrl = 'http://' + window.location.hostname + ':8080/tiles_topo/{z}/{x}/{y}.png';
        var cycleUrl = 'http://' + window.location.hostname + ':8080/tiles_cyclemap/{z}/{x}/{y}.pn';
        var satUrl= 'http://' + window.location.hostname + ':8080/tiles_sat/{z}/{x}/{y}.jpg';
        var mapnikZoom = 16 ;
        var landscapeZoom =16;
        var cycleZoom = 16;
        var satZoom = 11;

# OSM Tile Server (Docker | Ansible)
Der OSM Tile Server wird per Docker bereitgestellt. Der Container läd die Daten selbst herunter und wird nach dem Import ausgeschaltet. Bei nächsten Start wird der Container in den run mode versetzt und liefert eine Webserver mit den OSM Tiles.
Nach dem Start sollte einmal bis Zoom Level 9 die Tiles gerendert werden, das kann im Docker mit folgendem Befehl geschehen:
     render_list -m default -a -z 9 -Z 9

Docker Import:
    sudo docker run \
    --name openstreetmap-tile-server \
    -p 8080:80 \
    -e DOWNLOAD_PBF=https://download.geofabrik.de/europe/germany-latest.osm.pbf \
    -e DOWNLOAD_POLY=https://download.geofabrik.de/europe/germany.poly \
    -v map-data:/data/database/ \
    overv/openstreetmap-tile-server \
    import

Docker Run:
    sudo docker run \
    --name openstreetmap-tile-server \
    -p 8080:80 \
    -v map-data:/data/database/ \
    -d overv/openstreetmap-tile-server \
    run