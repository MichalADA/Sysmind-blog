+++
title = "Airbnb Clone"
date = "2025-05-05"
description = "Kompletny projekt CI/CD dla aplikacji React wykorzystującej najnowsze narzędzia DevOps"
tags = ["React", "DevOps", "AWS"]
categories = ["Projekty"]
draft = false
+++

# Airbnb Clone - Pipeline DevOps

## 📋 Opis projektu
Ten projekt zawiera kompletny zestaw narzędzi i konfiguracji do uruchomienia zautomatyzowanego pipeline'u CI/CD dla aplikacji React (klon Airbnb). Wykorzystuje architekturę opartą na AWS z serwerami Jenkins, bazą danych MySQL oraz systemem monitorowania Prometheus/Grafana.

## 🏗️ Architektura projektu
Projekt składa się z następujących komponentów:

- Aplikacja React: Klon Airbnb zbudowany w React
- Pipeline CI/CD: Automatyzacja procesu budowy, testów i wdrażania
- Infrastruktura: Skrypty Ansible do konfiguracji serwerów
- Konteneryzacja: Dockerfile i Docker Compose
- Monitoring: System Prometheus, Grafana i SmokePing
- Bezpieczeństwo: Skanowanie Trivy i analiza kodu SonarQube

## Infrastruktura AWS
Projekt wymaga następującej infrastruktury AWS:

- Węzeł kontrolny Ansible: Zarządza konfiguracją wszystkich pozostałych serwerów
- Serwer Jenkins: Obsługuje procesy CI/CD (budowanie, testowanie, wdrażanie)
- Serwer monitorujący: Hostuje Prometheus i Grafana do monitorowania
- Serwer bazy danych: Hostuje bazę danych MySQL dla aplikacji
- Serwer aplikacyjny: Uruchamia kontenery z aplikacją React

## 📁 Struktura folderów
```
REACT-AIRBNB-CLONE/
├── src/                       # Kod źródłowy aplikacji React
├── infrastructure/            # Pliki konfiguracyjne infrastruktury
│   └── ansible/
│       ├── ansible.sh         # Skrypt instalacyjny Ansible
│       ├── inventory.yml      # Konfiguracja hostów
│       └── playbooks/
│           ├── jenkins.yml    # Playbook instalacyjny Jenkins
│           ├── database.yml   # Playbook instalacyjny bazy danych
│           └── monitoring.yml # Playbook instalacyjny monitoringu
├── ci-cd/                     # Konfiguracja CI/CD
│   ├── Jenkinsfile            # Pipeline Jenkins
│   ├── Dockerfile             # Konfiguracja Docker
│   ├── docker-compose.yml     # Kompozycja usług Docker
│   └── app-setup.sh           # Skrypt konfiguracyjny serwera aplikacji
├── monitoring/                # Konfiguracja monitoringu
│   ├── prometheus.yml         # Konfiguracja Prometheus
│   └── grafana-dashboard-setup.sh # Konfiguracja dashboardu Grafana
└── docs/                      # Dokumentacja
    ├── DeploymentGuide.md     # Przewodnik wdrożenia
    ├── Pipeline-architecture.mmd # Diagram architektury
    └── README.md              # Ten plik
```

## 🚀 Fazy wdrożenia
Projekt podzielony jest na następujące fazy:

### Faza 1: Konfiguracja infrastruktury
- Uruchomienie instancji EC2 na AWS
- Instalacja Ansible na kontrolerze
- Konfiguracja serwerów (Jenkins, baza danych, monitoring)

### Faza 2: Konfiguracja CI/CD
- Instalacja wtyczek Jenkins
- Konfiguracja narzędzi (Docker, Node.js, Java)
- Konfiguracja pipeline'u z Jenkinsfile

### Faza 3: Monitoring
- Konfiguracja Prometheus
- Konfiguracja dashboardów Grafana
- Konfiguracja SmokePing

### Faza 4: Wdrożenie
- Zautomatyzowane wdrożenie przez Jenkins
- Konteneryzacja z Docker
- Weryfikacja wdrożenia

## 📋 Wymagania
AWS EC2 (min. 5 instancji):
- EC2 Węzeł kontrolny Ansible - Ubuntu 22.04, t3.micro (2 vCPU, 1 GB RAM)
- EC2 Jenkins - Ubuntu 22.04, t3.medium (2 vCPU, 4 GB RAM)
- EC2 Monitoring - Ubuntu 22.04, t3.small (2 vCPU, 2 GB RAM)
- EC2 Baza danych - Ubuntu 22.04, t3.small (2 vCPU, 2 GB RAM)
- EC2 Aplikacja - Ubuntu 22.04, t3.small (2 vCPU, 2 GB RAM)
- Odpowiednie grupy bezpieczeństwa AWS umożliwiające komunikację
- Ten sam klucz SSH do dostępu do wszystkich maszyn
- Konto Docker Hub

## 🛠️ Instalacja i wdrożenie
Pełne instrukcje instalacji można znaleźć w pliku DeploymentGuide.md.

### Przygotowanie infrastruktury
- Utwórz wszystkie wymagane instancje EC2 w AWS zgodnie z sekcją "Wymagania"
- Skonfiguruj grupy bezpieczeństwa, aby umożliwić komunikację między maszynami
- Zanotuj adresy IP wszystkich maszyn

### Konfiguracja Ansible
- Sklonuj to repozytorium na węzeł kontrolny Ansible:
```
git clone https://github.com/your-username/react-airbnb-clone.git
cd react-airbnb-clone
```
- Skonfiguruj serwer Ansible:
```
chmod +x infrastructure/ansible/ansible.sh
./infrastructure/ansible/ansible.sh
```
- Zaktualizuj plik inventory.yml rzeczywistymi adresami IP serwerów:
```
vi infrastructure/ansible/inventory.yml
```
- Uruchom playbooki Ansible, aby skonfigurować wszystkie serwery:
```
cd infrastructure/ansible
ansible-playbook -i inventory.yml jenkins.yml
ansible-playbook -i inventory.yml monitoring.yml
ansible-playbook -i inventory.yml database.yml
```

### Konfiguracja Jenkins
- Uzyskaj dostęp do Jenkins przez przeglądarkę: http://jenkins-server-ip:8080
- Zainstaluj sugerowane wtyczki oraz dodatkowe wymienione w DeploymentGuide.md
- Skonfiguruj pipeline, podając ścieżkę do repozytorium Git zawierającego Jenkinsfile

### Wykonanie pipeline'u
Po zakończeniu konfiguracji Jenkins automatycznie:
- Pobierze kod z repozytorium
- Zainstaluje zależności
- Uruchomi testy jednostkowe
- Przeprowadzi analizę kodu SonarQube
- Przeprowadzi skanowanie bezpieczeństwa z Trivy
- Zbuduje obraz Docker
- Wdroży na serwer aplikacyjny

## 📊 Monitoring
System monitorowania oparty jest na:
- Prometheus: Zbieranie metryk
- Grafana: Wizualizacja danych
- Node Exporter: Metryki systemowe
- SmokePing: Monitorowanie dostępności

Dostęp do dashboardów:
- Grafana: http://monitoring-server-ip:3000
- Prometheus: http://monitoring-server-ip:9090

## 🔒 Bezpieczeństwo
Pipeline zawiera automatyczne skanowanie bezpieczeństwa:
- Trivy: Skanowanie obrazów Docker
- SonarQube: Statyczna analiza kodu
- OWASP Dependency-Check: Sprawdzanie zależności

## 📝 Licencja
MIT

## 👨‍💻 Autor
MichalADA
