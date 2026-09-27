# KošarkaOS – zapisnik in informacijski sistem za košarkarske tekme

Ena sama HTML datoteka (`index.html`), ki deluje popolnoma samostojno v brskalniku – brez strežnika, brez namestitve. Vsi podatki (tekme, sezone, ekipe, nastavitve) se shranjujejo lokalno v brskalniku uporabnika (`localStorage`).

## Kaj vsebuje
- Uro tekme (start/stop, ±1s/±10s, štiri (ali poljubno) četrtine, popravek nazaj/naprej)
- Vse kategorije prekrškov in sodniških odločitev za hiter izbor + polje za poseben dogodek
- Polno statistiko vsakega igralca: točke, met 2/3/prosti met (%), napadalni/obrambni skoki, asistence, ukradene žoge, blokade, izgube žoge, osebni prekrški, minutaža, EFF indeks
- Kapetan, začetna peterka, menjave (z avtomatskim zapisom v živo zapisnik)
- Trenerja obeh ekip, datum, ura, kraj, tekmovanje/krog, sodnika, delegat, zapisnikar
- Enostaven Shot Chart (klik na igrišče + zadetek/zgrešen)
- Ekipna statistika v živo, box score po tekmi (arhiv), izvoz box scora v CSV
- Potrditev zapisnika s strani zapisnikarja pred zaključkom tekme
- Sezone brez omejitve let, lestvico (samodejni izračun Z/P/točke/razlika) in SVG grafikon
- Arhiv vseh tekem vseh sezon z razčlenjeno statistiko obeh ekip
- Nastavitve: logo, barva, dolžina/število četrtin, št. timeoutov
- Temni/svetli način
- Izvoz/uvoz celotne baze in posamezne tekme kot JSON, izvoz CSV, tisk/PDF izvoz vsake rubrike posebej

## Kaj (zavestno) ni vključeno
Naslednje zahteve iz specifikacije zahtevajo strežnik, bazo v oblaku, večuporabniško prijavo ali strojno opremo, zato jih v samostojni HTML datoteki brez zaledja ni mogoče zanesljivo narediti:
- Spletni portal/mobilna aplikacija za navijače, javni API za tretje osebe
- Povezava s fizičnim semaforjem, 24-sekundno napravo ali TV-grafiko
- Videoanaliza in sinhronizacija dogodkov s posnetki
- Uporabniške vloge/pravice in revizijska sled med več računalniki (ker ni skupne baze v oblaku, gre le za en brskalnik/napravo)
- Upravljanje celotne lige/zveze (registracije klubov, prestopi, generiranje razporedov)

Če kdaj želiš te zmogljivosti, potrebuješ pravo spletno aplikacijo s strežnikom in bazo (npr. na Cloudflare Workers + D1, kar si prej že omenjal za drug projekt) – to je lahko naslednji korak.

## 1) Namestitev na GitHub Pages (javna aplikacija)
1. Ustvari **javni (public)** repozitorij, npr. `kosarka-os`.
2. Naloži vanj datoteko `index.html` (iz tega paketa) v koren repozitorija.
3. Pojdi v **Settings → Pages**, pod "Build and deployment" izberi **Deploy from a branch**, veja `main`, mapa `/ (root)` → Save.
4. Po nekaj minutah bo aplikacija dostopna na `https://<uporabnik>.github.io/kosarka-os/`.

## 2) Shranjevanje podatkov (private repozitorij)
Aplikacija zaradi varnostnih omejitev brskalnika **ne more sama pisati** v GitHub, Google Drive ali na disk – nobena spletna stran tega ne more narediti neposredno. Deluje pa tale, dejansko preverjen postopek:

1. Ustvari **zaseben (private)** repozitorij, npr. `kosarka-os-data`, in ga klonira lokalno (npr. z GitHub Desktop) v mapo na C: disku.
2. V aplikaciji po vsaki tekmi ali na koncu dneva klikni **"Izvozi vso bazo (JSON)"** (v zavihku Nastavitve) in datoteko `kosarka_baza.json` shrani v to lokalno klonirano mapo (prepiši prejšnjo datoteko).
3. V GitHub Desktop (ali `git add . && git commit -m "posodobitev" && git push`) narediš commit + push → podatki so varno v tvojem zasebnem repozitoriju z zgodovino sprememb.
4. Če želiš podatke tudi na Google Drive: isto mapo, kamor shranjuješ JSON, nastavi kot mapo, ki jo sinhronizira **Google Drive for Desktop** – tako dobiš kopijo v oblaku popolnoma samodejno, brez ročnega nalaganja po spletu.
5. Ob naslednjem odprtju aplikacije (na drugem računalniku ali po brisanju predpomnilnika brskalnika) v Nastavitvah uporabi **"Uvozi JSON"** in izberi svojo shranjeno datoteko – vsi podatki se povrnejo.

## 3) Gumb "⚙ Admin" – neposredno shranjevanje iz aplikacije
Zgoraj desno je gumb **Admin**, kjer nastaviš tri neodvisne načine shranjevanja (lahko uporabiš enega, dva ali vse tri). Token/ključi se shranijo samo lokalno v tvojem brskalniku, ločeno od izvožene baze, in se nikoli ne delijo nikamor drugam.

### GitHub (zaseben repozitorij) – priporočeno
1. Na GitHub pojdi na **Settings → Developer settings → Personal access tokens → Fine-grained tokens** in ustvari nov token z dostopom samo do svojega zasebnega repozitorija (npr. `kosarka-os-data`) in pravico **Contents: Read and write**.
2. V Adminu vpiši token, uporabniško ime, ime repozitorija, vejo (`main`) in pot do datoteke (npr. `kosarka_baza.json`).
3. Klikni **"Shrani nastavitve"**, nato **"Naloži bazo na GitHub zdaj"** – aplikacija samodejno prebere obstoječo datoteko (če obstaja) in jo posodobi neposredno prek GitHub API (brez git ukazov, brez GitHub Desktop).

### Mapa na tem računalniku
Klikni **"Izberi mapo"** (deluje v Chrome/Edge – uporablja File System Access API) in nato **"Shrani bazo v mapo zdaj"** – datoteka `kosarka_baza.json` se zapiše neposredno v izbrano mapo na tvojem disku (lahko izbereš mapo, ki jo sinhronizira Google Drive for Desktop, in imaš s tem tudi samodejno kopijo v oblaku). Po vsakem ponovnem nalaganju strani je treba mapo znova izbrati (brskalniška varnostna omejitev).

### Google Drive (neposredno prek API-ja)
To zahteva tvoj lasten Google Cloud projekt (ker Google zahteva registracijo vsake aplikacije):
1. V [Google Cloud Console](https://console.cloud.google.com/) ustvari projekt, omogoči **Google Drive API** in ustvari **OAuth 2.0 Client ID** tipa "Web application".
2. Med "Authorized JavaScript origins" dodaj naslov, kjer bo aplikacija gostovana (npr. `https://<uporabnik>.github.io`).
3. V Adminu vnesi ta Client ID, klikni **"Prijava v Google"** (odpre se Googlovo okno za prijavo/dovoljenje), nato **"Shrani bazo na Drive"** – datoteka se naloži v tvoj Google Drive (vsak klik ustvari novo datoteko `kosarka_baza.json`, ker preprosta različica ne preverja/prepisuje obstoječe).

## Opomba
Ker gre za samostojno stran brez strežnika, si podatki niso deljeni med različnimi napravami/brskalniki avtomatsko – zato je zgornji izvoz/uvoz + git commit trenutno edini zanesljiv način za varnostno kopijo in prenos podatkov med napravami.
