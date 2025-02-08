+++

title = "Instalacja Bind"

date = 2025-02-08

description = "Instalacja Bind lokalnie"

tags = ["Linux", "DNS"]

categories = ["Projekty"]

draft = false

+++

  

# Przewodnik Konfiguracji Lokalnego Serwera DNS BIND

  

Ten przewodnik pomoże Ci skonfigurować lokalny serwer DNS przy użyciu BIND9 do obsługi własnej domeny (np. mydomain.local) z obsługą rozwiązywania nazw w przód i wstecz.

  

## Wymagania wstępne

  

- System Ubuntu/Debian

- Dostęp root lub sudo

- Podstawowa znajomość koncepcji DNS

  

## Instalacja

  

Zaktualizuj system i zainstaluj BIND9:

  

```bash

sudo apt update

sudo apt upgrade -y

sudo apt install bind9 bind9utils bind9-doc -y

```

  

Sprawdź status usługi BIND:

```bash

sudo systemctl status bind9

```

  

## Konfiguracja

  

### 1. Konfiguracja stref lokalnych

  

Edytuj plik konfiguracji lokalnej:

```bash

sudo nano /etc/bind/named.conf.local

```

  

Dodaj następujące konfiguracje stref:

```bash

zone "mydomain.local" {

    type master;

    file "/etc/bind/zones/mydomain.local.zone";

};

  

zone "0.0.127.in-addr.arpa" {

    type master;

    file "/etc/bind/zones/reverse.local.zone";

};

```

  

### 2. Tworzenie plików stref

  

Utwórz katalog na strefy:

```bash

sudo mkdir -p /etc/bind/zones

```

  

#### Plik strefy prostej (forward)

  

Utwórz i edytuj plik strefy prostej:

```bash

sudo nano /etc/bind/zones/mydomain.local.zone

```

  

Dodaj następującą konfigurację:

```bash

$TTL 86400

@   IN  SOA   ns1.mydomain.local. admin.mydomain.local. (

        2025011401 ; Numer seryjny

        3600       ; Odświeżanie

        1800       ; Ponawianie

        1209600    ; Wygaśnięcie

        86400 )    ; Minimalny TTL

  

@   IN  NS    ns1.mydomain.local.

@   IN  A     127.0.0.1

ns1 IN  A     127.0.0.1

www IN  A     127.0.0.1

app IN  A     127.0.0.2

```

  

#### Plik strefy odwrotnej (reverse)

  

Utwórz i edytuj plik strefy odwrotnej:

```bash

sudo nano /etc/bind/zones/reverse.local.zone

```

  

Dodaj następującą konfigurację:

```bash

$TTL 86400

@   IN  SOA   ns1.mydomain.local. admin.mydomain.local. (

        2025011401 ; Numer seryjny

        3600       ; Odświeżanie

        1800       ; Ponawianie

        1209600    ; Wygaśnięcie

        86400 )    ; Minimalny TTL

  

@   IN  NS    ns1.mydomain.local.

1   IN  PTR   ns1.mydomain.local.

1   IN  PTR   www.mydomain.local.

2   IN  PTR   app.mydomain.local.

```

  

### 3. Konfiguracja opcji BIND

  

Edytuj plik opcji:

```bash

sudo nano /etc/bind/named.conf.options

```

  

Dodaj następującą konfigurację:

```bash

options {

    directory "/var/cache/bind";

    recursion yes;

    allow-query { localhost; };

    forwarders {

        8.8.8.8;  // DNS Google

        1.1.1.1;  // DNS Cloudflare

    };

    dnssec-validation auto;

    listen-on { 127.0.0.1; };

    listen-on-v6 { ::1; };

};

```

  

## Weryfikacja i testowanie

  

### 1. Sprawdź składnię konfiguracji

  

```bash

sudo named-checkconf

sudo named-checkzone mydomain.local /etc/bind/zones/mydomain.local.zone

sudo named-checkzone 0.0.127.in-addr.arpa /etc/bind/zones/reverse.local.zone

```

  

### 2. Zrestartuj usługę BIND

  

```bash

sudo systemctl restart bind9

```

  

### 3. Przetestuj rozwiązywanie DNS

  

Test wyszukiwania do przodu:

```bash

dig @127.0.0.1 www.mydomain.local

```

  

Test wyszukiwania wstecznego:

```bash

dig -x 127.0.0.1 @127.0.0.1

```

  

## Rozwiązywanie problemów

  

1. Sprawdź logi BIND:

```bash

sudo tail -f /var/log/syslog | grep named

```

  

2. Sprawdź status usługi:

```bash

sudo systemctl status bind9

```

  

1. Sprawdź uprawnienia plików stref:

```bash

sudo ls -l /etc/bind/zones/

```

  

## Typowe problemy

  

- **Usługa nie uruchamia się**: Sprawdź błędy składni w plikach konfiguracyjnych

- **Rozwiązywanie DNS nie działa**: Sprawdź ustawienia zapory sieciowej i adresy nasłuchiwania BIND

- **Problemy z transferem stref**: Sprawdź uprawnienia plików i składnię plików stref

  

## Aspekty bezpieczeństwa

  

2. Ogranicz transfery stref do zaufanych serwerów

3. Skonfiguruj odpowiednie listy kontroli dostępu w `allow-query`

4. Aktualizuj BIND o najnowsze poprawki bezpieczeństwa

5. Użyj DNSSEC dla zwiększonego bezpieczeństwa

  

## Dodatkowe źródła

  

- [Dokumentacja BIND](https://www.isc.org/bind/)

- [Przewodnik BIND dla Ubuntu](https://ubuntu.com/server/docs/service-domain-name-service-dns)