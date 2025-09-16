# Onderzoeksplan: Technische Privacy-Analyse Outbrain Tracking Mechanismen

## Achtergrond en Context

### Introductie

-- TODO: write personal introduction

### Aanleiding

Op verzoek ben ik een onderzoek gestart naar Outbrain – een digitaal advertentienetwerk dat via aanbevelingswidgets zichtbaar is op onder meer Belgische nieuwswebsites zoals f1journaal.be. Outbrain specialiseert zich in het aanbieden van gesponsorde content en native advertising, en verzamelt hiervoor gebruikersgegevens om gepersonaliseerde aanbevelingen en advertenties te tonen.

De centrale vragen in dit onderzoek zijn: welke informatie verzamelt Outbrain, op welk moment gebeurt dit, en wat verandert er wanneer een gebruiker toestemming geeft of weigert? Daarbij kijk ik specifiek naar de datapunten die worden doorgestuurd, de betekenis van deze datapunten en de mate van zekerheid waarmee ik deze interpretaties kan maken.

Mijn aanpak is gebaseerd op reverse engineering. Daarbij analyseer ik het gedrag van de code “van buitenaf” (soms aangeduid als confiscated code), zonder toegang tot de volledige broncode van Outbrain. Dit is een gangbare methode in de industrie: absolute zekerheid is hierdoor niet haalbaar, maar door te observeren wat er verzonden wordt en door datapunten actief te variëren (bijvoorbeeld de schermresolutie) kan ik wel valideren welke informatie waarschijnlijk wordt gebruikt en hoe.

Waar mogelijk licht ik ook toe welke veelgebruikte afkortingen waarschijnlijk op slaan, zodat de betekenis van de parameters duidelijker wordt.### Relevante Kaders
- **EDPB Richtsnoeren 2/2023**: Verduidelijkt dat artikel 5(3) van de ePrivacy-richtlijn van toepassing is wanneer toegang wordt verkregen tot informatie op eindapparatuur                                                                                               j
### Onderzoeksdoel
Het doel van dit onderzoek is vast te stellen welke data Outbrain verzamelt, hoe deze data wordt verwerkt, en in hoeverre dit in lijn lijkt te zijn met de Europese privacywetgeving – in het bijzonder artikel 5(3) van de ePrivacy-richtlijn. De uiteindelijke juridische beoordeling laat ik echter over aan gespecialiseerde juristen.## 1. Probleemstelling


## 2. Methodologie

### 2.1 Technische Vastlegging (Best Practices cf. Desmet)

**Waarom**  
Zoals Desmet aangeeft (p. 21–22) vormen **HAR-bestanden** (*HTTP Archive*) de standaard voor reproduceerbaar technisch onderzoek. Ze geven een volledig overzicht van al het netwerkverkeer en maken het mogelijk om observaties transparant te onderbouwen.  

**Implementatie**  
- **Dataverzameling**: voor elke test wordt een HAR-bestand aangemaakt met al het netwerkverkeer.  
- **Testmatrix**:  
  - Browser: Google Chrome  
  - Apparaat: Desktop (macOS)  
  - Consent-statussen:  
    - Geen consent  
    - Volledige consent  
    - Gedeeltelijke consent  
    - Consent ingetrokken  

Hoewel het technisch mogelijk is dat websites verschillend gedrag vertonen per browser of besturingssysteem, gebeurt dit doorgaans enkel voor browser-specifieke optimalisaties. Voor dit eerste onderzoek beperk ik me daarom tot één besturingssysteem (macOS) en één browser (Google Chrome, de populairste browser).  

Naast HAR-bestanden maak ik ook **screenshots** om de context te verduidelijken. Waar relevant verwijs ik naar specifieke **regel- of requestnummers in het HAR-bestand**, zodat bevindingen eenvoudig te verifiëren zijn.  

Om externe invloeden uit te sluiten, gebruik ik bij elke test een **nieuw browserprofiel**. Zo voorkom ik dat eerder gegeven toestemmingen of afwijzingen impact hebben op het gedrag van de onderzochte websites.  

**Sanitised HAR-bestanden**  
Bij dit onderzoek maak ik gebruik van de **sanitised HAR-bestanden** die standaard door moderne browsers worden aangemaakt. Deze bestanden registreren alle netwerkverzoeken en -antwoorden die relevant zijn voor analyse van datastromen. Om privacyredenen worden daarbij gevoelige gegevens zoals wachtwoorden, sessiecookies of tokens automatisch weggelaten.  

Dit heeft **geen impact op de analyse van Outbrain-verkeer**, omdat de parameters die ik bestudeer (zoals consent-status, schermresolutie, en andere metadata in de requests) volledig zichtbaar blijven. Sanitised HAR-bestanden zijn een erkende **best practice** voor reproduceerbaar technisch onderzoek.

> ⚠️ **Waarschuwing**  
> Dit onderzoek start met de analyse van **één specifieke website**. In een vervolgstap wordt nagegaan of het waargenomen gedrag enkel daar voorkomt, of ook op andere websites met Outbrain-integratie.  
> Net zoals in wetenschappelijk onderzoek is het doel om de kans op uitschieters zo klein mogelijk te houden. Hoeveel extra websites hiervoor onderzocht moeten worden, moet echter nog bepaald worden.  

# Sectie: Pageload zonder interactie (Scenario A)

**Doel**  
Vaststellen welk Outbrain-verkeer automatisch afvuurt bij een eerste pageload **zonder** enige gebruikersactie en **zonder** actieve consent (CMP).

> **In gewone taal**  
> We meten wat de Outbrain-widget doet **zodra u een pagina opent**. Dus nog **voor** u klikt of kiest in een cookiebanner: welke servers worden aangeroepen en welke informatie gaat er heen?

**Bronbestand**  
`f1journaal.be-outbrain-minimal.har` (19 requests).

> **In gewone taal**  
> Een HAR-bestand is een “zwarte doos” van de netwerkverkeer-log van uw browser. Zo kan iedereen nadien **controleren** wat er verzonden is.

**Meetopzet (samenvatting)**  
- Nieuw browserprofiel, desktop (macOS), Google Chrome.  
- Geen klikken, geen scrolls door de tester.  
- Analyse op basis van HAR-tijden t.o.v. eerste Outbrain-scriptrequest (T0).

> **In gewone taal**  
> We starten **schoon** (nieuw profiel) en we **doen niets** op de pagina. Daardoor zie je puur wat de site en de Outbrain-widget **zelf** verzenden.

---

## 1.1 Tijdlijn (eerste ~2 seconden)

- **T+0 ms** – Laad Outbrain hoofdbibliotheek: `widgets.outbrain.com/outbrain.js` **[0]**  
- **T+247–350 ms** – Init-pings en Browsi bootstrap:  
  - `tcheck.outbrainimg.com/.../check` **[1]**  
  - `widget-pixels.outbrain.com/widget/detect/px.gif` **[2]**  
  - **Eventlog POST** “BrowsiInjection”: `eventlog.outbrain.com/logger/v1/widget/` **[3]**  
  - Browsi supply: `yield-manager.browsiprod.com/supply/v5` **[4]**  
  - Browsi script: `cdn.browsiprod.com/.../PreEngine_desktop_*.js` **[5]**
- **T+817 ms** – **Multivac API** (content/recommendations): `mv.outbrain.com/Multivac/api/get?...` **[6]**  
  *(met o.a. `cmpStat=0`, `ccnsnt=false`, `ccpaStat=0` in de query)*  
- **T+1348–1372 ms** – **CHEQ anti-fraud logs** en **mcdp-logging**:  
  - `log.outbrainimg.com/loggerServices/dwce_cheq_events?...` **[7], [11], [13], [18]**  
  - **Outbrain logging “/l”**: `mcdp-chidc2.outbrain.com/l?...cnsnt=no_consent...` **[9], [12], [15]**  
  - UI/asset calls: `widgets.outbrainimg.com/images/...` **[8]**, `widgets.outbrain.com/nanoWidget/.../clip.js` **[14]**  
  - Creatives/media: `images.outbrainimg.com/transform/...mp4` **[10], [17]**
- **T+1379 ms** – Tweede Multivac call (met base64-`t` → `pvId`): **[16]**

> **In gewone taal**  
> Binnen **1 à 2 seconden** na het openen van de pagina praat de widget al met meerdere servers (Outbrain zelf én partners). Dat gebeurt **zonder dat u iets doet**.  
> **Hebben ze hiervoor informatie van u nodig?** Kort gezegd: voor een **eenvoudige, contextuele weergave** niet. De uitgever kan het blok **lokaal/statisch** renderen (of via eigen server) **zonder** unieke bezoekers-ID’s of pre-consent meetdata naar derden te sturen.
>
> **Wat Outbrain mogelijk zal aanvoeren als noodzaak vóór consent**  
> • *Anti-fraude/botdetectie*: basiscontrole om nepverkeer te weren.  
> • *Viewability/rapportage*: adverteerders willen weten of een advertentie zichtbaar was.  
> • *Prestatie-optimalisatie*: snel/traag netwerk bepalen om juiste variant te laden.  
> • *Frequentie-capping/deduplicatie*: niet te vaak dezelfde advertentie tonen.  
> • *Brand safety/compliance*: zeker weten dat de context geschikt is.  
>
> 
> **Waarom dit niet rechtvaardigt dataverwerking vóór consent**  
> • *Anti-fraude*: dit is een **commercieel belang** (bescherming advertentiebudget), maar géén **strikt noodzakelijke** voorwaarde om de dienst aan de gebruiker te leveren. De ePrivacy-uitzondering geldt enkel voor **technische levering** (bv. een login-cookie), niet voor fraudebestrijding.  
> • *Viewability/rapportage*: dit is relevant voor adverteerders, niet voor de gebruiker. Het kan dus **pas na toestemming** gebeuren.  
> • *Prestatie-optimalisatie*: kan client-side of **na consent** geregeld worden; niet essentieel voor het tonen van de widget.  
> • *Frequentie-capping*: vereist identifiers en valt buiten de noodzakelijkheidsexceptie. Alleen mogelijk met **opt-in**.  
> • *Brand safety*: kan publisher-side geregeld worden zonder Outbrain-calls vóór consent.
>
> **Kernpunt**: De wet maakt geen uitzondering omdat een partij “het nodig heeft voor fraude, inkomsten of optimalisatie”.  
> Alleen wat **strikt technisch vereist** is om de gevraagde inhoud te tonen, mag zonder toestemming. Alles daarbuiten valt onder de **toestemmingsplicht**.

---

## 1.2 Actieve domeinen in dit scenario

**Outbrain/OB-CDN:**  
- `widgets.outbrain.com` **[0], [14]**  
- `mv.outbrain.com` **[6], [16]**  
- `mcdp-chidc2.outbrain.com` **[9], [12], [15]**  
- `eventlog.outbrain.com` **[3]**  
- `log.outbrainimg.com` (CHEQ) **[7], [11], [13], [18]**  
- `tcheck.outbrainimg.com` **[1]**  
- `widget-pixels.outbrain.com` **[2]**  
- `images.outbrainimg.com` **[10], [17]**  
- `widgets.outbrainimg.com` **[8]**

**Derden gelinkt aan Outbrain-stack:**  
- **Browsi**: `yield-manager.browsiprod.com` **[4]**, `cdn.browsiprod.com` **[5]**


> Meerdere **derde partijen** worden betrokken. Dat is belangrijk, want **Outbrain is niet de eigenaar** van de site; het levert advertenties/widgets **op veel verschillende sites**. Daardoor kan er **cross‑site zicht** ontstaan.

> **Wat is “cross-site zicht”?**
Omdat Outbrain-servers op veel websites worden aangeroepen, ziet Outbrain over meerdere sites heen dat dezelfde
browser/huishoud-combinatie langs-komt (via IP, tijdstempel, UA, publisher-ID en evt. pageview/sessie-ID’s). Zo ontstaat
een schaduwprofiel (gedragspatroon) zónder dat je ergens bent ingelogd of een third-party cookie is gezet.
> Dit valt niet te bewijzen zonder de source code van het backend systeem van outbrain.

---

## 1.3 Consent-status in de requests (vóór consent)

- **Multivac** **[6], [16]**: `cmpStat=0`, `ccpaStat=0`, `ccnsnt=false`  
- **Outbrain mcdp-logging** **[9], [12], [15]**: `cnsnt=no_consent` (+ o.a. `pVis`, `lsd`, `widgetX/Y/Width/Height`)

> **In gewone taal**  
> De verzoeken **zeggen zelf** dat er (nog) **geen toestemming** is. Toch wordt er **al data uitgewisseld** (zie hieronder wat).  

TODO: wat is het verschil als je accept klikt? Dezelfde data? 


---

## 1.4 Welke gegevens worden (zonder interactie) verzonden?

### 1.4.1 Unieke identifiers & koppelingen
- **PageView ID (pvId)** via base64‑parameter `t` in Multivac **[16]**  
  `t=YWNiOGEzYTNjZDcxMjFmZjBhZjdlMDRhMjY0NmMwYjY=` → `acb8a3a3cd7121ff0af7e04a2646c0b6`  
- **Session ID** (CHEQ) **[7], [11], [13], [18]**: `sessionId=345a84d5-6e18-cd15-ba61-a76e59148044`  
- **mcdp `/l` tokens** **[9], [12], [15]** (hash_publisher_epoch_seq)

> **In gewone taal**  
> Dit zijn **etiketten** om dit bezoek en deze sessie te herkennen (en aan elkaar te koppelen).  
> **Waarom verzamelen?** Om **zichtbaarheid/rapportage** te doen en fouten op te sporen.  
> **Noodzakelijk vóór consent?** **Nee.** Je kunt een widget **contextueel** tonen zonder deze ID’s vóór uw keuze te versturen.  
> **Outbrain ≠ eigenaar:** als derde partij op veel sites kunnen zulke ID’s, samen met IP/tijd, **over sites heen** worden herkend.

### 1.4.2 Scherm/venster/positie & zichtbaarheid
- `widgetX`, `widgetY`, `widgetWidth`, `widgetHeight`, `pVis` (visibility), `lsd` (last scroll depth) in mcdp **[9], [12], [15]**

> **In gewone taal**  
> Dit geeft door **waar** het blok staat en of het **in beeld** is (of was).  
> **Waarom verzamelen?** Om te **meten** of een advertentie **zichtbaar** was (dit bepaalt vaak de betaling).  
> **Noodzakelijk vóór consent?** **Twijfelachtig.** Je kan dit **lokaler** berekenen en pas **na** toestemming doorsturen.  
> **Outbrain ≠ eigenaar:** door dit overal uit te wisselen kan een derde partij **globaal** leren wanneer en waar u content te zien krijgt.

### 1.4.3 Netwerk/kwaliteits‑metingen & vensterhints
- `cet=4g`, `rtt`, `ttfb`, `winW`, `winH`, `vpd` in Multivac **[6], [16]**
- 

> **In gewone taal**  
> Het systeem ziet of uw verbinding **snel** of **traag** is en hoe groot uw **venster** is.  
> **Waarom verzamelen?** Om **juiste varianten** te sturen (licht/zwaar) en prestaties te **analyseren**.  
> **Noodzakelijk vóór consent?** **Deels.** Vaak kan een site lokaal beslissen en is **server‑side** detail niet nodig vóór uw keuze.  
> **Outbrain ≠ eigenaar:** bij een derde partij vergroten extra metingen de **data‑oppervlakte** buiten de uitgever om.

### 1.4.4 A/B‑testen en optimalisatievelden

 **Wat betekenen deze velden waarschijnlijk?**  
 • `cet=4g` → netwerkprofiel (snel of traag).  
 • `rtt`, `ttfb` → gemeten laadtijden en vertraging.  
 • `winW`, `winH` → breedte en hoogte van het browservenster.  
 • `vpd` → viewport-informatie, bv. hoe ver of hoeveel van het blok zichtbaar is.

> Al deze parameters zijn **prestatie- en optimalisatie-signalen**. Ze helpen Outbrain bepalen welke variant te serveren
en hoe snel content laadt.  
> **Noodzakelijk vóór consent?** Nee, behalve dat een site *lokaal* moet weten hoe groot het venster is. Het versturen
van deze details naar een derde partij vóór uw toestemming is dus **niet vereist** om de widget te laten functioneren.

## 1.5 “No consent” maar wel datastromen (technische duiding)

- Multivac **[6], [16]**, mcdp-logging **[9], [12], [15]** en CHEQ **[7], [11], [13], [18]** wachten niet op “consent =
  true”.
- Identifiers en context gaan **pre-consent** al naar derden.

> **In gewone taal**  
> **Zonder keuze** gaat er al informatie naar Outbrain en partners.  
> **Noodzakelijk?** Sommige **basisdingen** zijn onvermijdelijk voor transport (bv. IP), maar **veel
optimalisatie/metingen** kunnen **na** uw keuze.  
> **Outbrain ≠ eigenaar:** omdat Outbrain op **veel sites** draait, kan dit een **cross-site beeld** geven.

### Welke parameters zien we in deze pre-consent datastromen?

- **`pvId`** – *PageView ID*: uniek ID voor dit pagina-bezoek.
- **`sessionId`** – *Session ID*: groepeert meerdere events in één browsersessie.
- **`token`** – samengesteld ID (hash + publisher-ID + timestamp + sequence).
- **`widgetX`, `widgetY`, `widgetWidth`, `widgetHeight`** – exacte positie en afmetingen van de widget op de pagina.
- **`pVis`** – *Page Visibility*: of de widget zichtbaar is.
- **`lsd`** – *Last Scroll Depth*: tot waar er gescrold is.
- **`tm`** – *Timing metric*: milliseconden sinds page-load.
- **`eT`** – *Event Type*: type event (bv. init, scroll, zichtbaarheid).
- **`ab`, `abwl`, `clss`, `settings`** – A/B-testvariabelen en configuratie.
- **`cet`, `rtt`, `ttfb`** – netwerksnelheid en laadtijdmetingen.
- **`winW`, `winH`, `vpd`** – vensterbreedte/-hoogte en viewportdata.
- **`oo=true`** – mogelijk “out of opt-out” flag (nog niet bevestigd).

### Interpretatie

> **In gewone taal**  
> Wat hier verstuurd wordt is een **mix van identifiers (bezoek/sessie)**, **positie/zichtbaarheid van het
advertentieblok**, en **prestatie-/netwerkmetingen**.
>
> **Waarom verzamelen?**  
> – Om advertenties correct te rapporteren (“was het zichtbaar?”).  
> – Om prestaties en A/B-testen te volgen.  
> – Om fraude of afwijkingen te detecteren.
>
> **Noodzakelijk vóór consent?**  
> – **Nee** voor de meeste velden: identifiers, zichtbaarheid, scroll-diepte en A/B-variabelen zijn *niet* essentieel om
de widget te tonen.  
> – **Ja, deels**: enkel het ophalen van de advertentie-inhoud zelf kan als noodzakelijk worden gezien.
>
> **Kernpunt**  
> De parameters die vóór consent al worden verstuurd, gaan duidelijk verder dan “strikt noodzakelijk”. Ze geven Outbrain
als derde partij de mogelijkheid om gebruikers over **meerdere sites** te volgen en te analyseren.
---

## 1.6 Tokenanalyse (mcdp `/l`)

Voorbeeldtokens (binnen ~20 ms) **[9], [12], [15]**:  
```
e4094a9bf04597f66722740012bf72ca_230797_1757329462220_1
4d93b05d698fbf82fb488578cacbb639_230797_1757329462294_1
cff3c671a0b3f7c95aa2c5606f63645b_230797_1757329462493_1
```
→ patroon: **hash** \+ **publisher/site‑ID (230797)** \+ **epoch‑ms** \+ **sequence**.

> **In gewone taal**  
> Dit lijkt op een **log‑/diagnose‑ID**.  
> **Waarom verzamelen?** Voor **foutenanalyse** en **routering**.  
> **Noodzakelijk vóór consent?** **Nee.** Dit kan ook **na** toestemming of met **ephemeral** (kortlevende) ID’s.

---

## 1.7 Derde partijen/integraties

- **CHEQ (anti‑fraude)** – events met `sessionId` en `pvId` **[7], [11], [13], [18]**  
- **Browsi** – supply call + script vóór interactie **[4], [5]**  
- **Eventlog** “BrowsiInjection” via Outbrain **[3]**

> **In gewone taal**  
> Nog **vóór** u iets kiest, zijn al **meerdere** partijen actief.  
> **Noodzakelijk?** **Alleen** als de uitgever kan aantonen dat dit **strikt noodzakelijk** is vóór consent (bv. harde anti‑fraude).  
> **Outbrain ≠ eigenaar:** extra **datadeling** met derden vergroot het risico op **cross‑site** groepering.

---

## 1.8 Reproduceerbaarheid / controleerbaarheid

- **T0**: eerste Outbrain‑script **[0]**  
- **Consentsignalen**: `ccnsnt=false`/`cmpStat=0` (Multivac **[6], [16]**), `cnsnt=no_consent` (mcdp **[9], [12], [15]**)  
- **Koppeling**: `pvId` in CHEQ **[7], [11], [13], [18]** = base64 `t` uit Multivac **[16]**

> **In gewone taal**  
> We geven voor elke claim **controleerbare** HAR‑regels. Iedereen kan dit **natrekken**.

---

## 1.9 Beperkingen & voorzorg

- Eén site en één scenario (geen interactie, geen consent).  
- HAR is **sanitised**: gevoelige secrets gestript; relevante parameters zichtbaar.  
- Geen broncode‑inspectie; conclusies op **netwerkverkeer**.

> **In gewone taal**  
> Dit is een **momentopname**. Het toont **feitelijk gedrag** op één site. Juridische duiding hoort bij juristen.

---

## 1.10 Aanbevolen vervolgstappen (direct uitvoerbaar)

1. **Consent‑timing exact meten** (CMP ready vs. Outbrain init vs. eerste calls).  
2. **Matrix uitbreiden** (volledige/gedeeltelijke/ingetrokken consent; incognito; cross‑site).  
3. **Token‑persistency** (blijft het hash‑deel gelijk per profiel/site?).  
4. **Scopecheck** op 2–3 andere Belgische publishers.

> **In gewone taal**  
> Deze stappen maken duidelijk **wat echt vóór/na** uw keuze gebeurt en **hoe breed** het patroon is.

---

## 1.11 Schaduwprofiel op basis van IP (server‑side realiteit)

**IP‑feit:** elke server die u aanroept, ziet uw **IP‑adres** (ook al staat het niet in de HAR). Dus Outbrain, CHEQ en Browsi krijgen bij elk verzoek uw **bron‑IP**.

**Waarom is IP genoeg?**  
- **Herkenning per huishouden/bedrijf** over tijd (stabiele IP/prefix).  
- **Koppeling** met tijdstip, User‑Agent en paginacontent = **pseudo‑profiel per IP/device**.  
- **Cross‑site**: Outbrain is **derde partij** op veel sites → **overzichten** per IP zonder cookies.  
- **Interne ID‑linking**: IP ↔ pvId/sessie in server‑logs geeft **continuïteit** over bezoeken.

> **In gewone taal**  
> Alleen al via **IP + timing + browser‑info** kan een partij u over meerdere sites **herkennen/groeperen**. Dat heet een **schaduwprofiel**.  
> **Noodzakelijk?** IP is nodig om **technisch te antwoorden**, maar **bewaren/aggregatie** (zeker **cross‑site**) vóór consent is **niet** noodzakelijk voor louter context tonen.
> **Outbrain ≠ eigenaar:** precies daarom is de **derde‑partijpositie** hier cruciaal voor trackingrisico.

---

## 1.12 Noodzaak‑per‑datapunt (technische proportionaliteit)

| Datapunt | Typisch doel (leverancier) | Strikt noodzakelijk vóór consent? | Risico op tracking over tijd | Outbrain is derde partij (cross‑site)? | Opmerkingen |
|---|---|---|---|---|---|
| **IP-adres (server‑side, altijd)** | Terugzenden response, geolocatie op land/stad, fraudedetectie | **Deels** (transport is noodzakelijk; opslag/verdere verwerking niet per se) | **Hoog** | **Ja** | Server ziet IP bij elke call; opslag/aggregatie vergroot risico. |
| **Tijdstempel** | Meting, volgorde events, frequentie | **Deels** | **Middel/Hoog** | **Ja** | In combinatie met IP/UA sterk. |
| **URL/Referrer** | Context (welke pagina/onderwerp), brand safety | **Ja** (voor contextuele rendering) | **Middel** | **Ja** | Context tonen kan zonder langdurige opslag. |
| **User‑Agent** | Compatibiliteit, rendering, debug | **Deels** | **Middel** | **Ja** | UA + IP helpt groeperen. |
| **PageView ID (pvId)** | Zichtbaarheid/rapportage | **Nee** | **Middel/Hoog** | **Ja** | Niet vereist voor enkel context tonen. |
| **Session ID** | Groeperen events binnen sessie | **Nee** | **Middel/Hoog** | **Ja** | Alternatief: kortstondige, lokale counters. |
| **mcdp `/l` token** | Logging/diagnostiek/AB-routing | **Nee** | **Middel/Hoog** | **Ja** | Bevat (hash,publisher,timestamp,seq). |
| **Widget posities/zichtbaarheid** | Viewability‑meting, lazy‑load | **Twijfelachtig** | **Middel** | **Ja** | Verzenden **na** consent of geaggregeerd. |
| **Netwerkhints (`cet`, `rtt`, `ttfb`)** | Variantkeuze/prestaties | **Deels** | **Laag/Middel** | **Ja** | Vaak lokaal beslisbaar. |
| **A/B‑testvelden** | Optimalisatie/inkomsten | **Nee** | **Middel** | **Ja** | Uitstellen tot na consent. |

> **In gewone taal**  
> Sommige data is **onvermijdelijk** (IP voor transport), maar veel optimalisatie‑ en meetdata kan **later** of **lokaler**. Omdat Outbrain **derde partij** is, is vroegtijdige datadeling **extra gevoelig**.

# Samenvatting

### T+ ~817 ms — Multivac-aanroep (content/recommendations)

Rond **T+ 817 ms** vraagt de browser via `mv.outbrain.com/Multivac/api/get` aanbevelingen op bij Outbrain. In dezelfde
aanvraag staan expliciete consentsignalen die aangeven dat er **(nog) geen toestemming** is: `cmpStat=0`, `ccpaStat=0`,
`ccnsnt=false`.

Naast deze consentsignalen bevat de call een **brede set aanvullende gegevens** die het bezoek herkenbaar maakt, de
weergave aanstuurt en metingen mogelijk maakt:

#### 1) Identificatie & koppeling

- **Page-view ID** via `t` (base64) → decodeert naar een 32-tekens ID (bijv. `acb8a3a3cd7121ff0af7e04a2646c0b6`).
- **Widget-/variantvelden** zoals `widgetJSId`, `idx`, `rand`, `sig`, `key`, `format`, `settings`, `recs`, `va`, `et`,
  `clss`, `version`.
- **Clientversievelden**: `clientType`, `clientVer`.

#### 2) Paginacontext

- **Bezochte URL en origin**: `url`, `ogn` (bijv. `https://f1journaal.be/`).

#### 3) Zichtbaarheid/positie & venstermaten

- **Venster/scherm**: `winW`, `winH`, `scrW`, `scrH`, `dpr` (bijv. `winW=957`, `winH=857`, `scrW=1512`, `scrH=982`,
  `dpr=2`).
- **Viewport/positie**: `vpd` (bijv. `2959`, `857`) en coördinaten `px`, `py` (bijv. `px=673`, `py=1714`).
- **Tab/veiligheid**: `activeTab=true`, `secured=true`.

#### 4) Netwerk & prestatie

- **Laadtijdmeting**: `ttfb` (bijv. `795` milliseconden).
- **Bandbreedte-indicatie**: `bandwidth` (bijv. `10`).

#### 5) Browser- en platformkenmerken (Client Hints)

- **Architectuur/bit-diepte**: `cha=arm`, `chb=64`.
- **Platform/versie**: `chp=macOS`, `chpv=15.6.0`.
- **Browsermerken/versies**: `chfv` (JSON-lijst met o.a. Chromium/Chrome-versies).

#### 6) Detectie-/besturingsflags

- **Adblock-indicatie**: `adblck=0` (geen adblock gedetecteerd).
- **Touch-ondersteuning**: `tch=0`.
- **Attributie/compatibiliteit**: `wdr-attribution-src=1`, `wdr-cosc=1`.
- **Overig**: `abwl` (A/B-test whitelist), `apv`, `lsl`.

---

#### Relevantie vóór toestemming

De genoemde gegevens ondersteunen meting, optimalisatie en koppeling (zoals zichtbaarheid/positie, variantenkeuze,
prestatieprofilering en diagnostiek). Zij zijn **niet strikt noodzakelijk** om louter **contextuele aanbevelingen** te
tonen. Technisch kan de widget worden weergegeven zonder het **vroegtijdig** doorsturen van een pa



---

# Opvolging momenteel zonder .har file voor gesprek:

Bij **afgewezen cookies** voegt Outbrain **extra browser fingerprinting-parameters** toe die **niet aanwezig** zijn wanneer cookies worden geaccepteerd:

**Aanvullende parameters bij "cookies not accepted"**:

- **`wdr-cosc=1`** – "Without Cookies" flag  
- **`cha=arm, chb=64`** – CPU-architectuur (ARM, 64-bit)  
- **`chfv=[...]`** – Gedetailleerde browser-versie fingerprint  
- **`chpv=15.6.0, chp=macOS`** – Platform-specificatie (macOS versie)

**Wat blijft gelijk in beide scenario's**:
- Session/page identifiers blijven aanwezig  
- Performance metrics worden nog steeds verzameld  
- Zichtbaarheids- en positiedata ongewijzigd  
- Consent-strings aanwezig in beide gevallen  

**Technische Analyse van Fingerprinting-Parameters**:

```
cha=arm     // CPU-architectuur (ARM vs x86)
chb=64      // Bit-diepte (32/64-bit)
```
**Trackingwaarde**: CPU-architectuur is **relatief stabiel** en helpt bij **device-herkenning** over sessies heen, ook zonder persistente identifiers.

```javascript
chfv=[
  {"brand":"Chromium","version":"140.0.7339.133"},
  {"brand":"Not=A?Brand","version":"24.0.0.0"},
  {"brand":"Google Chrome","version":"140.0.7339.133"}
]
```
**Trackingwaarde**: Specifieke versie-combinaties zijn **uniek genoeg** voor fingerprinting, vooral in combinatie met andere parameters.

```
chp=macOS           // Besturingssysteem
chpv=15.6.0        // Specifieke OS-versie
```
**Trackingwaarde**: OS-versies updaten **traag genoeg** om als **semi-persistente identifier** te fungeren.


