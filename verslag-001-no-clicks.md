# Onderzoeksplan: Technische Privacy-Analyse Outbrain Tracking Mechanismen

## Achtergrond en Context

### Belangrijk

Dag Ywein, Bart, de huidig geschreven versie is specifiek voor jullie geschreven voor context.
Niet als verslag om voor te leggen. Het lijkt me gemakklijker om nu nog even algemeen te focussen.

### Aanleiding

Op verzoek ben ik een onderzoek gestart naar Outbrain – een digitaal advertentienetwerk dat via aanbevelingswidgets
zichtbaar is op onder meer Belgische nieuwswebsites zoals f1journaal.be. Outbrain specialiseert zich in het aanbieden
van gesponsorde content en native advertising, en verzamelt hiervoor gebruikersgegevens om gepersonaliseerde
aanbevelingen en advertenties te tonen.

De centrale vragen in dit onderzoek zijn: welke informatie verzamelt Outbrain, op welk moment gebeurt dit, en wat
verandert er wanneer een gebruiker toestemming geeft of weigert? Daarbij kijk ik specifiek naar de datapunten die worden
doorgestuurd, de betekenis van deze datapunten en de mate van zekerheid waarmee ik deze interpretaties kan maken.

Mijn aanpak is gebaseerd op reverse engineering. Daarbij analyseer ik het gedrag van de code “van buitenaf” (soms
aangeduid als confiscated code), zonder toegang tot de volledige broncode van Outbrain. Dit is een gangbare methode in
de industrie: absolute zekerheid is hierdoor niet haalbaar, maar door te observeren wat er verzonden wordt en door
datapunten actief te variëren (bijvoorbeeld de schermresolutie) kan ik wel valideren welke informatie waarschijnlijk
wordt gebruikt en hoe.

## 1. Request 1

**T+1348 ms** - Uitgebreide logging naar meerdere partijen:

- CHEQ anti-fraud systeem krijgt session/pageview IDs
- Outbrain logging met widget-positie en zichtbaarheid
- Media/advertentie content wordt geladen

> **In gewone taal:** Binnen 2 seconden praat de Outbrain-code al met minstens 9 verschillende servers en stuurt
> informatie over uw bezoek, scherm en browser - terwijl de code zelf aangeeft dat u nog geen toestemming heeft gegeven.

### Welke informatie wordt verzonden?

#### 1. Unieke identificatoren (wie bent u?)

| Wat         | Voorbeeld                              | Waarom problematisch?                                                                     |
|-------------|----------------------------------------|-------------------------------------------------------------------------------------------|
| PageView ID | `acb8a3a3cd7121ff0af7e04a2646c0b6`     | Uniek nummer voor dit specifieke bezoek, is deze nodig als deze niet wordt opgeslagen?    |
| Session ID  | `345a84d5-6e18-cd15-ba61-a76e59148044` | Koppelt meerdere acties aan elkaar. Waarom genereren we dit als we dit niet gaan opslaan? |

> **In gewone taal:** Dit zijn digitale "naamkaartjes" die aan uw bezoek worden gehangen. Hiermee kan Outbrain uw acties
> volgen, zelfs zonder cookies.

#### 2. Scherm en positie-informatie

- Exacte widget-positie: `widgetX=674, widgetY=3825`
- Widget-afmetingen: `widgetWidth=223, widgetHeight=43`
- Schermgrootte: `scrW=1512, scrH=982`
- Venstergrootte: `winW=957, winH=857`
- Zichtbaarheid: `pVis=0` (niet zichtbaar) of `pVis=1` (zichtbaar)

> **In gewone taal:** Outbrain weet precies waar hun advertentieblok staat, hoe groot het is, en of u het kunt zien -
> nog voordat u toestemming geeft.

#### 3. Browser vingerafdruk

- Platform: `chp=macOS, chpv=15.6.0`
- Architectuur: `cha=arm, chb=64`
- Browser versies: Gedetailleerde Chrome/Chromium info
- Netwerksnelheid: `cet=4g, bandwidth=10`
- Laadtijd: `ttfb=795` milliseconden

> **In gewone taal:** Dit is als een digitale vingerafdruk van uw computer. Zelfs zonder cookies kan Outbrain u mogelijk
> herkennen door deze unieke combinatie.

### Het Cross-Site Tracking Probleem

**Concreet voorbeeld:**
Stel, u bezoekt 's ochtends nieuwssite A (De Standaard) en 's middags sportsite B (Sporza). Beide gebruiken Outbrain.
Wat ziet Outbrain?

1. **09:00** - IP 192.168.1.1 + macOS + Chrome + scherm 1512x982 bezoekt De Standaard
2. **14:00** - IP 192.168.1.1 + macOS + Chrome + scherm 1512x982 bezoekt Sporza

**Conclusie van Outbrain:** "Dezelfde persoon/computer bezocht beide sites"

Dit is een simpel voorbeeld. Maar dit zijn de gegevens die outbrain momenteel verzamelt:

### Wat Outbrain Verzamelt

| Categorie    | Datapunten   | Voorbeeld uit HAR      |
|--------------|--------------|------------------------|
| **Hardware** | CPU cores    | `hwc=16`               |
|              | Geheugen     | `devMem=8` GB          |
|              | Pixel ratio  | `dpr=2` (Retina)       |
|              | Scherm       | `1512x982` pixels      |
| **Software** | OS versie    | `macOS 15.6.0`         |
|              | Browser      | `Chrome 140.0.7339.80` |
|              | Architectuur | `arm64`                |
|              | Taal         | `en-GB`                |
| **Netwerk**  | Type         | `4g`                   |
|              | Bandbreedte  | `bandwidth=10`         |
|              | Laadtijd     | `ttfb=795ms`           |
| **Context**  | Tijdzone     | Via timestamps         |
|              | Viewport     | `957x857` pixels       |
|              | Oriëntatie   | `landscape`            |

Deze tracking sets is voldoende om devices te onderscheiden op korte termijn, maar minder stabiel over lange
termijn (bv.
viewport en bandwidth veranderen).

**Voorbeeld**

- Hoeveel mensen in België draaien macOS 15.6.0 op een arm64-chip?
  → Dat beperkt je meteen tot een subset van Apple Silicon gebruikers, niet Intel.
- Hoeveel daarvan gebruiken Chrome 140.0.7339.80 in plaats van Safari (de standaard op Mac)?
  → De meeste Mac-gebruikers blijven bij Safari; Chrome op Mac is al minderheid.
- Hoeveel van die Chrome-op-Mac gebruikers hebben hun taalinstelling op en-GB in plaats van NL of FR?
  → De meerderheid van Belgische Macs staat ingesteld op Nederlands of Frans. en-GB is een niche.
- Hoeveel van die mensen hebben een 14" MacBook Pro schermresolutie van 3024x1964, mét devicePixelRatio 2?
  → Dat komt neer op een heel specifieke laptop (MacBook Pro 14").
- Hoeveel zitten tegelijk in tijdzone Europe/Brussels, met een viewport van 1440x980 (dus niet fullscreen maar
  verkleind)?
  → Zelfs binnen dezelfde hardwaregroep onderscheiden viewport en vensterafmetingen je verder.
- Hoeveel van die gebruikers zitten toevallig op een Proximus-IP in Vlaanderen, met wifi rond 50 Mbps en een TTFB van
  ±680
  ms?
  → Je netwerk legt er nog een extra laag unieke context bovenop.

Stel: je hoeft niet altijd exact één persoon te herkennen.  
Je kunt mensen ook **in buckets plaatsen** op basis van gedeelde kenmerken.

- Eén bucket bevat bijvoorbeeld **5 tot 10 mensen** met vergelijkbare fingerprint-data  
  (zelfde OS, browser, taal, schermtype, enz.).
- In eerste instantie gebruik je iets simpels als **IP-adres**: alle gebruikers achter dat IP gaan in dezelfde bucket.
- Daarna kun je de buckets **verfijnen of samenvoegen** op basis van hardware/software-patronen.

Het mooie is: ook al weet je niet precies *wie* die ene gebruiker is, je weet wel:

- tot welke **groep** ze horen,
- op welke **sites** die groep komt,
- en dus welke **advertenties** waarschijnlijk relevant zijn.

Zo ontstaat een soort **schaduwprofiel**:  
geen direct profiel van *“Jan Peeters, 34 jaar uit Gent,”*  
maar een **digitale afdruk van een groep** waarin Jan valt. Deze groep kan 1 of meerdere mensen bevatten.

Dat profiel kan over tijd steeds meer context verzamelen (bezochte sites, interesses, gewoontes). En dan is de stap naar
advertenties eenvoudig: je richt niet per se op **één persoon**,  
maar op de **groep**.  
Als er maar 10 mensen in een bucket zitten, en je weet waar die bucket komt,  
kan je alsnog heel gericht adverteren — bijna alsof het een individu was.

#### Ook je IP adres wordt verstuurd, maar hoe werkt dat dan?

Belgische ISPs (Telenet, Proximus, Orange) delen IPv4-adressen uit via **DHCP** (Dynamic Host Configuration Protocol):

- **Consumenten**: Krijgen dynamische IP's die periodiek veranderen (24-72 uur)
- **Zakelijk**: Vaak statische IP's die nooit veranderen
- **Mobiel**: Gedeelde IP's via NAT (Network Address Translation)

### Betrokken partijen (wie krijgt deze data?)

1. **Outbrain zelf** - 9 verschillende subdomeinen
2. **Browsi** - yield optimization/viewability metingen
3. **CHEQ** - bot detectie en fraud preventie

> **In gewone taal:** Uw data wordt niet alleen met Outbrain gedeeld, maar ook met hun partners. Elk van deze partijen
> kan theoretisch eigen profielen opbouwen.

## Is dit noodzakelijk?

### Wat Outbrain waarschijnlijk zal zeggen:

- "We moeten fraude detecteren"
- "Adverteerders willen weten of ads zichtbaar zijn"
- "We optimaliseren voor snelheid"
- "We voorkomen dat u dezelfde ad te vaak ziet"

### Waarom deze argumenten waarschijnlijk niet opgaan vóór consent:

| Argument          | Realiteit                                        | Alternatief                                              |
|-------------------|--------------------------------------------------|----------------------------------------------------------|
| Fraudedetectie    | Commercieel belang, geen noodzaak voor gebruiker |                                                          |
| Viewability       | Relevant voor adverteerder, niet voor gebruiker  | Meten NA consent                                         |
| Optimalisatie     | Kan client-side bepaald worden                   | Lokale detectie en omwisseling zonder data uitwisseling. |
| Frequency capping | Vereist tracking over tijd                       | Alleen met opt-in                                        |

## Wat betekent dit?

### De misleidende cookiebanner

De cookiebanner vraagt: "Accepteert u cookies?" maar:

- Outbrain heeft al uw pageview-ID aangemaakt
- Uw scherminfo is al verstuurd
- Uw browser-vingerafdruk is al gedeeld
- Meerdere bedrijven hebben al data ontvangen
- Fingerpinting is niet bewijsbaar zonder hun code

## Interessante Notities uit HAR-Analyse

### Kritieke Bevindingen

- **T+385ms**: Data verzameling actief voordat cookiebanner zelfs zichtbaar is
- **"no_consent" paradox**: Ondanks `ccnsnt=false` in alle requests wordt toch alle data verstuurd
- **9 verschillende subdomeinen** contacteren elkaar binnen 2 seconden
- **3 externe bedrijven** (Outbrain, CHEQ, Browsi) ontvangen data zonder toestemming
- **Token-based tracking**: Unieke tokens maken cookie-loze tracking mogelijk (deze veranderd bij een nieuwe sessie,
  engiste tracking cross-sessies is fingerprinting)

### Verborgen Third-Party Integraties

- **CHEQ**: Anti-fraud dienst ontvangt session/pageview IDs zonder consent
- **Browsi**: Yield optimization krijgt complete browser/scherm configuratie
- Beide partijen kunnen eigen gebruikersprofielen opbouwen

## Beperkingen van dit onderzoek

- Analyse van één website (f1journaal.be)
- Één browser/OS combinatie getest
- Geen toegang tot server-side verwerking
- Geen analyse van wat er NA consent gebeurt (volgt)

## Vragen en notities

- Zijn de gegevens die ze verzamelen genoeg als bewijs?
- Een eerste analyse toont wel aan dat outbrain geen herhalende ID's gebruikt. Als ze dus tracking doen dan doen ze
  dit via shadow profiles.
- Moeten we bewijzen dat ze die shaduw profielen opmaken?
- Wat gebeurd er als ik de cookies accepteer op A maar weiger op B? Ik ga uit van de cookies van DPG media, niet van
  outbrain? 