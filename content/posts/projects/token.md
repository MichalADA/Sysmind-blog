+++
title = "Don't Buy This Token"
date = "2025-05-05"
description = "Projekt edukacyjny tworzenia i wdrażania własnego tokena na blockchainie Solana"
tags = ["Blockchain", "Solana", "Rust", "Docker"]
categories = ["Projekty"]
draft = false
+++

# Don't Buy This Token - Projekt edukacyjny na Solanie

## 📋 Przegląd projektu
"Don't Buy This Token" to projekt edukacyjny mający na celu stworzenie i wdrożenie własnego tokena na blockchainie Solana. Celem tego projektu jest zdobycie praktycznego doświadczenia w zakresie rozwoju blockchain, Dockera, Rusta i Solana CLI, jednocześnie tworząc funkcjonalny token, który może zostać wdrożony na mainnecie Solany.

## 🎯 Cele
* **Stworzenie tokena na devnecie Solany**: Wykorzystanie testowego środowiska Solany do zaprojektowania i wdrożenia tokena.
* **Wdrożenie tokena na mainnecie**: Wydanie około 20$ na opublikowanie tokena na mainnecie Solany.
* **Dokumentacja procesu**: Dostarczenie jasnych instrukcji i spostrzeżeń dla innych zainteresowanych tworzeniem tokenów.
* **Rozwój projektu portfolio**: Zaprezentowanie umiejętności w zakresie rozwoju blockchain i nowoczesnych technologii.

## 🛠️ Kluczowe funkcje
* **Tworzenie tokenów**: Generowanie tokena na devnecie i mainnecie Solany.
* **Integracja metadanych**: Dodawanie metadanych, takich jak nazwa tokena, symbol i ikona.
* **Zdecentralizowane hostowanie**: Wykorzystanie zdecentralizowanego przechowywania zasobów tokena (np. Pinata/IPFS).
* **Ukierunkowanie na naukę**: Priorytetowe zrozumienie technologii blockchain i narzędzi.

## 💻 Wykorzystane technologie
* **Blockchain Solana**: Platforma do tworzenia i zarządzania tokenem.
* **Docker**: Konteneryzacja środowiska deweloperskiego dla zwiększenia przenośności.
* **Rust**: Wykorzystanie Rusta do pracy z Solana CLI.
* **Pinata/IPFS**: Hostowanie metadanych i zasobów tokena.

## 📝 Kroki do wykonania

### 1. Konfiguracja środowiska
* Instalacja Dockera.
* Konfiguracja kontenera z Solana CLI i Rustem.

### 2. Tworzenie tokena na devnecie
* Generowanie kont dla uprawnienia do emisji i portfela.
* Tworzenie i konfiguracja tokena na devnecie.
* Przesyłanie metadanych do zdecentralizowanego przechowywania.

### 3. Wdrożenie tokena na mainnecie
* Pozyskanie SOL do wdrożenia na mainnecie.
* Powtórzenie procesu tworzenia tokena na mainnecie.

### 4. Dokumentacja i publikacja
* Utworzenie repozytorium GitHub ze szczegółowymi instrukcjami.
* Udostępnianie spostrzeżeń i zdobytych doświadczeń.

## 🤔 Dlaczego "Don't Buy This Token"?
Nazwa podkreśla eksperymentalny charakter projektu i służy jako żartobliwe przypomnienie o wielu projektach tokenów, które nie przynoszą oczekiwanych rezultatów. Ten projekt ma być doświadczeniem edukacyjnym, a nie przedsięwzięciem spekulacyjnym.

## 💎 Szczegóły tokena
* **Nazwa**: Don't Buy This Token
* **Symbol**: DBTt

## 📚 Jak stworzyć własny token - Przewodnik krok po kroku

### 1. Instalacja Dockera

Docker to mój ulubiony sposób wdrażania praktycznie wszystkiego. Jest dostępny na wszystkich platformach i bardzo łatwy w instalacji.

#### Dla Mac
- Link: [https://docs.docker.com/desktop/setup/install/mac-install/](https://docs.docker.com/desktop/setup/install/mac-install/)

#### Dla Linux/Windows (WSL)
- Link: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/) (link dla wszystkich innych dystrybucji)

Jeśli używasz Ubuntu (główny system operacyjny lub WSL), wykonaj te kroki:
- Dokumentacja: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

##### Konfiguracja repozytorium apt Dockera
```
# Dodaj oficjalny klucz GPG Dockera:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Dodaj repozytorium do źródeł Apt:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

##### Instalacja najnowszej wersji Dockera
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

##### Sprawdź, czy działa
```
sudo docker run hello-world
```

### 2. Konfiguracja kontenera Docker dla Solany

#### Utwórz nowy folder dla swojego projektu tokenowego
```
mkdir your-token-name
cd your-token-name
```

#### Utwórz plik Dockerfile
```
nano Dockerfile
```

Skopiuj i wklej poniższy kod Dockerfile:
```
# Użyj lekkiego obrazu bazowego
FROM debian:bullseye-slim

# Ustaw nieinteraktywny frontend dla apt
ENV DEBIAN_FRONTEND=noninteractive

# Zainstaluj wymagane zależności i Rust
RUN apt-get update && apt-get install -y \
    curl build-essential libssl-dev pkg-config nano \
    && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Dodaj Rust do PATH
ENV PATH="/root/.cargo/bin:$PATH"

# Sprawdź instalację Rust
RUN rustc --version

# Zainstaluj Solana CLI
RUN curl -sSfL https://release.anza.xyz/stable/install | sh \
    && echo 'export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"' >> ~/.bashrc

# Dodaj Solana CLI do PATH
ENV PATH="/root/.local/share/solana/install/active_release/bin:$PATH"

# Sprawdź instalację Solana CLI
RUN solana --version

# Ustaw konfigurację Solana dla Devnet
RUN solana config set -ud

# Ustaw katalog roboczy
WORKDIR /solana-token

# Domyślne polecenie do uruchomienia powłoki
CMD ["/bin/bash"]
```

#### Zbuduj obraz Docker
```
docker build -t heysolana .
```

#### Uruchom kontener
```
docker run -it --rm -v $(pwd):/solana-token -v $(pwd)/solana-data:/root/.config/solana heysolana
```

### 3. Tworzenie tokena na Solana devnet

#### Utwórz konto dla uprawnień do emisji
```
solana-keygen grind --starts-with dad:1
```

#### Ustaw konto jako domyślną parę kluczy
```
solana config set --keypair dad-your-token-acount.json
```

#### Przełącz się na devnet
```
solana config set --url devnet
```

#### Sprawdź swoją konfigurację
```
solana config get
```

#### Zdobądź trochę SOL z faucetu
Aby emitować nowe tokeny na sieci Solana, potrzebujesz SOL.

1. Sprawdź swój adres Solana:
```
solana address
```

2. Przejdź do [https://faucet.solana.com/](https://faucet.solana.com/), aby otrzymać airdrop SOL na utworzone konto. Wklej adres Solana i wprowadź ilość SOL, jakiej potrzebujesz. 2,5 będzie WIĘCEJ niż wystarczająco.

3. Sprawdź swoje saldo Solana:
```
solana balance
```

#### Utwórz adres mint
```
solana-keygen grind --starts-with mnt:1
```

#### Emituj swój TOKEN!
```
spl-token create-token \
--program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb \
--enable-metadata \
--decimals 9 \
mnt-your-mint-address.json
```

### 4. Przesyłanie metadanych

#### Przygotuj ikonę tokena
Upewnij się, że jest:
- Kwadratowa
- 512x512 lub 1024x1024 pikseli
- mniej niż 100kb

#### Prześlij obraz do zdecentralizowanego magazynu
1. Utwórz konto na [https://app.pinata.cloud/](https://app.pinata.cloud/)
2. Przejdź do IPFS Files i prześlij swój plik
3. Otwórz plik i skopiuj URL

#### Utwórz plik metadanych
```
nano metadata.json
```

Skopiuj i wklej poniższy format, uzupełniając swoje dane:
```json
{
  "name": "Don't Buy This Token",
  "symbol": "DBTt",
  "description": "Projekt edukacyjny tworzenia tokena na blockchainie Solana.",
  "image": "TU_WKLEJ_URL_SWOJEGO_OBRAZU"
}
```

#### Prześlij plik metadanych do Pinata
Wykonaj te same kroki co wcześniej. Prześlij do Pinata i skopiuj URL.

#### Dodaj metadane do tokena
```
spl-token initialize-metadata \
TWÓJ_ADRES_TOKENA \
"Don't Buy This Token" \
"DBTt" \
URL_DO_TWOICH_METADANYCH
```

#### Sprawdź swój token!
1. Przejdź do [Solana Explorer](https://explorer.solana.com/) i wybierz devnet jako swój klaster
2. Wklej adres swojego tokena w pasku wyszukiwania

### 5. Tworzenie tokenów

#### Utwórz konto tokena
```
spl-token create-account TWÓJ_ADRES_TOKENA
```

#### Wybij tokeny
```
spl-token mint TWÓJ_ADRES_TOKENA 1000
```

#### Sprawdź saldo swojego portfela
```
spl-token balance TWÓJ_ADRES_TOKENA
```

#### Wyślij tokeny do znajomego
```
spl-token transfer TWÓJ_ADRES_TOKENA 10 ADRES_PORTFELA_ODBIORCY --fund-recipient --allow-unfunded-recipient
```

### 6. Przejście na mainnet (opcjonalnie)
Aby stworzyć token na głównej sieci Solana, powtórz powyższe kroki, ale zamiast przełączania na devnet, użyj:
```
solana config set --url mainnet-beta
```

Pamiętaj, że na mainnecie będziesz potrzebować prawdziwego SOL, który musisz kupić.

#### Zabezpieczenie tokena
Po zakończeniu konfiguracji tokena, możesz wyłączyć uprawnienia do emisji i zamrażania:
```
spl-token authorize TWÓJ_ADRES_TOKENA mint --disable
spl-token authorize TWÓJ_ADRES_TOKENA freeze --disable
```

#### Aktualizacja metadanych (w razie potrzeby)
```
spl-token update-metadata TWÓJ_ADRES_TOKENA uri NOWY_URL_DO_METADANYCH
```

## 🔮 Plany na przyszłość
* Dodanie przykładów wykorzystania tokena w małych zdecentralizowanych aplikacjach.
* Eksploracja integracji z innymi ekosystemami blockchain.

## ⚠️ Zastrzeżenie
Ten projekt służy wyłącznie celom edukacyjnym. Token utworzony w tym projekcie nie jest przeznaczony do handlu ani inwestycji. Twórca nie ponosi odpowiedzialności za jakiekolwiek niewłaściwe wykorzystanie tego tokena lub jakiekolwiek poniesione straty finansowe.
