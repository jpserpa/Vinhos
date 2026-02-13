
<html lang="pt">
<head>
<meta charset="UTF-8">
<title>Mapa Interativo - Fornecedor e Vinhos</title>
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
// ----- DADOS -----
const dados = [
  {regiao:"Alentejo", fornecedor:"Tiago Cabaço", cidade:"Évora", produto:"Blog Tour", lat:38.57, lng:-7.9},
  {regiao:"Alentejo", fornecedor:"Tiago Cabaço", cidade:"Évora", produto:"Reserva Especial", lat:38.57, lng:-7.9},
  {regiao:"Alentejo", fornecedor:"Herdade do Esporão", cidade:"Reguengos", produto:"Monte Velho", lat:38.30, lng:-7.50},
  {regiao:"Herdade do Esporão", fornecedor:"Herdade do Esporão", cidade:"Reguengos", produto:"Reserva", lat:38.30, lng:-7.50},
  {regiao:"Setúbal", fornecedor:"José Maria da Fonseca", cidade:"Setúbal", produto:"Periquita", lat:38.52, lng:-8.89},
  {regiao:"Setúbal", fornecedor:"José Maria da Fonseca", cidade:"Setúbal", produto:"Lágrima", lat:38.52, lng:-8.89},
  {regiao:"Douro", fornecedor:"Graham’s", cidade:"Vila Nova de Gaia", produto:"Six Grapes", lat:41.14, lng:-8.61},
  {regiao:"Douro", fornecedor:"Graham’s", cidade:"Vila Nova de Gaia", produto:"Late Bottled", lat:41.14, lng:-8.61}
];

// ----- CORES -----
const cores = { "Alentejo":"#d62728", "Setúbal":"#1f77b4", "Douro":"#2ca02c" };

// ----- MAPA -----
const map = L.map('map').setView([39.5, -8],6);
L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png',{ attribution:'&copy; OpenStreetMap &copy; CARTO', subdomains:'abcd', maxZoom:19 }).addTo(map);

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

// ----- FILTRO -----
function filtrarPorRegiao(regiao){ return regiao==="Todos"? dados : dados.filter(d=>d.regiao===regiao); }

// ----- ATUALIZAR LISTA -----
function atualizarLista(filtrados){
  const lista = document.getElementById("fornecedoresLista"); lista.innerHTML="";
  const nomes = [...new Set(filtrados.map(d=>d.fornecedor))];
  nomes.forEach(f=>{
    const btn=document.createElement("button"); btn.innerText=f; btn.className="fornecedor";
    btn.onclick=()=> mostrarVinhos(f);
    lista.appendChild(btn);
    // adicionar lista de vinhos do fornecedor
    const ul=document.createElement("ul"); ul.className="vinhos"; lista.appendChild(ul);
  });
  document.getElementById("contador").innerText=`Fornecedores visíveis: ${nomes.length}`;
}

// ----- ATUALIZAR MAPA -----
function atualizarMapa(filtrados){
  Object.values(markers).forEach(m=>map.removeLayer(m));
  const usados = [...new Set(filtrados.map(d=>d.fornecedor))];
  usados.forEach(f=>markers[f].addTo(map));
  if(filtrados.length>0){ const group=L.featureGroup(filtrados.map(d=>markers[d.fornecedor])); map.fitBounds(group.getBounds().pad(0.3)); }
}

// ----- MOSTRAR VINHOS -----
function mostrarVinhos(fornecedor){
  // focar no marcador
  const ponto = dados.find(d=>d.fornecedor===fornecedor);
  if(ponto){ map.setView([ponto.lat,ponto.lng],8,{animate:true}); markers[fornecedor].openPopup(); }
  // atualizar a lista de vinhos
  const btns=document.querySelectorAll("#fornecedoresLista button.fornecedor");
  btns.forEach((btn,i)=>{
    const ul = btn.nextElementSibling;
    if(btn.innerText===fornecedor){
      ul.innerHTML="";
      dados.filter(d=>d.fornecedor===fornecedor).forEach(d=>{ const li=document.createElement("li"); li.innerText=`${d.produto} (${d.cidade})`; ul.appendChild(li); });
    } else { btn.nextElementSibling.innerHTML=""; }
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
