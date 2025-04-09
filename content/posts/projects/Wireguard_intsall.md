+++ 
title = "Własny VPN z WireGuard na VPS (Ubuntu)"
date = "2025-04-09"
description = "Konfiguracja bezpiecznego tunelu VPN z WireGuard na serwerze Ubuntu"
tags = ["Linux", "VPN", "Bezpieczeństwo", "WireGuard"]
categories = ["Projekty"]
draft = false 
+++

# 🛡️ Własny VPN z WireGuard na VPS (Ubuntu)

> Bezpieczny tunel sieciowy z nowoczesnym WireGuardem – szybka i lekka alternatywa dla OpenVPN. Pokażę Ci krok po kroku, jak postawić własny VPN i połączyć się z telefonu.

---

## 📋 Wymagania

- VPS z Ubuntu 20.04/22.04
- Uprawnienia `sudo`
- (Opcjonalnie) domena – ułatwia konfigurację klienta

---

## 🔧 Instalacja WireGuard

```bash
sudo apt update
sudo apt install wireguard qrencode
```

---

## 🔑 Tworzenie kluczy serwera

```bash
cd /etc/wireguard
umask 077
wg genkey | tee server.key | wg pubkey > server.pub
```

---

## ⚙️ Konfiguracja serwera (/etc/wireguard/wg0.conf)

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <zawartość server.key>
SaveConfig = true

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

---

## 🔥 Włączenie przekierowywania ruchu

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 🟢 Uruchomienie WireGuard

```bash
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

---

## 📱 Dodanie klienta (np. telefon z Android/iOS)

### 1. Wygeneruj klucze klienta:

```bash
wg genkey | tee client.key | wg pubkey > client.pub
```

### 2. Stwórz plik client.conf:

```ini
[Interface]
PrivateKey = <zawartość client.key>
Address = 10.0.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <zawartość server.pub>
Endpoint = twoja.domena.com:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

### 3. Dodaj klienta do serwera:

```bash
sudo wg set wg0 peer $(<client.pub) allowed-ips 10.0.0.2
```

### 4. Wygeneruj QR kod do zeskanowania:

```bash
qrencode -t ansiutf8 < client.conf
```

---

## ♻️ Bonus: Automatyczne odnawianie kluczy klienta

### Skrypt /usr/local/bin/renew-wg-keys.sh:

```bash
#!/bin/bash
cd /etc/wireguard
umask 077

# Nowe klucze
wg genkey | tee client.key | wg pubkey > client.pub

# Update klienta w WireGuardzie
wg set wg0 peer $(<client.pub) allowed-ips 10.0.0.2
systemctl restart wg-quick@wg0
```

### Cron (co miesiąc):

```bash
(crontab -l ; echo "@monthly /usr/local/bin/renew-wg-keys.sh") | crontab -
```

---

## ✅ Gotowe!

Masz teraz w pełni działający, szybki i bezpieczny VPN z WireGuardem! Możesz dodać więcej klientów (np. laptopa z Linuxem/Windows), korzystając z podobnego schematu.
