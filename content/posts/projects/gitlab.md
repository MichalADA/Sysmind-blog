+++

title = "Instalacja Gitlab"

date = "2025-02-10"

description = "Instalacja Gitlaba"

tags = ["Linux", "CI/CD"]

categories = ["Projekty"]

draft = false

+++


# Instrukcja instalacji GitLab CE na Ubuntu/Debian

## Wprowadzenie
Ten przewodnik przeprowadzi Cię przez proces instalacji GitLab Community Edition (CE) na serwerze Ubuntu/Debian. GitLab CE to darmowa, samodzielnie hostowana platforma do zarządzania kodem źródłowym i projektami.

## Wymagania systemowe

Przed rozpoczęciem instalacji upewnij się, że Twój serwer spełnia następujące wymagania:

- Ubuntu 20.04 lub nowszy / Debian 10 lub nowszy
- Minimum 4 GB RAM (zalecane 8 GB)
- Minimum 10 GB wolnego miejsca na dysku
- Dostęp root lub użytkownik z uprawnieniami sudo
- Skonfigurowany adres IP (publiczny lub prywatny)

## Instrukcja krok po kroku

### 1. Aktualizacja systemu

Najpierw zaktualizuj system operacyjny:

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Instalacja wymaganych pakietów

Zainstaluj niezbędne zależności:

```bash
sudo apt install -y curl openssh-server ca-certificates
```

Jeśli planujesz korzystać z powiadomień email, zainstaluj Postfix:

```bash
sudo apt install -y postfix
```

> **Uwaga**: Podczas instalacji Postfix wybierz opcję "Internet Site" i wprowadź nazwę swojej domeny.

### 3. Dodanie repozytorium GitLab

Dodaj oficjalne repozytorium GitLab:

```bash
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

### 4. Instalacja GitLab CE

Zainstaluj GitLab CE, określając docelowy URL:

```bash
sudo EXTERNAL_URL="http://gitlab.twojadomena.com" apt install -y gitlab-ce
```

> **Ważne**: Zamień `gitlab.twojadomena.com` na właściwy adres swojego serwera.

### 5. Podstawowa konfiguracja

Uruchom wstępną konfigurację:

```bash
sudo gitlab-ctl reconfigure
```

### 6. Pierwsze uruchomienie

1. Otwórz przeglądarkę i przejdź pod skonfigurowany adres
2. Ustaw hasło dla konta administratora (root)
3. Zaloguj się używając loginu `root` i ustawionego hasła

### 7. Przydatne komendy administracyjne

Zarządzanie usługą GitLab:

```bash
# Sprawdzenie statusu
sudo gitlab-ctl status

# Restart usługi
sudo gitlab-ctl restart

# Zatrzymanie usługi
sudo gitlab-ctl stop

# Uruchomienie usługi
sudo gitlab-ctl start
```

### 8. Konfiguracja SSL (opcjonalnie)

Aby zabezpieczyć GitLab przez HTTPS:

1. Edytuj plik konfiguracyjny:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

2. Dodaj lub zmodyfikuj następujące linie:
```ruby
external_url "https://gitlab.twojadomena.com"
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['admin@twojadomena.com']
```

3. Zastosuj zmiany:
```bash
sudo gitlab-ctl reconfigure
```

### 9. Tworzenie kopii zapasowych

Aby utworzyć kopię zapasową:

```bash
sudo gitlab-backup create
```

> **Informacja**: Kopie zapasowe są przechowywane w katalogu `/var/opt/gitlab/backups/`

## Rozwiązywanie problemów

Jeśli napotkasz problemy podczas instalacji:

1. Sprawdź logi:
```bash
sudo gitlab-ctl tail
```

2. Upewnij się, że wszystkie usługi działają:
```bash
sudo gitlab-ctl status
```

3. Zweryfikuj konfigurację:
```bash
sudo gitlab-ctl check-config
```

## Wsparcie

- Dokumentacja GitLab: https://docs.gitlab.com/
- Forum społeczności: https://forum.gitlab.com/
- Issue tracker: https://gitlab.com/gitlab-org/gitlab/issues

