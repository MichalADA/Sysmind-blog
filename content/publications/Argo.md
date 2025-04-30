---
title: 'Argo CD - Kompletny Przewodnik dla DevOps'
date: 2025-04-30
draft: false
tags: ['kubernetes', 'devops', 'argocd', 'gitops', 'cicd', 'cloud-native']
---

# Argo CD - Kompletny Przewodnik dla DevOps

Dzisiaj jest 30.04.2025, a ja chciałbym podzielić się z Wami kompleksowym przewodnikiem po Argo CD - narzędziu, które zrewolucjonizowało wdrażanie aplikacji w Kubernetes dzięki podejściu GitOps.

## 1. Czym jest Argo CD?

Argo CD to deklaratywne narzędzie GitOps do ciągłego dostarczania (CD) zaprojektowane dla Kubernetes. Działa jako kontroler Kubernetes, który stale monitoruje bieżący stan aplikacji w klastrze i porównuje go z pożądanym stanem zdefiniowanym w repozytorium Git.

Główne zalety Argo CD:

- **GitOps** - Git jako single source of truth dla stanu infrastruktury
- **Automatyzacja** - automatyczna synchronizacja stanu klastra z repozytorium
- **Audytowalność** - pełna historia zmian zapisana w Git
- **Samonaprawa** - automatyczne wykrywanie i naprawianie dryfu konfiguracji
- **Wieloklastrowość** - zarządzanie aplikacjami na wielu klastrach z jednego miejsca
- **Bezpieczeństwo** - RBAC i integracja z SSO

## 2. Architektura Argo CD

Argo CD składa się z kilku kluczowych komponentów:

```
argocd/
  ├── API Server          # Serwer API RESTful i gRPC
  ├── Repository Server   # Serwer do obsługi repozytoriów Git
  ├── Application Controller  # Kontroler główny do synchronizacji aplikacji
  ├── Dex                 # Opcjonalna integracja z SSO
  └── Redis               # Pamięć podręczna
```

### 2.1. API Server

API Server zapewnia następujące funkcje:

- Udostępnia interfejs API (RESTful i gRPC) dla klientów (UI, CLI, CI)
- Implementuje uwierzytelnianie i autoryzację
- Zarządza konfiguracją aplikacji i projektów
- Udostępnia interfejs webowy

### 2.2. Repository Server

Repository Server jest odpowiedzialny za:

- Obsługę połączeń z repozytoriami Git
- Buforowanie zawartości repozytoriów
- Generowanie manifestów Kubernetes z szablonów (Helm, Kustomize)
- Weryfikację zmian w aplikacjach

### 2.3. Application Controller

Application Controller jest sercem Argo CD i odpowiada za:

- Monitorowanie stanu aplikacji w klastrze
- Porównywanie stanu bieżącego ze stanem pożądanym
- Wykonywanie synchronizacji aplikacji
- Zgłaszanie statusu aplikacji do API Server

## 3. Kluczowe koncepcje

### 3.1. Aplikacja (Application)

Aplikacja to podstawowe pojęcie w Argo CD, reprezentujące grupę zasobów Kubernetes, które należą do jednej aplikacji. Aplikacja definiuje:

- Źródło manifestów (repozytorium Git, ścieżka)
- Docelowy klaster i przestrzeń nazw
- Strategię synchronizacji
- Ustawienia promocji

Przykład:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 3.2. Projekt (AppProject)

Projekt to logiczne grupowanie aplikacji Argo CD, które definiuje:

- Dozwolone źródła (repozytoria Git)
- Dozwolone docelowe klastry i przestrzenie nazw
- Rodzaje zasobów, które mogą być wdrażane
- RBAC dla członków projektu

Przykład:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-alpha
  namespace: argocd
spec:
  description: Projekt dla Zespołu Alpha
  sourceRepos:
  - https://github.com/orgs/team-alpha/*
  destinations:
  - namespace: team-alpha-*
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  roles:
  - name: developer
    description: Developer role
    policies:
    - p, proj:team-alpha:developer, applications, get, team-alpha/*, allow
```

### 3.3. ConfigManagement (Narzędzia konfiguracji)

Argo CD obsługuje różne narzędzia konfiguracji:

- **Kustomize** - nakładki na manifesty Kubernetes
- **Helm** - pakiety dla Kubernetes
- **Jsonnet** - szablonowanie JSON z logiką
- **Zwykłe manifesty YAML** - statyczne pliki
- **Plugin** - własne generatory manifestów

## 4. Synchronizacja aplikacji

### 4.1. Strategie synchronizacji

Argo CD oferuje różne strategie synchronizacji:

- **Manualna** - synchronizacja tylko na żądanie
- **Automatyczna** - synchronizacja przy każdej zmianie
- **Automatyczna z self-healing** - automatyczne naprawianie dryfu konfiguracji

### 4.2. Strategia pruning

Pruning określa, czy Argo CD powinno usuwać zasoby, które nie są już zdefiniowane w repozytorium:

- **Włączone** - usuwa zasoby, które nie istnieją w Git
- **Wyłączone** - zachowuje zasoby, nawet jeśli zostały usunięte z Git

### 4.3. Kolejność synchronizacji

Argo CD umożliwia kontrolowanie kolejności synchronizacji zasobów:

- **Wave** - grupowanie zasobów w fale synchronizacji (metadata.annotations."argocd.argoproj.io/sync-wave")
- **Hook** - definiowanie haków pre/post synchronizacji (metadata.annotations."argocd.argoproj.io/hook")

Przykład adnotacji wave:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "5"
```

Przykład hooka:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

## 5. Zarządzanie konfiguracją

### 5.1. Konfiguracja aplikacji

Konfiguracja aplikacji może być zarządzana za pomocą:

- **values.yaml** - dla chartów Helm
- **kustomization.yaml** - dla Kustomize
- **environment overlays** - dla różnych środowisk
- **ConfigMaps i Secrets** - dla danych konfiguracyjnych

### 5.2. Zarządzanie sekretami

Argo CD obsługuje różne metody zarządzania sekretami:

- **Sealed Secrets** - szyfrowanie sekretów w Git
- **Vault** - integracja z HashiCorp Vault
- **SOPS** - szyfrowanie plików konfiguracyjnych
- **Bitnami Sealed Secrets** - kontroler CRD dla szyfrowanych sekretów
- **AWS Secrets Manager** - integracja z AWS

Przykład użycia Sealed Secrets:

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: mynamespace
spec:
  encryptedData:
    password: AgBy8hCJVQXXXXXXXXXX
```

## 6. Argo CD w praktyce

### 6.1. Instalacja Argo CD

Instalacja podstawowa:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Instalacja z Helm:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd --create-namespace
```

### 6.2. Dostęp do interfejsu webowego

Po instalacji, interfejs webowy jest dostępny przez port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Lub przez Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-secret
```

### 6.3. Zarządzanie aplikacjami z CLI

Instalacja CLI:

```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

Logowanie do Argo CD:

```bash
argocd login argocd.example.com
```

Tworzenie aplikacji:

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Synchronizacja aplikacji:

```bash
argocd app sync guestbook
```

### 6.4. Aplikacje jako definicje aplikacji

Argo CD umożliwia definiowanie aplikacji jako pliki YAML w repozytorium Git:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
```

### 6.5. App of Apps Pattern

Wzorzec "App of Apps" pozwala na zarządzanie wieloma aplikacjami za pomocą jednej aplikacji Argo CD:

```
apps-repo/
  ├── apps/
  │   ├── templates/
  │   │   ├── app1.yaml     # Definicja aplikacji 1
  │   │   ├── app2.yaml     # Definicja aplikacji 2
  │   │   └── app3.yaml     # Definicja aplikacji 3
  │   └── values.yaml       # Wspólne wartości
  └── Chart.yaml            # Metadane chartu
```

## 7. Integracja z CI/CD

### 7.1. GitOps z Jenkins

Integracja Argo CD z Jenkins:

1. Jenkins buduje i testuje aplikację
2. Jenkins aktualizuje wartości w repozytorium GitOps
3. Argo CD wykrywa zmiany i wdraża nową wersję

```groovy
pipeline {
    agent any
    stages {
        stage('Build and Test') {
            steps {
                sh 'docker build -t myapp:$GIT_COMMIT .'
                sh 'docker push myapp:$GIT_COMMIT'
            }
        }
        stage('Update GitOps Repo') {
            steps {
                sh '''
                git clone https://github.com/myorg/gitops.git
                cd gitops
                yq e '.image.tag = "$GIT_COMMIT"' -i apps/myapp/values.yaml
                git commit -am "Update myapp to $GIT_COMMIT"
                git push
                '''
            }
        }
    }
}
```

### 7.2. GitOps z GitHub Actions

Integracja Argo CD z GitHub Actions:

```yaml
name: Build and Update GitOps

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v3
      with:
        push: true
        tags: myapp:${{ github.sha }}
        
    - name: Update GitOps repository
      uses: actions/checkout@v3
      with:
        repository: myorg/gitops
        token: ${{ secrets.PAT }}
        path: gitops
        
    - name: Update image tag
      run: |
        cd gitops
        yq e '.image.tag = "${{ github.sha }}"' -i apps/myapp/values.yaml
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git commit -am "Update myapp to ${{ github.sha }}"
        git push
```

## 8. Zaawansowane funkcje

### 8.1. Health Checks

Argo CD umożliwia monitorowanie stanu zdrowia aplikacji:

- **Wbudowane health checks** dla standardowych zasobów Kubernetes
- **Custom health checks** dla niestandardowych zasobów
- **Lua scripts** dla zaawansowanej logiki

Przykład niestandardowego health check:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
spec:
  # ...
  health:
    - resource:
        group: apps
        kind: Deployment
      jsonPointers:
        - /status/availableReplicas
        - /spec/replicas
```

### 8.2. Resource Hooks

Resource Hooks pozwalają na wykonywanie operacji przed, w trakcie i po synchronizacji:

- **PreSync** - przed synchronizacją
- **Sync** - podczas synchronizacji
- **PostSync** - po synchronizacji
- **SyncFail** - gdy synchronizacja nie powiedzie się

Przykład:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: migration
        image: myapp-migration:latest
      restartPolicy: Never
```

### 8.3. Notification Controller

Argo CD Notification Controller umożliwia wysyłanie powiadomień o zdarzeniach:

- Powiadomienia o zmianach statusu aplikacji
- Integracja z Slack, Teams, Email, Webhook
- Konfigurowane triggery i szablony

Przykład:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  annotations:
    notifications.argoproj.io/subscribe.on-sync-succeeded.slack: devops-alerts
```

### 8.4. ApplicationSet

ApplicationSet to kontroler do automatycznego generowania wielu aplikacji Argo CD:

- **Generators** - tworzenie aplikacji na podstawie szablonów
- **Multi-cluster** - wdrażanie na wielu klastrach
- **Multi-tenant** - zarządzanie wieloma zespołami

Przykład:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
spec:
  generators:
  - list:
      elements:
      - cluster: production
        url: https://kubernetes.example.com
      - cluster: staging
        url: https://kubernetes.staging.example.com
  template:
    metadata:
      name: '{{cluster}}-guestbook'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/argocd-example-apps.git
        targetRevision: HEAD
        path: guestbook
      destination:
        server: '{{url}}'
        namespace: guestbook
```

## 9. Best Practices

### 9.1. Organizacja repozytoriów

Zalecane podejścia do organizacji repozytoriów:

- **Monorepo** - wszystkie aplikacje w jednym repozytorium
- **Repo per app** - osobne repozytorium dla każdej aplikacji
- **Repo per team** - repozytorium dla każdego zespołu
- **Repo per environment** - osobne repozytoria dla różnych środowisk

### 9.2. Struktura katalogu GitOps

Przykładowa struktura katalogu GitOps:

```
gitops/
  ├── apps/                  # Definicje aplikacji
  │   ├── app1/
  │   │   ├── base/          # Podstawowa konfiguracja
  │   │   ├── overlays/      # Nakładki dla środowisk
  │   │   │   ├── dev/
  │   │   │   ├── staging/
  │   │   │   └── prod/
  │   ├── app2/
  │   └── ...
  ├── clusters/              # Konfiguracja klastrów
  │   ├── dev/
  │   ├── staging/
  │   └── prod/
  └── platform/              # Komponenty platformy
      ├── monitoring/
      ├── logging/
      └── security/
```

### 9.3. Strategie promocji

Strategie promocji aplikacji między środowiskami:

- **GitOps Promotion** - aktualizacja Git dla każdego środowiska
- **Image Promotion** - promocja obrazów między rejestrami
- **Environment Overlays** - różne konfiguracje dla różnych środowisk

### 9.4. Bezpieczeństwo

Zalecenia dotyczące bezpieczeństwa:

- Używaj kontroli dostępu RBAC
- Integruj z SSO (OIDC, LDAP)
- Szyfruj sekrety w repozytorium
- Używaj podpisu obrazów Docker
- Wdrażaj polityki bezpieczeństwa (OPA, Kyverno)

### 9.5. Skalowalność

Zalecenia dotyczące skalowalności:

- Używaj ApplicationSets dla wielu aplikacji
- Optymalizuj buforowanie repozytoriów
- Rozważ High Availability dla produkcji
- Monitoruj wydajność Argo CD
- Używaj progresywnej synchronizacji dla dużych aplikacji

## 10. Podsumowanie

Argo CD to potężne narzędzie, które zmieniło sposób, w jaki zespoły DevOps wdrażają i zarządzają aplikacjami w Kubernetes. Dzięki podejściu GitOps, Argo CD zapewnia:

- Deklaratywną konfigurację aplikacji
- Automatyczną synchronizację z repozytorium Git
- Pełną historię zmian i audytowalność
- Samonaprawiający się system
- Skalowalność dla wielu aplikacji i klastrów

W miarę jak organizacje przechodzą na architekturę mikroserwisową i adopcję Kubernetes, Argo CD staje się niezbędnym narzędziem w arsenale każdego zespołu DevOps. Mam nadzieję, że ten przewodnik pomoże wam zacząć przygodę z Argo CD i w pełni wykorzystać jego potencjał w waszych projektach!

