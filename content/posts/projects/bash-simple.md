+++
title = "Automonitoring serwera Linux - prosty projekt bash"
date = "2025-05-05"
description = "Praktyczny projekt: tworzenie skryptu bash do monitorowania zasobów serwera Linux z powiadomieniami e-mail"
tags = ["Linux", "Bash", "Monitoring", "Serwer"]
categories = ["Projekty"]
draft = false
+++

# Automonitoring serwera Linux - prosty projekt bash

## 📋 Opis projektu

Ten projekt zawiera skrypt bash do monitorowania podstawowych zasobów serwera Linux (CPU, pamięć RAM, przestrzeń dyskowa) oraz wysyłania powiadomień e-mail w przypadku przekroczenia zdefiniowanych progów. To świetne narzędzie dla administratorów serwerów oraz entuzjastów Linuksa, którzy chcą mieć pod kontrolą wydajność swoich systemów.

## 🎯 Cele projektu

* **Monitoring podstawowych zasobów**: Śledzenie wykorzystania CPU, pamięci RAM i przestrzeni dyskowej
* **Automatyczne powiadomienia**: Wysyłanie alertów e-mail w przypadku przekroczenia progów
* **Prostota wdrożenia**: Łatwy w implementacji skrypt bash bez dodatkowych zależności
* **Możliwość rozbudowy**: Modułowa struktura umożliwiająca łatwe rozszerzanie funkcjonalności

## 💻 Wymagania

* Serwer Linux (testowano na Ubuntu, Debian, CentOS)
* Bash (dostępny domyślnie w większości dystrybucji Linux)
* Dostęp do polecenia mail (lub podobnego) do wysyłania e-maili
* Podstawowa znajomość edytorów tekstu (nano, vim, itp.)
* Uprawnienia do ustawiania zadań cron

## 📝 Implementacja

### Krok 1: Tworzenie skryptu monitorującego

Utwórz plik `system_monitor.sh` i otwórz go w edytorze:

```bash
#!/bin/bash

# Konfiguracja
EMAIL="twoj-email@domena.pl"
SUBJECT="Alert: Problem na serwerze $(hostname)"
LOG_FILE="/var/log/system_monitor.log"

# Progi alertów (w procentach)
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80

# Funkcja do wysyłania powiadomień
send_alert() {
    local resource=$1
    local usage=$2
    local threshold=$3
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    
    echo "$timestamp - ALERT: $resource użycie przekroczyło próg! Obecne użycie: $usage%, Próg: $threshold%" >> "$LOG_FILE"
    
    # Przygotowanie wiadomości e-mail
    local email_body="
    Wykryto przekroczenie progu wykorzystania zasobów na serwerze $(hostname)
    
    Zasób: $resource
    Obecne użycie: $usage%
    Próg: $threshold%
    Czas: $timestamp
    
    Prosimy o sprawdzenie serwera.
    "
    
    # Wysłanie e-maila
    echo "$email_body" | mail -s "$SUBJECT" "$EMAIL"
}

# Funkcja do sprawdzania CPU
check_cpu() {
    # Pobieranie aktualnego użycia CPU
    local cpu_idle=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/")
    local cpu_usage=$(echo "100 - $cpu_idle" | bc)
    cpu_usage=${cpu_usage%.*} # Usunięcie części dziesiętnej
    
    echo "Aktualne użycie CPU: $cpu_usage%"
    
    # Sprawdzenie progu
    if [ "$cpu_usage" -ge "$CPU_THRESHOLD" ]; then
        send_alert "CPU" "$cpu_usage" "$CPU_THRESHOLD"
    fi
}

# Funkcja do sprawdzania pamięci RAM
check_memory() {
    # Pobieranie aktualnego użycia pamięci
    local memory_usage=$(free | awk '/Mem/ {printf("%.1f", $3/$2 * 100)}')
    memory_usage=${memory_usage%.*} # Usunięcie części dziesiętnej
    
    echo "Aktualne użycie pamięci: $memory_usage%"
    
    # Sprawdzenie progu
    if [ "$memory_usage" -ge "$MEMORY_THRESHOLD" ]; then
        send_alert "Pamięć" "$memory_usage" "$MEMORY_THRESHOLD"
    fi
}

# Funkcja do sprawdzania miejsca na dysku
check_disk() {
    # Pobieranie aktualnego użycia dysku dla głównej partycji
    local disk_usage=$(df -h / | awk '/\// {print $(NF-1)}')
    disk_usage=${disk_usage%\%} # Usunięcie znaku procentu
    
    echo "Aktualne użycie dysku: $disk_usage%"
    
    # Sprawdzenie progu
    if [ "$disk_usage" -ge "$DISK_THRESHOLD" ]; then
        send_alert "Dysk" "$disk_usage" "$DISK_THRESHOLD"
    fi
}

# Główna funkcja
main() {
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    echo "-----------------------------------" >> "$LOG_FILE"
    echo "$timestamp - Uruchomienie monitoringu systemu" >> "$LOG_FILE"
    
    # Sprawdzenie zasobów
    check_cpu
    check_memory
    check_disk
    
    echo "$timestamp - Zakończenie monitoringu systemu" >> "$LOG_FILE"
    echo "-----------------------------------" >> "$LOG_FILE"
}

# Uruchomienie głównej funkcji
main
```

### Krok 2: Nadanie uprawnień wykonywania

```bash
chmod +x system_monitor.sh
```

### Krok 3: Konfiguracja automatycznego uruchamiania (cron)

Otwórz edytor crontab:

```bash
crontab -e
```

Dodaj linię, aby uruchamiać skrypt co 15 minut:

```
*/15 * * * * /ścieżka/do/system_monitor.sh
```

## 🚀 Rozbudowa projektu

Skrypt można z łatwością rozbudować o dodatkowe funkcjonalności:

### Monitorowanie usług

Dodaj do skryptu funkcję sprawdzającą, czy kluczowe usługi są uruchomione:

```bash
check_service() {
    local service_name=$1
    systemctl is-active --quiet "$service_name"
    if [ $? -ne 0 ]; then
        send_alert "Usługa" "$service_name nie działa" "Aktywna"
        # Opcjonalnie - automatyczny restart usługi
        systemctl restart "$service_name"
    fi
}

# Przykład użycia w funkcji main
check_service "nginx"
check_service "mysql"
```

### Monitoring ataku brute force

Dodaj sprawdzanie prób logowania:

```bash
check_brute_force() {
    local attempts=$(grep "Failed password" /var/log/auth.log | wc -l)
    if [ "$attempts" -gt 10 ]; then
        send_alert "Bezpieczeństwo" "$attempts nieudanych prób logowania" "10"
    fi
}
```

### Dashboard w HTML

Możesz również rozszerzyć skrypt, aby generował prosty raport HTML:

```bash
generate_html_report() {
    local report_file="/var/www/html/status.html"
    local timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    
    cat > "$report_file" << EOF
<!DOCTYPE html>
<html>
<head>
    <title>Status serwera $(hostname)</title>
    <meta charset="UTF-8">
    <meta http-equiv="refresh" content="300">
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .status { padding: 10px; margin: 10px 0; border-radius: 5px; }
        .ok { background-color: #d4edda; }
        .warning { background-color: #fff3cd; }
        .critical { background-color: #f8d7da; }
    </style>
</head>
<body>
    <h1>Status serwera $(hostname)</h1>
    <p>Ostatnia aktualizacja: $timestamp</p>
    
    <div class="status $([ "$cpu_usage" -ge "$CPU_THRESHOLD" ] && echo "critical" || echo "ok")">
        <h2>CPU</h2>
        <p>Użycie: $cpu_usage%</p>
    </div>
    
    <div class="status $([ "$memory_usage" -ge "$MEMORY_THRESHOLD" ] && echo "critical" || echo "ok")">
        <h2>Pamięć RAM</h2>
        <p>Użycie: $memory_usage%</p>
    </div>
    
    <div class="status $([ "$disk_usage" -ge "$DISK_THRESHOLD" ] && echo "critical" || echo "ok")">
        <h2>Dysk</h2>
        <p>Użycie: $disk_usage%</p>
    </div>
</body>
</html>
EOF
}
```

## 📊 Wnioski

Ten prosty skrypt monitorujący może okazać się nieoceniony dla każdego administratora systemu Linux. Dzięki niemu będziesz otrzymywać natychmiastowe powiadomienia o potencjalnych problemach, zanim staną się one krytyczne. 

Skrypt jest również dobrym punktem wyjścia do nauki i rozwijania umiejętności w zakresie programowania w bashu oraz administracji systemami Linux.

## 📝 Zastrzeżenia

Ten skrypt jest przeznaczony głównie do celów edukacyjnych i podstawowego monitoringu. W środowiskach produkcyjnych warto rozważyć bardziej zaawansowane rozwiązania monitorujące, takie jak Nagios, Zabbix czy Prometheus.
