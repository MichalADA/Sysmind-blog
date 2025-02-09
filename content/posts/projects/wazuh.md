+++

title = "Wazuh"

date = "2025-02-09"

description = "Instalacja Wazuha"

tags = ["Linux", "Cybersecurity"]

categories = ["Projekty"]

draft = false

+++
# Instalacja Wazuh - kompletny przewodnik dla początkujących

Cześć! W dzisiejszym wpisie przeprowadzę Cię przez proces instalacji Wazuh - potężnego narzędzia do monitorowania bezpieczeństwa. Niezależnie od tego, czy zabezpieczasz małą firmę czy własne projekty, Wazuh może znacząco poprawić Twoje bezpieczeństwo IT.

## Co to jest Wazuh?

Wazuh to wszechstronne narzędzie open-source, które łączy w sobie:
- Monitoring bezpieczeństwa w czasie rzeczywistym
- Wykrywanie włamań
- Monitorowanie integralności plików
- Reagowanie na zagrożenia
- Zgodność z przepisami (GDPR, PCI DSS)

## Wymagania systemowe

Zanim zaczniemy, upewnij się, że Twój serwer spełnia minimalne wymagania:

### Dla serwera Wazuh:
- CPU: minimum 4 rdzenie
- RAM: minimum 8GB (zalecane 16GB)
- Dysk: minimum 50GB (zalecane 100GB)
- System: Ubuntu 22.04 LTS (zalecany)

### Dla agentów Wazuh:
- CPU: 2 rdzenie
- RAM: 2GB
- Dysk: 20GB
- Wspierane systemy: Linux, Windows, macOS

## Krok 1: Przygotowanie systemu

Najpierw zaktualizujmy system i skonfigurujmy podstawowe zabezpieczenia:

```bash
# Aktualizacja systemu
sudo apt update && sudo apt upgrade -y

# Konfiguracja firewalla
sudo ufw allow 1514/tcp  # Komunikacja agent-server
sudo ufw allow 1515/tcp  # Rejestracja agentów
sudo ufw allow 443/tcp   # Panel webowy
sudo ufw enable
```

## Krok 2: Instalacja serwera Wazuh

Teraz przechodzimy do właściwej instalacji:

```bash
# Pobieranie instalatora
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Uruchomienie instalacji (opcja -a instaluje wszystkie komponenty)
sudo bash ./wazuh-install.sh -a
```

Po instalacji sprawdź, czy wszystkie usługi działają poprawnie:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

## Krok 3: Instalacja agentów

### Dla Ubuntu/Debian:
```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.5-1_amd64.deb
sudo WAZUH_MANAGER='ip_twojego_serwera' WAZUH_AGENT_NAME='nazwa_hosta' dpkg -i ./wazuh-agent_4.7.5-1_amd64.deb
sudo systemctl start wazuh-agent
```

### Dla Windows:
1. Pobierz instalator ze strony Wazuh
2. Uruchom instalator jako administrator
3. Podaj adres IP serwera Wazuh podczas instalacji

## Krok 4: Podstawowa konfiguracja

Główny plik konfiguracyjny serwera znajduje się w `/var/ossec/etc/ossec.conf`. Oto przykładowa podstawowa konfiguracja:

```xml
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <email_to>twoj@email.com</email_to>
    <smtp_server>smtp.twojserwer.com</smtp_server>
    <email_from>wazuh@twojadomena.com</email_from>
  </global>

  <syscheck>
    <frequency>43200</frequency>
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes">/var/www,/var/lib</directories>
  </syscheck>
</ossec_config>
```

## Krok 5: Pierwsze logowanie

4. Otwórz przeglądarkę i przejdź do `https://ip_twojego_serwera`
5. Zaloguj się używając domyślnych danych (znajdziesz je w `/var/ossec/etc/wazuh-dashboard/credentials.txt`)
6. Zmień hasło przy pierwszym logowaniu

## Najczęstsze problemy i rozwiązania

7. **Problem z połączeniem agenta:**
```bash
# Sprawdź status agenta
sudo systemctl status wazuh-agent

# Sprawdź logi
tail -f /var/ossec/logs/ossec.log
```

8. **Problemy z serwerem:**
```bash
# Weryfikacja statusu
sudo systemctl status wazuh-manager

# Sprawdzenie logów
tail -f /var/ossec/logs/ossec.log
```

## Co dalej?

Po podstawowej instalacji warto:
9. Skonfigurować dodatkowe reguły monitorowania
10. Ustawić powiadomienia (np. przez Slack lub email)
11. Skonfigurować regular backups
12. Dostosować dashboardy do swoich potrzeb

## Wskazówki bezpieczeństwa

13. Regularnie aktualizuj system i komponenty Wazuh
14. Zmień domyślne hasła
15. Ogranicz dostęp do panelu administracyjnego
16. Włącz uwierzytelnianie dwuskładnikowe
17. Regularnie twórz kopie zapasowe konfiguracji

## Przydatne linki

- [Oficjalna dokumentacja Wazuh](https://documentation.wazuh.com)
- [Forum społeczności Wazuh](https://groups.google.com/g/wazuh)
- [GitHub Wazuh](https://github.com/wazuh/wazuh)

Mam nadzieję, że ten przewodnik pomoże Ci w sprawnej instalacji Wazuh! Jeśli masz pytania, zostaw komentarz poniżej.

*Ostatnia aktualizacja: Luty 2025*