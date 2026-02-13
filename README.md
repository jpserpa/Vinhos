
<html lang="pt">
<head>
<meta charset="UTF-8">
<title>Mapa Interativo dos Vinhos - Layout Lado a Lado</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<style>
body { margin:0; font-family: Arial; display: flex; height: 100vh; }
#sidebar {
  width: 350px;
  min-width: 250px;
  max-width: 40%;
  height: 100%;
  padding: 15px;
  background: #f4f4f4;
  overflow-y: auto;
  box-shadow: 2px 0 5px rgba(0,0,0,0.1);
}
#map { flex-grow:1; height:100%; }
button.fornecedor {
  width: 100%;
  padding: 12px;
  margin-bottom: 5px;
  cursor: pointer;
  text-align: left;
  font-size: 14px;
}
button.fornecedor:hover { background-color: #ddd; }
h2 { margin-top: 0; font-size: 18px; }
label { font-weight: bold; }
@media (max-width: 768px){
  body { flex-direction: column; }
  #sidebar { width: 100%; max-height: 40%; }
  #map { flex-grow: 1; height: 60%; }
}
</style>
</head>
<body>

<div id="sidebar">
  <h2>Vinhos de Portugal</h2>
  <label>Região</label>
  <select id="regiao"></select>
  <div id="fornecedoresLista"></div>
  <p id="contador"></p>
</div>

<div id="map"></div>

<script>
// ----- DADOS -----
const dados = [
 {regiao:"Alentejo", fornecedor:"Tiago Cabaço", cidade:"Évora", produto:"Blog Tour", lat:38.57, lng:-7.9},
 {regiao:"Alentejo", fornecedor:"Herdade do Esporão", cidade:"Reguengos", produto:"Monte Velho", lat:38.30, lng:-7.50},
 {regiao:"Setúbal", fornecedor:"José Maria da Fonseca", cidade:"Setúbal", produto:"Periquita", lat:38.52, lng:-8.89},
 {regiao:"Setúbal", fornecedor:"Bacalhôa", cidade:"Azeitão", produto:"Catarina", lat:38.55, lng:-8.88},
 {regiao:"Douro", fornecedor:"Graham’s", cidade:"Vila Nova de Gaia", produto:"Six Grapes", lat:41.14, lng:-8.61},
 {regiao:"Douro", fornecedor:"Sogrape", cidade:"Peso da Régua", produto:"Papa Figos", lat:41.16, lng:-7.79},
 {regiao:"Vinho Verde", fornecedor:"Aveleda", cidade:"Penafiel", produto:"Casal Garcia", lat:41.21, lng:-8.28},
 {regiao:"Lisboa", fornecedor:"Casa Santos Lima", cidade:"Lisboa", produto:"Colossal", lat:39.05, lng:-9.00},
 {regiao:"Algarve", fornecedor:"Quinta dos Vales", cidade:"Lagoa", produto:"Grace Vineyard", lat:37.12, lng:-8.45},
 {regiao:"Açores", fornecedor:"Azores Wine Company", cidade:"Ponta Delgada", produto:"Terrantez", lat:38.47, lng:-28.40},
 {regiao:"Madeira", fornecedor:"Henriques & Henriques", cidade:"Funchal", produto:"10 Years", lat:32.65, lng:-16.90}
];

// ----- MAPA MINIMALISTA -----
const map = L.map('map').setView([39.5, -8], 6);
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> &copy; <a href="https://carto.com/">CARTO</a>',
    subdomains: 'abcd',
    maxZoom: 19
}).addTo(map);

let markers = [];
const regiaoSelect = document.getElementById("regiao");
const fornecedoresLista = document.getElementById("fornecedoresLista");
const contador = document.getElementById("contador");

// ----- CORES POR REGIÃO -----
const cores = {
  "Alentejo":"#d62728",
  "Setúbal":"#1f77b4",
  "Douro":"#2ca02c",
  "Vinho Verde":"#ff7f0e",
  "Lisboa":"#9467bd",
  "Algarve":"#8c564b",
  "Açores":"#e377c2",
  "Madeira":"#7f7f7f"
};

// ----- FUNÇÕES -----
function atualizarListaFornecedores() {
  fornecedoresLista.innerHTML = "";
  const r = regiaoSelect.value;
  const filtrados = dados.filter(d => r==="Todos" || d.regiao===r);
  const nomesUnicos = [...new Set(filtrados.map(d => d.fornecedor))];
  nomesUnicos.forEach(f => {
    const btn = document.createElement("button");
    btn.innerText = f;
    btn.className = "fornecedor";
    btn.onclick = () => focarFornecedor(f);
    fornecedoresLista.appendChild(btn);
  });
  contador.innerText = `Fornecedores visíveis: ${nomesUnicos.length}`;
}

function atualizarMapa() {
  markers.forEach(m => map.removeLayer(m));
  markers = [];
  const r = regiaoSelect.value;
  const filtrados = dados.filter(d => r==="Todos" || d.regiao===r);
  filtrados.forEach(d => {
    const m = L.circleMarker([d.lat, d.lng], {
      radius: 8,
      color: cores[d.regiao] || "#000",
      fillOpacity: 0.8
    }).addTo(map)
      .bindPopup(`<b>${d.fornecedor}</b><br>${d.produto}<br>${d.cidade}<br>${d.regiao}`);
    markers.push(m);
  });
  if(filtrados.length>0){
    const group = L.featureGroup(markers);
    map.fitBounds(group.getBounds().pad(0.3));
  }
}

function focarFornecedor(fornecedorNome) {
  const ponto = dados.find(d => d.fornecedor === fornecedorNome);
  if (ponto) {
    map.setView([ponto.lat, ponto.lng], 8);
    L.popup()
      .setLatLng([ponto.lat, ponto.lng])
      .setContent(`<b>${ponto.fornecedor}</b><br>${ponto.produto}<br>${ponto.cidade}<br>${ponto.regiao}`)
      .openOn(map);
  }
}

// ----- INICIALIZAÇÃO -----
["Todos", ...new Set(dados.map(d=>d.regiao))].forEach(r=>{
  regiaoSelect.add(new Option(r,r));
});

regiaoSelect.onchange = () => {
  atualizarListaFornecedores();
  atualizarMapa();
};

atualizarListaFornecedores();
atualizarMapa();
</script>

</body>
</html>
