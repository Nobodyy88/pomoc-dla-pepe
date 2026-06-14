# Pomoc dla PEPE

Instrukcja dla kompletnego nowicjusza: jak ustawiać filtry w autobrr, co oznaczają najważniejsze pola i jak czytać gotowy filtr JSON.

Źródło: dokumentacja autobrr Filters: https://autobrr.com/filters

---

## 1. Co robi filtr w autobrr?

Filtr to zestaw warunków typu:

„Jeśli tracker ogłosi torrent, który spełnia moje zasady, to wykonaj akcję”.

Przykład:

- tracker ogłasza film,
- autobrr sprawdza nazwę, kategorię, rozdzielczość, kodek, rozmiar, język itd.,
- jeśli torrent pasuje do filtra, autobrr wysyła go np. do qBittorrenta, Radarra albo Sonarra.

Najważniejsze: filtr sam z siebie nic nie pobiera, jeśli nie ma ustawionej akcji.

---

## 2. Najprostszy schemat pracy

1. Wejdź w autobrr.
2. Otwórz `Filters`.
3. Kliknij `Create Filter`.
4. Ustaw nazwę filtra.
5. Wybierz indexer/tracker.
6. Ustaw warunki, np. filmy 1080p, tylko H.264, bez Remuxów.
7. Dodaj akcję, np. qBittorrent albo Radarr.
8. Zapisz filtr.
9. Przetestuj na kilku nazwach release’ów albo obserwuj logi.

---

## 3. Najważniejsza zasada: puste pole oznacza „pasuje wszystko”

Jeśli jakieś pole zostawisz puste, autobrr zwykle traktuje je jako brak ograniczenia.

Przykład:

- `Resolutions` puste → pasuje 720p, 1080p, 2160p itd.
- `Years` puste → pasują wszystkie lata.
- `Codecs` puste → pasują wszystkie kodeki.

Dlatego nie zostawiaj pustego pola, jeśli chcesz coś naprawdę ograniczyć.

---

## 4. Match vs Except

W wielu miejscach są dwa typy pól:

- `Match ...` — co ma pasować,
- `Except ...` — czego nie wolno przepuścić.

`Except` ma pierwszeństwo.

Przykład:

- `Match releases`: `*Movie*`
- `Except releases`: `*Remux*`

Release `Some.Movie.2025.1080p.Remux` zostanie odrzucony, mimo że zawiera `Movie`, bo złapał go wyjątek `Remux`.

---

## 5. Wielkość liter nie ma znaczenia

Filtry autobrr są case-insensitive.

To znaczy, że:

- `remux`,
- `REMUX`,
- `ReMuX`

są traktowane tak samo.

---

## 6. Wildcardy: gwiazdka i znak zapytania

W wielu polach można używać wildcardów.

### `*` oznacza dowolny ciąg znaków

Przykłady:

- `*movies*` — pasuje do wszystkiego, co zawiera `movies`,
- `*HD*` — pasuje do wszystkiego, co zawiera `HD`,
- `*WEB*` — pasuje do `WEB`, `WEB-DL`, `WEBRip` itd., zależnie od pola.

### `?` oznacza dokładnie jeden znak

Przykład:

- `movies?HD` pasuje do `movies/HD`, `movies-HD`, `movies HD`, jeśli w danym miejscu jest jeden znak między `movies` a `HD`.

Dlatego w Twoim przykładzie:

```text
*movies?HD*
```

znaczy mniej więcej:

„kategoria ma zawierać `movies`, potem jeden dowolny znak, potem `HD`, a przed i po może być cokolwiek”.

To może złapać np.:

- `Movies/HD`,
- `movies-HD`,
- `Movies HD`.

---

## 7. Regex: tryb dla bardziej zaawansowanych

W autobrr możesz włączyć `Use regex`.

Regex to mocniejsze dopasowanie niż wildcardy. Pozwala pisać wzory typu:

```regex
(-GROUP1|-GROUP2|-GROUP3)
```

To oznacza:

„release ma zawierać `-GROUP1` albo `-GROUP2` albo `-GROUP3`”.

W JSON-ie backslash trzeba często zapisywać podwójnie, np.:

```text
"except_releases": "(720p\\.|Remux|2160p\\.)"
```

W interfejsie autobrr widzisz to zwykle jako:

```regex
(720p\.|Remux|2160p\.)
```

A znaczenie jest takie:

- `720p\.` — dopasuj `720p.` z kropką,
- `2160p\.` — dopasuj `2160p.` z kropką,
- `Remux` — dopasuj słowo `Remux`.

Kropka w regexie normalnie oznacza „dowolny znak”, więc jeśli chcesz prawdziwą kropkę, trzeba ją uciec jako `\.`.

---

## 8. Pola ogólne

### Filter name / name

Nazwa filtra. Daj nazwę, po której za miesiąc nadal zrozumiesz, co robi.

Dobre nazwy:

- `Filmy HD 1080p H264 bez PL`
- `Seriale 1080p WEB Sonarr`
- `Freeleech buffer 24h`

Słabe nazwy:

- `test`
- `nowy`
- `filtr1`

### Enabled

Czy filtr jest aktywny.

- `true` / włączone — działa,
- `false` / wyłączone — nie działa.

### Indexers

Trackery/indexery, na których filtr ma działać.

Jeśli wybierzesz zły indexer albo żadnego, filtr nie złapie tego, czego oczekujesz.

### Announce Type

Najczęściej używasz:

- `NEW` — nowe uploady.

Inne typy zależą od trackera, np. staff checked, promo/freeleech, resurrected itd.

Dla początkującego: zacznij od `NEW`.

---

## 9. Rules: rozmiar, priorytet i limity

### Min. size

Minimalny rozmiar torrenta.

Przykłady:

- `1GB` — odrzuć wszystko poniżej 1 GB,
- `4GB` — dobre minimum dla filmów 1080p, jeśli nie chcesz małych encode’ów,
- `15GB` — sensowne minimum dla większych wydań 4K.

Uwaga: jeśli tracker nie podaje rozmiaru w announce, autobrr może musieć pobrać plik `.torrent`, żeby rozmiar sprawdzić.

### Max. size

Maksymalny rozmiar.

Przykłady:

- `12GB` — film 1080p, bez bardzo dużych wydań,
- `35GB` — 4K, ale bez ogromnych remuxów,
- `3GB` — odcinki seriali 1080p.

### Priority

Filtry są sprawdzane według priorytetu.

- wyższa liczba = wyższy priorytet,
- może być dodatnia albo ujemna,
- domyślnie `0`.

Przykład:

- filtr bardzo ważny: `10`,
- zwykły filtr: `0`,
- filtr awaryjny: `-5`.

### Max downloads / Max downloads per

Limit ile razy filtr może coś pobrać w danym czasie.

Przykład:

- `Max downloads`: `3`
- `Max downloads per`: `day`

Oznacza: maksymalnie 3 pobrania dziennie z tego filtra.

---

## 10. TV & Movies

### Movies/Shows

Lista tytułów filmów albo seriali.

Przykłady:

```text
Dune, Matrix, The?Last?of?Us
```

Lepiej używać `?` między słowami, bo release’y mogą mieć kropki, spacje albo inne separatory.

### Years

Lata produkcji.

Przykłady:

```text
2025
```

```text
2024,2025,2026
```

```text
2020-2026
```

### Seasons

Sezony serialu.

Przykłady:

```text
1
```

```text
1-5
```

### Episodes

Odcinki serialu.

Przykłady:

```text
1-99
```

Tylko paczki sezonowe:

```text
Seasons: 1-99
Episodes: 0
```

Tylko pojedyncze odcinki, bez paczek sezonowych:

```text
Seasons: 1-99
Episodes: 1-99
```

---

## 11. Jakość: resolution, source, codec, HDR, other

### Resolutions

Rozdzielczość.

Najczęstsze:

- `720p`,
- `1080p`,
- `2160p`.

Przykład dla filmów HD:

```text
1080p
```

Przykład dla HD + 720p:

```text
720p, 1080p
```

### Sources

Źródło wydania.

Typowe wartości:

- `WEB`,
- `WEB-DL`,
- `WEBRip`,
- `BluRay`,
- `UHD BluRay` / podobne, zależnie od indexera.

Dla Radarr/Sonarr dokumentacja autobrr sugeruje lekkie filtrowanie i zostawienie reszty aplikacjom typu Radarr/Sonarr.

### Codecs

Kodek wideo.

Najczęstsze:

- `H.264`,
- `x264`,
- `H.265`,
- `x265`.

Przykład:

```text
H.264, x264
```

To przepuszcza klasyczne wydania 1080p H.264/x264.

### Containers

Kontener, np. `mkv`, `mp4`.

Dla początkującego: zwykle zostaw puste.

Powód: nie każdy tracker ogłasza kontener, więc możesz przypadkiem odrzucać dobre torrenty.

### Match HDR / Except HDR

Jeśli chcesz tylko HDR/DV:

```text
Match HDR: HDR, DV
```

Jeśli nie chcesz HDR/DV:

```text
Except HDR: HDR, DV
```

Jeśli nie wiesz: zostaw puste i pozwól Radarr/Sonarr zdecydować.

### Match Other / Except Other

Tu często trafiają oznaczenia typu:

- `REMUX`,
- `PROPER`,
- `REPACK`,
- inne specjalne tagi.

Przykład:

```text
Except Other: REMUX
```

Oznacza: nie bierz remuxów.

---

## 12. Kategorie

`Match categories` ogranicza filtr do kategorii z trackera.

Przykład:

```text
*movies?HD*
```

Dla większości osób wystarczą proste wildcardy typu:

```text
*Movie*
```

```text
*TV*, *Episode*
```

```text
*HD*
```

Kategorie zależą od trackera. Jeden tracker może mieć `Movies/HD`, inny `MovieHD`, a jeszcze inny polskie nazwy.

Jeśli nie wiesz, jakie kategorie widzi autobrr, można je wyciągnąć z bazy:

```bash
sqlite3 /path/to/autobrr.db "SELECT DISTINCT indexer, category FROM \"release\" ORDER BY indexer, category;" ".exit" > dump.txt
```

---

## 13. Języki

`Except language` pozwala odrzucać języki, których nie chcesz.

Przykład:

```text
TURKiSH, HiNDi, BRAZiLiAN, BALTIC, CHiNESE, MANDARiN, RUSSiAN
```

To oznacza: odrzuć wydania oznaczone jako tureckie, hinduskie, brazylijskie, bałtyckie, chińskie, mandaryńskie i rosyjskie.

Uwaga: nazwy języków i tagów zależą od tego, co tracker podaje w announce.

---

## 14. Release groups

Grupa release’owa to końcówka nazwy, np.:

```text
Some.Movie.2025.1080p.WEB-DL.x264-GROUP
```

Tutaj grupa to:

```text
GROUP
```

Możesz używać:

- `Match release groups` — bierz tylko konkretne grupy,
- `Except release groups` — odrzucaj konkretne grupy.

Jeśli grupa jest w samej nazwie release’a, możesz też łapać ją przez `Match releases` albo `Except releases`, np. regexem `-GROUP`.

---

## 15. Akcje

Filtr bez akcji tylko dopasowuje. Żeby coś się stało, dodaj akcję.

Najczęstsze akcje:

- qBittorrent,
- Deluge,
- Transmission,
- Radarr,
- Sonarr,
- Watch folder,
- Webhook,
- Exec,
- Test.

Dla początkującego najbezpieczniej:

1. Najpierw dodaj akcję `Test`.
2. Sprawdź logi, czy filtr łapie dobre rzeczy.
3. Dopiero potem dodaj qBittorrent/Radarr/Sonarr.

Po dodaniu lub zmianie akcji zawsze zapisz filtr.

---

## 16. Analiza przykładowego filtra

Oto filtr od PEPE:

```json
{
  "name": "Główne FIlmy HD",
  "version": "1.0",
  "data": {
    "enabled": true,
    "min_size": "4GB",
    "priority": 3,
    "match_releases": "(-TV4TG|-inTGrity|-presa|-SLiM|-BluzgiTeam|-CiNEF0X|-Arbsom|-Hunt3r|-4PT)",
    "except_releases": "(720p\\.|Remux|2160p\\.|\\.PL\\.|TrueHD|\\.MAX\\.)",
    "use_regex": true,
    "announce_types": [
      "NEW"
    ],
    "resolutions": [
      "1080p"
    ],
    "codecs": [
      "H.264",
      "x264"
    ],
    "except_other": [
      "REMUX"
    ],
    "years": "2025,2026",
    "match_categories": "*movies?HD*",
    "except_language": [
      "TURKiSH",
      "HiNDi",
      "BRAZiLiAN",
      "BALTIC",
      "CHiNESE",
      "MANDARiN",
      "RUSSiAN"
    ],
    "is_auto_updated": false,
    "release_profile_duplicate": null
  }
}
```

### Co ten filtr robi po ludzku?

Ten filtr szuka nowych filmów HD 1080p z lat 2025–2026, zakodowanych jako H.264/x264, większych niż 4 GB, z wybranych grup release’owych, ale odrzuca remuxy, 720p, 2160p, polskie wydania oznaczone `.PL.`, TrueHD, `.MAX.` i kilka języków obcych.

### Szczegółowo

#### `enabled: true`

Filtr jest włączony.

#### `min_size: 4GB`

Torrent musi mieć co najmniej 4 GB.

To pomaga odrzucić bardzo małe encode’y.

#### `priority: 3`

Filtr ma priorytet 3. Jeśli masz inne filtry, ten będzie ważniejszy niż filtry z priorytetem 0, 1 albo 2.

#### `match_releases`

```regex
(-TV4TG|-inTGrity|-presa|-SLiM|-BluzgiTeam|-CiNEF0X|-Arbsom|-Hunt3r|-4PT)
```

Filtr przepuszcza tylko release’y zawierające jedną z tych końcówek/grup:

- `-TV4TG`,
- `-inTGrity`,
- `-presa`,
- `-SLiM`,
- `-BluzgiTeam`,
- `-CiNEF0X`,
- `-Arbsom`,
- `-Hunt3r`,
- `-4PT`.

To jest lista dozwolonych grup.

#### `except_releases`

```regex
(720p\.|Remux|2160p\.|\.PL\.|TrueHD|\.MAX\.)
```

Filtr odrzuca release, jeśli nazwa zawiera:

- `720p.` — nie chce 720p,
- `Remux` — nie chce remuxów,
- `2160p.` — nie chce 4K,
- `.PL.` — nie chce wydań oznaczonych jako PL,
- `TrueHD` — nie chce ścieżek TrueHD,
- `.MAX.` — nie chce tagu MAX.

#### `use_regex: true`

Pola `match_releases` i `except_releases` są traktowane jako regex.

#### `announce_types: ["NEW"]`

Filtr reaguje na nowe torrenty.

#### `resolutions: ["1080p"]`

Tylko 1080p.

#### `codecs: ["H.264", "x264"]`

Tylko H.264/x264.

#### `except_other: ["REMUX"]`

Dodatkowe zabezpieczenie przed remuxami.

#### `years: "2025,2026"`

Tylko filmy z 2025 albo 2026.

#### `match_categories: "*movies?HD*"`

Kategoria musi wyglądać jak Movies/HD, Movies-HD, Movies HD itp.

#### `except_language`

Odrzuca wskazane języki.

---

## 17. Przykład: prosty filtr filmów HD 1080p

Cel:

- filmy 1080p,
- H.264/x264,
- minimum 4 GB,
- bez remuxów,
- lata 2025–2026,
- kategoria filmowa HD.

Ustawienia:

```text
Name: Filmy HD 1080p
Enabled: true
Announce Type: NEW
Min size: 4GB
Resolution: 1080p
Codecs: H.264, x264
Years: 2025,2026
Match categories: *movies?HD*
Except other: REMUX
Except releases: Remux, 720p, 2160p
```

Gotowy JSON do skopiowania/importu:

```json
{
  "name": "Filmy HD 1080p",
  "version": "1.0",
  "data": {
    "enabled": true,
    "min_size": "4GB",
    "priority": 0,
    "announce_types": [
      "NEW"
    ],
    "resolutions": [
      "1080p"
    ],
    "codecs": [
      "H.264",
      "x264"
    ],
    "years": "2025,2026",
    "match_categories": "*movies?HD*",
    "except_other": [
      "REMUX"
    ],
    "except_releases": "(720p\\.|Remux|2160p\\.)",
    "use_regex": true,
    "is_auto_updated": false,
    "release_profile_duplicate": null
  }
}
```

Ten sam przykład jest też jako plik:

```text
examples/filmy-hd-1080p.json
```

---

## 18. Przykład: filtr filmów 4K, ale bez remuxów

Cel:

- tylko 2160p,
- WEB albo BluRay,
- bez remuxów,
- rozmiar 10–35 GB.

Ustawienia:

```text
Name: Filmy 4K bez remux
Enabled: true
Announce Type: NEW
Min size: 10GB
Max size: 35GB
Resolution: 2160p
Sources: WEB, WEB-DL, WEBRip, BluRay
Except other: REMUX
Except releases: Remux
Match categories: *Movie*, *4K*, *UHD*
```

Gotowy JSON do skopiowania/importu:

```json
{
  "name": "Filmy 4K bez remux",
  "version": "1.0",
  "data": {
    "enabled": true,
    "min_size": "10GB",
    "max_size": "35GB",
    "priority": 0,
    "announce_types": [
      "NEW"
    ],
    "resolutions": [
      "2160p"
    ],
    "sources": [
      "WEB",
      "WEB-DL",
      "WEBRip",
      "BluRay"
    ],
    "match_categories": "*Movie*,*4K*,*UHD*",
    "except_other": [
      "REMUX"
    ],
    "except_releases": "Remux",
    "use_regex": false,
    "is_auto_updated": false,
    "release_profile_duplicate": null
  }
}
```

Ten sam przykład jest też jako plik:

```text
examples/filmy-4k-bez-remux.json
```

---

## 19. Przykład: seriale 1080p WEB dla Sonarr

Cel:

- odcinki seriali,
- 1080p,
- WEB/WEB-DL/WEBRip,
- bez paczek sezonowych,
- wysyłka do Sonarr.

Ustawienia:

```text
Name: Seriale 1080p WEB Sonarr
Enabled: true
Announce Type: NEW
Resolution: 1080p
Sources: WEB, WEB-DL, WEBRip
Seasons: 1-99
Episodes: 1-99
Match categories: *TV*, *Episode*
```

Akcja:

```text
Action: Sonarr
Client: Twoja instancja Sonarr
```

Gotowy JSON warunków filtra do skopiowania/importu:

```json
{
  "name": "Seriale 1080p WEB Sonarr",
  "version": "1.0",
  "data": {
    "enabled": true,
    "priority": 0,
    "announce_types": [
      "NEW"
    ],
    "resolutions": [
      "1080p"
    ],
    "sources": [
      "WEB",
      "WEB-DL",
      "WEBRip"
    ],
    "seasons": "1-99",
    "episodes": "1-99",
    "match_categories": "*TV*,*Episode*",
    "is_auto_updated": false,
    "release_profile_duplicate": null
  }
}
```

Uwaga: akcję Sonarr nadal trzeba wskazać w autobrr, bo zależy od Twojej konkretnej konfiguracji klienta.

Ten sam przykład jest też jako plik:

```text
examples/seriale-1080p-web-sonarr.json
```

---

## 20. Przykład: tylko paczki sezonowe

Cel:

- tylko season packi,
- bez pojedynczych odcinków.

Ustawienia:

```text
Name: Seriale season packi
Enabled: true
Announce Type: NEW
Resolution: 1080p
Sources: WEB, WEB-DL, WEBRip, BluRay
Seasons: 1-99
Episodes: 0
Match categories: *TV*, *Season*
```

Gotowy JSON do skopiowania/importu:

```json
{
  "name": "Seriale season packi",
  "version": "1.0",
  "data": {
    "enabled": true,
    "priority": 0,
    "announce_types": [
      "NEW"
    ],
    "resolutions": [
      "1080p"
    ],
    "sources": [
      "WEB",
      "WEB-DL",
      "WEBRip",
      "BluRay"
    ],
    "seasons": "1-99",
    "episodes": "0",
    "match_categories": "*TV*,*Season*",
    "is_auto_updated": false,
    "release_profile_duplicate": null
  }
}
```

Ten sam przykład jest też jako plik:

```text
examples/seriale-season-packi.json
```

---

## 21. Przykład: filtr freeleech na budowanie bufora

Cel:

- łapać tylko freeleech,
- nie zalać klienta torrentami.

Ustawienia:

```text
Name: Freeleech buffer
Enabled: true
Announce Type: NEW
Freeleech: true
Max downloads: 5
Max downloads per: day
```

Dodatkowo w kliencie qBittorrent/Deluge ustaw limit aktywnych pobrań w regułach klienta autobrr, np. `Max active downloads: 2`.

Uwaga: nie myl tego z wewnętrzną kolejką qBittorrenta. Wysyłanie wielu torrentów jako paused może szkodzić ratio, bo torrent trafia do klienta, ale nie seeduje od razu.

---

## 22. Jak testować filtr

Najbezpieczniejsza kolejność:

1. Utwórz filtr.
2. Dodaj akcję `Test` zamiast qBittorrent.
3. Zapisz.
4. Poczekaj na announce albo sprawdź logi.
5. Jeśli filtr łapie dobre rzeczy, dodaj właściwą akcję.
6. Jeśli łapie śmieci, dodaj `Except releases`, `Except other`, `Except language` albo doprecyzuj kategorię.

---

## 23. Najczęstsze błędy początkujących

### Błąd 1: za szeroki filtr

Przykład:

```text
Match categories: *Movie*
```

Bez rozdzielczości, kodeka, rozmiaru i wyjątków filtr może łapać prawie wszystko z filmów.

### Błąd 2: za ciasny filtr

Przykład:

```text
Resolution: 1080p
Source: WEB-DL
Codec: x264
Container: mkv
Category: Movies/HD
Group: JednaGrupa
Year: 2026
```

Jeśli tracker nie poda kontenera albo nazwie kategorię inaczej, filtr nic nie złapie.

### Błąd 3: filtrowanie po kontenerze

Wiele trackerów nie ogłasza `mkv`/`mp4`. Zostaw puste, jeśli nie masz pewności.

### Błąd 4: brak akcji

Filtr pasuje, ale nic się nie dzieje, bo nie ma akcji.

### Błąd 5: zmiana akcji bez zapisania filtra

Po dodaniu albo edycji akcji trzeba zapisać filtr.

### Błąd 6: regex bez uciekania kropki

Źle:

```regex
.PL.
```

To oznacza: dowolny znak + `PL` + dowolny znak.

Lepiej:

```regex
\.PL\.
```

To oznacza dosłownie `.PL.`.

---

## 24. Szybka ściąga

### Chcę tylko 1080p

```text
Resolution: 1080p
```

### Nie chcę 720p i 4K

```text
Except releases: 720p, 2160p
```

albo regex:

```regex
(720p\.|2160p\.)
```

### Nie chcę remuxów

```text
Except other: REMUX
Except releases: Remux
```

### Chcę tylko nowe uploady

```text
Announce Type: NEW
```

### Chcę lata 2025 i 2026

```text
Years: 2025,2026
```

### Chcę tylko konkretne grupy

Regex w `Match releases`:

```regex
(-GROUP1|-GROUP2|-GROUP3)
```

### Chcę odrzucić konkretne grupy

Regex w `Except releases`:

```regex
(-BADGROUP1|-BADGROUP2)
```

---

## 25. Proponowana metoda budowania dobrego filtra

Nie zaczynaj od skomplikowanego regexa.

Buduj filtr warstwami:

1. Kategoria: film/serial.
2. Rozdzielczość: 1080p albo 2160p.
3. Źródło: WEB/BluRay.
4. Kodek: x264/H.264 albo x265/H.265.
5. Rozmiar: minimum i maksimum.
6. Wyjątki: Remux, TrueHD, niechciane języki, grupy.
7. Akcja testowa.
8. Dopiero potem akcja pobierania.

Dzięki temu łatwo zobaczysz, który warunek psuje filtr.

---

## 26. Minimalny dobry filtr dla początkującego

```text
Name: Filmy 1080p start
Enabled: true
Announce Type: NEW
Min size: 4GB
Max size: 15GB
Resolution: 1080p
Sources: WEB, WEB-DL, WEBRip, BluRay
Codecs: H.264, x264
Match categories: *Movie*, *movies?HD*
Except other: REMUX
Except releases: Remux, 720p, 2160p
Action: Test
```

Jeśli testy wyglądają dobrze, zmień akcję z `Test` na qBittorrent/Radarr.

---

## 27. Gotowy JSON na bazie przykładu PEPE

Plik z przykładowym filtrem jest też w repozytorium:

```text
examples/glowne-filmy-hd.json
```

Możesz go potraktować jako wzór i zmieniać:

- lata,
- grupy,
- kategorie,
- rozmiary,
- wyjątki.

Najważniejsze: po każdej większej zmianie najpierw testuj, potem pobieraj.
