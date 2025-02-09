+++

title = "Split-HorizonDNS"

date = 2025-02-08

description = "Instalacja Split-Horizon"

tags = ["Linux", "DNS"]

categories = ["Projekty"]

draft = false

+++
# Konfiguracja Split-Horizon DNS na CentOS/AlmaLinux

Ten poradnik przeprowadzi Cię przez proces konfiguracji Split-Horizon DNS przy użyciu BIND na systemach CentOS lub AlmaLinux. Split-Horizon DNS pozwala serwerowi DNS zwracać różne rekordy w zależności od źródła zapytania (sieć wewnętrzna vs zewnętrzna).

## 1. Wymagania wstępne
* CentOS, AlmaLinux lub RedHat z dostępem root lub sudo
* Zainstalowany BIND (`bind` i `bind-utils`)

## 2. Instalacja BIND
```bash
sudo dnf install bind bind-utils -y
```

## 3. Przygotowanie plików stref
Utworzenie katalogu stref:
```bash
sudo mkdir -p /etc/named/zones
```

### Plik strefy wewnętrznej
Utwórz plik `/etc/named/zones/internal.example.com.zone`:
```conf
$TTL 86400
@   IN  SOA   ns1.example.com. admin.example.com. (
            2025011501 ; Numer seryjny
            3600       ; Odświeżanie
            1800       ; Ponawianie
            1209600    ; Wygaśnięcie
            86400 )    ; Minimalny TTL
@   IN  NS    ns1.example.com.
ns1 IN  A     192.168.1.1
www IN  A     192.168.1.2
mail IN  A     192.168.1.3
```

### Plik strefy zewnętrznej
Utwórz plik `/etc/named/zones/external.example.com.zone`:
```conf
$TTL 86400
@   IN  SOA   ns1.example.com. admin.example.com. (
            2025011501 ; Numer seryjny
            3600       ; Odświeżanie
            1800       ; Ponawianie
            1209600    ; Wygaśnięcie
            86400 )    ; Minimalny TTL
@   IN  NS    ns1.example.com.
ns1 IN  A     203.0.113.1
www IN  A     203.0.113.2
mail IN  A     203.0.113.3
```

## 4. Konfiguracja BIND
Edytuj plik `/etc/named.conf` aby zdefiniować widoki dla zapytań wewnętrznych i zewnętrznych.

### Widok wewnętrzny
```conf
view "internal" {
    match-clients { 192.168.1.0/24; localhost; }; // Sieć wewnętrzna
    zone "example.com" {
        type master;
        file "/etc/named/zones/internal.example.com.zone";
    };
};
```

### Widok zewnętrzny
```conf
view "external" {
    match-clients { any; }; // Wszystkie pozostałe źródła
    zone "example.com" {
        type master;
        file "/etc/named/zones/external.example.com.zone";
    };
};
```

## 5. Walidacja konfiguracji
Sprawdź plik konfiguracyjny:
```bash
sudo named-checkconf
```

Sprawdź pliki stref:
```bash
sudo named-checkzone example.com /etc/named/zones/internal.example.com.zone
sudo named-checkzone example.com /etc/named/zones/external.example.com.zone
```

## 6. Restart BIND
Włącz i zrestartuj usługę BIND:
```bash
sudo systemctl enable named
sudo systemctl restart named
```

Sprawdź status:
```bash
sudo systemctl status named
```

## 7. Konfiguracja firewalla
Zezwól na ruch DNS przez firewall:
```bash
sudo firewall-cmd --permanent --add-service=dns
sudo firewall-cmd --reload
```

## 8. Testowanie
### Test sieci wewnętrznej
Uruchom to polecenie z maszyny w sieci wewnętrznej:
```bash
dig @localhost www.example.com
```
Oczekiwany wynik: IP z `internal.example.com.zone` (np. `192.168.1.2`)

### Test sieci zewnętrznej
Uruchom to polecenie z maszyny zewnętrznej:
```bash
dig @<PUBLICZNE_IP_SERWERA_BIND> www.example.com
```
Oczekiwany wynik: IP z `external.example.com.zone` (np. `203.0.113.2`)

## 9. Opcjonalnie: Włączenie logowania
### Dodaj konfigurację logowania
Edytuj `/etc/named.conf` aby dodać logowanie:
```conf
logging {
    channel query_log {
        file "/var/log/named_queries.log";
        severity dynamic;
    };
    category queries { query_log; };
};
```

### Utwórz plik logów
```bash
sudo touch /var/log/named_queries.log
sudo chmod 640 /var/log/named_queries.log
sudo chown named:named /var/log/named_queries.log
```

### Zrestartuj BIND
```bash
sudo systemctl restart named
```

Serwer Split-Horizon DNS jest teraz skonfigurowany i gotowy do użycia. W przypadku problemów, sprawdź dokładnie pliki konfiguracyjne i logi w celu rozwiązania problemów.

### Porady dotyczące rozwiązywania problemów
1. Sprawdź status usługi BIND używając `systemctl status named`
2. Przejrzyj logi systemowe: `journalctl -u named`
3. Upewnij się, że porty są otwarte: `netstat -tulpn | grep named`
4. Sprawdź uprawnienia do plików konfiguracyjnych
