# Szablony wyjscia

Dwa szablony: wynik analizy (idzie W ROZMOWIE, 5-7 krotkich punktow) i mail. Skryptu do nagrywania NIE robimy (decyzja Bartka 11.08.2026: nagrywa sam z punktow, wlasnymi slowami). Jezyk polski z ogonkami, zero em-dash, zero "audyt/doradztwo/consulting" w materialach dla leada.

---

## 1. Wynik analizy - ZAWSZE jako wiadomosc w rozmowie

**Zero plikow .md. Zero `SendUserFile`. Zero artefaktow.** Bartek przeskanuje to w 30 sekund, zapisze sobie i opowie na wideo (decyzje 19.08.2026 i 19.09.2026). Karta techniczna, weryfikacja i ryzyka zostaja u ciebie.

```markdown
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

### Zasady

- **5-7 punktow razem**, kazdy w jednej linii, liczba albo cytat w nawiasie.
- Kategorie w stalej kolejnosci: Strona, Analityka, Google, Reklamy Meta. Pusta kategoria wypada.
- W kategorii najmocniejszy punkt pierwszy.
- **Bez** "Co powiedziec", "Pokaz", "Na czym to postawione", "Czego nie mowic", "Nie zweryfikowalem". Czego nie potwierdzisz dwiema metodami, tego po prostu nie ma na liscie.
- Minimum polowa punktow dotyka pola **"Co ich uwiera"**.
- Gdy strona jest dobra, punktem moze byc szansa, nie blad. Nie chwal tego, co lead sam zglosil jako problem.
- Wyjatek: jedna linia **Uwaga:** na koncu, tylko gdy zmienia to, czy i do kogo nagrywac.

### Przyklad (Szkoła Językowa X, 19.09.2026)

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

---

## 2. Mail (draft Gmail, dopiero gdy Bartek poda link do nagrania)

**Ponizszy szablon wypelniasz od razu w `create_draft`, nie w rozmowie.** Zero kopii maila w czacie: ani calosci, ani bulletow, ani tematu.

Wzorzec potwierdzony odpowiedziami leadow (kilku leadów). Adres: ZE ZGLOSZENIA. Forma ZAWSZE Pan/Pani, takze gdy w wideo Bartek mowi na Ty (decyzja 19.09.2026). Akapit po powitaniu od WIELKIEJ litery.

**Temat:** `Pani/Panie <imie>, nagralem wideo o stronie (<hak: "dlaczego teksty nie przekonuja" / "N rzeczy">)`
(NIE uzywac slowa "audyt" w temacie ani tresci - starsze maile je mialy, regula VAT z 11.08.2026 to zmienia)

```
Dzien Dobry <Pani Imię / Panie Imię w wołaczu>,

Zostawil/a Pan/Pani zgloszenie, ze <doslownie z ankiety>.

Wszedlem na <domena>, przeszedlem przez strone jak <typ odwiedzajacego: rodzic szukajacy kursu / osoba szukajaca kursu> i zajrzalem tez w kod strony.

Nagralem wideo z tego, co znalazlem.

<LINK DO WIDEO>

Pare rzeczy wymieniam ponizej, co jest w wideo:

- <bullet 1: punkt zatwierdzony, z liczba/cytatem>
- <bullet 2>
- <bullet 3>
- <bullet 4>
- <bullet 5, opcjonalnie cos pozytywnego: "fundament jest w porzadku, problem lezy w..." o ile lead NIE zglosil tego jako problemu>

Szczegoly tych rzeczy sa w nagraniu.

Jedna rzecz, o ktorej warto wiedziec: <killer insight spinajacy JEGO odpowiedz z formularza z pomiarem, np. "napisal/a Pan/Pani, ze X. Zmierzylem: Y">

Jesli chcialby/chcialaby Pan/Pani przejsc przez to razem podczas darmowej 30 minutowej rozmowy, tutaj mozna umowic spotkanie:
https://cal.com/bartbakowski/system-sprzedazy-wiedzy

Pozdrawiam,
Bartosz Bakowski
bartbakowski.com/szkoly-jezykowe
```

Zasady twarde:
- "strona internetowa" / "strona www", nie samo "strona" (regula rules-tools)
- link LP `bartbakowski.com/szkoly-jezykowe` POD podpisem
- **Wideo z Looma (19.09.2026):** w miejscu `<LINK DO WIDEO>` klikalna miniatura plus link pod nia. Miniature bierzesz z `https://www.loom.com/v1/oembed?url=<share>` (`thumbnail_url`, np. `...-f22284365a51e699.gif`) i zamieniasz koncowke na `-full-play.gif` (wersja z przyciskiem play, 513x289). `<a href="<share>"><img src="...-full-play.gif" width="513" height="289" alt=""></a>`. BEZ tytulu i opisu wygenerowanego przez Looma (zadne "Tytul - Watch Video", zadne AI-streszczenie), iframe nie dziala w Gmailu. Bullety piszesz sam z transkrypcji nagrania (`source_url` transkrypcji jest w HTML strony share) i z zmierzonych liczb.
- `create_draft`, nigdy wysylka; wysylka tylko po jawnym "wyslij" Bartka
- lead nieufny wobec linkow (przypadek z poprzedniej kampanii)? Wideo jako zalacznik mp4 albo tekst zamiast wideo
- po drafcie: Notion "Nastepny krok" zaktualizowany; "Wideo wyslane" dopiero po potwierdzeniu wysylki
