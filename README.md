
<html lang="pt">
<head>
<meta charset="UTF-8">
<title>Mapa Interativo de Vinhos - Portugal</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<style>
body, html { margin:0; padding:0; height:100%; font-family: Arial; }
#container { display:grid; grid-template-columns:350px 1fr; height:100%; }
#sidebar { padding:15px; background:#f4f4f4; overflow-y:auto; box-shadow:2px 0 5px rgba(0,0,0,0.1); }
#sidebar h2 { margin-top:0; font-size:18px; }
#sidebar label { font-weight:bold; margin-top:10px; display:block; }
#map { width:100%; height:100%; }
button.fornecedor { width:100%; padding:14px; margin-bottom:6px; cursor:pointer; text-align:left; font-size:14px; border:none; border-radius:4px; background-color:#fff; transition:0.2s; }
button.fornecedor:hover { background-color:#e0e0e0; }
ul.vinhos { margin:5px 0 10px 15px; padding:0; list-style-type:disc; color:#333; font-size:13px; }
#contador { margin-top:10px; font-style:italic; color:#333; }
.sidebar-toggle { display:none; padding:10px; cursor:pointer; background:#ddd; margin-bottom:10px; border-radius:4px; text-align:center; }
@media(max-width:700px){ #container { grid-template-columns:1fr; } #sidebar { position:absolute; z-index:1000; width:70%; height:100%; transform:translateX(-100%); transition:transform 0.3s; } #sidebar.show { transform:translateX(0); } .sidebar-toggle { display:block; } }
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
// ----- DADOS COMPLETOS -----
const dados = [
  // ALENTEJO
  {regiao:"Alentejo", fornecedor:"Tiago Cabaço", cidade:"Évora", produto:"Blog Tour", lat:38.57, lng:-7.9},
  {regiao:"Alentejo", fornecedor:"Tiago Cabaço", cidade:"Évora", produto:"Reserva Especial", lat:38.57, lng:-7.9},
  {regiao:"Alentejo", fornecedor:"Herdade do Esporão", cidade:"Reguengos", produto:"Monte Velho", lat:38.30, lng:-7.50},
  {regiao:"Alentejo", fornecedor:"Herdade do Esporão", cidade:"Reguengos", produto:"Reserva", lat:38.30, lng:-7.50},
  {regiao:"Alentejo", fornecedor:"Cortes de Cima", cidade:"Vila Nova de Milfontes", produto:"Cortes Branco", lat:37.44, lng:-8.79},
  {regiao:"Alentejo", fornecedor:"Cortes de Cima", cidade:"Vila Nova de Milfontes", produto:"Cortes Tinto", lat:37.44, lng:-8.79},
  {regiao:"Alentejo", fornecedor:"Adega Mayor", cidade:"Redondo", produto:"Redondo Reserva", lat:38.74, lng:-7.68},

  // SETÚBAL
  {regiao:"Setúbal", fornecedor:"José Maria da Fonseca", cidade:"Setúbal", produto:"Periquita", lat:38.52, lng:-8.89},
  {regiao:"Setúbal", fornecedor:"José Maria da Fonseca", cidade:"Setúbal", produto:"Lágrima", lat:38.52, lng:-8.89},
  {regiao:"Setúbal", fornecedor:"Bacalhôa", cidade:"Azeitão", produto:"Catarina", lat:38.55, lng:-8.88},
  {regiao:"Setúbal", fornecedor:"Bacalhôa", cidade:"Azeitão", produto:"Vinha da Bacalhôa", lat:38.55, lng:-8.88},
  {regiao:"Setúbal", fornecedor:"Herdade do Mouchão", cidade:"Montemor-o-Novo", produto:"Mouchão Reserva", lat:38.78, lng:-8.50},

  // DOURO
  {regiao:"Douro", fornecedor:"Graham’s", cidade:"Vila Nova de Gaia", produto:"Six Grapes", lat:41.14, lng:-8.61},
  {regiao:"Douro", fornecedor:"Graham’s", cidade:"Vila Nova de Gaia", produto:"Late Bottled", lat:41.14, lng:-8.61},
  {regiao:"Douro", fornecedor:"Quinta do Crasto", cidade:"Sabrosa", produto:"Crasto Tinto", lat:41.19, lng:-7.71},
  {regiao:"Douro", fornecedor:"Quinta do Crasto", cidade:"Sabrosa", produto:"Crasto Branco", lat:41.19, lng:-7.71},
  {regiao:"Douro", fornecedor:"Quinta do Noval", cidade:"Pinhão", produto:"Noval Black", lat:41.16, lng:-7.52},

  // VINHO VERDE
  {regiao:"Vinho Verde", fornecedor:"Aveleda", cidade:"Penafiel", produto:"Casal Garcia", lat:41.21, lng:-8.28},
  {regiao:"Vinho Verde", fornecedor:"Aveleda", cidade:"Penafiel", produto:"Vinho Verde Branco", lat:41.21, lng:-8.28},
  {regiao:"Vinho Verde", fornecedor:"Quinta de Azevedo", cidade:"Amarante", produto:"Azevedo Branco", lat:41.27, lng:-8.01},
  {regiao:"Vinho Verde", fornecedor:"Quinta de Azevedo", cidade:"Amarante", produto:"Azevedo Tinto", lat:41.27, lng:-8.01},

  // LISBOA
  {regiao:"Lisboa", fornecedor:"Casa Santos Lima", cidade:"Lisboa", produto:"Colossal", lat:39.05, lng:-9.00},
  {regiao:"Lisboa", fornecedor:"Casa Santos Lima", cidade:"Lisboa", produto:"Reserva", lat:39.05, lng:-9.00},
  {regiao:"Lisboa", fornecedor:"Adega Mãe", cidade:"Torres Vedras", produto:"Mãe Tinto", lat:39.08, lng:-9.26},

  // ALGARVE
  {regiao:"Algarve", fornecedor:"Quinta dos Vales", cidade:"Lagoa", produto:"Grace Vineyard", lat:37.12, lng:-8.45},
  {regiao:"Algarve", fornecedor:"Quinta dos Vales", cidade:"Lagoa", produto:"Tinto Reserva", lat:37.12, lng:-8.45},
  {regiao:"Algarve", fornecedor:"Adega do Cantor", cidade:"Portimão", produto:"Cantor Tinto", lat:37.13, lng:-8.54},

  // AÇORES
  {regiao:"Açores", fornecedor:"Azores Wine Company", cidade:"Ponta Delgada", produto:"Terrantez", lat:38.47, lng:-28.40},
  {regiao:"Açores", fornecedor:"Azores Wine Company", cidade:"Ponta Delgada", produto:"Verdelho", lat:38.47, lng:-28.40},

  // MADEIRA
  {regiao:"Madeira", fornecedor:"Henriques & Henriques", cidade:"Funchal", produto:"10 Years", lat:32.65, lng:-16.90},
  {regiao:"Madeira", fornecedor:"Henriques & Henriques", cidade:"Funchal", produto:"15 Years", lat:32.65, lng:-16.90},
  {regiao:"Madeira", fornecedor:"Blandy's", cidade:"Funchal", produto:"Malvasia", lat:32.65, lng:-16.90}
];

// ----- CORES POR REGIÃO -----
const cores = { "Alentejo":"#d62728", "Setúbal":"#1f77b4", "Douro":"#2ca02c", "Vinho Verde":"#ff7f0e", "Lisboa":"#9467bd", "Algarve":"#8c564b", "Açores":"#e377c2", "Madeira":"#7f7f7f" };

// ----- MAPA -----
const map = L.map('map');
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',{
  attribution:'&copy; OpenStreetMap &copy; CARTO', subdomains:'abcd', maxZoom:19
}).addTo(map);

// Limites de Portugal para evitar barras cinza
const portugalBounds = [[32.5, -9.5],[42.2, -6.0]];
map.setMaxBounds(portugalBounds);
map.fitBounds(portugalBounds);

// ----- MARCADORES -----
let markers = {};
function criarMarcadores(){
  dados.forEach(d=>{
    if(!markers[d.fornecedor]){
      markers[d.fornecedor]=L.circleMarker([d.lat,d.lng],{radius:8,color:cores[d.regiao]||"#000",fillOpacity:0.8})
        .bindPopup(`<b>${d.fornecedor}</b>`);
    }
  });
}

// ----- FILTRO POR REGIÃO -----
function filtrarPorRegiao(regiao){ return regiao==="Todos"? dados : dados.filter(d=>d.regiao===regiao); }

// ----- ATUALIZAR LISTA FORNECEDORES -----
function atualizarLista(filtrados){
  const lista=document.getElementById("fornecedoresLista"); lista.innerHTML="";
  const nomes=[...new Set(filtrados.map(d=>d.fornecedor))];
  nomes.forEach(f=>{
    const btn=document.createElement("button"); btn.innerText=f; btn.className="fornecedor";
    btn.onclick=()=> mostrarVinhos(f);
    lista.appendChild(btn);
    const ul=document.createElement("ul"); ul.className="vinhos"; lista.appendChild(ul);
  });
  document.getElementById("contador").innerText=`Fornecedores visíveis: ${nomes.length}`;
}

// ----- ATUALIZAR MAPA -----
function atualizarMapa(filtrados){
  Object.values(markers).forEach(m=>map.removeLayer(m));
  const usados=[...new Set(filtrados.map(d=>d.fornecedor))];
  usados.forEach(f=>markers[f].addTo(map));
}

// ----- MOSTRAR VINHOS POR FORNECEDOR -----
function mostrarVinhos(fornecedor){
  const ponto=dados.find(d=>d.fornecedor===fornecedor);
  if(ponto){ map.setView([ponto.lat,ponto.lng],8,{animate:true}); markers[fornecedor].openPopup(); }
  const btns=document.querySelectorAll("#fornecedoresLista button.fornecedor");
  btns.forEach((btn)=>{
    const ul=btn.nextElementSibling;
    if(btn.innerText===fornecedor){
      ul.innerHTML="";
      dados.filter(d=>d.fornecedor===fornecedor).forEach(d=>{ const li=document.createElement("li"); li.innerText=`${d.produto} (${d.cidade})`; ul.appendChild(li); });
    } else { ul.innerHTML=""; }
  });
}

// ----- INICIALIZAÇÃO -----
function inicializarMapa(){
  criarMarcadores();
  const regSelect=document.getElementById("regiao");
  ["Todos", ...new Set(dados.map(d=>d.regiao))].forEach(r=>regSelect.add(new Option(r,r)));
  function atualizarTudo(){ const filtrados=filtrarPorRegiao(regSelect.value); atualizarLista(filtrados); atualizarMapa(filtrados); }
  regSelect.onchange=atualizarTudo;
  atualizarTudo();
}

// ----- SIDEBAR COLLAPSIBLE -----
function toggleSidebar(){ document.getElementById("sidebar").classList.toggle("show"); }

inicializarMapa();
</script>
</body>
</html>
