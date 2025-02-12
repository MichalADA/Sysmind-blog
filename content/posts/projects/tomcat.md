+++
title = "Instalacja Apache Tomcat"
date = "2025-02-10"
description = "Instalacja i konfiguracja Apache Tomcat na Ubuntu"
tags = ["Linux", "Java", "Server"]
categories = ["Projekty"]
draft = false
+++

# Instalacja i konfiguracja Apache Tomcat na Ubuntu

Ten przewodnik opisuje kroki niezbędne do instalacji i konfiguracji Apache Tomcat na Ubuntu z Java 17, skonfigurowania go jako usługi systemowej oraz włączenia HTTPS z certyfikatem self-signed.

## 1. Instalacja Java 17

Aktualizacja menedżera pakietów i instalacja Java 17:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Weryfikacja instalacji Java:

```bash
java -version
```

Konfiguracja domyślnej wersji Java (jeśli zainstalowanych jest wiele wersji):

```bash
sudo update-alternatives --config java
```

Ustawienie zmiennej środowiskowej `JAVA_HOME`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

Utrwalenie zmiennej `JAVA_HOME`:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

Weryfikacja zmiennej `JAVA_HOME`:

```bash
echo $JAVA_HOME
```

## 2. Instalacja Apache Tomcat

Pobranie Apache Tomcat:

```bash
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz
```

Rozpakowanie archiwum:

```bash
tar -xvzf apache-tomcat-11.0.2.tar.gz
```

Przeniesienie Tomcat do katalogu `/opt`:

```bash
sudo mv apache-tomcat-11.0.2 /opt/tomcat
```

Ustawienie uprawnień do katalogu Tomcat:

```bash
sudo chown -R $USER:$USER /opt/tomcat
```

Uruchomienie Tomcat:

```bash
/opt/tomcat/bin/startup.sh
```

## 3. Konfiguracja Tomcat jako usługi systemowej

Utworzenie pliku usługi systemd:

```bash
sudo nano /etc/systemd/system/tomcat.service
```

Dodanie następującej zawartości do pliku usługi:

```ini
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Przeładowanie systemd i włączenie usługi:

```bash
sudo systemctl daemon-reload
sudo systemctl enable tomcat
```

Uruchomienie usługi Tomcat:

```bash
sudo systemctl start tomcat
```

Sprawdzenie statusu usługi:

```bash
sudo systemctl status tomcat
```

## 4. Włączenie HTTPS z certyfikatem self-signed

Generowanie certyfikatu self-signed:

```bash
keytool -genkey -alias tomcat -keyalg RSA -keystore /opt/tomcat/conf/keystore.jks -keysize 2048
```

* Podczas generowania certyfikatu należy podać wymagane informacje (np. hasło, nazwa organizacji, itp.).

Konfiguracja Tomcat dla HTTPS:

1. Otwarcie pliku `server.xml`:

```bash
sudo nano /opt/tomcat/conf/server.xml
```

2. Znalezienie elementu `<Connector>` dla HTTPS i odkomentowanie lub zmodyfikowanie go:

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="200" SSLEnabled="true">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/keystore.jks"
                     type="RSA" certificateKeystorePassword="twoje_haslo_keystore"/>
    </SSLHostConfig>
</Connector>
```

3. Zapisanie i zamknięcie pliku.

Restart Tomcat, aby zastosować zmiany:

```bash
sudo systemctl restart tomcat
```

Test HTTPS:
Otwórz przeglądarkę i przejdź do:

```
https://twoj-adres-ip:8443
```

## 5. Dodatkowe komendy

Zatrzymanie Tomcat:

```bash
sudo systemctl stop tomcat
```

Restart Tomcat:

```bash
sudo systemctl restart tomcat
```

Sprawdzenie logów:

```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

## Rozwiązywanie problemów

1. Jeśli Tomcat nie uruchamia się, sprawdź logi:
```bash
journalctl -u tomcat.service
```

2. Upewnij się, że porty nie są zajęte:
```bash
sudo netstat -tulpn | grep LISTEN
```

3. Sprawdź uprawnienia do katalogów:
```bash
ls -la /opt/tomcat
```

## Wskazówki bezpieczeństwa

1. Zmień domyślne hasła w pliku `tomcat-users.xml`
2. Usuń przykładowe aplikacje w środowisku produkcyjnym
3. Regularnie aktualizuj Tomcat do najnowszej wersji
4. Rozważ użycie firewalla do ograniczenia dostępu

Tomcat jest teraz w pełni zainstalowany i skonfigurowany do działania jako usługa systemowa z obsługą HTTPS!
