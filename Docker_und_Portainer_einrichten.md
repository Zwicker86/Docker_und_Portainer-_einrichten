[Docker installieren](https://www.bogotobogo.com/DevOps/Docker/Docker_Install_On_Amazon_Linux_AMI.php)
[Portainer installieren](https://www.howtoforge.de/anleitung/wie-man-portainer-fur-die-docker-verwaltung-mit-dem-nginx-proxy-manager-installiert-und-nutzt/)

#### per SSH mit der EC2 Instanz verbinden

    ssh -i Zwick_Apache.pem ec2-user@3.75.184.142
    
    Offene Ports: 80  443  9443  81
    
#### source list updaten

    sudo yum update
    
#### Docker installieren 

    sudo yum install -y docker
    
#### Version kontrollieren

    docker -v
    
#### Docker starten

    sudo service docker start
    
#### Firewall konfigurieren

    sudo firewall-cmd --state
    
#### Ports öffnen

    sudo firewall-cmd --permanent --add-service=http
    sudo firewall-cmd --permanent --add-service=https
    sudo firewall-cmd --permanent --add-port=9443/tcp
    sudo firewall-cmd --permanent --add-port=81/tcp
    
#### Firewall neustarten

    sudo firewall-cmd --reload
    
#### Docker istallieren Ubuntu

    sudo apt install ca-certificates curl gnupg lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io
    
#### Docker-Dienst starten

    sudo systemctl start docker --now
    
#### Benutzernamen zur Docker-Gruppe hinzufügen

    sudo usermod -aG docker $USER
    
    danach ab- und wieder anmelden
    
#### Docker Compose installieren

    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    
#### Berechtigung zur Ausführung der Binärdatei

    sudo chmod +x /usr/local/bin/docker-compose

#### Portainer-Verzeichnis erstellen und ins Verzeichnis wechseln

    mkdir ~/portainer
    cd ~/portainer
    
#### Erstellen und öffnen von Docker Compose Datei

    nano docker-compose.yaml
    
#### Code in die Datei einfügen:

    version: "3.3"
    services:
        portainer:
           image: portainer/portainer-ce:latest
           container_name: portainer
           restart: always
           privileged: true
           volumes:
             - ./data:/data:Z
             - /var/run/docker.sock:/var/run/docker.sock:Z
           ports:
           - 9443:9443
           
#### Portainer starten

    docker-compose up -d
    
#### Status des Containers überprüfen

    docker ps
  CONTAINER ID   IMAGE                           COMMAND        CREATED         STATUS         PORTS                                                           NAMES
916411e8d12e   portainer/portainer-ce:latest   "/portainer"   5 seconds ago   Up 4 seconds   8000/tcp, 9000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   portainer

#### Portainer aufrufen und konfigurieren

Öffne die URL https://<yourserverIP>:9443 in deinem Browser und du erhältst den folgenden Bildschirm

#### Portainer

Du wirst aufgefordert, einen neuen Administrator-Benutzer anzulegen. Gib deine Benutzerdaten ein.
Deaktiviere das Kontrollkästchen Sammlung anonymer Statistiken zulassen, wenn dir der Datenschutz wichtig ist.
Klicke auf die Schaltfläche Benutzer erstellen, um die Installation zu starten und ein neues Administratorkonto zu erstellen.

Als Nächstes wirst du zum folgenden Dashboard-Bildschirm weitergeleitet.

Nach ein paar Sekunden wird er automatisch aktualisiert und zeigt dir den folgenden Bildschirm an.

Klicke auf die lokale Umgebung, um loszulegen.

Die meisten Abschnitte sind selbsterklärend. Der Abschnitt Stacks hilft dir bei der Erstellung von Containern mithilfe von Docker-Compose-Dateien.
Du kannst Container direkt über die Kategorie Container in der Seitenleiste bereitstellen. Im Bereich Hosts kannst du die aktuelle Docker-Umgebung konfigurieren.
Im Bereich App-Vorlagen findest du vorinstallierte Docker Compose-Dateien für die Installation der gängigsten Anwendungen. Du kannst auch eigene Vorlagen erstellen.
Im Bereich Einstellungen kannst du verschiedene Einstellungen vornehmen, z. B. benutzerdefinierte Docker-Registries hinzufügen, mehrere Hosts für Docker Swarm hinzufügen, den Benutzerzugang konfigurieren, Daten sichern und Portainer anpassen.

#### Portainer mit dem Nginx Proxy Manager (NPM) hinter einen Reverse Proxy setzen

#### NPM installieren

Der erste Schritt besteht darin, ein Netzwerk für den Nginx Proxy Manager (NPM) zu erstellen. 
Öffne den Bereich Netzwerke und klicke auf die Schaltfläche Netzwerk hinzufügen, um ein neues Netzwerk zu erstellen.
Gib dem Netzwerk einen Namen und lass alle Einstellungen unverändert. Klicke auf die Schaltfläche Netzwerk erstellen, um den Vorgang abzuschließen.
Gehe zu den Stacks und erstelle mit der Schaltfläche Stack hinzufügen einen neuen Stack.

#### Stack benennen

    nginx-proxy-manager
    
#### Code hinzufügen

    version: "3.3"
    services:
    npm-app:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm-app
    restart: unless-stopped
    ports:
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    environment:
      DB_MYSQL_HOST: "npm-db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: ${DB_MYSQL_PASSWORD}
      DB_MYSQL_NAME: "npm"
      # Uncomment the line below if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - ./npm-data:/data:Z
      - ./letsencrypt:/etc/letsencrypt:Z
    depends_on:
      - npm-db
    networks:
      - npm-network
      - npm-internal

    npm-db:
    image: 'mariadb:latest'
    container_name: npm-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: ${DB_MYSQL_PASSWORD}
    volumes:
      - ./npm-data/mysql:/var/lib/mysql:Z
    networks:
      - npm-internal

    networks:
    npm-internal:
    npm-network:
    external: true

Wir haben zwei Umgebungsvariablen gesetzt, um Datenbank- und Root-MySQL-Passwörter festzulegen.

Portainer kann verwendet werden, um Geheimnisse mithilfe von Umgebungsvariablen zu setzen. Scrolle auf der Seite nach unten und klicke auf die Schaltfläche Umgebungsvariable hinzufügen, um starke Passwörter hinzuzufügen.

Klicke auf die Schaltfläche Deploy the stack, um den NPM-Container zu erstellen und zu starten.

#### Zugriff auf MPM

Öffne die URL https://<yourserverIP>:81 in deinem Browser und du erhältst den folgenden Bildschirm.
Gib die folgenden Standard-Anmeldedaten ein, um dich anzumelden.

E-Mail Adresse

    admin@example.com
    
Passwort

    changeme
    
Als Nächstes wirst du sofort aufgefordert, einen Namen und eine E-Mail-Adresse anzugeben. 
Klicke auf die Schaltfläche Speichern und du wirst aufgefordert, ein neues Passwort zu erstellen. 
Klicke erneut auf die Schaltfläche Speichern, um loszulegen.
Gehe zu Hosts >> Proxy Hosts und klicke auf die Schaltfläche Proxy Host hinzufügen
Gib den Domainnamen portainer.example.com ein. Wähle das Schema als https. 
Gib den Namen des Containers als Forward Hostname und 9443 als Forward Port ein. 
Aktiviere die Optionen Block Common Exploits und Websockets Support.
Wechsle zur Registerkarte SSL und wähle im Dropdown-Menü die OptionNeues SSL-Zertifikat anfordern.
Aktiviere die OptionenForce SSL und HTTP/2 Support, um deine SSL-Verbindung zu sichern und zu optimieren. 
Gib die E-Mail-Adresse ein, an die du die Erneuerungsbenachrichtigungen senden möchtest, und stimme den Nutzungsbedingungen zu.    
Klicke auf die Schaltfläche Speichern, um die Einrichtung des Proxy-Hosts für Portainer abzuschließen.

#### Portainer mit dem NPM-Container verbinden

Gehe zurück zum Portainer-Dashboard, besuche den Bereich Container und wähle den Portainer-Container aus.
Wähle npm-network aus dem Dropdown-Menü unter dem Abschnitt Verbundene Netzwerke und klicke auf die SchaltflächeNetzwerk beitreten, um den Portainer-Container zum Netzwerk des Proxy-Managers hinzuzufügen.
Es kann sein, dass du eine Fehlermeldung erhältst, aber wenn du die Seite aktualisierst, solltest du sehen, dass der Container zum NPM-Netzwerk hinzugefügt wurde.
Du solltest nun in der Lage sein, Portainer über die URL https://portainer.example.com in deinem Browser aufzurufen.
Du kannst ähnlich vorgehen, um NPM hinter einer öffentlich zugänglichen URL wie https://npm.example.com zu platzieren, wie in https://www.howtoforge.com/how-to-install-and-use-nginx-proxy-manager/ beschrieben.

Nachdem du nun eine öffentliche URL für Portainer festgelegt hast, kannst du den exponierten Port 9443 entfernen. Gehe dazu zurück zum Terminal und wechsle in das Portainer-Verzeichnis.

    cd ~/portainer
    nano docker-compose.yaml
    
Entferne den Ports-Abschnitt, indem du ihn auskommentierst, wie unten gezeigt.

version: "3.3"
services:
    portainer:
      image: portainer/portainer-ce:latest
      container_name: portainer
      restart: always
      privileged: true
      volumes:
        - ./data:/data:Z
        - /var/run/docker.sock:/var/run/docker.sock:Z
      #ports:
      #  - 9443:9443
      networks:
        - npm-network

networks:
  npm-network:
    external: true
    
Hier haben wir die Details des NPM-Netzwerks hinzugefügt, weil wir den Portainer-Container neu starten müssen.

Halte den Portainer-Container an.

    docker-compose down --remove-orphans
    
Starte den Container erneut mit der aktualisierten Konfiguration

    docker-compose up -


