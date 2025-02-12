+++
title = "Instalacja WildFly (JBoss)"
date = "2025-02-10"
description = "Instalacja i konfiguracja WildFly (JBoss) na Ubuntu"
tags = ["Linux", "Java", "Server", "JBoss"]
categories = ["Projekty"]
draft = false
+++

# Instalacja i konfiguracja WildFly (JBoss) na Ubuntu

Ten przewodnik opisuje proces instalacji i konfiguracji serwera WildFly (dawniej JBoss) na Ubuntu, w tym zmianę domyślnych portów, utworzenie użytkownika administratora oraz skonfigurowanie go jako usługi systemowej.

## 1. Pobranie i rozpakowanie WildFly

### Pobranie WildFly:
```bash
wget https://github.com/wildfly/wildfly/releases/download/35.0.0.Final/wildfly-35.0.0.Final.tar.gz
```

### Rozpakowanie pobranego archiwum:
```bash
tar -xvzf wildfly-35.0.0.Final.tar.gz
```

### Przeniesienie WildFly do `/opt/jboss`:
```bash
sudo mv wildfly-35.0.0.Final /opt/jboss
```

### Ustawienie uprawnień do katalogu WildFly:
```bash
sudo chown -R $USER:$USER /opt/jboss
```

## 2. Zmiana domyślnych portów

### Edycja pliku konfiguracyjnego WildFly:
```bash
nano /opt/jboss/standalone/configuration/standalone.xml
```

### Zmiana portu HTTP na `8181`:
```xml
<socket-binding name="http" port="${jboss.http.port:8181}"/>
```

### Zmiana portu HTTPS na `8543`:
```xml
<socket-binding name="https" port="${jboss.https.port:8543}"/>
```

### Zapisz i zamknij plik (`CTRL + O`, następnie `CTRL + X`).

## 3. Uruchomienie WildFly

### Przejście do katalogu `bin` WildFly:
```bash
cd /opt/jboss/bin
```

### Uruchomienie serwera w trybie standalone:
```bash
./standalone.sh
```

### Weryfikacja w przeglądarce:
```
http://twoj-adres-ip:8181
```

## 4. Utworzenie użytkownika administratora

### Uruchomienie skryptu do tworzenia użytkowników:
```bash
/opt/jboss/bin/add-user.sh
```

### Postępuj zgodnie z instrukcjami, aby utworzyć użytkownika z rolą `ManagementRealm`.

## 5. Konfiguracja WildFly jako usługi systemowej

### Utworzenie pliku usługi systemd:
```bash
sudo nano /etc/systemd/system/jboss.service
```

### Dodanie następującej konfiguracji:
```ini
[Unit]
Description=WildFly Application Server
After=network.target

[Service]
User=twoja-nazwa-uzytkownika
Group=twoja-nazwa-uzytkownika
ExecStart=/opt/jboss/bin/standalone.sh -b 0.0.0.0
ExecStop=/opt/jboss/bin/jboss-cli.sh --connect command=:shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```

### Zapisz i zamknij plik (`CTRL + O`, następnie `CTRL + X`).

### Przeładowanie systemd i włączenie usługi:
```bash
sudo systemctl daemon-reload
sudo systemctl enable jboss
```

### Uruchomienie usługi WildFly:
```bash
sudo systemctl start jboss
```

### Sprawdzenie statusu usługi:
```bash
sudo systemctl status jboss
```

## 6. Rozwiązywanie problemów

### Sprawdzanie logów:
```bash
tail -f /opt/jboss/standalone/log/server.log
```

### Sprawdzanie statusu portów:
```bash
netstat -tulpn | grep LISTEN
```

### Sprawdzanie firewalla:
```bash
sudo ufw status
```

## 7. Wskazówki bezpieczeństwa

1. Zmień domyślne hasła dla wszystkich kont administratora
2. Ogranicz dostęp do konsoli administracyjnej poprzez firewall
3. Regularnie aktualizuj WildFly do najnowszej wersji
4. Używaj HTTPS dla konsoli administracyjnej
5. Skonfiguruj odpowiednie uprawnienia dla plików i katalogów

## 8. Dodatkowe komendy

### Zatrzymanie WildFly:
```bash
sudo systemctl stop jboss
```

### Restart WildFly:
```bash
sudo systemctl restart jboss
```

### Sprawdzenie wersji WildFly:
```bash
/opt/jboss/bin/standalone.sh -v
```

WildFly (JBoss) jest teraz zainstalowany, skonfigurowany i uruchomiony jako usługa systemowa. Możesz uzyskać dostęp do serwera przez zaktualizowane porty HTTP lub HTTPS.

## 9. Konfiguracja pamięci

Aby zoptymalizować działanie WildFly, możesz dostosować ustawienia pamięci JVM. Edytuj plik `standalone.conf`:

```bash
nano /opt/jboss/bin/standalone.conf
```

Znajdź i zmodyfikuj następujące linie:

```bash
JAVA_OPTS="-Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m"
```

Dostosuj wartości zgodnie z dostępną pamięcią na serwerze.
