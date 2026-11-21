# VRT Radio voor XENEON EDGE

Een Nederlandstalige radiowidget voor de Corsair XENEON EDGE met Radio 1 en StuBru (Studio Brussel) live en het VRT NWS-nieuwsbulletin. Je hebt Windows, iCUE 5.54 of hoger en een internetverbinding nodig.

## Installeren

1. Download [vrt-radio.icuewidget](./vrt-radio.icuewidget).
2. Open **Widgets** in iCUE, kies **+** en importeer het bestand.
3. Kies **Small (S)** voor de bediening en het frequentiespectrum, of **Medium (M)** voor dezelfde bediening met **Nu op de radio** erbij. Daar verschijnen nummer en artiest wanneer de feed van Radio 1 of StuBru die levert.

Tik op **Radio 1**, **VRT NWS** of **StuBru** om die zender te starten. Gebruik de grote knop om af te spelen of te pauzeren.

In **Medium** wordt **Nu op de radio** ongeveer elke 15 seconden bijgewerkt. Bij het starten of wisselen van zender kan het even duren voor de titel en artiest verschijnen. Als een update geen nummerinformatie oplevert, blijft het laatst bevestigde nummer ongeveer een minuut zichtbaar.

Tik op **Kopieer** rechts naast een nummer om `artiest – titel` op je klembord te zetten. Plak de tekst in bijvoorbeeld YouTube Music. iCUE vraagt hiervoor toegang tot het klembord.

## Instellingen

| Instelling | Werking |
| --- | --- |
| Standaardzender | Kies Radio 1, VRT NWS of StuBru. Radio 1 is vooraf geselecteerd. De widget onthoudt de laatst gekozen zender. |
| Automatisch afspelen | Standaard uit. Als iCUE automatisch afspelen blokkeert, verschijnt **Tik op afspelen**. |
| Volume | Start op 65%. De schuifregelaar onthoudt het laatst gekozen volume. |
| Accentkleur | Past de afspeelknop, zenderselectie en volumeregelaar aan. |
| Streamadres Radio 1, VRT NWS en StuBru | Elke zender heeft een eigen veld met een vooraf ingevuld standaardadres. Pas het aan met een volledig HTTPS-streamadres. Maak het veld leeg om terug te keren naar het standaardadres. |

## Verbinding en machtigingen

iCUE vraagt luidsprekertoegang en HTTPS-toegang tot `icecast.vrtcdn.be`, `quantumcast.vrtcdn.be` en `streamabc.net` voor de livestreams en hun doorverwijzingen. Tijdens het afspelen van Radio 1 of StuBru leest Medium de nummerinformatie rechtstreeks uit korte HLS-fragmenten via `radio.vrtcdn.be` en subdomeinen van `streamabc.net`. Tijdens gesproken uitzendingen of jingles is er soms geen nummerinformatie. Voor een zelf ingesteld adres kan iCUE tijdens het afspelen extra netwerktoegang vragen.

Voor VRT NWS speelt de widget het MP3-nieuwsbulletin rechtstreeks af via `progressive-audio.vrtcdn.be`, zodat de browser het signaal kan analyseren. Als die verbinding mislukt, probeert de widget de officiële VRT-link. Bij die omweg blijven de spectrumbalken stil. Na afloop haalt de widget het bulletin opnieuw op.

Bij verbindingsverlies probeert de widget opnieuw na 2, 5, 10 en daarna telkens 30 seconden. De widget toont wanneer de volgende poging volgt.

## Zelf bouwen

De widgetbron staat in [radio-widget/](./radio-widget/) en heeft geen runtime-afhankelijkheden. iCUE 5.54.46 verwacht `clipboard-read-write` in het manifest. WidgetBuilder CLI 0.5.2 verwacht hier underscores en weigert die spelling. Maak het installatiebestand daarom rechtstreeks vanuit de bronmap.

```powershell
$zip = Join-Path $env:TEMP ("vrt-radio-" + [guid]::NewGuid() + ".zip")
[IO.Compression.ZipFile]::CreateFromDirectory((Resolve-Path radio-widget).Path, $zip)
Move-Item -LiteralPath $zip -Destination vrt-radio.icuewidget -Force
```
