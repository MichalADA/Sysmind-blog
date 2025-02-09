+++

title = "SElinux-Wprowadzenie"

date = "2025-02-09"

description = "Podstawy SElinuxa"

tags = ["Linux", "Cybersecurity"]

categories = ["Projekty"]

draft = false

+++



# SELinux - Kompleksowy przewodnik

## Spis treści
- [Wprowadzenie do SELinux](#wprowadzenie-do-selinux)
- [Architektura SELinux](#architektura-selinux)
- [Tryby pracy](#tryby-pracy)
- [Konteksty bezpieczeństwa](#konteksty-bezpieczeństwa)
- [Polityki SELinux](#polityki-selinux)
- [Podstawowe komendy](#podstawowe-komendy)
- [Booleany](#booleany)
- [Rozwiązywanie problemów](#rozwiązywanie-problemów)
- [Najlepsze praktyki](#najlepsze-praktyki)

## Wprowadzenie do SELinux

Security-Enhanced Linux (SELinux) to zaawansowany system kontroli dostępu (MAC - Mandatory Access Control) zaimplementowany w jądrze Linux. Został pierwotnie opracowany przez NSA i później zintegrowany z wieloma dystrybucjami Linux.

### Główne cechy:
- Kontrola dostępu na poziomie jądra systemu
- Szczegółowa polityka bezpieczeństwa
- Ochrona przed eskalacją uprawnień
- Izolacja procesów i usług
- Minimalizacja potencjalnych zagrożeń

## Architektura SELinux

SELinux działa na zasadzie etykietowania (labeling) wszystkich obiektów w systemie i kontrolowania dostępu między nimi na podstawie zdefiniowanych reguł.

### Komponenty:
- Kernel SELinux
- Polityki bezpieczeństwa
- Narzędzia użytkownika
- System logowania

```bash
# Sprawdzenie czy SELinux jest zainstalowany
rpm -qa | grep selinux
```

## Tryby pracy

SELinux może działać w trzech trybach, które określają jak system reaguje na naruszenia polityki.

### 1. Enforcing
- Polityka jest aktywnie egzekwowana
- Naruszenia są blokowane i logowane
- Zalecany w środowisku produkcyjnym

```bash
# Włączenie trybu Enforcing
sudo setenforce 1
```

### 2. Permissive
- Polityka nie jest egzekwowana
- Naruszenia są tylko logowane
- Przydatny do debugowania i testowania

```bash
# Włączenie trybu Permissive
sudo setenforce 0
```

### 3. Disabled
- SELinux jest całkowicie wyłączony
- Brak kontroli dostępu
- Niezalecany w produkcji

```bash
# Sprawdzenie aktualnego trybu
sestatus
```

## Konteksty bezpieczeństwa

Każdy obiekt w systemie (pliki, procesy, porty) posiada kontekst bezpieczeństwa składający się z czterech elementów:

### Format kontekstu:
```
user:role:type:level
```

- **user**: Tożsamość SELinux
- **role**: Przypisana rola w systemie
- **type**: Typ obiektu lub domeny
- **level**: Poziom bezpieczeństwa (używany w MLS)

```bash
# Sprawdzenie kontekstu pliku
ls -Z /ścieżka/do/pliku

# Sprawdzenie kontekstu procesu
ps auxZ | grep httpd
```

### Zarządzanie kontekstami:

```bash
# Zmiana kontekstu pliku
sudo chcon -t httpd_sys_content_t /var/www/html/index.html

# Przywrócenie domyślnego kontekstu
sudo restorecon -Rv /var/www/html/

# Dodanie reguły trwałej
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/custom(/.*)?"
```

## Polityki SELinux

SELinux oferuje różne typy polityk dostosowane do różnych przypadków użycia.

### Targeted (domyślna)
- Chroni tylko wybrane usługi systemowe
- Najprostsza w konfiguracji
- Zalecana dla większości przypadków

### MLS (Multi-Level Security)
- Zaawansowana kontrola dostępu
- Używana w środowiskach o wysokich wymaganiach bezpieczeństwa
- Wymaga szczegółowej konfiguracji

```bash
# Sprawdzenie aktywnej polityki
sestatus | grep "Loaded policy"

# Lista dostępnych polityk
semodule -l
```

## Podstawowe komendy

### Sprawdzanie statusu i konfiguracji:
```bash
# Status SELinux
sestatus

# Aktywne porty
semanage port -l

# Lista modułów polityki
semodule -l
```

### Zarządzanie kontekstami:
```bash
# Zmiana kontekstu
chcon -t httpd_sys_content_t plik.html

# Przywracanie domyślnych kontekstów
restorecon -Rv /ścieżka

# Dodawanie reguł trwałych
semanage fcontext -a -t httpd_sys_content_t "/custom/path(/.*)?"
```

## Booleany

Booleany to przełączniki umożliwiające szybką modyfikację zachowania polityki SELinux.

```bash
# Lista wszystkich booleanów
getsebool -a

# Sprawdzenie konkretnego booleana
getsebool httpd_enable_homedirs

# Włączenie booleana (tymczasowo)
setsebool httpd_enable_homedirs on

# Włączenie booleana (trwale)
setsebool -P httpd_enable_homedirs on
```

### Popularne booleany:
- `httpd_enable_homedirs`: Dostęp Apache do katalogów domowych
- `ftpd_anon_write`: Zapis przez anonimowego FTP
- `httpd_can_network_connect`: Połączenia sieciowe dla Apache

## Rozwiązywanie problemów

### Analiza logów:
```bash
# Przeszukiwanie logów audytu
ausearch -m AVC -ts recent

# Analiza naruszeń
audit2why < /var/log/audit/audit.log

# Generowanie modułu polityki
audit2allow -a -M mymodule
```

### Typowe problemy:
1. Nieprawidłowe konteksty plików
2. Brakujące porty w polityce
3. Wymagane booleany są wyłączone
4. Konflikty z innymi mechanizmami bezpieczeństwa

### Rozwiązywanie:
```bash
# Sprawdzenie kontekstów
ls -Z

# Dodanie portu
semanage port -a -t http_port_t -p tcp 8080

# Włączenie wymaganego booleana
setsebool -P httpd_can_network_connect on
```

## Najlepsze praktyki

1. **Używaj trybu Enforcing**
   - Zawsze w środowisku produkcyjnym
   - Permissive tylko do debugowania

2. **Regularnie monitoruj logi**
   ```bash
   ausearch -m AVC -ts recent
   ```

3. **Twórz własne moduły polityki**
   ```bash
   audit2allow -a -M custom_policy
   semodule -i custom_policy.pp
   ```

4. **Dokumentuj zmiany**
   - Zapisuj wszystkie modyfikacje kontekstów
   - Przechowuj listę własnych modułów polityki

5. **Automatyzuj zarządzanie**
   - Używaj narzędzi do zarządzania konfiguracją
   - Twórz skrypty do typowych operacji

6. **Regularne aktualizacje**
   ```bash
   # Aktualizacja polityk
   yum update selinux-policy
   ```

### Przykładowa automatyzacja:
```bash
#!/bin/bash
# Skrypt do konfiguracji kontekstów dla nowej aplikacji web

# Ustawienie kontekstów
semanage fcontext -a -t httpd_sys_content_t "/var/www/app(/.*)?"
restorecon -Rv /var/www/app

# Włączenie wymaganych booleanów
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on

# Sprawdzenie konfiguracji
sestatus
getsebool -a | grep httpd
ls -Z /var/www/app
```