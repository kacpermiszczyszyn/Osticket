# Tampermonkey Installer – Handoff

> Wklej ten plik (lub jego treść) w pierwszej wiadomości nowej sesji, żeby Claude od razu wiedział, co zostało ustalone i co dalej robić — bez ponownego tłumaczenia.

## Cel projektu
Wewnętrzny "1-click installer" skryptów Tampermonkey dla pracowników firmy **MDP Serwis** (Maszyny do palet), hostowany na własnym VPS:
- pracownik wchodzi na stronę z listą skryptów
- klika "Zainstaluj" przy wybranym → Tampermonkey otwiera ekran instalacji
- skrypty aktualizują się automatycznie (mechanizm wbudowany w Tampermonkey)

## Podjęte decyzje techniczne
1. **Aktualizacje**: zostajemy przy domyślnym mechanizmie Tampermonkey (`@updateURL` + `@downloadURL` + `@version` w nagłówkach skryptu). TM sprawdza co 24h i przy starcie przeglądarki. **Nie** budujemy pollera/WebSocketu.
2. **Hosting**: na VPS, plan to serwer statyczny (najpewniej nginx). Adres: prawdopodobnie po IP, ale rozważyć darmową subdomenę (DuckDNS/no-ip) + Let's Encrypt — bez HTTPS pojawia się problem mixed-content kiedy skrypty mają działać na stronach `https://`.
3. **Architektura strony**: statyczna, **driven by `manifest.json`**. Dodanie nowego skryptu = jeden wpis w JSON + plik `.user.js` w `scripts/`. Bez backendu, bez build-stepu.
4. **Branding**: kolorystyka MDP Serwis — zielony `#2D8E3F` (primary), brązowy `#8B5A2B` (akcent), szarości. Font: Inter. Logo w `assets/logo.png` z fallbackiem tekstowym jeśli plik nie istnieje.

## Stan obecny (branch `claude/tampermonkey-auto-installer-K8xl4` na `kacpermiszczyszyn/osticket`)

```
Osticket/
├── index.html          # landing page – fetch'uje manifest.json, renderuje karty
├── assets/
│   ├── style.css       # styles w brand colors, responsive
│   └── logo.png        # ⚠️ DO WGRANIA – obecnie brak pliku
├── scripts/            # ⚠️ PUSTY – tu trafiają pliki .user.js
└── manifest.json       # { "scripts": [] } – pusta lista
```

**Co działa**: landing page renderuje się poprawnie, pusty stan pokazuje instrukcję, fallback tekstowy logo działa.

## Format wpisu w `manifest.json`
```json
{
  "id": "foo",
  "name": "Nazwa skryptu",
  "description": "Co robi w jednym zdaniu.",
  "version": "1.0.0",
  "file": "scripts/foo.user.js",
  "icon": "🛠️",
  "updated_at": "2026-05-10"
}
```

## Wymagane nagłówki w pliku `.user.js` (pod auto-update)
```
// ==UserScript==
// @name         Nazwa
// @namespace    mdp-serwis
// @version      1.0.0
// @description  ...
// @match        https://example.com/*
// @updateURL    https://VPS_HOST/scripts/foo.user.js
// @downloadURL  https://VPS_HOST/scripts/foo.user.js
// @grant        none
// ==/UserScript==
```
Bumpowanie `@version` przy każdej zmianie kodu = TM podchwytuje update.

## Otwarte sprawy / blokujące

- [ ] **Złe repo**: ta sesja powstała na `kacpermiszczyszyn/osticket`, ale właściwe skrypty Tampermonkey są w innym projekcie/repo (do potwierdzenia gdzie). Trzeba przenieść pracę.
- [ ] **Logo**: właściciel ma plik PNG z logiem MDP Serwis (gear/saw blade + boards + napis "MASZYNY DO PALET / MDP SERWIS"). Wrzucić jako `assets/logo.png`.
- [ ] **Skrypty**: zebrać istniejące `.user.js`, wrzucić do `scripts/` i dopisać wpisy do `manifest.json`.
- [ ] **VPS**: brak danych dostępowych w sesji — potrzebne IP, user, sposób auth (klucz/hasło), ścieżka docelowa, info czy nginx/apache jest skonfigurowane.
- [ ] **HTTPS**: zdecydować czy zostajemy przy IP (HTTP, ryzyko mixed-content) czy dorzucamy darmową subdomenę + Let's Encrypt.

## Następne kroki w nowej sesji
1. Potwierdzić właściwe repo / przenieść pliki landing page (`index.html`, `assets/style.css`, `manifest.json`) do niego.
2. Wgrać logo i pierwsze skrypty.
3. Zaplanować deploy na VPS (skopiowanie plików + konfiguracja serwera www + ewentualnie cert SSL).
4. Sprawdzić instalację na realnej przeglądarce (link `.user.js` → ekran TM).

## Skąd wziąć dotychczasowy kod
Gałąź `claude/tampermonkey-auto-installer-K8xl4` w repo `kacpermiszczyszyn/osticket`. Jeśli docelowe repo jest inne, można:
```bash
git remote add old https://github.com/kacpermiszczyszyn/osticket.git
git fetch old claude/tampermonkey-auto-installer-K8xl4
git checkout old/claude/tampermonkey-auto-installer-K8xl4 -- index.html assets/ manifest.json
```
albo po prostu zassać surowe pliki z UI GitHuba.
