---
title: 'Helm Charts - Kompletny Przewodnik dla DevOps'
date: 2025-04-30
draft: false
tags: ['kubernetes', 'devops', 'helm', 'containers', 'cloud-native']
---

# Helm Charts - Kompletny Przewodnik dla DevOps


## 1. Czym są Helm Charts?

Helm Charts to pakiety zasobów Kubernetes, które można zainstalować w klastrze Kubernetes. Można je porównać do paczek `.deb` dla Debiana lub `.rpm` dla RedHat - są to gotowe, skonfigurowane i łatwe do instalacji aplikacje dla Kubernetes.

Główne zalety Helm Charts:
- **Powtarzalność** - ten sam chart zawsze tworzy te same zasoby
- **Konfigurowalność** - jeden chart może być używany do wielu środowisk dzięki parametryzacji
- **Łatwość aktualizacji** - prosta ścieżka do aktualizacji i wycofywania zmian
- **Zarządzanie zależnościami** - automatyczne wdrażanie innych wymaganych chartów

## 2. Struktura Helm Chart

Typowa struktura katalogu Helm Chart wygląda następująco:

```
mychart/
  ├── Chart.yaml              # Metadane chartu
  ├── values.yaml             # Domyślne wartości konfiguracyjne
  ├── templates/              # Szablony zasobów Kubernetes
  │   ├── _helpers.tpl        # Pomocnicze funkcje szablonów
  │   ├── deployment.yaml     # Szablon deploymentu
  │   ├── service.yaml        # Szablon usługi
  │   ├── ingress.yaml        # Szablon ingressu
  │   └── ...
  ├── charts/                 # Podcharty (zależności)
  ├── .helmignore             # Pliki ignorowane przez Helm
  └── README.md               # Dokumentacja chartu
```

## 3. Kluczowe pliki

### 3.1. Chart.yaml

Zawiera metadane opisujące chart, takie jak:
- `apiVersion` - wersja API Helm (v1 dla Helm 2, v2 dla Helm 3)
- `name` - nazwa chartu
- `version` - wersja chartu (np. 1.2.3)
- `appVersion` - wersja aplikacji (np. 1.0.0)
- `description` - opis chartu
- `maintainers` - lista opiekunów
- `dependencies` - zależności od innych chartów

Przykład:
```yaml
apiVersion: v2
name: my-app
version: 0.1.0
appVersion: "1.0.0"
description: A Helm chart for my application
```

### 3.2. values.yaml

Zawiera domyślne wartości konfiguracyjne, które mogą być nadpisane podczas instalacji:
- Skalowanie (liczba replik)
- Konfiguracja obrazu (repozytorium, tag)
- Zasoby (CPU, pamięć)
- Parametry sieciowe
- Zmienne środowiskowe

Przykład:
```yaml
replicaCount: 2
image:
  repository: example/app
  tag: latest
  pullPolicy: Always
service:
  type: ClusterIP
  port: 80
```

### 3.3. templates/_helpers.tpl

Zawiera funkcje pomocnicze używane przez inne szablony:
- Generowanie nazw (np. pełna nazwa aplikacji)
- Tworzenie wspólnych etykiet
- Obsługa selektorów
- Obsługa kont usługowych

Przykład:
```yaml
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

## 4. Szablony zasobów Kubernetes

### 4.1. Deployment (deployment.yaml)

Definiuje Deployment Kubernetes, który zarządza podami:
- Liczba replik
- Strategia aktualizacji
- Selektory podów
- Konfiguracja kontenerów
- Wolumeny
- Health checks

Przykład:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
```

### 4.2. Service (service.yaml)

Definiuje Service Kubernetes, który zapewnia dostęp do podów:
- Typ usługi (ClusterIP, NodePort, LoadBalancer)
- Selektory podów
- Porty (wewnętrzne i zewnętrzne)

Przykład:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.name" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
  selector:
    {{- include "my-app.selectorLabels" . | nindent 4 }}
```

### 4.3. ConfigMap (configuration.yaml)

Przechowuje konfigurację w postaci par klucz-wartość:
- Pliki konfiguracyjne
- Zmienne środowiskowe
- Flagi uruchomieniowe

Przykład:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-app.name" . }}
data:
  application.yaml: |-
    {{- toYaml .Values.configuration.data | nindent 4 }}
```

### 4.4. Ingress (ingress.yaml)

Definiuje reguły routingu HTTP/HTTPS do usług:
- Hosty wirtualne
- Ścieżki URL
- Konfiguracja TLS
- Adnotacje dla kontrolera Ingress

Przykład:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-app.name" . }}
spec:
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ include "my-app.name" . }}
                port:
                  number: {{ .Values.service.port }}
```

### 4.5. RBAC (security.yaml)

Definiuje uprawnienia dla podów:
- Role - zestaw uprawnień
- RoleBinding - przypisanie roli do konta usługowego
- ServiceAccount - tożsamość dla podów

Przykład:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: {{ include "my-app.name" . }}
rules:
  - apiGroups: [""]
    resources: ["services", "endpoints", "pods"]
    verbs: ["get", "list", "watch"]
```

## 5. Składnia szablonów Helm

Helm używa składni Go Templates do tworzenia dynamicznych szablonów:

### 5.1. Podstawowa składnia

- `{{ .Values.klucz }}` - dostęp do wartości z values.yaml
- `{{ include "nazwa.funkcji" . }}` - wywołanie funkcji pomocniczej
- `{{- }}` i `{{- }}` - usuwanie białych znaków przed i po
- `{{ .Release.Name }}` - dostęp do informacji o wydaniu
- `{{ .Chart.Name }}` - dostęp do informacji o charcie

### 5.2. Funkcje

- `default` - wartość domyślna (`{{ default "domyślna" .Values.opcja }}`)
- `toYaml` - konwersja do YAML (`{{ toYaml .Values.obiekt | nindent 2 }}`)
- `nindent` - wcięcie z nową linią (`{{ ... | nindent 4 }}`)
- `indent` - tylko wcięcie (`{{ ... | indent 4 }}`)
- `quote` - dodaje cudzysłowy (`{{ .Values.nazwa | quote }}`)
- `trunc` - przycina tekst (`{{ .Values.nazwa | trunc 63 }}`)

### 5.3. Konstrukcje warunkowe

```yaml
{{- if .Values.warunek }}
# Kod gdy warunek jest spełniony
{{- else }}
# Kod gdy warunek nie jest spełniony
{{- end }}
```

### 5.4. Pętle

```yaml
{{- range .Values.lista }}
- {{ . }}
{{- end }}

{{- range $klucz, $wartość := .Values.mapa }}
{{ $klucz }}: {{ $wartość }}
{{- end }}
```

## 6. Best Practices

### 6.1. Struktura zasobów

- Używaj prefiksów nazw, aby uniknąć konfliktów
- Grupuj powiązane zasoby w jednym pliku
- Używaj znaczników `---` do oddzielania zasobów

### 6.2. Formatowanie

- Używaj `nindent` zamiast `indent` dla lepszej czytelności
- Utrzymuj spójne wcięcia (zwykle 2 lub 4 spacje)
- Używaj `{{-` i `-}}` do kontrolowania białych znaków

### 6.3. Wartości

- Dostarczaj sensowne wartości domyślne
- Dokumentuj wszystkie wartości
- Grupuj powiązane wartości w struktury zagnieżdżone
- Używaj typów danych odpowiednich dla wartości

### 6.4. Etykiety i selektory

- Używaj wspólnych funkcji dla etykiet i selektorów
- Zawsze dołączaj standardowe etykiety Kubernetes
- Używaj adnotacji dla metadanych, które nie są selektorami

### 6.5. Health Checks

- Zawsze definiuj probes (readiness, liveness)
- Używaj odpowiednich parametrów timeoutów i opóźnień
- Oddzielaj porty dla health checks jeśli to możliwe

## 7. Przykładowy proces modernizacji Helm Chart

Podczas modernizacji Helm Charts warto zastosować następujące usprawnienia:

1. **Standaryzacja selektorów**:
   - Dodanie funkcji `selectorLabels` w _helpers.tpl
   - Usunięcie duplikacji selektorów w różnych plikach

2. **Poprawa formatowania**:
   - Zamiana `indent` na `nindent` dla lepszej czytelności
   - Ujednolicenie wcięć w całym projekcie

3. **Rozszerzenie konfigurowalności**:
   - Dodanie obsługi adnotacji dla wszystkich zasobów
   - Dodanie konfiguracji RBAC
   - Ulepszenie struktury konfiguracji Ingress

4. **Lepsza obsługa portów**:
   - Użycie nazw portów zamiast numerów w health checks
   - Poprawne odwołania do portów usług

5. **Wzbogacenie values.yaml**:
   - Dodanie sekcji dla wszystkich nowych funkcji
   - Ułatwienie konfiguracji przez użytkownika końcowego

## 8. Praca z Helm Charts

### 8.1. Instalacja chartu

```bash
helm install [nazwa-wydania] ./my-app
```

### 8.2. Aktualizacja chartu

```bash
helm upgrade [nazwa-wydania] ./my-app
```

### 8.3. Używanie własnych wartości

```bash
helm install [nazwa-wydania] ./my-app -f wlasne-wartosci.yaml
```

### 8.4. Walidacja szablonów

```bash
helm template ./my-app
```

### 8.5. Usuwanie zainstalowanego chartu

```bash
helm uninstall [nazwa-wydania]
```

## 9. Podsumowanie

Helm Charts to potężne narzędzie do standaryzacji i automatyzacji wdrażania aplikacji w Kubernetes. Dzięki zastosowaniu szablonów, charty pozwalają na:

- Wielokrotne używanie tej samej konfiguracji
- Łatwe dostosowanie do różnych środowisk
- Spójne zarządzanie całym cyklem życia aplikacji
- Lepszą organizację i dokumentację zasobów Kubernetes

W miarę jak chmury natywne stają się standardem w branży IT, umiejętność efektywnego zarządzania i tworzenia Helm Charts staje się niezbędna dla każdego specjalisty DevOps. Mam nadzieję, że ten przewodnik pomoże Wam w codziennej pracy z Kubernetes i przyspieszy Wasze wdrożenia!

---


