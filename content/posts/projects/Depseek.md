+++
title = "Kubernetes Setup: Ollama + OpenWebUI + DeepSeek-R1"
date = "2025-02-13"
description = "Wdrożenie Ollamy z modelem DeepSeek-R1 na klastrze Kubernetes"
tags = ["Linux", "Kubernetes", "AI"]
categories = ["Projekty"]
draft = false
+++

# Kubernetes Setup: Ollama + OpenWebUI + DeepSeek-R1

## Opis projektu

Ten projekt przedstawia wdrożenie Ollamy, OpenWebUI i modelu DeepSeek-R1 na klastrze Kubernetes. Jest to kompletne rozwiązanie umożliwiające uruchomienie własnego środowiska AI z interfejsem webowym.

## Wykorzystane narzędzia

- Kubernetes
- Devbox
- Taskfile
- Docker

## Architektura rozwiązania

Projekt składa się z trzech głównych komponentów:

- **Ollama** – backend AI obsługujący modele językowe
- **OpenWebUI** – przyjazny interfejs użytkownika do interakcji z modelami
- **DeepSeek-R1** – model językowy zoptymalizowany pod kątem wydajności

### Struktura projektu

```
.
├── manifests/
│   ├── Ingress.yaml
│   ├── Namespace.yaml
│   ├── Ollama.yaml
│   ├── OllamaService.yaml
│   ├── OpenWebUI.yaml
│   ├── Service.yaml
│   └── Volume.yaml
├── scripts/
├── devbox.json
├── devbox.lock
├── README.md
└── Taskfile.yaml
```

## Instrukcja instalacji

### Wymagania wstępne

Przed rozpoczęciem instalacji upewnij się, że masz zainstalowane:

- Klaster Kubernetes (np. Kind)
- Devbox
- Docker
- Task

### Krok po kroku

1. **Inicjalizacja i konfiguracja Devbox:**
   ```bash
   devbox init
   devbox add kubectl
   devbox add k9s
   devbox add helm
   devbox add task
   devbox add jq
   devbox add yq
   ```

2. **Uruchom środowisko Devbox:**
   ```bash
   devbox shell
   ```

2. **Utwórz dedykowaną przestrzeń nazw:**
   ```bash
   task create-namespace
   ```

3. **Wdróż wszystkie komponenty:**
   ```bash
   task deploy-all
   ```

4. **Poczekaj na pobranie modelu DeepSeek-R1:**
   ```bash
   task fetch-model
   ```

5. **Skonfiguruj przekierowanie portów:**
   ```bash
   task port-forward
   ```

6. **Otwórz aplikację w przeglądarce:**
   ```
   http://localhost:3000
   ```

## Przykłady działania

### Przykład odpowiedzi modelu na pytanie historyczne:
![DeepSeek - Odpowiedź na pytanie](https://azure-obliged-ox-268.mypinata.cloud/ipfs/bafkreiftk74p4yzp23vqqf7pikwr4qpofcsbrjpec2t26iwwn3offcliam)

### Generowanie kodu przez model:
![DeepSeek - Kod Pythona](https://azure-obliged-ox-268.mypinata.cloud/ipfs/bafkreiaxr3il5fkltizusxs4ukubvuhdeph5ht2l365ecbhrjrsbgjeafa)

## Konfiguracja zadań (Taskfile)

Poniżej znajduje się kompletna konfiguracja zadań w pliku `Taskfile.yaml`:

```yaml
version: "3"

tasks:
  create-namespace:
    desc: "Create namespace"
    cmds:
      - kubectl create namespace ai-stack || true

  deploy-volume:
    desc: "Deploy volume"
    cmds:
      - kubectl apply -f manifests/Volume.yaml -n ai-stack

  deploy-ollama:
    desc: "Deploy Ollama"
    cmds:
      - kubectl apply -f manifests/Ollama.yaml -n ai-stack

  deploy-openwebui:
    desc: "Deploy OpenWebUI"
    cmds:
      - kubectl apply -f manifests/OpenWebUI.yaml -n ai-stack

  deploy-service:
    desc: "Deploy Service"
    cmds:
      - kubectl apply -f manifests/Service.yaml -n ai-stack
  
  deploy-service-ollama:
    desc: "Deploy Service-ollama"
    cmds:
      - kubectl apply -f manifests/OllamaService.yaml -n ai-stack
  
  deploy-ingress:
    desc: "Deploy Ingress"
    cmds:
      - kubectl apply -f manifests/Ingress.yaml -n ai-stack

  fetch-model:
    desc: "Pobranie modelu DeepSeek dla Ollamy"
    cmds:
      - |
        echo "Czekam na start kontenera Ollama..."
        until kubectl get pod -n ai-stack -l app=ollama -o jsonpath="{.items[0].status.phase}" | grep -q "Running"; do
          sleep 5
          echo "Czekam dalej..."
        done
        echo "Kontener uruchomiony, pobieram model..."
        kubectl exec -n ai-stack $(kubectl get pod -n ai-stack -l app=ollama -o jsonpath="{.items[0].metadata.name}") -- bash -c "
          ollama run deepseek-r1:1.5b"
    silent: false

  port-forward:
    desc: "Port forwarding for OpenWebUI and Ollama"
    cmds:
      - kubectl port-forward -n ai-stack svc/openwebui 3000:80 &
      - kubectl port-forward -n ai-stack svc/ollama 11434:11434 &
    silent: false

  deploy-all:
    desc: "Deploy all components"
    cmds:
      - task: create-namespace
      - task: deploy-volume
      - task: deploy-ollama
      - task: deploy-openwebui
      - task: deploy-service
      - task: deploy-service-ollama
      - task: deploy-ingress
      - task: fetch-model

  clean:
    desc: "Clean up the environment"
    cmds:
      - kubectl delete namespace ai-stack
```

## Rozwiązywanie problemów

1. **Sprawdzanie statusu wdrożenia:**
   ```bash
   kubectl get pods -n ai-stack
   ```

2. **Sprawdzanie logów:**
   ```bash
   kubectl logs -n ai-stack -l app=openwebui
   kubectl logs -n ai-stack -l app=ollama
   ```

3. **Wymagania sprzętowe:**
   - Minimum 8GB RAM
   - 4 rdzenie CPU
   - 20GB przestrzeni dyskowej

## Uwagi końcowe

- Model DeepSeek-R1 jest zoptymalizowany pod kątem wydajności, ale może działać wolniej na słabszych maszynach
- Model jak widac dziala , troche zajmuje mu odpowiadanie na pytania 
- W przypadku problemów z dostępem do interfejsu, sprawdź status przekierowania portów
- Regularnie monitoruj zużycie zasobów klastra

---

 
🚀 **Powodzenia w implementacji!**
