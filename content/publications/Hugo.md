---
title: 'Hugo - Kompletny Przewodnik do Budowania Nowoczesnych Stron Statycznych'
date: 2025-04-30
draft: false
tags: ['hugo', 'webdev', 'jamstack', 'static-site', 'golang', 'devops']
---

# Hugo - Kompletny Przewodnik do Budowania Nowoczesnych Stron Statycznych

To właśnie na Hugo oparty jest blog Sysmind, na którym właśnie czytasz ten artykuł!

## 1. Czym jest Hugo?

Hugo to generator stron statycznych napisany w języku Go, zaprojektowany z myślą o szybkości, prostocie i elastyczności. W przeciwieństwie do tradycyjnych systemów CMS (jak WordPress), Hugo generuje statyczne pliki HTML, CSS i JavaScript, które można hostować na dowolnym serwerze lub platformie CDN.

Główne zalety Hugo:

- **Wyjątkowa szybkość** - generuje tysiące stron w milisekundach
- **Brak bazy danych** - eliminuje problemy z bezpieczeństwem i wydajnością
- **Prostota wdrażania** - gotowe pliki statyczne można hostować wszędzie
- **Markdown** - treść tworzona w łatwym do opanowania formacie
- **Szablony i motywy** - elastyczny system tworzenia layoutów
- **Wielojęzyczność** - wbudowana obsługa wielu języków
- **Zarządzanie zasobami** - przetwarzanie SCSS, JS, obrazów

## 2. Instalacja i konfiguracja

### 2.1. Instalacja Hugo

Hugo można zainstalować na różnych systemach operacyjnych:

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install hugo
```

**macOS (Homebrew):**
```bash
brew install hugo
```

**Windows (Chocolatey):**
```bash
choco install hugo -confirm
```

**Alternatywnie**, można pobrać gotowy pakiet binarny ze [strony GitHub](https://github.com/gohugoio/hugo/releases).

### 2.2. Tworzenie nowej strony

Aby utworzyć nową stronę Hugo:

```bash
hugo new site moja-strona
cd moja-strona
```

To polecenie utworzy podstawową strukturę katalogu:

```
moja-strona/
  ├── archetypes/
  ├── assets/
  ├── content/
  ├── data/
  ├── layouts/
  ├── static/
  ├── themes/
  └── config.toml
```

### 2.3. Konfiguracja podstawowa

Plik `config.toml` zawiera podstawową konfigurację strony:

```toml
baseURL = "https://example.org/"
languageCode = "pl-pl"
title = "Moja strona Hugo"
theme = "nazwa-motywu"

# Konfiguracja paginacji
paginate = 10

# Menu nawigacyjne
[menu]
  [[menu.main]]
    name = "Strona główna"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "O mnie"
    url = "/about/"
    weight = 2
  [[menu.main]]
    name = "Blog"
    url = "/posts/"
    weight = 3

# Parametry motywu
[params]
  description = "Moja wspaniała strona Hugo"
  author = "Twoje Imię"
  github = "twojeimie"
  twitter = "twojeimie"
```

## 3. Struktura projektu Hugo

### 3.1. Kluczowe katalogi

**content/** - zawiera treść strony w formacie Markdown
```
content/
  ├── _index.md        # Strona główna
  ├── about/           # Sekcja "O mnie"
  │   └── _index.md
  └── posts/           # Sekcja z wpisami na blogu
      ├── _index.md
      ├── post1.md
      └── post2.md
```

**layouts/** - zawiera szablony HTML
```
layouts/
  ├── _default/        # Domyślne szablony
  │   ├── baseof.html  # Bazowy szablon
  │   ├── list.html    # Szablon listy (np. lista wpisów)
  │   └── single.html  # Szablon pojedynczej strony
  ├── partials/        # Komponenty wielokrotnego użytku
  │   ├── header.html
  │   ├── footer.html
  │   └── sidebar.html
  └── index.html       # Szablon strony głównej
```

**static/** - pliki statyczne (obrazy, CSS, JS)
```
static/
  ├── css/
  ├── js/
  └── images/
```

**themes/** - motywy
```
themes/
  └── nazwa-motywu/
```

**data/** - dane w formatach YAML, JSON, TOML
```
data/
  ├── authors.yaml
  └── social.json
```

**assets/** - zasoby przetwarzane przez potok zasobów
```
assets/
  ├── scss/
  └── js/
```

### 3.2. Pliki treści

Pliki Markdown zawierają frontmatter z metadanymi oraz treść:

```markdown
---
title: "Mój pierwszy post"
date: 2025-04-30
draft: false
tags: ["hugo", "webdev"]
categories: ["poradniki"]
---

# Mój pierwszy post w Hugo

To jest treść mojego pierwszego postu napisanego w Markdown.

## Podtytuł

Lista zalet Hugo:
- Szybkość
- Prostota
- Elastyczność
```

## 4. Szablony i layouts

### 4.1. Podstawowa struktura szablonu

Szablon bazowy (`layouts/_default/baseof.html`):

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{{ block "title" . }}{{ .Site.Title }}{{ end }}</title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    {{ partial "header.html" . }}
    <main>
        {{ block "main" . }}{{ end }}
    </main>
    {{ partial "footer.html" . }}
</body>
</html>
```

Szablon pojedynczej strony (`layouts/_default/single.html`):

```html
{{ define "title" }}
    {{ .Title }} | {{ .Site.Title }}
{{ end }}

{{ define "main" }}
<article>
    <h1>{{ .Title }}</h1>
    <div class="metadata">
        <time>{{ .Date.Format "2006-01-02" }}</time>
        {{ with .Params.tags }}
        <ul class="tags">
            {{ range . }}
            <li><a href="{{ "tags/" | absURL }}{{ . | urlize }}">{{ . }}</a></li>
            {{ end }}
        </ul>
        {{ end }}
    </div>
    <div class="content">
        {{ .Content }}
    </div>
</article>
{{ end }}
```

### 4.2. Komponenty wielokrotnego użytku

Partials to komponenty, które można używać wielokrotnie:

```html
<!-- layouts/partials/header.html -->
<header>
    <div class="logo">
        <a href="/">{{ .Site.Title }}</a>
    </div>
    <nav>
        <ul>
            {{ range .Site.Menus.main }}
            <li><a href="{{ .URL }}">{{ .Name }}</a></li>
            {{ end }}
        </ul>
    </nav>
</header>
```

### 4.3. Funkcje szablonów

Hugo oferuje liczne funkcje do manipulacji danymi w szablonach:

- **range** - iteracja po kolekcjach
- **if/else** - warunkowe renderowanie
- **with** - ustalanie kontekstu
- **partial** - wstawianie komponentów
- **block/define** - definiowanie bloków

Przykład listy wpisów:

```html
{{ define "main" }}
<h1>{{ .Title }}</h1>
{{ .Content }}
<div class="posts">
    {{ range .Pages }}
    <article class="post-summary">
        <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
        <time>{{ .Date.Format "02.01.2006" }}</time>
        <div class="summary">
            {{ .Summary }}
            {{ if .Truncated }}
            <a href="{{ .Permalink }}">Czytaj więcej...</a>
            {{ end }}
        </div>
    </article>
    {{ end }}
</div>
{{ end }}
```

## 5. Zaawansowane funkcje Hugo

### 5.1. Taksonomie

Hugo obsługuje taksonomie (tagi, kategorie):

```toml
# config.toml
[taxonomies]
  tag = "tags"
  category = "categories"
  author = "authors"
```

### 5.2. Shortcodes

Shortcodes to skróty do dodawania złożonej treści w plikach Markdown:

```html
<!-- layouts/shortcodes/figure.html -->
<figure>
    <img src="{{ .Get "src" }}" alt="{{ .Get "alt" }}">
    <figcaption>{{ .Get "caption" }}</figcaption>
</figure>
```

Użycie w treści:
```markdown
{{< figure src="/images/hugo-logo.png" alt="Logo Hugo" caption="Oficjalne logo Hugo" >}}
```

### 5.3. Wielojęzyczność

Hugo obsługuje wiele języków:

```toml
# config.toml
defaultContentLanguage = "pl"
defaultContentLanguageInSubdir = true

[languages]
  [languages.pl]
    title = "Moja strona Hugo"
    weight = 1
  [languages.en]
    title = "My Hugo Website"
    weight = 2
```

### 5.4. Przetwarzanie zasobów

Hugo może przetwarzać SCSS, minifikować JS:

```html
{{ $style := resources.Get "scss/main.scss" | toCSS | minify }}
<link rel="stylesheet" href="{{ $style.RelPermalink }}">

{{ $js := resources.Get "js/main.js" | minify }}
<script src="{{ $js.RelPermalink }}"></script>
```

### 5.5. Moduły Hugo

Hugo obsługuje system modułów do importowania innych projektów Hugo:

```toml
# config.toml
[module]
  [[module.imports]]
    path = "github.com/themefisher/educenter-hugo"
```

## 6. Wdrażanie strony Hugo

### 6.1. Generowanie strony

Aby wygenerować statyczną stronę:

```bash
hugo
```

Pliki wyjściowe znajdą się w katalogu `public/`.

Aby uruchomić serwer deweloperski:

```bash
hugo server -D
```

### 6.2. Wdrażanie na Netlify

Netlify to popularna platforma do hostowania stron Hugo:

1. Utwórz plik `netlify.toml` w głównym katalogu projektu:
   ```toml
   [build]
     publish = "public"
     command = "hugo --minify"
   
   [build.environment]
     HUGO_VERSION = "0.115.0"
   
   [context.production.environment]
     HUGO_ENV = "production"
   ```

2. Podłącz repozytorium Git do Netlify
3. Skonfiguruj ustawienia budowania
4. Dodaj własną domenę

### 6.3. Wdrażanie na GitHub Pages

Aby wdrożyć na GitHub Pages:

1. Utwórz plik `.github/workflows/hugo.yml`:
   ```yaml
   name: Deploy Hugo site

   on:
     push:
       branches:
         - main

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Setup Hugo
           uses: peaceiris/actions-hugo@v2
           with:
             hugo-version: 'latest'
             extended: true
         - name: Build
           run: hugo --minify
         - name: Deploy
           uses: peaceiris/actions-gh-pages@v3
           with:
             github_token: ${{ secrets.GITHUB_TOKEN }}
             publish_dir: ./public
   ```

2. Włącz GitHub Pages w ustawieniach repozytorium

### 6.4. Własny serwer

Aby wdrożyć na własny serwer:

1. Wygeneruj stronę: `hugo --minify`
2. Prześlij zawartość katalogu `public/` na serwer
3. Skonfiguruj serwer HTTP (np. Nginx):
   ```nginx
   server {
       listen 80;
       server_name example.com;
       root /var/www/html/public;
       index index.html;
   
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

## 7. Motywy i zasoby

### 7.1. Wybór motywu

Hugo oferuje setki gotowych motywów: [Hugo Themes](https://themes.gohugo.io/)

Aby zainstalować motyw:

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

Następnie zaktualizuj `config.toml`:
```toml
theme = "ananke"
```

### 7.2. Tworzenie własnego motywu

Struktura własnego motywu:

```
themes/moj-motyw/
  ├── archetypes/
  ├── assets/
  ├── layouts/
  ├── static/
  ├── theme.toml
  └── README.md
```

Plik `theme.toml`:
```toml
name = "Mój Motyw"
license = "MIT"
licenselink = "https://github.com/twojnick/moj-motyw/LICENSE"
description = "Mój własny motyw Hugo"
```

### 7.3. Przydatne zasoby

- [Dokumentacja Hugo](https://gohugo.io/documentation/)
- [Hugo Forum](https://discourse.gohugo.io/)
- [Hugo GitHub](https://github.com/gohugoio/hugo)
- [Awesome Hugo](https://github.com/theNewDynamic/awesome-hugo)

## 8. Case Study: Sysmind Blog

Blog Sysmind, na którym czytasz ten artykuł, jest zbudowany przy użyciu Hugo. Wybrałem Hugo z kilku powodów:

1. **Szybkość i wydajność** - Hugo generuje stronę w milisekundach, co pozwala na szybkie iteracje podczas rozwoju.

2. **Bezpieczeństwo** - brak bazy danych eliminuje typowe wektory ataków.

3. **Prostota zarządzania treścią** - pliki Markdown są wygodne w edycji i można je przechowywać w systemie kontroli wersji.

4. **Elastyczność** - Hugo pozwala na pełną kontrolę nad strukturą i wyglądem strony.

5. **Wielojęzyczność** - wbudowana obsługa wielu języków umożliwia łatwe tworzenie treści po polsku i angielsku.

6. **Integracja z Netlify** - automatyczne wdrażanie przy każdym pushu do repozytorium.

Sysmind Blog wykorzystuje:

- Niestandardowy motyw oparty na [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- Wielojęzyczność (pl/en) z oddzielnymi domenami
- System taksonomii dla tagów i kategorii
- Automatyczne wdrażanie przez Netlify
- Zoptymalizowane obrazy i CSS dzięki procesowi budowania Hugo

## 9. Porównanie z innymi generatorami stron statycznych

### 9.1. Hugo vs Jekyll

| Cecha | Hugo | Jekyll |
|-------|------|--------|
| Język | Go | Ruby |
| Szybkość | Bardzo wysoka | Średnia |
| Ekosystem | Rosnący | Dojrzały |
| Łatwość instalacji | Pojedynczy plik binarny | Wymaga Ruby i gemów |
| Wbudowane funkcje | Wiele | Podstawowe (rozszerzane przez pluginy) |

### 9.2. Hugo vs Gatsby

| Cecha | Hugo | Gatsby |
|-------|------|--------|
| Język | Go | JavaScript (React) |
| Podejście | Statyczne pliki | Hydratacja React |
| Źródła danych | Pliki lokalne | GraphQL (wiele źródeł) |
| Krzywa uczenia | Średnia | Stroma |
| Wydajność generowania | Wysoka | Średnia |

### 9.3. Hugo vs Next.js

| Cecha | Hugo | Next.js |
|-------|------|--------|
| Język | Go | JavaScript (React) |
| Podejście | Czysto statyczne | SSG/SSR/CSR |
| Funkcje dynamiczne | Ograniczone | Zaawansowane |
| Złożoność | Niska do średniej | Średnia do wysokiej |
| Typowe zastosowanie | Blogi, dokumentacja | Aplikacje webowe |

## 10. Podsumowanie

Hugo to potężne narzędzie do tworzenia nowoczesnych stron statycznych, które wyróżnia się szybkością, prostotą i elastycznością. Dzięki architektury opartej na plikach statycznych, strony Hugo są:

- Szybkie w ładowaniu
- Łatwe w utrzymaniu
- Bezpieczne
- Tanie w hostowaniu
- Skalowalne

W erze JAMstack, Hugo jest jednym z liderów dzięki swojej wydajności i prostocie. Niezależnie czy tworzysz blog, portfolio, stronę firmową czy dokumentację, Hugo oferuje wszystkie narzędzia potrzebne do szybkiego i efektywnego wdrożenia.



---

**PS.** Jeśli chcesz zobaczyć źródło tego bloga, zapraszam do mojego [repozytorium na GitHubie](https://github.com/MichalADA/Sysmind-blog)
