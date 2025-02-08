---
title: "Instalacja Docker"
date: 2024-02-08
---

# Przewodnik Instalacji Dockera

Docker to platforma konteneryzacji, która umożliwia uruchamianie aplikacji w lekkich, izolowanych środowiskach. Poniżej znajdziesz instrukcje instalacji dla różnych dystrybucji Linuxa.

---

## 1. Ubuntu/Debian

1. **Aktualizacja systemu**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Instalacja wymaganych pakietów**:
   ```bash
   sudo apt install -y curl apt-transport-https ca-certificates software-properties-common
   ```

3. **Dodanie repozytorium Dockera**:
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
   ```

4. **Instalacja Dockera**:
   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

5. **Włączenie i uruchomienie Dockera**:
   ```bash
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

6. **Weryfikacja instalacji**:
   ```bash
   sudo docker run hello-world
   ```

---

## 2. Red Hat/CentOS

1. **Aktualizacja systemu**:
   ```bash
   sudo yum update -y
   ```

2. **Dodanie repozytorium Dockera**:
   ```bash
   sudo yum install -y yum-utils
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

3. **Instalacja Dockera**:
   ```bash
   sudo yum install -y docker-ce docker-ce-cli containerd.io
   ```

4. **Włączenie i uruchomienie Dockera**:
   ```bash
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

5. **Weryfikacja instalacji**:
   ```bash
   sudo docker run hello-world
   ```

---

## 3. Fedora

1. **Aktualizacja systemu**:
   ```bash
   sudo dnf update -y
   ```

2. **Dodanie repozytorium Dockera**:
   ```bash
   sudo dnf install -y dnf-plugins-core
   sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
   ```

3. **Instalacja Dockera**:
   ```bash
   sudo dnf install -y docker-ce docker-ce-cli containerd.io
   ```

4. **Włączenie i uruchomienie Dockera**:
   ```bash
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

5. **Weryfikacja instalacji**:
   ```bash
   sudo docker run hello-world
   ```

---

## 4. Arch Linux

1. **Aktualizacja systemu**:
   ```bash
   sudo pacman -Syu
   ```

2. **Instalacja Dockera**:
   ```bash
   sudo pacman -S docker
   ```

3. **Włączenie i uruchomienie Dockera**:
   ```bash
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

4. **Dodanie użytkownika do grupy Docker (opcjonalne)**:
   ```bash
   sudo usermod -aG docker $USER
   ```

5. **Weryfikacja instalacji**:
   ```bash
   sudo docker run hello-world
   ```

---

## Uwagi

- **Oficjalna dokumentacja**:
  - [Ubuntu/Debian](https://docs.docker.com/engine/install/ubuntu/)
  - [Red Hat/CentOS](https://docs.docker.com/engine/install/centos/)
  - [Fedora](https://docs.docker.com/engine/install/fedora/)
  - [Arch Linux](https://wiki.archlinux.org/title/docker)

- Upewnij się, że Twoja dystrybucja Linuxa wspiera wymaganą wersję jądra.

Docker jest teraz gotowy do użycia! 🚀