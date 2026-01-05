# Kriminalitatearen Mapa Interaktiboa  
**Leaflet erabiliz egindako Web Dashboard-a**

## 1. Sarrera

Dokumentu honek Leaflet liburutegian oinarritutako web aplikazio baten funtzionamendua azaltzen du. Aplikazioaren helburua da **kriminalitate-adierazleak modu bisual eta interaktiboan erakustea**, GeoJSON datu geografikoak erabiliz.

Erabiltzaileak honako ekintzak egin ditzake:
- Urtea hautatu
- Delitu-kategoria filtratu
- Udalerri bakoitzeko informazio zehatza popup bidez kontsultatu

Aplikazioa egokia da analisi estatistikorako, hezkuntza-inguruneetarako eta administrazio publikoan erabiltzeko.

## 2. Teknologia eta Mendekotasunak

### Erabilitako teknologiak
- **HTML5**: Edukiaren egitura semantikoa
- **CSS3**: Diseinu bisuala eta layout-a
- **JavaScript (ES6)**: Logika eta interakzio dinamikoa
- **Leaflet.js**: Mapen bistaratzea
- **OpenStreetMap**: Oinarri kartografikoa
- **Font Awesome**: Ikono grafikoak

### Kanpoko baliabideak
```html
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" />
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
```

## 3. HTML Egitura

### Elementuak
- **map**: Leaflet maparen edukia
- **sidebar**: Iragazkien panela
- **toggleFilters**: Mugikorrentzako sidebar botoia
- **yearFilter**: Urtearen hautaketa
- **categoryFilter**: Delitu kategorien hautaketa

### Maparen hasieraketa
```html
let map = L.map('map').setView([43.235, -2.885], 9);

Hasierako kokapena: Euskadi ingurua

Zoom maila: 9

Oinarri mapa: OpenStreetMap

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
}).addTo(map);
```
### Datuen Karga (GeoJSON)

Aplikazioak kriminalitate-datu geografikoak GeoJSON fitxategi batetik kargatzen ditu modu asinkronoan.
```html

fetch("data.geojson")
    .then(r => r.json())
    .then(data => {
        geoData = data;
        applyFilters();
    });
```

data.geojson fitxategiak udalerri bakoitzeko informazioa dauka

Datuak geoData aldagaian gordetzen dira

Karga amaitzean, iragazkiak automatikoki aplikatzen dira

### Utilitate Funtzioak
**val(v)**

Balioa null edo undefined bada, 0 itzultzen du, erroreak saihesteko.

```html

const val = v => v ?? 0;

getTotal(feature, category, year)
```
Aukeratutako urte eta kategoriaren araberako delitu kopuru totala kalkulatzen du.
```html
function getTotal(f, cat, year) {
    if (cat === "all")
        return val(f.properties.datos.total_infracciones_penales[year]);
    return val(f.properties.datos[cat]?.total?.[year]);
}
```

Kategoria orokorra edo espezifikoa onartzen du

GeoJSON egiturara egokituta dago

**getColor(value)**

Delitu kopuruaren arabera kolore bat esleitzen du.

Balioa	Kolorea
> 800	Gorria iluna
> 500	Gorria
> 200	Laranja iluna
> 100	Laranja
> 50	Horia
≤ 50	Hori argia
```html
function getColor(d) {
    return d > 800 ? '#800026' :
           d > 500 ? '#BD0026' :
           d > 200 ? '#E31A1C' :
           d > 100 ? '#FC4E2A' :
           d > 50  ? '#FD8D3C' :
                     '#FEB24C';
}
```

**getRadius(value)**

Zirkulu-markatzaileen tamaina kalkulatzen du, delitu kopuruaren arabera.
```html
function getRadius(d) {
    return Math.max(6, Math.sqrt(d) / 2);
}
```

### Maparen Renderizatzea
**render(data, year, category)**

Funtzio honek mapa eguneratzen du hautatutako iragazkien arabera.
```html
function render(data, year, category) {
    if (geoLayer) map.removeLayer(geoLayer);
    if (legend) map.removeControl(legend);

geoLayer = L.geoJSON(data, {
        pointToLayer: (f, latlng) => {
            const total = getTotal(f, category, year);
            return L.circleMarker(latlng, {
                radius: getRadius(total),
                fillColor: getColor(total),
                color: "#222",
                weight: 1,
                fillOpacity: 0.85
            });
        },
        onEachFeature: (f, l) => {
            l.bindPopup(buildPopup(f, year, category), { maxWidth: 350 });
        }
    }).addTo(map);

buildLegend();
}
```

Eginkizunak:

Aurreko geruzak ezabatzea

GeoJSON geruza berria sortzea

Zirkulu-markatzaileak bistaratzea

Popup-ak gehitzea

Legenda sortzea

### Popup-en Eraikuntza
**buildPopup(feature, year, category)**

Udalerri bakoitzeko informazio zehatza erakusten duen popup-a sortzen du.
```html
function buildPopup(f, year, category) {
    const p = f.properties;
    let html = `<h3>${p.municipio}</h3>
<b>Urtea:</b> ${p.year_columns[year]}<hr>`;

    if (category === "all") {
        html += `
<table class="popup-table">
<tr><td>Total penala</td><td>${p.datos.total_infracciones_penales[year]}</td></tr>
<tr><td>Tasa x 1000 Biz</td><td>${p.datos.tasa_x_1000_hab_total_penal[year]}</td></tr>
</table>`;
        return html;
    }

    const cat = p.datos[category];
    html += `<b>Total Kategoria:</b> ${val(cat.total?.[year])}<br><br>`;
    html += `<table class="popup-table">`;

    Object.keys(cat).forEach(k => {
        if (k === "total") return;
        html += `<tr>
<td>${k.replaceAll("_", " ")}</td>
<td>${val(cat[k][year])}</td>
</tr>`;
    });

    html += `</table>`;
    return html;
}
```

Popup-ak honako informazioa eskaintzen du:

Udalerriaren izena

Hautatutako urtea

Delitu kopuru orokorra

Kategoriaren azpidatuak taula moduan

### Legenda Dinamikoa
**buildLegend()**

Maparen behe-eskuinean legenda bisuala gehitzen du.
```html
function buildLegend() {
    legend = L.control({ position: 'bottomright' });

    legend.onAdd = () => {
        const div = L.DomUtil.create('div', 'legend');
        const grades = [0, 50, 100, 200, 500, 800];

        div.innerHTML = "<b>Infrakzio totalak</b><br>";
        grades.forEach((g, i) => {
            div.innerHTML +=
            `<i style="background:${getColor(g + 1)}"></i>
            ${g}${grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+'}`;
        });
        return div;
    };

    legend.addTo(map);
}
```
### Iragazkien Sistema
**applyFilters()**

Erabiltzaileak hautatutako iragazkiak aplikatzen ditu.
```html
function applyFilters() {
    const year = yearFilter.value;
    const category = categoryFilter.value;

    render({
        type: "FeatureCollection",
        features: geoData.features.filter(f =>
            category === "all" || f.properties.datos[category]
        )
    }, year, category);
}
```


Urtea eta kategoria kontuan hartzen ditu

Datuak filtratu eta mapa eguneratzen du

### Mugikorretarako Egokitzapena (Responsive)

Aplikazioa pantaila txikietara egokitzen da.
```html
toggleFilters.onclick = () =>
    sidebar.classList.toggle("hidden");

if (innerWidth < 700)
    toggleFilters.style.display = "block";
```

Sidebar-a ezkutatzen da mugikorretan

Botoi bidez erakutsi edo ezkutatu daiteke

Erabilerraztasuna hobetzen du

## Ondorioa

Web aplikazio honek:

Kriminalitate-datuak modu bisual eta argian aurkezten ditu

Interfaze intuitibo eta interaktiboa eskaintzen du

Erraz heda daiteke (urte edo kategoria berriak)

GIS oinarritutako dashboard sendoa eskaintzen du

Egokia da ikerketa, hezkuntza eta administrazio publikoan erabiltzeko.
