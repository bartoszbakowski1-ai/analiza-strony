---
name: analiza-strony
description: 'Analiza strony internetowej LEADA (kampania szkoły językowe albo inny lead biznesowy) pod kątem sprzedaży. Wynik ZAWSZE w rozmowie, nigdy jako plik: 5-7 krótkich punktów do poprawy w 4 kategoriach (Strona, Analityka, Google, Reklamy Meta), każdy w jednej linii z liczbą, plus jedno zdanie propozycji. Bez sekcji "czego nie mówić", "nie zweryfikowałem" i bez rozpisywania. Pomiary robione wielometodowo (Playwright, Firecrawl, realny SERP, Ad Library), ale do rozmowy idzie tylko krótka lista. Użyj gdy Bartek mówi "przeanalizuj stronę leada", "zrób analizę strony", "lecimy z analizą", "/analiza-strony", podaje URL strony szkoły albo leada z kampanii, albo chce wideo dla leada. NIE dotyczy oceny estetyki/AI-look (to audyt-anti-ai) ani stron własnych projektów (to sprawdz-kod). Pipeline: kontekst z Notion+Gmail PRZED stroną, render ze scrollem i screenshotem, pomiary, CHECKPOINT w rozmowie (Bartek zatwierdza punkty i sam nagrywa), draft maila.'
---

# Skill: analiza-strony

Analiza strony leada, ktorej wynik trafia do wideo i maila sprzedazowego. Kazde zdanie, ktore dostanie lead, musi byc wczesniej zmierzone dwiema niezaleznymi metodami. Ten skill powstal po sesji 11.08.2026, w ktorej do materialow trafily trzy falszywe twierdzenia (lazy-load, SERP, zmyslona dzielnica).

**Bartek nie czyta raportow. Bartek nagrywa wideo.** Caly pomiar sluzy jednemu: podac mu 5-7 krotkich punktow do poprawy, ktore przeskanuje wzrokiem, zapisze sobie i opowie przed kamera. Wszystko, co nie trafi do tych punktow, jest praca wewnetrzna i NIE pokazuje sie w rozmowie (decyzje Bartka 19.08.2026 i 19.09.2026: "w krotkich punktach, jak najprostsze, bez rozwodzenia sie nad tematem").

## ROBI / NIE ROBI

ROBI:
- zbiera kontekst leada (Notion, Gmail, Instantly) ZANIM otworzy strone
- mierzy strone przez Playwright (render, scroll, screenshot) + Firecrawl jako drugie zrodlo
- sprawdza SERP w prawdziwym Google, reklamy w Meta Ad Library, mobile, podstrony, analityke, platforme, pomiar, zgode, SEO, sciezke do kontaktu
- **oddaje wynik W ROZMOWIE jako 5-7 krotkich punktow w 4 kategoriach (Strona, Analityka, Google, Reklamy Meta) plus jedno zdanie propozycji. Nic wiecej (format w KROKU 6)**
- po checkpoincie Bartek NAGRYWA SAM z zatwierdzonych punktow; po nagraniu skill pisze draft Gmail i aktualizuje Notion

NIE ROBI:
- **NIE tworzy pliku .md z analiza, NIE wysyla przez SendUserFile, NIE robi artefaktu.** Wynik idzie w rozmowie i tyle
- **NIE pokazuje tabel, karty technicznej, sekcji ZMIERZONE/WNIOSKI, list sygnalow za i przeciw.** To sa narzedzia myslenia, nie output
- **NIE pisze sekcji "Czego nie mowic", "Nie zweryfikowalem", "Na czym to postawione" ani linii "Co powiedziec" / "Pokaz"** (decyzja Bartka 19.09.2026). Rzecz ryzykowna albo niepotwierdzona dwiema metodami po prostu nie trafia na liste
- nie ocenia estetyki/AI-look (deleguj do `audyt-anti-ai`, mozna wywolac jako uzupelnienie)
- nie pisze skryptu do nagrywania (Bartek nagrywa z listy punktow wlasnymi slowami, decyzja 11.08.2026)
- nie wysyla NICZEGO (mail tylko jako draft, wysylka wylacznie po jawnym "wyslij" Bartka)
- nie uzywa slow "audyt", "doradztwo", "consulting" w zadnym materiale dla leada (VAT, regula 2026-08-11)
- nie pisze draftu przed zatwierdzeniem punktow przez Bartka i przed linkiem do nagrania

## Zanim zaczniesz (obowiazkowe)

1. Przeczytaj `~/.claude/projects/-Users-bartekbakowski/memory/feedback_analiza_stron_fakt_vs_wniosek.md` (NIE laduje sie sam).
2. Przeczytaj `references/protokol-pomiaru.md` (gotowe, przetestowane snippety) i `references/szablony.md`.
3. Do pomiaru uzyj gotowych skryptow z `lokalnego folderu ze skryptami dc/` (instrukcja `dc/INSTRUKCJA.md`): `dc_site.py` (render z ukrytym webdriverem, baner zgody, siec przed zgoda, po kliknieciu i po przeladowaniu, static HTML jako metoda 2), `dc_adlib.py` + MCP `ads_library_search` (liczba reklam z API, adresy docelowe ze scrapera), `dc_serper.py`, `dc_psi.py`. Headless bez ukrytego webdrivera nie widzi czesci banerow (lekcja jednego z leadów 19.09.2026).

## Twarde reguly (lamanie ktorejkolwiek = material do kosza)

- **Adres e-mail leada bierzesz ze ZGLOSZENIA** (mail z formularza w Gmail, naglowek "Szkoly: zgloszenie na wideo"), nigdy ze strony. Lekcja: Julia podala w formularzu inny adres niz na stronie.
- **Nic z glowy.** Lokalizacje, liczby, nazwy, dzielnice tylko doslownie z ich strony albo zgloszenia. Lekcja: "Bielany" vs Zoliborz.
- **"W Google wyglada tak" mowisz tylko po sprawdzeniu W GOOGLE** (`site:domena.pl`), nie po tagu `<title>`. Google przepisuje tytuly i opisy.
- **Pomiar tekstu dopiero PO scrollu do dna i odczekaniu.** Sekcje lazy-load i animowane liczniki nie istnieja przed scrollem. Lekcja: "zero liczb" gdy na dole staly 12+/4000+/50+/7+.
- **Server-side (CAPI itp.) tylko z configu wtyczki** (`pysOptions.facebook.serverApiEnabled`, `eventID`), nigdy z ruchu przegladarki.
- **NIE WNIOSKUJ O KONFIGURACJI KAMPANII Z TEGO, CO WIDAC W PRZEGLADARCE** (lekcja Avalon, 13.08.2026). Z zewnatrz NIE WIDAC: zawartosci kontenera GTM, liczby akcji konwersji w Google Ads i tego, ktora steruje kampaniami, konwersji importowanych offline z CRM, ani zdarzen, ktorych sam nie wywolales (nie klikales w telefon, nie wysylales formularza). Zdania typu "kupuja wejscia zamiast kontaktow", "optymalizuja sie pod odslony", "ich reklamy nie widza zapisow" to WNIOSKI, ktore moga byc falszywe. Zmierzony fakt: "przy zaladowaniu strony leci tylko PageView, a etykieta konwersji Google Ads odpala sie na 3 z 3 sprawdzonych podstron". Do leada idzie PYTANIE ("czy kampanie maja ustawiona konwersje na zgloszenie, czy na wejscie"), nie diagnoza.
- **Liczniki animowane sprawdzaj PRAWDZIWYM scrollem** (petla `window.scrollTo` co ~200 px z opoznieniem), nie `scrollIntoView` i nie ze zrzutu `fullPage`. Zrzut calej strony i Firecrawl czesto lapia je na wartosci poczatkowej (zera), mimo ze uzytkownik widzi poprawne liczby. Zanim nazwiesz licznik zepsutym, porownaj `data-end` z tym, co pokazuje po realnym scrollu (lekcja Avalon: "+ 0 tys." na zrzucie, faktycznie 22 tys.).
- **Norma rynkowa to nie wada:** baner cookies, brak cen w uslugach premium, formularz zamiast koszyka. Zanim nazwiesz cos bledem, poszukaj na stronie zdania, ktore to tlumaczy ("cene podajemy na rozmowie" = model sprzedazy).
- Kazde twierdzenie w materiale dla leada = 2 niezalezne metody pomiaru. Jedna metoda = WNIOSEK, nie fakt.

## KROK 0 - kontekst PRZED dotknieciem strony

Bez tego kroku ZAKAZ pisania czegokolwiek.

1. **Notion Leady** (`collection://3b815601-6bef-8166-95c4-000bb4160451`): czy lead istnieje, status, historia, notatki. Pole **"Co ich uwiera" = os calej analizy** (wideo ma odpowiadac na JEGO bol, nie na moja liste).
2. **Gmail**: szukaj po domenie/imieniu/adresie (`search_threads`). Znajdz zgloszenie z formularza (stamtad adres i tresc ankiety) oraz poprzednie watki. Z wyslanych maili Bartka odczytaj ton (Pan/Pani, forma).
3. **Instantly** (gdy zrodlo = cold mail): `GET /api/v2/emails?lead=<email>` z kluczem z `mcp-servers.md`. Ustal, co lead juz dostal i co odpisal.
4. Zanotuj w naglowku analizy: imie, adres ze zgloszenia, telefon, skala szkoly, "Co ich uwiera", zrodlo, dotychczasowa korespondencja.

## KROK 1 - render (Playwright, nigdy WebFetch/curl do tresci)

Kolejnosc jest czescia protokolu:

1. `browser_navigate` na strone glowna.
2. **Akceptuj cookies** (snippet w protokole). Bez tego analityka moze byc uspiona i pomiar sklamie.
3. **Scroll do samego dna + wait 2,5 s, potem scroll na 70% + wait 1,5 s** (lazy-load, liczniki animowane dobiegaja do wartosci koncowych).
4. **Screenshot fullPage i OBEJRZYJ go przez Read.** Wzrok lapie sekcje, ktore skrypty gubia. To jest obowiazkowy punkt, nie ozdoba.
5. **Sprawdz robots.txt i sitemape** (protokol pkt 12b). Wczesnie, bo to bywa cala analiza: u szkoły dla dzieci sitemapa miala 1496 z 1496 adresow o kasynach, opublikowanych pod domena szkoly jezykowej dla dzieci.
6. Dopiero teraz pomiary DOM.

## KROK 2 - KARTA TECHNICZNA (obowiazkowa, ale WEWNETRZNA)

To jest fundament calej analizy. Wypelnij KAZDY wiersz u siebie. Nie wolno zostawic pustego pola: jak czegos nie da sie sprawdzic z zewnatrz, wpisz **NIE ZWERYFIKOWANO** i napisz dlaczego. Zgadywanie = material do kosza.

**Karty NIE pokazujesz Bartkowi.** Z niej biora sie tylko: platforma w nawiasie przy nazwie szkoly i liczby w punktach. Reszta zostaje u ciebie jako zabezpieczenie przed falszywym twierdzeniem.

Snippet zbierajacy wieksza czesc naraz: `references/protokol-pomiaru.md` pkt 13.

### A. Fundament

| Pozycja | Co zmierzyc | Uwagi |
|---|---|---|
| Platforma / CMS | `meta[name=generator]`, sciezki plikow (`wp-content`, `staticContent`), ciasteczka (`ww_worker` = WebWave), naglowek `Server` | Podaj nazwe I wersje |
| Wersja aktualna? | porownaj z aktualna wersja platformy | WordPress: podaj wersje i ile lat ma galaz. **Wersja z lataniem bezpieczenstwa to NIE to samo co porzucona** |
| Builder / motyw | Elementor, Divi, WebWave, Landingi, Next.js | |
| Hosting | naglowek `Server`, CDN | |
| HTTPS i przekierowania | `curl -sIL`, liczba przeskokow, czas | 1 przeskok 301 = OK, nie robic z tego problemu |

### B. Pomiar (najwazniejsza czesc)

| Pozycja | Stan | Jak sprawdzic |
|---|---|---|
| Google Analytics 4 | ID `G-XXXX` albo **BRAK** | request `gtag/js?id=`, `g/collect?tid=`, ciasteczko `_ga_XXXX`. **`gtag` undefined NIE dowodzi braku GA4**, gdy leci przez GTM |
| Google Tag Manager | ID `GTM-XXXX` albo BRAK | |
| Meta Pixel | ID + **ile sztuk** | `fbq('init')` inline oraz requesty `signals/config/`. Wiecej niz jedno ID = duplikat zdarzen |
| Conversion API | WLACZONE / WYLACZONE / **NIE ZWERYFIKOWANO** | TYLKO z configu wtyczki (`pysOptions.facebook.serverApiEnabled`) albo `eventID`. Nigdy z ruchu przegladarki |
| Inne narzedzia | Clarity, Hotjar, Matomo | |
| Zdarzenia konwersji | czy klik w `tel:`, `mailto:`, submit sa zdarzeniami | Zwykle BRAK. To osobny, mocny punkt |
| **Pomiar tam, gdzie lada reklamy** | czy strona docelowa reklam ma pixel | Sprawdz cel reklam w Ad Library i porownaj z pomiarem NA TEJ stronie. Lekcja z leada B: pomiar byl na innej domenie niz ruch |
| **Formularz w innej domenie** | czy formularz wychodzi na langlion/Google Forms/inna domene | Jesli tak: sprawdz pomiar TAM. Lekcja z leada C: zero pomiaru na formularzu = zero konwersji w Meta |
| **GRANICA POMIARU (zawsze wpisz)** | co da sie stwierdzic z zewnatrz, a czego NIE | Wpisz wprost: "z przegladarki widac tagi odpalane przy zaladowaniu; NIE widac zawartosci GTM, akcji konwersji w Google Ads ani konwersji offline". Ten wiersz chroni przed naciaganym wnioskiem i **sam w sobie jest dobrym punktem do filmu** (pokazuje, ze wiesz, czego nie wiesz) |

### B2. Lejek i lista (sprawdzaj ZAWSZE, gdy strona jest technicznie dobra)

Gdy karta techniczna wychodzi czysto, to znak, ze pieniadze leza nie w stronie, tylko w lejku. Wtedy te wiersze decyduja, czy jest co sprzedac:

| Pozycja | Co zmierzyc |
|---|---|
| System mailingowy | skan HTML pod `mailerlite`, `mailchimp`, `freshmail`, `getresponse`, `convertkit`, `activecampaign`, `brevo`, `klaviyo`. Brak wszystkich = brak listy albo lista poza strona |
| Zapis na liste | czy istnieje formularz newsletter/lead magnet i gdzie lezy |
| Lead magnet | test poziomujacy, quiz, e-book, kupon. Czy ZBIERA e-mail (`type="email"` na jego stronie) i co obiecuje w zamian |
| Co sie dzieje z adresem | czy widac slad sekwencji, czy adres tylko trafia do skrzynki. Zwykle NIE ZWERYFIKOWANO z zewnatrz: pytanie na rozmowe |
| Social jako zrodlo | linki do FB/IG/YT/TikTok ze strony (kanaly, w ktorych mozna puscic ruch) |

Lekcja z leada E 13.08.2026: strona techniczna bez zarzutu, ale test poziomujacy zbiera adresy i obiecuje kod rabatowy, a w kodzie nie ma ZADNEGO systemu mailingowego. To jest realna oferta (lejek plus mailing plus kampanie), podczas gdy sama strona nie dawala sie uczciwie sprzedac.

### C. Zgoda i temat prawny

| Pozycja | Stan |
|---|---|
| Baner zgody | jest (jaka wtyczka) / BRAK |
| Czy `_ga` i `_fbp` leca PRZED zgoda | sprawdz przy PIERWSZEJ wizycie, przed klikaniem |
| Consent Mode | czy kategorie nie sa zahardkodowane na `granted` |
| Polityka prywatnosci | jest / brak, gdzie |

Formuluj jako "temat do sprawdzenia z Waszym prawnikiem", NIGDY jako "lamiecie RODO".

### D. SEO podstawy (to, co Bartek ma wymienic na wideo)

| Pozycja | Co podac |
|---|---|
| **Meta title** | DOSLOWNIE + liczba znakow. Norma 50-60. Zaznacz, czy niesie miasto i usluge, czy zaczyna sie od smiecia typu "Home" |
| **Meta description** | DOSLOWNIE + liczba znakow albo **BRAK**. Norma 140-160. Sprawdz popsute znaki i emoji |
| **H1** | ile sztuk (0? 3?) + tresc. 0 albo >1 to konkret do wymienienia |
| Struktura H2/H3 | czy opisowe, czy puste albo same liczby |
| Canonical | jest / brak / wskazuje gdzie |
| **Realny SERP** | `site:domena.pl`, zacytuj co Google FAKTYCZNIE pokazuje dla strony glownej i 2-3 podstron |
| Podstrony bez meta description | ile z ilu sprawdzonych |
| **Sitemap: integralnosc domeny** | **Adres sitemapy WEZ Z robots.txt** (bywa niestandardowy). Policz adresy i sprawdz, ile jest nie-na-temat. Liczba razaco wieksza niz menu = czerwona flaga, patrz protokol pkt 12b |
| Sitemap i robots | `/sitemap.xml`, `/robots.txt` status |
| **Brakujace strony pod fraze** | czy istnieje URL pod glowna usluge (np. "online"). Lekcja z leada B: 8 pozycji menu "online", zero adresow z "online" |

### E. Konwersja

| Pozycja | Co podac |
|---|---|
| Formularz na stronie glownej | jest / **BRAK** |
| Najkrotsza sciezka do kontaktu | ile klikniec, ile pol, ktore wymagane |
| Pola odstraszajace | adres domowy, PESEL, data urodzenia, liczba zgod |
| Telefon klikalny (`tel:`) | jest / brak, na jakiej wysokosci strony |
| CTA | tresc i pozycja w procentach wysokosci; czy najnizszy prog jest wyzej niz najwyzszy |

### F. Wydajnosc i mobile

| Pozycja | Co podac |
|---|---|
| Waga HTML | rozmiar dokumentu |
| **PageSpeed (mobile)** | wynik wydajnosci z PageSpeed Insights. API: `https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=<URL>&strategy=mobile&category=performance`. **Limit dzienny bywa wyczerpany (HTTP 429)** - wtedy Bartek odpala recznie w przegladarce i podaje wynik. Ponizej 60 to konkret do wymienienia, powyzej 85 to argument PRZECIW przebudowie |
| Mobile 390x844 | poziomy scroll, CTA nad zagieciem, tap targets < 40 px |

## KROK 3 - pomiary (kategoria = przypisana metoda + double check)

| Kategoria | Metoda 1 | Metoda 2 (double check) |
|---|---|---|
| Tresc, liczby, daty, ceny | `innerText` PO scrollu | Firecrawl markdown (`firecrawl_scrape.sh`); rozbieznosc = badaj, nie zgaduj |
| Analityka | obiekty globalne (gtag, fbq, dataLayer, clarity, hj) | requesty sieciowe do domen analitycznych + skan inline pod ID (`G-`, `GTM-`, `fbq('init'`) |
| Server-side / CAPI | config wtyczki (`pysOptions`) | obecnosc `eventID` przy zdarzeniach (deduplikacja) |
| SERP | `google.com/search?q=site:domena.pl` przez Playwright | poszczegolne frazy komercyjne, jesli istotne |
| Reklamy Meta | Ad Library: aktywne | Ad Library: wszystkie (zakresy dat, kto reklamodawca) |
| Mobile | 390x844: fold, pozycja CTA, tap targets, fonty | screenshot mobile + Read |
| Podstrony | fetch w kontekscie strony: /cennik/, /kontakt/, oferta, 2-3 strony kursow | daty i ceny cytuj doslownie |
| Formularze | pola, required, action, AJAX vs redirect | tresc strony podziekowania, jesli jest |

Wszystkie snippety gotowe w `references/protokol-pomiaru.md`. Firecrawl 408 = fallback Playwright (udokumentowany quirk VPS).

## KROK 4 - klasyfikacja (rdzen skilla)

Kazdy punkt przechodzi filtr 3 pytan:
1. Zmierzone czy zalozone?
2. Usterka czy swiadoma decyzja wlasciciela? (szukaj zdania, ktore to tlumaczy)
3. Czy dotyka pieniedzy leada?

Punkt, ktory nie przechodzi wszystkich trzech, **wypada z listy**. Nie przenosisz go do zadnej innej sekcji, bo takich sekcji w wyniku nie ma.

Rozdziel u siebie dwa koszyki (to praca wewnetrzna, do rozmowy idzie tylko wynik):
- **ZMIERZONE**: punkt | metoda 1 | metoda 2 | doslowny cytat lub liczba. **Tylko z tego koszyka robisz punkty do wideo.**
- **WNIOSEK**: interpretacje ("mam hipoteze, ze..."). Do wyniku NIE ida.

## KROK 5 - WERDYKT: nowa strona czy poprawki

Cel handlowy Bartka to sprzedac postawienie strony od nowa, ale **werdykt musi wynikac z karty technicznej, nie z checi sprzedazy**. Naciagany werdykt konczy sie tak jak leada A: trzy argumenty, dwa do kosza, lead stracony.

Policz sygnaly z KROKU 2.

**Sygnaly ZA postawieniem od nowa** (kazdy musi byc zmierzony):
- platforma przestarzala albo porzucona (stary CMS bez aktualizacji, kreator z sufitem funkcji)
- **dwa lub wiecej rownoleglych systemow** z osobnymi abonamentami, niepolaczonych (Lekcja z leada B: WebWave + Landingi)
- brak GA4 albo brak pixela na stronie, na ktora leci platny ruch
- formularz albo koszyk w obcej domenie bez pomiaru (Lekcja z leada C: langlion)
- brak H1 albo brak meta description na wiekszosci podstron
- **domena zaspamowana wstrzyknieta trescia** (kasyna, pigulki, pozyczki) opublikowana pod ich marka i zaindeksowana. To nie jest kosmetyka, to utrata domeny w Google. Sprzedawaj jako odzyskanie domeny, nie jako ladniejsza strone
- **brak stron pod glowna usluge** (usluga jest w menu i w nazwie firmy, nie ma jej pod zadnym adresem)
- brak formularza na stronie glownej albo sciezka do kontaktu dluzsza niz 2 klikniecia
- strona nie da sie rozbudowac bez placenia kreatorowi albo agencji za kazda zmiane

**Sygnaly PRZECIW** (jak sa, powiedz Bartkowi wprost, ze rebuild bedzie trudny do sprzedania):
- hero mowi o rezultacie, dowody spoleczne wysoko, teksty konkretne
- zgoda na cookies i pomiar zrobione poprawnie
- strona swiezo zrobiona albo wlasnie po rebrandingu
- w stopce jest agencja, ktora to zrobila (konkurujesz z ich swiezym wyborem)
- lead nie zglosil problemu ze strona, tylko z konwersja albo ruchem

Werdykt ustal u siebie w jednej z trzech form (w rozmowie zostaje z niego tylko jedna linia **Propozycja:**, patrz KROK 6):

1. **REBUILD UZASADNIONY** - wymien 3 najmocniejsze zmierzone sygnaly ZA, to sa punkty do filmu.
2. **REBUILD DO OBRONY** - sygnaly sa mieszane. Podaj, czego brakuje do pewnosci i jakie pytanie na rozmowie to rozstrzyga.
3. **REBUILD NIEUZASADNIONY** - napisz to Bartkowi WPROST plus zaproponuj, co sprzedac zamiast tego (pomiar, lejek, produkt cyfrowy, opieka). Lekcja z leada B: strona byla dobra, pieniadze lezaly w monetyzacji publicznosci.

**Zanim zaproponujesz "strona z AI, ktora sami ogarniecie": sprawdz, czym oni edytuja teraz.** Ta rama dziala przy kreatorach z sufitem i podwojnym abonamencie (WebWave plus Landingi u leada B). NIE dziala przy WordPressie z edytorem blokowym (Kadence Blocks, Elementor Pro, Divi), bo klient **juz** edytuje sam i wychwyci to w pierwszym zdaniu. Lekcja z leada E 13.08.2026.

**Gdy rebuild odpada, przejdz na oferte performance (to czesto wiecej pieniedzy niz strona):** kampanie Meta i Google, lejek, lead magnet, system mailingowy i sekwencje, pomiar konwersji. Materialu do tej rozmowy dostarcza sekcja B2 karty technicznej. Rama dla leada: "strona jest zrobiona dobrze, wiec nie ma sensu jej ruszac. Pieniadze leza w tym, co dzieje sie PO wejsciu na strone".

Nigdy nie naciagaj werdyktu do gory. Uczciwe "nieuzasadniony" jest wiecej warte niz sprzedany rebuild, ktory sie posypie na pierwszym pytaniu klienta.

## KROK 6 - WYNIK (twardy stop, W ROZMOWIE)

**Zaden plik. Zaden SendUserFile. Zaden artefakt.** Bartek ma to przeskanowac w 30 sekund, zapisac sobie i opowiedziec na wideo. Piszesz dokladnie to i nic wiecej:

```
**<Szkoła>** (<platforma, np. WordPress + Elementor>)

**Strona**
1. <co jest do poprawy, jedno krótkie zdanie> (<liczba albo dosłowny cytat>)
2. ...

**Analityka**
3. ...

**Google**
4. ...

**Reklamy Meta**
5. ...

**Propozycja:** <jedno zdanie: nowa strona albo co sprzedać zamiast niej>
```

Zasady:
- **5-7 punktow razem.** Nigdy wiecej. Mniej, jesli wiecej nie ma uczciwie.
- **4 kategorie w tej kolejnosci:** Strona (tresc, pierwszy ekran, zapis i formularz, telefon, mobile, szybkosc, opinie na stronie), Analityka (GA4, GTM, piksel Meta, zdarzenia konwersji, zgoda na cookies), Google (indeksowanie, tytul i opis w wynikach, H1, sitemap, smieci w indeksie, wizytowka i opinie w Mapach, pozycje na frazy), Reklamy Meta (Ad Library: aktywne reklamy, dokad prowadza, czy tam jest pomiar). Kategoria bez punktu wypada w calosci.
- **Punkt = jedna linia**, najlepiej do 15 slow, plus liczba albo doslowny cytat w nawiasie. Bez "Co powiedziec", bez "Pokaz", bez uzasadnien i bez zdan pobocznych.
- W kazdej kategorii **najmocniejszy punkt pierwszy**.
- **Jezyk jak z notatki na kartce:** "Brak GA4 na stronie, na ktora ida reklamy (0 zapytan do Google)", a nie opis procesu pomiaru.
- Punkty maja dotykac pola **"Co ich uwiera"**, gdy tylko sie da.
- Gdy strona jest dobra, punktem moze byc szansa, nie blad ("1. miejsce na 'szkola jezykowa Katowice', wiec problem nie lezy w Google").
- **Czego nie zmierzyles dwiema metodami, tego nie ma na liscie.** Zadnych sekcji "Czego nie mowic", "Nie zweryfikowalem", "Na czym to postawione", zadnego werdyktu w kilku zdaniach.
- Wyjatek: jedna linia **Uwaga:** na samym koncu, tylko gdy zmienia to, czy i do kogo nagrywac (np. zgloszenie od osoby spoza szkoly, strona pod inna domena niz w zgloszeniu). Poza tym nic.

Przyklad (Szkoła Językowa X, 19.09.2026):

```
**Szkoła Językowa X** (WordPress + WPBakery, wykonawca: agencja)

**Strona**
1. Na telefonie pierwszy ekran jest pusty, slider ma 0 px wysokości (544 px na komputerze)
2. Ikony Facebooka, Instagrama i Google prowadzą na "Strony nie znaleziono" (16 zepsutych linków)
3. Żaden numer telefonu na stronie głównej nie jest klikalny (0 z 6)

**Analityka**
4. Zapisy idą przez formularze LangLion bez GA4 i piksela (26 pól, 0 tagów)
5. Drugi kontener Google Tag Managera ładuje się z błędnego adresu (404)
6. Dwie wtyczki zgody naraz, przy części wejść dwa banery (2 z 4 wejść)

**Propozycja:** nowa strona z zapisami i pomiarem w jednym miejscu.
```

**CZEKAJ.** Bartek wywala albo dopisuje punkty i nagrywa sam, wlasnymi slowami. Bez jego "ok" nie powstaje draft.

Przy analizie wielu leadow naraz ten sam krotki format trafia do tresci rekordu leada w Notion (baza "Leady - wszystkie kampanie").

## KROK 7 - draft Gmail (gdy Bartek poda link do nagrania)

**Draft powstaje WYLACZNIE w Gmailu.** W rozmowie nie wklejasz tresci maila, nie streszczasz bulletow, nie cytujesz tematu ani puenty. Zostaje jedna linia: do kogo poszedl draft i ze nie zostal wyslany, plus ewentualnie to, czego swiadomie w mailu nie ma i dlaczego. Regula z 19.08.2026, patrz `memory/feedback_draft_w_gmailu_nie_w_rozmowie.md`.

- Wzorzec maila z `memory/project_lp_szkoly_jezykowe.md` (2 potwierdzone odpowiedzi leadow): nawiazanie do jego slow ze zgloszenia -> "wszedlem na [strona], poklikalem jak [typ odwiedzajacego], nagralem wideo" -> 5 pogrubionych bulletow -> "szczegoly w nagraniu" -> killer insight spinajacy jego odpowiedz z formularza -> CTA `cal.com/bartbakowski/system-sprzedazy-wiedzy` -> podpis "Bartosz Bakowski" + `bartbakowski.com/szkoly-jezykowe` pod podpisem.
- Adres: ZE ZGLOSZENIA. Forma: ZAWSZE Pan/Pani ("Dzień Dobry Panie/Pani X,", temat "Panie X, ..."), takze gdy Bartek w wideo mowi na Ty (decyzja 19.09.2026, `memory/feedback_mail_forma_pan_pani.md`). "Strona internetowa", nie "strona".
- `create_draft`. NIGDY nie wysylaj.
- Notion: zaktualizuj "Nastepny krok" po analizie i po drafcie. Date "Wideo wyslane" wpisz dopiero, gdy Bartek potwierdzi wysylke.

## Samoocena przed KROKIEM 6 i przed KROKIEM 7 (obowiazkowa)

Przejdz material zdanie po zdaniu i odpowiedz:
1. Czy **wynik miesci sie w formacie z KROKU 6**: 5-7 punktow po jednej linii, 4 kategorie, jedna linia propozycji i NIC poza tym (zero sekcji "czego nie mowic", "nie zweryfikowalem", "na czym to postawione", zero tabel)? Plik albo `SendUserFile` = blad.
2. Czy **kazdy punkt ma liczbe albo doslowny cytat** w nawiasie i da sie go pokazac na ekranie? Nie ma = wypada z listy.
3. Czy za kazdym punktem stoja **dwie niezalezne metody** pomiaru (u mnie, nie w wiadomosci)? Jedna metoda = punkt wypada albo wraca do weryfikacji.
4. Czy **wypelnilem u siebie kazdy wiersz pomiaru** z KROKU 2, a tam gdzie sie nie dalo, mam "NIE ZWERYFIKOWANO" z powodem? Puste pole = wracaj do pomiaru, nawet jesli do rozmowy to nie trafi.
5. Czy werdykt wynika z policzonych sygnalow, czy z checi sprzedazy? Jesli sygnaly PRZECIW przewazaja, napisz to wprost w jednym zdaniu.
6. Czy jakies zdanie zawiera "audyt", "doradztwo", "consulting"? Zamien (wideo o stronie, analiza, sesja).
7. Czy jakas liczba, nazwa, lokalizacja pochodzi spoza ich strony/zgloszenia? Usun.
8. Czy chwale cos, co lead sam zglosil jako problem? (Lekcja: "strona zrobiona porzadnie" u osoby, ktora napisala "wyglada przestarzale".) Przeformuluj.

## Granice z innymi skillami

- Estetyka, AI-look, art direction: `audyt-anti-ai` (mozna odpalic jako uzupelnienie i dolaczyc wnioski do sekcji WNIOSEK).
- Strony wlasnych projektow przed deployem: `sprawdz-kod` / `bezpieczenstwo`.
- Gleboki research firmy/rynku: `deep-research-pro`.

## Pliki

- `references/protokol-pomiaru.md` - snippety JS/bash (przetestowane 11.08.2026)
- `references/szablony.md` - format wyjscia do rozmowy + szablon maila
- **Wyniki NIE ida do pliku.** Wynik = wiadomosc w rozmowie wg KROKU 6. Jedyny trwaly zapis to Notion Leady (pole "Nastepny krok")
