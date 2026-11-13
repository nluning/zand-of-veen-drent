# Wat voor Drent ben jij?

Type a place in Drenthe and get one sentence about the soil under it: *Je bent een zwak-lemig-fijn-zand-Drent!*

Live: https://nluning.github.io/zand-of-veen-drent/

## How it works

1. **Place → point.** The PDOK Locatieserver turns the place name into the centroid of the village.
2. **Point → soil.** The BRO Bodemkaart (via PDOK WMS GetFeatureInfo) returns the soil name at that point.
3. **Built-up areas.** The soil map has no data in towns. In that case the BRO REST API is searched for the nearest geological (BHR-G) or geotechnical (BHR-GT) borehole, with a radius that grows from 0.5 to 5 km. The first layer that isn't fill or paving gives the name.

The soil name is shown as it comes from the source: semicolons dropped, spaces replaced by dashes.

## Run it

It is a single HTML file with plain JavaScript. Open `index.html` in a browser; there is no build step and no server.

## Data

- [PDOK Locatieserver](https://api.pdok.nl/bzk/locatieserver/search/v3_1/doc)
- [BRO Bodemkaart](https://www.pdok.nl/) (WMS)
- [BRO REST API](https://basisregistratieondergrond.nl/werken-bro/producten-diensten/rest-api/): BHR-G and BHR-GT

## Limitations

- Drenthe only.
- In towns the borehole can be some distance from the centroid (up to 5 km), so the answer says less about the exact spot.
- Boreholes name layers in their own vocabulary (e.g. `siltigZand`), which is shown unchanged.
