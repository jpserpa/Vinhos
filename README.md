
<html lang="pt">
<head>
<meta charset="UTF-8">
<title>Mapa Interativo dos Vinhos - Profissional</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<style>
body, html { margin:0; padding:0; height:100%; font-family: Arial; }
#container {
  display: grid;
  grid-template-columns: 350px 1fr;
  height: 100%;
}
#sidebar {
  padding: 15px;
  background: #f4f4f4;
  overflow-y: auto;
  box-shadow: 2px 0 5px rgba(0,0,0,0.1);
}
#sidebar h2 { margin-top:0; font-size:18px; }
#sidebar label { font-weight:bold; margin-top:10px; display:block; }
#map { width:100%; height:100%; }
button.fornecedor {
  width: 100%;
  padding: 14px;
  margin-bottom: 6px;
  cursor: pointer;
  text-align: left;
  font-size: 14px;
  border: none;
  border-radius: 4px;
  background-color: #fff;
  transition: 0.2s;
}
button.fornecedor:hover { background-color: #e0e0e0; }
#contador { margin-top: 10px; font-style: italic; color: #333; }
.sidebar-toggle {
  display:none;
  padding:10px;
  cursor:pointer;
  background:#ddd;
  margin-bottom:10px;
  border-radius:4px;
  text-align:center;
}
@media(max-width: 700px){
  #container { grid-template-columns: 1fr; }
  #sidebar { position: absolute; z-index:1000; width:70%; height:100%; transform: translateX(-100%); transition: transform 0.3s; }
  #sidebar.show { transform: translateX(0); }
  .sidebar-toggle { display:block; }
}
</style>
</head>
<body>

<div class="sidebar-toggle" onclick="toggleSidebar()">☰ Abrir/Fechar Sidebar</div>

<div id="container">
  <div id="sidebar">
    <h2>Vinhos de Portugal</h2>
    <label>Região</label>
    <select id="regiao"></select>
    <div id="fornecedoresLista"></div>
    <p id="contador"></p>
  </div>
  <div id="map"></div>
</div>

<script>
// ----- DADOS (simulado JSON externo) -----
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

// ----- CORES POR REGIÃO -----
const cores = {
  "Alentejo":"#d62728", "Setúbal":"#1f77b4", "Douro":"#2ca02c", "Vinho Verde":"#ff7f0e",
  "Lisboa":"#9467bd", "Algarve":"#8c564b", "Açores":"#e377c2", "Madeira":"#7f7f7f"
};

// ----- MAPA -----
const map = L.map('map').setView([39.5, -8], 6);
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',{
  attribution:'&copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> &copy; <a href="https://carto.com/">CARTO</a>',
  subdomains:'abcd', maxZoom:19
}).addTo(map);

// ----- MARCADORES REUTILIZÁVEIS -----
let markers = {};
function criarMarcadores() {
  dados.forEach(d => {
    const m = L.circleMarker([d.lat,d.lng],{
      radius:8,
      color: cores[d.regiao] || "#000",
      fillOpacity:0.8
    }).bindPopup(`<b>${d.fornecedor}</b><br>${d.produto}<br>${d.cidade}<br>${d.regiao}`);
    markers[d.fornecedor] = m;
    m.addTo(map);
  });
}

// ----- FILTRAR -----
function filtrarPorRegiao(regiao){
  return regiao==="Todos"? dados : dados.filter(d=>d.regiao===regiao);
}

// ----- ATUALIZAR LISTA -----
function atualizarLista(filtrados){
  const lista = document.getElementById("fornecedoresLista");
  lista.innerHTML="";
  const nomes = [...new Set(filtrados.map(d=>d.fornecedor))];
  nomes.forEach(f=>{
    const btn = document.createElement("button");
    btn.innerText = f;
    btn.className="fornecedor";
    btn.onclick=()=> focarFornecedor(f);
    lista.appendChild(btn);
  });
  document.getElementById("contador").innerText=`Fornecedores visíveis: ${nomes.length}`;
}

// ----- ATUALIZAR MAPA -----
function atualizarMapa(filtrados){
  Object.values(markers).forEach(m=>map.removeLayer(m));
  filtrados.forEach(d=> markers[d.fornecedor].addTo(map));
  if(filtrados.length>0){
    const group = L.featureGroup(filtrados.map(d=>markers[d.fornecedor]));
    map.fitBounds(group.getBounds().pad(0.3));
  }
}

// ----- FOCO NO FORNECEDOR -----
function focarFornecedor(fornecedor){
  const ponto = dados.find(d=>d.fornecedor===fornecedor);
  if(ponto){
    const currentZoom = map.getZoom();
    const targetZoom = Math.max(currentZoom, 8);
    map.setView([ponto.lat,ponto.lng], targetZoom, {animate:true});
    markers[fornecedor].openPopup();
  }
}

// ----- INICIALIZAÇÃO -----
function inicializarMapa(){
  criarMarcadores();
  const regSelect = document.getElementById("regiao");
  ["Todos", ...new Set(dados.map(d=>d.regiao))].forEach(r=>{
    regSelect.add(new Option(r,r));
  });
  function atualizarTudo(){
    const filtrados = filtrarPorRegiao(regSelect.value);
    atualizarLista(filtrados);
    atualizarMapa(filtrados);
  }
  regSelect.onchange = atualizarTudo;
  atualizarTudo();
}

// ----- SIDEBAR COLLAPSIBLE MOBILE -----
function toggleSidebar(){
  document.getElementById("sidebar").classList.toggle("show");
}

// ----- INICIALIZAÇÃO FINAL -----
inicializarMapa();
</script>

</body>
</html>
