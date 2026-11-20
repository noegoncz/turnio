# Turnio – turnajový systém a živé skóre pro nohejbal

Projekt pro okresní nohejbalový turnaj (cca 10 týmů, jeden den, restaurace s nespolehlivým Wi-Fi). Komunikuj česky. Pracuj iterativně po checkpointech, po každém se zastav a nech mě ověřit. Dodávej kompletní kód bez zástupných míst.

## Cíl
Aplikace v telefonu, kterou organizátor ovládá jednou rukou: zadá týmy, rozlosuje skupiny, vygeneruje rozpis zápasů, zapisuje výsledky, počítá tabulky a KO pavouka. Živý stav se promítá přes Chromecast na projektor. Aplikace se jmenuje Turnio. Zatím je jen pro vlastní turnaj, ale základ má vydržet případné pozdější nabídnutí dalším klubům.

## Architektura
- Ovládací stránka (PWA) na GitHub Pages (noegoncz.github.io/turnio). Drží celý stav turnaje v localStorage v telefonu. Zapisuje po každé akci, drží posledních cca 20 snímků pro Zpět a má tlačítko Export (JSON do telefonu nebo sdílení). Při startu si vyžádá trvalé úložiště (navigator.storage.persist()).
- Custom Web Receiver (druhá statická stránka na GitHub Pages) běží na Chromecastu a jen vykresluje stav, který mu telefon pošle přes Cast SDK (Web Sender). Receiver nic neukládá. Po připojení telefon vždy znovu pošle celý stav.
- Žádná databáze ani server. Online zálohu řeší jen ruční export. Případný zápis do soukromého Gistu až jako pozdější volitelný krok.
- Cast SDK vyžaduje jednorázovou registraci receiveru v Google Cast Developer Console (asi 5 USD) a registraci sériového čísla Chromecastu jako testovacího zařízení. Chromecast potřebuje internet a stejnou síť jako telefon (hotspot z druhého telefonu stačí).
- Ověřit prototypem na začátku, že tlačítko Cast ve vlastní stránce funguje v Chromu na Androidu.
- Záložní plán bez Cast SDK: zobrazovací stránka na celou obrazovku a zrcadlení telefonu (Smart View).

## Promítání
Plánuje se nový projektor se dvěma vstupy a funkcí obraz v obraze (GoPro na jednom HDMI, Chromecast s tabulkou na druhém). Stávající Epson EB-X25 PiP nemá (jen jeden HDMI), proto se mění. Před koupí ověřit v manuálu konkrétního modelu, že PiP umí HDMI + HDMI a že jde nastavit velikost a roh podokna.
Důsledek pro vzhled: receiver se v PiP zobrazí jen jako malé podokno, takže musí být čitelný při velikosti zhruba čtvrtiny obrazu. Velká písma, vysoký kontrast, málo informací najednou, střídání pohledů (skóre zápasu, tabulky, pavouk).

## Rozhraní v telefonu
Celá plocha, velká tlačítka, jednoduchá grafika, žádné složité menu.
1. Týmy: zadání názvů (například 10 týmů).
2. Losování se třemi režimy, mezi kterými jde kdykoli přepínat a výsledek ručně upravit:
   - A) Náhodně: tlačítko, všechny týmy náhodně do skupin.
   - B) S nasazením: označím silné týmy (například 2 nebo 4), ty se rozlosují zvlášť rovnoměrně do skupin, zbytek se rozlosuje náhodně.
   - C) Ručně: sám přiřadím týmy do skupin.
3. Rozpis zápasů (viz níže), seznam k vytištění jako štítky.
4. Zadávání výsledků: u zápasu označím, kdo vyhrál 2:0 nebo 1:1 nebo 0:2, a zapíšu body v každém setu.
5. Tabulky skupin (matice týmů, diagonála přeškrtnutá) a pořadí.
6. KO pavouk.

## Pravidla
Oficiální pravidla ČNS (nohejbal.org, platnost od 8. 3. 2015) potvrzují: set se hraje do 10 bodů bez nutnosti dvoubodového rozdílu (pokud Rozpis soutěže nestanoví jinak), zápas na dva vítězné sety, v rozhodujícím setu se strany mění v polovině, tedy při 5. bodu. Skupiny, tabulky a pořadí při shodě pravidla ČNS neřeší, to jsou naše vlastní pravidla níže.

Skupiny: do 7 týmů jedna skupina, od 8 týmů dvě. Velikosti se liší nejvýš o jeden tým (10 týmů = 5 + 5). Každý hraje s každým.
Zápas ve skupině: vždy 2 sety do 10. 2:0 = 3 body, 1:1 = 1 bod každému, 0:2 = 0 bodů. Zadávají se body v každém setu.
Pořadí ve skupině:
1. Body.
2. Při shodě vzájemný zápas (vyhrál ten, kdo vyhrál mezi nimi).
3. Při remíze ve vzájemném zápase celkem získané body za celou skupinu.
4. Při další shodě to označit a rozhodne organizátor ručně.
Aplikace vždy zobrazí důvod, proč je pořadí takové (například „rozhodl vzájemný zápas“), protože o tom bývají spory.

KO fáze (v hovoru „Kočka“): postupují max. 4 týmy z každé skupiny, tedy pavouk o 8 týmech. První kolo kříží skupiny: A1 – B4, B2 – A3, B1 – A4, A2 – B3 (ověřit podle pavouka, který pošle organizátor). Zápas se hraje na dva vítězné sety (2:0 nebo 2:1), sety do 10, v rozhodujícím třetím setu se strany mění při 5. bodu. Další kola pavouka a případný zápas o 3. místo se dodefinují později.
Pozor: „Kočka“ je KO fáze (pavouk), ne útěšná větev. Dřívější vlákno ji chybně považovalo za útěchu.

## Rozpis zápasů ve skupině
Cíl: nikdo nehraje dva zápasy hned po sobě a nikdo nečeká zbytečně dlouho (rovnoměrné rozložení zápasů po celý turnaj).
- Základ: Bergerova tabulka (kruhová metoda) pro kola. Lichý počet týmů = volno navíc.
- Poté seřadit zápasy prohledáváním (backtracking nebo vylepšování) podle hodnoticí funkce: velká penalizace za dva zápasy po sobě, penalizace za dlouhé čekání a nerovnoměrné mezery, výsledek deterministický se seedem.
- Úplně vyhnout se hře dvakrát po sobě nejde vždy (například 4 týmy na jednom hřišti je to matematicky nemožné), proto minimalizovat a zobrazit.
- Hřiště: nyní jedno hřiště, dvě skupiny se na něm střídají zápas po zápasu. Aplikace má být připravená i na variantu se dvěma hřišti (venkovní turnaj), kdy obě skupiny hrají současně. Počet hřišť je tedy nastavení turnaje.
- Výstup: seznam zápasů v pořadí s číslem zápasu, názvy týmů a hřištěm, pro vytištění. Štítek na zápas obsahuje číslo zápasu, týmy, políčka pro body obou setů a označení vítěze.

## Stávající kód
V dřívějším vlákně „Nohejbalový turnaj a živé skóre“ (13. 11. 2026) vzniklo jádro src/tournament.js (čistý JavaScript bez závislostí, losování, nasazení, zápasy, tabulky, pavouk) s testy test/tournament.test.js (21 testů) a PLAN.md. Soubory se stáhnou z toho vlákna do tohoto repozitáře a jádro se použije beze změny, doplní se podle sekcí výše (režimy losování, rozpis zápasů, KO podle ČNS, pořadí).

## Checkpointy
1. CP0: repozitář, GitHub Pages, tento soubor, jádro logiky a npm test.
2. CP1: prototyp Cast: dva týmy, skóre, tlačítko Cast, receiver na Chromecastu (včetně registrace).
3. CP2: týmy a losování (A, B, C) a rozpis zápasů s tiskovým výpisem.
4. CP3: zadávání výsledků, tabulky skupin a pořadí s důvodem.
5. CP4: perzistence (localStorage, snímky, export, obnova po restartu a připojení).
6. CP5: KO pavouk a zobrazení pro PiP.
7. CP6: zkušební turnaj s fiktivními týmy, test na hardwaru (projektor, Chromecast, GoPro).

## Otevřená rozhodnutí
- Přesné rozlosování dalších kol KO pavouka a zápas o 3. místo.
- Obrázek pavouka od organizátora k ověření prvního kola.
- Počet týmů nad 14 a pravidla nasazení (kolik potů).
- Model nového projektoru (PiP, dva HDMI).
