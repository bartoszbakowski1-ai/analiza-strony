# Protokol pomiaru - gotowe snippety

Wszystkie snippety przetestowane 11.08.2026 na zywych stronach (trzech stronach szkół językowych). Wykonuj przez `mcp__plugin_playwright_playwright__browser_evaluate` (parametr `function`), chyba ze napisano inaczej. Kolejnosc krokow ma znaczenie.

## 1. Akceptacja cookies (ZAWSZE pierwszy krok po nawigacji)

```js
() => {
  const btns = [...document.querySelectorAll('button,a')].filter(b=>/akceptuj|zgadzam|zgoda|accept/i.test(b.innerText||''));
  if (btns.length) { btns[0].click(); return {clicked: btns[0].innerText.trim()}; }
  return {clicked: null, uwaga: 'BRAK BANERA - zanotuj, bo brak baneru przy dzialajacej analityce to osobny sygnal (RODO)'};
}
```

Po kliknieciu przeladuj strone (`browser_navigate` na ten sam URL), zeby analityka odpalila jak u realnego uzytkownika po zgodzie.

## 2. Scroll-and-settle (przed KAZDYM pomiarem tekstu)

Lazy-load i animowane liczniki nie istnieja przed scrollem. Liczniki lap po odczekaniu (inaczej zlapiesz klatke w trakcie animacji, np. 3998+ zamiast 4000+).

```js
async () => {
  window.scrollTo(0, document.body.scrollHeight);
  await new Promise(r=>setTimeout(r, 2500));
  window.scrollTo(0, document.body.scrollHeight*0.7);
  await new Promise(r=>setTimeout(r, 1500));
  window.scrollTo(0, 0);
  return {docHeight: document.documentElement.scrollHeight, textLen: document.body.innerText.length};
}
```

Zanotuj `textLen` przed i po scrollu. Roznica = tresc doladowana, ktorej nie bylo w pierwszym renderze.

**Liczniki animowane:** nawet po odczekaniu mozesz zlapac klatke w trakcie animacji (test 11.08: `3998+` gdy na ekranie konczy sie na `4000+`). Wartosc koncowa odczytaj ze SCREENSHOTA fullPage (krok 3), nie z innerText. Cytujac licznik, podawaj wartosc ze screenshota.

## 3. Screenshot fullPage + OBEJRZEC

```
browser_take_screenshot {filename: "analiza-<lead>-full.png", scale: "css", fullPage: true}
```

Potem `Read` na pliku (szukaj w `~/.playwright-mcp/` albo sciezce zwroconej przez narzedzie; jesli plik nie istnieje, zrob screenshot viewportu sekcjami). Obejrzenie jest obowiazkowe: wzrok lapie sekcje liczb, badge, bannery, ktore skrypty gubia.

**UWAGA, zrzut fullPage KLAMIE w dwie strony (lekcja z leada D, 18.08.2026).** Gubi tresc `iframe` (osadzone YouTube renderuja sie jako CZARNE PROSTOKATY) oraz tla CSS (`background-image`), przez co sekcja ze zdjeciem wyglada na pusta ramke. Zanim nazwiesz cokolwiek zepsutym na podstawie fullPage, **przewin do tej sekcji i zrob zrzut VIEWPORTU**. Przy lead D 9 wideo-opinii uczniow wygladalo na 9 czarnych dziur, a dzialaly poprawnie z miniaturami. To bylby falszywy zarzut wobec ich najmocniejszego dowodu spolecznego.

## 4. Tresc, liczby, daty, ceny (po scrollu)

```js
() => {
  const t = document.body.innerText;
  // liczbyZKontekstem: KAZDA liczba z otoczeniem 60 znakow. NIE filtruj keywordami
  // (regex keywordowy gubi "4000+ zadowolonych kursantow" i ogonki typu "czlonkow").
  const liczbyZKontekstem = [];
  const seen = new Set();
  for (const m of t.matchAll(/\d[\d\s]*\+?/g)) {
    const val = m[0].trim();
    if (val.length < 1 || seen.has(val + '@' + m.index)) continue;
    seen.add(val + '@' + m.index);
    liczbyZKontekstem.push({liczba: val, kontekst: t.slice(Math.max(0, m.index-40), m.index + val.length + 45).replace(/\s+/g,' ')});
  }
  return {
    textLen: t.length,
    liczbyZKontekstem: liczbyZKontekstem.slice(0, 40),
    ceny: [...new Set((t.match(/\d[\d\s.,]{0,8}\s*(zl|zł|PLN)/gi)||[]))],
    daty: [...new Set((t.match(/\d{1,2}\s+(stycznia|lutego|marca|kwietnia|maja|czerwca|lipca|sierpnia|wrzesnia|września|pazdziernika|października|listopada|grudnia)|20\d\d(\/\d\d)?/gi)||[]))],
    naglowki: [...document.querySelectorAll('h1,h2,h3')].map(h=>h.tagName+': '+h.innerText.trim().replace(/\s+/g,' ').slice(0,75)).slice(0,30)
  };
}
```

Czytajac `liczbyZKontekstem` sam ocen, ktore liczby to dowody spoleczne (lat na rynku, kursanci, opinie), a ktore to smieci (kod pocztowy, telefon, rok w stopce). Regexem tego nie rozstrzygniesz.

**Double check tresci: Firecrawl** (drugie, niezalezne zrodlo):

```bash
bash ~/.claude/skills/deep-research-pro/scripts/firecrawl_scrape.sh "https://domena.pl" /tmp/fc-out.md
```

Self-hosted `scrape.bartbakowski.com/v1/scrape`, klucz `fc-roast-internal-2026`. Quirki: NIE akceptuje `onlyMainContent`; 408 pod obciazeniem = fallback zostaje przy Playwright (zanotuj w analizie "Firecrawl niedostepny, jedno zrodlo"); pusty markdown na ciezkich SPA. Porownaj liczby i daty z wynikiem z Playwright. Rozbieznosc = badaj recznie, nie wybieraj wygodniejszej wersji.

## 5. Mapa strony (gdzie co lezy, w procentach wysokosci)

Pozwala powiedziec "dowody sa na 95% wysokosci strony" zamiast ogolnikow.

```js
() => {
  const H = document.documentElement.scrollHeight;
  const find = (txt) => {
    const e = [...document.querySelectorAll('h1,h2,h3,p,div')].find(x=>(x.innerText||'').trim().startsWith(txt));
    return e ? Math.round((e.getBoundingClientRect().top+scrollY)/H*100) : null;
  };
  // podmien frazy na naglowki z pkt 4
  return {wysokosc: H, mapa: {'FRAZA1': find('FRAZA1'), 'FRAZA2': find('FRAZA2')}};
}
```

## 6. Analityka - 3 metody NARAZ (po akceptacji cookies i przeladowaniu)

```js
() => {
  const r = {};
  const inline = [...document.querySelectorAll('script:not([src])')].map(s=>s.textContent).join('\n');
  r.m1_globals = {gtag:typeof window.gtag, dataLayer:Array.isArray(window.dataLayer)?window.dataLayer.length:'brak',
    fbq:typeof window.fbq, clarity:typeof window.clarity, hj:typeof window.hj, pys:typeof window.pysOptions};
  r.m2_requests = performance.getEntriesByType('resource').map(e=>e.name)
    .filter(n=>/google-analytics|googletagmanager|connect\.facebook|doubleclick|clarity\.ms|hotjar/i.test(n))
    .map(n=>n.split('?')[0].slice(0,80));
  r.m3_inlineIDs = {
    ga4: [...new Set(inline.match(/G-[A-Z0-9]{6,}/g)||[])],
    gtm: [...new Set(inline.match(/GTM-[A-Z0-9]{5,}/g)||[])],
    fbPixel: [...new Set((inline.match(/fbq\s*\(\s*['"]init['"]\s*,\s*['"](\d+)['"]/g)||[]))]
  };
  r.cookies = document.cookie.split(';').map(c=>c.trim().split('=')[0]).filter(Boolean);
  return r;
}
```

Interpretacja:
- 3x pusto po akceptacji cookies = brak analityki ruchu (ale sprawdz cookies `sbjs_*` = WooCommerce Order Attribution, wtedy NIE mow "nie mierza niczego", tylko "nie mierza ruchu na stronie").
- Wiecej niz jeden pixel ID w `m3` + wiecej niz jeden request `signals/config` = duplikat pixela (czesty przy agencji + wtyczce).
- Cookies `_ga`/`_fbp` USTAWIONE bez klikniecia zgody (sprawdz przy pierwszej wizycie, przed krokiem 1) = temat RODO. Formuluj ostroznie, jako "temat prawny".

## 7. Server-side / CAPI (NIGDY z ruchu przegladarki)

```js
() => (typeof pysOptions !== 'undefined')
  ? {facebook: pysOptions.facebook, uwaga: 'serverApiEnabled=true oznacza CAPI WLACZONE - nie podnosic tematu braku CAPI'}
  : {pys: 'brak wtyczki PixelYourSite - CAPI nieweryfikowalne z zewnatrz, napisz NIE ZWERYFIKOWANO'};
```

Kluczowe pola: `facebook.serverApiEnabled`, `facebook.advancedMatchingEnabled`, `facebook.pixelIds`. `eventID` (UUID) przy zdarzeniach = deduplikacja pixel/CAPI, czyli CAPI zywe. **Prefiks `ob3_plugin-set_...` przy `eventID` plus zaladowany `openbridge3.js` = Meta Openbridge, czyli CAPI DZIALA rowniez bez PixelYourSite** (lekcja z leada D, 18.08.2026). Wtedy temat braku CAPI wypada z materialu. PixelYourSite Free MA CAPI. Bez wtyczki z czytelnym configiem: napisz "nie zweryfikowalem", nie zgaduj.

## 8. SERP - realne Google, nie tag title

```
browser_navigate: https://www.google.com/search?q=site:domena.pl&hl=pl&gl=pl
```

```js
() => {
  const out = [];
  document.querySelectorAll('#search a h3, #rso a h3').forEach(h3 => {
    const a = h3.closest('a');
    const block = h3.closest('div.g, div[data-hveid], div.MjjYud') || h3.parentElement;
    let desc = '';
    if (block) {
      const cand = [...block.querySelectorAll('div,span')].map(e=>e.innerText||'').filter(s=>s.length>60 && !s.includes(h3.innerText));
      desc = cand.length ? cand[cand.length-1] : '';
    }
    out.push({tytul: h3.innerText.trim(), url: (a?a.href:'').split('&')[0].slice(0,90), opis: desc.replace(/\s+/g,' ').slice(0,260)});
  });
  return out.slice(0,8);
}
```

Cytuj DOSLOWNIE to, co zwrocilo Google. Tag `<title>` i meta description z kodu to tylko kontekst (Google je przepisuje). Blad wzorcowy: "tytul brzmi Home", gdy w SERP stalo "Home -> Nazwa Firmy".

## 9. Meta Ad Library

Dwa przebiegi: `active_status=active` oraz `active_status=all`.

```
browser_navigate: https://www.facebook.com/ads/library/?active_status=all&ad_type=all&country=PL&q=%22NAZWA%20FIRMY%22&search_type=keyword_exact_phrase
```

Uzywaj `keyword_exact_phrase` z nazwa w cudzyslowie (bez tego dostaniesz smieci z calego rynku). Parser:

```js
() => {
  const t = document.body.innerText;
  const blocks = t.split(/Library ID:/).slice(1);
  return {
    count: (t.match(/~?\s?[\d,\s]+ results?/i)||['brak'])[0],
    noResults: /No ads match your search/i.test(t),
    ads: blocks.map(b=>({
      id: b.split('\n').map(s=>s.trim()).filter(Boolean)[0],
      okres: (b.slice(0,300).match(/\d{1,2} \w{3} 20\d\d\s?-\s?\d{1,2} \w{3} 20\d\d/)||b.slice(0,300).match(/Started running on[^\n]{0,45}/i)||['?'])[0],
      reklamodawca: (b.match(/\n([^\n]{5,70})\nSponsored/)||[])[1]||'?',
      tekst: b.replace(/\s+/g,' ').slice(0,200)
    })).slice(0,12)
  };
}
```

Sprawdz, DOKAD prowadza reklamy (linki l.facebook.com), zanim powiesz "pala budzet na zle strony". Reklamy moga prowadzic na strony, ktore akurat sa poprawne (lekcja z leada A). Wlasne kampanie Bartka: Graph API z System User Token (`memory/knowledge/stack/meta-marketing-api.md`).

## 10. Mobile (390x844)

```
browser_resize {width: 390, height: 844}
```

```js
() => {
  const r = {};
  r.horizontalScroll = document.documentElement.scrollWidth > innerWidth;
  r.fold = [...document.querySelectorAll('h1,h2,p,a,button,img')]
    .filter(el=>{const b=el.getBoundingClientRect(); return b.top<844 && b.bottom>0 && b.width>0;})
    .map(el=>({tag:el.tagName, txt:(el.innerText||el.alt||'').trim().replace(/\s+/g,' ').slice(0,45), top:Math.round(el.getBoundingClientRect().top)}))
    .filter(x=>x.txt).slice(0,18);
  const tap=[...document.querySelectorAll('a,button')].filter(el=>{const b=el.getBoundingClientRect(); return b.width>0&&b.height>0&&b.top<3000;});
  r.tapTargets = {total:tap.length, ponizej40px: tap.filter(el=>el.getBoundingClientRect().height<40).length};
  const fonts={}; [...document.querySelectorAll('p,span,li,a')].slice(0,250).forEach(el=>{const f=getComputedStyle(el).fontSize; fonts[f]=(fonts[f]||0)+1;});
  r.fonts = Object.entries(fonts).sort((a,b)=>parseFloat(a[0])-parseFloat(b[0])).slice(0,5);
  return r;
}
```

Baner cookies na mobile NIE jest argumentem sprzedazowym (pokazuje sie raz, ma go kazda strona). Mozna zmierzyc, nie wolno na tym budowac punktu.

## 11. Podstrony (fetch w kontekscie strony)

```js
async () => {
  const paths = ['/cennik/','/kontakt/','/oferta/','/zapisy/']; // + 2-3 strony kursow z menu
  const out = {};
  for (const p of paths) {
    try {
      const q = await fetch(p); const h = await q.text();
      const txt = h.replace(/<script[\s\S]*?<\/script>/g,' ').replace(/<style[\s\S]*?<\/style>/g,' ').replace(/<[^>]+>/g,' ').replace(/&nbsp;/g,' ').replace(/\s+/g,' ');
      out[p] = {
        status: q.status,
        title: (h.match(/<title>(.*?)<\/title>/)||[])[1]||'',
        ceny: [...new Set((txt.match(/\d[\d\s]{0,7}(zl|zł|PLN)/gi)||[]))].slice(0,15),
        zapisy: [...new Set((txt.match(/[^.!?]{0,90}(zapisy|nabor|nabór|startuj|rusza)[^.!?]{0,90}[.!]/gi)||[]))].slice(0,3)
      };
    } catch(e){ out[p] = {err: String(e).slice(0,50)}; }
  }
  return out;
}
```

UWAGA interpretacyjna: brak ceny na podstronie to FAKT, "brak ceny to blad" to WNIOSEK. Najpierw poszukaj zdania w stylu "informacje o cenie podczas rozmowy" (lekcja z leada A).

## 12. Formularze

```js
() => [...document.querySelectorAll('form')].map(f=>({
  action: f.action, ajax: /ajax/i.test(f.className),
  pola: [...f.querySelectorAll('input,select,textarea')].filter(i=>i.type!=='hidden').map(i=>({
    name: i.name||'(brak)', type: i.type||i.tagName.toLowerCase(), required: i.required,
    label: (i.labels&&i.labels[0]?i.labels[0].innerText:i.placeholder||'').trim().replace(/\s+/g,' ').slice(0,60)
  }))
}))
```

NIGDY nie wysylaj formularza leada, nawet testowo.

## 12b. SITEMAP I ROBOTS - kontrola integralnosci domeny (ROB TO ZAWSZE, WCZESNIE)

Dodane 20.08.2026 po sprawie szkoły dla dzieci. Ten krok znalazl 1496 stron o kasynach opublikowanych pod domena szkoly jezykowej dla dzieci. Bez niego cala analiza skonczylaby sie na nieklikalnym telefonie i braku H1, czyli na drobiazgach, podczas gdy realny problem leadu byl gdzie indziej. **Rob to zaraz po renderze, przed pomiarami SEO.** Kosztuje trzy polecenia, a bywa cala analiza.

```bash
# 1. robots.txt: skad Google bierze liste podstron
curl -s https://DOMENA/robots.txt

# 2. sitemapa - sprawdz WSZYSTKIE warianty, nie tylko /sitemap.xml
for s in "/sitemap.xml" "/sitemap_index.xml" "/wp-sitemap.xml"; do
  echo "[$(curl -sL -o /tmp/s.xml -w '%{http_code}' "https://DOMENA$s")] $s -> $(grep -o '<loc>' /tmp/s.xml | wc -l) adresow"
done
# WAZNE: adres sitemapy bierz z robots.txt, bo bywa niestandardowy
# (u szkoły dla dzieci bylo "?feed=xmlsitemap19", a /sitemap.xml zwracalo tylko 2 adresy)

# 3. wyciagnij adresy i policz, ile jest nie-na-temat
curl -sL "ADRES_SITEMAPY_Z_ROBOTS" -o /tmp/sm.xml
grep -o '<loc>[^<]*</loc>' /tmp/sm.xml | sed 's/<[^>]*>//g' > /tmp/urls.txt
echo "lacznie: $(wc -l < /tmp/urls.txt)"
grep -icE "kasyn|casino|bonus|depozyt|spin|slot|blackjack|ruletka|poker|hazard|jackpot|automat|bingo|keno|baccarat|rtp|viagra|cialis|payday|loan|replica|escort" /tmp/urls.txt
```

Interpretacja i **obowiazkowy double check**:

- Liczba adresow w sitemapie **razaco wieksza niz liczba pozycji w menu** = czerwona flaga. Szkola jezykowa ma 10-30 podstron, nie 1500.
- Zanim cokolwiek powiesz, **potwierdz, ze strony zyja**: `curl -sL -o /tmp/x.html -w "%{http_code}" "<adres>"` plus wyciagnij `<title>`. Kod 200 i tytul konczacy sie nazwa firmy = tresc jest realnie opublikowana pod ich marka, a nie tylko wisi w pliku XML.
- **Ustal date wrzutu** z JSON-LD na stronie spamowej: `grep -oE '"datePublished":"[^"]*"'`. Sprawdz na 5-8 probkach. Identyczny znacznik co do sekundy = jeden masowy wrzut, i masz date, od kiedy to trwa.
- **Sprawdz, czy to jest podlinkowane z menu.** Jesli nie (typowo nie jest), to najmocniejszy punkt rozmowy: wlasciciel nie zobaczy tego, klikajac po wlasnej stronie. Widac to tylko w Google i w sitemapie. To tlumaczy, czemu wisi miesiacami.
- **Domknij realnym Google** (pkt 8): `site:domena.pl`. Licznik wynikow plus ile pozycji na pierwszej stronie to spam. To jest dowod, ktory lead zobaczy sam, u siebie, w 10 sekund.

**Jak to formulowac wobec leada:** zmierzony jest STAN (tyle a tyle stron, opublikowane wtedy, indeksowane przez Google). PRZYCZYNA nie jest zmierzona. Nie mow "zostaliscie zhakowani" jako pewnika, tylko pokaz stan i zapytaj, kto jeszcze ma dostep do panelu. Rozroznienie to samo co przy konfiguracji kampanii (lekcja Avalon).

## 12c. FAVICON - czy strona zostala skonczona (szybki, mocny sygnal)

Dodane 20.08.2026 po jednej ze stron. Bartek zauwazyl to golym okiem w karcie przegladarki, a pomiar potwierdzil twardo. Domyslny favicon z `create-next-app` (czarne kolo z bialym trojkatem, logo Vercel/Next.js) na produkcyjnej stronie firmy oznacza, ze ktos postawil szablon i nie domknal detali. Kosztuje 10 sekund, a jest widoczny dla kazdego odwiedzajacego.

```bash
curl -s https://DOMENA/favicon.ico -o /tmp/fav.ico
ls -la /tmp/fav.ico && md5 /tmp/fav.ico
# domyslny create-next-app: 25931 bajtow, MD5 c30c7d42707a47a3f4591831641e50dc
# porownaj z wlasnymi projektami:
find <katalog-projektu> -name favicon.ico -path "*app*" -exec md5 {} \;

# czy w ogole podmienili zestaw ikon (Next.js App Router):
for f in /icon.png /icon.svg /apple-icon.png; do
  printf "%-16s " "$f"; curl -sI "https://DOMENA$f" | head -1
done
```

Trzy metody potwierdzenia, uzyj co najmniej dwoch: **hash MD5** identyczny z domyslnym, **rozmiar 25931 bajtow**, oraz **404 na `/icon.png` i `/apple-icon.png`** (nikt nie podmienial zestawu). Domknij **wzrokiem**: `sips -s format png fav.ico --out fav.png` i `Read` na pliku. Trojkat w czarnym kole = domyslny.

Formuluj jako drobiazg, nie jako zarzut: "ikona strony w karcie przegladarki to domyslny obrazek z szablonu, ten sam, ktory dostaje kazdy nowo zalozony projekt". **Nie mow "Vercel" ani nazwy hostingu**, dopoki nie zmierzysz hostingu osobno (przy dinozaurywkosmosie favicon byl domyslny z Next.js, ale hosting to DigitalOcean App Platform za Cloudflare, nie Vercel).

## 13. KARTA TECHNICZNA jednym strzalem (uruchom PO akceptacji cookies i przeladowaniu)

Zbiera wiekszosc pol karty z KROKU 2 SKILL.md. Reszte (SERP, Ad Library, podstrony, mobile) dobierasz osobno.

```js
async () => {
  const inline = [...document.querySelectorAll('script:not([src])')].map(s=>s.textContent).join('\n');
  const res = performance.getEntriesByType('resource').map(e=>e.name);
  const H = document.documentElement.scrollHeight;
  const poz = el => Math.round((el.getBoundingClientRect().top + window.scrollY)/H*100);
  const q = s => document.querySelector(s);
  const title = document.title || '';
  const desc = (q('meta[name="description"]')||{}).content || null;
  const h1 = [...document.querySelectorAll('h1')].map(h=>h.innerText.trim().replace(/\s+/g,' ').slice(0,80));
  const ga4 = [...new Set([...(inline.match(/G-[A-Z0-9]{6,}/g)||[]), ...res.map(n=>{try{const u=new URL(n);return u.searchParams.get('id')||u.searchParams.get('tid');}catch(e){return null;}}).filter(x=>x&&/^G-/.test(x))])];
  const gtm = [...new Set([...(inline.match(/GTM-[A-Z0-9]{5,}/g)||[]), ...res.map(n=>{try{return new URL(n).searchParams.get('id');}catch(e){return null;}}).filter(x=>x&&/^GTM-/.test(x))])];
  const pixele = [...new Set([...(inline.match(/fbq\s*\(\s*['"]init['"]\s*,\s*['"](\d+)['"]/g)||[]).map(s=>(s.match(/(\d{8,})/)||[])[1]), ...res.filter(n=>/signals\/config\//.test(n)).map(n=>(n.match(/signals\/config\/(\d+)/)||[])[1])].filter(Boolean))];
  const formy = [...document.querySelectorAll('form')];
  return {
    A_fundament: {
      generator: (q('meta[name="generator"]')||{}).content || 'brak',
      slady: ['webwave','landingi','wix','elementor','wp-content','squarespace','shopify','_next'].filter(t=>document.documentElement.outerHTML.toLowerCase().includes(t)),
      wagaHTML: document.documentElement.outerHTML.length, wysokoscStrony: H
    },
    B_pomiar: {
      GA4: ga4.length?ga4:'BRAK', GTM: gtm.length?gtm:'BRAK',
      MetaPixel: pixele.length?pixele:'BRAK', ILE_PIXELI: pixele.length,
      CAPI: (typeof pysOptions!=='undefined'&&pysOptions.facebook) ? (pysOptions.facebook.serverApiEnabled?'WLACZONE':'WYLACZONE') : 'NIE ZWERYFIKOWANO (brak wtyczki z czytelnym configiem)',
      inne: {clarity:typeof window.clarity, hotjar:typeof window.hj},
      globals: {gtag:typeof window.gtag, fbq:typeof window.fbq, dataLayer:Array.isArray(window.dataLayer)?window.dataLayer.length:'brak'},
      UWAGA: 'gtag undefined NIE dowodzi braku GA4 gdy leci przez GTM. Patrz requesty.'
    },
    C_zgoda: {
      cookies: document.cookie.split(';').map(c=>c.trim().split('=')[0]).filter(Boolean),
      banerWidoczny: [...document.querySelectorAll('button,a')].filter(b=>/akceptuj|zgadzam|zgoda|accept|rozumiem/i.test(b.innerText||'')).map(b=>b.innerText.trim().slice(0,30)),
      wtyczkaZgody: [...document.querySelectorAll('script[src],link[href]')].map(e=>e.src||e.href).filter(u=>/cookie|consent|gdpr|cmplz|cookiebot|cookieyes|iubenda/i.test(u)).map(u=>u.split('/').pop().slice(0,40)).slice(0,3),
      consentMode: (typeof pysOptions!=='undefined'&&pysOptions.gdpr)?{analytics:pysOptions.gdpr.analytics_storage,ads:pysOptions.gdpr.ad_storage}:'nd'
    },
    D_seo: {
      metaTitle: title, dlugoscTitle: title.length, ocenaTitle: (title.length<30?'ZA KROTKI':title.length>62?'ZA DLUGI':'w normie 50-60'),
      metaDescription: desc || 'BRAK META DESCRIPTION', dlugoscDesc: desc?desc.length:0,
      ocenaDesc: !desc?'BRAK':(desc.length<110?'ZA KROTKI':desc.length>165?'ZA DLUGI':'w normie 140-160'),
      popsuteZnakiWDesc: desc?/\?\?\?|�/.test(desc):false,
      liczbaH1: h1.length, trescH1: h1,
      liczbaH2: document.querySelectorAll('h2').length, pusteH2: [...document.querySelectorAll('h2')].filter(h=>!h.innerText.trim()).length,
      canonical: (q('link[rel="canonical"]')||{}).href || 'BRAK',
      lang: document.documentElement.lang || 'BRAK'
    },
    E_konwersja: {
      formularzeNaStronieGlownej: formy.length,
      polaFormularzy: formy.map(f=>[...f.querySelectorAll('input,select,textarea')].filter(i=>i.type!=='hidden').length),
      telefonKlikalny: [...document.querySelectorAll('a[href^="tel:"]')].map(a=>({nr:a.innerText.trim().slice(0,20), poz:poz(a)})),
      mailtoKlikalny: [...document.querySelectorAll('a[href^="mailto:"]')].map(a=>({poz:poz(a)})),
      ctaZPozycja: [...new Map([...document.querySelectorAll('a,button')].filter(e=>{const t=(e.innerText||'').trim();return t.length>3&&t.length<40;}).map(e=>[e.innerText.trim().replace(/\s+/g,' '),{txt:e.innerText.trim().replace(/\s+/g,' '),href:(e.href||'').slice(-40),poz:poz(e)}])).values()].slice(0,20)
    }
  };
}
```

Do karty dobierz osobno: `/sitemap.xml` i `/robots.txt` (curl status), realny SERP (pkt 8), Ad Library (pkt 9), mobile (pkt 10), podstrony (pkt 11), przekierowania (`curl -sL -o /dev/null -w "%{num_redirects} %{url_effective}"`).

## Kolejnosc wykonania (checklist)

1. KROK 0 ze SKILL.md (Notion + Gmail + Instantly)
2. navigate -> pierwsza wizyta: sprawdz cookies USTAWIONE bez zgody (pkt 6, tylko `r.cookies`)
3. akceptuj cookies (1) -> przeladuj
4. scroll-and-settle (2) -> screenshot fullPage + Read (3)
4b. **SITEMAP I ROBOTS (12b)** plus **FAVICON (12c)** - kontrola integralnosci domeny. Robisz to WCZESNIE, bo bywa cala analiza
5. **KARTA TECHNICZNA jednym strzalem (13)** -> to wypelnia wiekszosc KROKU 2 ze SKILL.md
6. tresc (4) + Firecrawl double check + mapa strony (5)
7. analityka (6) + pysOptions (7) -> uzupelnij karte
8. SERP (8) -> uzupelnij sekcje D karty
9. Ad Library (9) -> sprawdz, czy strona docelowa reklam ma pomiar
10. mobile (10) + screenshot mobile
11. podstrony (11) + formularze (12) -> policz podstrony bez meta description i bez H1
12. sitemap, robots, przekierowania
13. WERDYKT rebuild (KROK 5 ze SKILL.md), klasyfikacja i raport wg `szablony.md`
