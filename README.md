# Veen of Zand?

Een poging om wat diepte toe te voegen aan het onderscheid tussen Veen- en Zand-Drenten.

Dit is een leerproject voor mij als developer met een interesse in de bodem om met echte Nederlandse geodata te werken (PDOK en de Basisregistratie Ondergrond, BRO): typ een plaats in Drenthe of klik op de kaart waar je vandaan komt, en de app zegt wat voor Drent je bent.

Live: https://nluning.github.io/zand-of-veen-drent/

## Hoe het werkt

1. **Plaats of klik naar punt.** De PDOK Locatieserver zet de plaatsnaam om in het zwaartepunt (centroïde) van de woonplaats. Bij een klik op de kaart wordt het aangeklikte punt gebruikt.
2. **Punt naar bodem.** De BRO Bodemkaart (via PDOK WMS GetFeatureInfo) geeft de bodemnaam op dat punt.
3. **Bebouwd gebied.** In steden heeft de bodemkaart geen gegevens. Dan zoekt de app in de BRO REST API naar de dichtstbijzijnde geologische (BHR-G) of geotechnische (BHR-GT) boring, met een straal die groeit van 0,5 tot 5 km. De eerste laag die geen ophoging of verharding is, geeft de naam.

Zoeken en klikken delen vanaf stap 2 dezelfde code (`answerForPoint`): beide leveren een punt in WGS84 en RD, en daarna is het antwoord hetzelfde. Als je snel achter elkaar zoekt of klikt, telt alleen de laatste actie. Een oudere, tragere zoekopdracht kan het antwoord of de markering dus niet meer overschrijven.

De bodemnaam wordt getoond zoals de bron hem geeft: puntkomma's weggelaten, spaties vervangen door streepjes.

## De kaart

- De achtergrondkaart is de BRT van PDOK, met Leaflet.
- Een klik op de kaart zet een markering. Pas na de knop "Zoek hier" wordt de bodem opgezocht.
- Klikken buiten Drenthe worden geweigerd. Dat controleert de app met de `reverse`-functie van de Locatieserver.
- De klik komt binnen als WGS84 (lengte en breedtegraad). De bodemkaart wil RD New (EPSG:28992), dus Proj4js rekent het punt om.

## Starten

Het is één HTML-bestand met gewone JavaScript. Open `index.html` in een browser. Er is geen build en geen server nodig. Leaflet en Proj4js worden via een CDN geladen, dus je hebt internet nodig.

## Data

- [PDOK Locatieserver](https://api.pdok.nl/bzk/locatieserver/search/v3_1/doc)
- [BRO Bodemkaart](https://service.pdok.nl/tno/bro-bodemkaart/wms/v1_0?request=getCapabilities&service=WMS)
- [BRO REST API](https://basisregistratieondergrond.nl/werken-bro/producten-diensten/rest-api/): BHR-G en BHR-GT
- [BRT achtergrondkaart](https://service.pdok.nl/kadaster/brt-achtergrondkaart/wmts/v2_0?request=getcapabilities&service=WMTS)

## Beperkingen

- Alleen Drenthe.
- Het zwaartepunt van een woonplaats ligt soms buiten de bebouwing (bij Zuidwolde ongeveer 1,1 km). Dan antwoordt de bodemkaart voor het land eromheen en wordt er geen boring gebruikt. Voor een leerproject is dat zo gelaten.
- In steden kan de boring ver van het punt liggen (tot 5 km), dus het antwoord zegt minder over de exacte plek.
- Boringen noemen lagen in hun eigen woordenschat (bijvoorbeeld `siltigZand`). Die wordt ongewijzigd getoond.
- De kaart toont alleen de achtergrond en een markering. Het bodemvlak wordt niet gemarkeerd. Dat is geprobeerd met vectortegels en weer weggehaald.

## Over de code

De code is geschreven met Claude (Anthropic). De rolverdeling:

- **AI:** schreef de code voor alle fasen, en was ook mentor en leraar. Ik leerde stap voor stap mee: bounding boxes, coördinatenstelsels, hoe WMS en de BRO-bevraging werken.
- **Ik:** bepaalde wat de app moest doen, las en testte de code, vond de fouten en maakte de keuzes onder "Ontwerpkeuzes".
- **Voorbeeld van eigen afweging:** de AI stelde voor om in bebouwd gebied de bodem te schatten uit een ring van punten rond de plek. Ik koos ervoor om in plaats daarvan nabije boringen uit de BRO te gebruiken.

## Geleerde concepten

- **WMS GetFeatureInfo:** een kaartplaatje tekenen en vragen wat onder één pixel zit.
- **Coördinatenstelsels:** WGS84 (klik), RD New (bodemkaart) en Web Mercator (kaarttegels), en omrekenen met Proj4js.
- **Kaartlaag versus puntwaarneming:** de bodemkaart toont vlakken, boringen zijn metingen op één plek.
- **Pedologie versus lithologie:** de bodemkaart beschrijft hoe de bovenste 1,2 m is gevormd, een boring beschrijft waaruit de grond bestaat, laag voor laag. Een boring is dus niet 1-op-1 een bodemtype.
- **BHR-G en BHR-GT:** geologisch en geotechnisch booronderzoek in de BRO.
- **Antropogene lagen:** opgebrachte grond en verharding, die je overslaat om bij de natuurlijke bodem te komen.
- **Woonplaats als bestuurlijk gebied:** het zwaartepunt (centroïde) kan buiten de bebouwing liggen.
- **CORS en preflight:** waarom de browser de PDOK- en BRO-diensten rechtstreeks mag aanroepen, zonder proxy.

## Ontwerpkeuzes

1. **WMS voor de bodemkaart.** WFS ("welk vlak bevat dit punt?") zou de nettere vraag zijn, maar PDOK biedt voor de BRO Bodemkaart geen WFS aan, en de OGC API heeft alleen Tiles (gecontroleerd op 13-11-2026). GetFeatureInfo is dus de enige directe puntbevraging. Ik controleer eerst wat een dienst aanbiedt voordat ik een aanpak kies.
2. **Boringen in plaats van een steekproef.** In dorpen en steden geeft de bodemkaart niets terug (gecontroleerd voor Assen, Emmen, Hoogeveen, Meppel en Exloo). Een steekproef rond het punt zou het gat omzeilen, een boring geeft een antwoord uit een meting op die plek.
3. **Straal die groeit, dichtstbijzijnde boring.** Zoeken in BHR-G en BHR-GT met 0,5, 1, 2 en 5 km, dan de dichtstbijzijnde. Voorbeeld: in Assen ligt een BHR-GT-boring op ongeveer 640 m, met als eerste natuurlijke laag siltig zand. In Meppel ligt de dichtstbijzijnde op 2,7 km, en die zegt minder over de grond onder je voeten.
4. **Een klik zoekt niet meteen.** Een klik zet een markering, de knop "Zoek hier" start de zoekopdracht. Klikken buiten Drenthe weigert de app.
5. **Zuidwolde laten staan.** Het zwaartepunt van de woonplaats ligt ongeveer 1,1 km buiten het dorp. Voor een leerproject heb ik dat bewust zo gelaten.
