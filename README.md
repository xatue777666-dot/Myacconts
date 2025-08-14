<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Compra Coletiva de Assinaturas</title>
<style>
  :root{--ink:#e5e7eb;--muted:#9ca3af;--card:#0b1223;--line:#1f2937;--brand:#2563eb;--danger:#ef4444;--ok:#22c55e}
  *{box-sizing:border-box}
  body{margin:0;font-family:system-ui,Segoe UI,Roboto,Arial;background:linear-gradient(180deg,#0b1223,#0f172a);color:var(--ink)}
  header{position:sticky;top:0;background:rgba(11,18,35,.9);backdrop-filter:blur(8px);border-bottom:1px solid var(--line);padding:14px 16px}
  header h1{margin:0;font-size:18px;letter-spacing:.3px}
  main{max-width:820px;margin:18px auto;padding:0 16px;display:grid;gap:16px}
  .card{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:16px;box-shadow:0 10px 30px rgba(0,0,0,.25)}
  .title{margin:0 0 8px;font-size:18px;font-weight:700}
  .sub{margin:0 0 10px;color:var(--muted);font-size:14px}
  .grid{display:grid;gap:10px}
  .grid-2{grid-template-columns:1fr 1fr}
  @media (max-width:640px){.grid-2{grid-template-columns:1fr}}
  input,button{width:100%;padding:12px 14px;border-radius:12px}
  input{border:1px solid var(--line);background:#0f172a;color:var(--ink)}
  input::placeholder{color:#94a3b8}
  button{border:none;background:linear-gradient(90deg,#2563eb,#1d4ed8);color:white;font-weight:700}
  button.secondary{background:#0f172a;border:1px solid var(--line);color:var(--ink);font-weight:600}
  button:disabled{opacity:.6;filter:grayscale(40%)}
  .group{display:grid;gap:10px;border:1px solid var(--line);border-radius:14px;padding:12px;background:#0e1530}
  .row{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
  .pill{font-size:12px;padding:4px 8px;border:1px solid var(--line);border-radius:999px;color:#cbd5e1}
  .ok{border-color:var(--ok);color:#bbf7d0}
  .bad{border-color:var(--danger);color:#fecaca}
  .note{color:var(--muted);font-size:12px}
  .sep{height:1px;background:var(--line);margin:6px 0}
</style>
</head>
<body>
<header><h1>Compra Coletiva de Assinaturas</h1></header>
<main>
  <!-- Criar grupo -->
  <section class="card">
    <h2 class="title">Criar um novo grupo</h2>
    <p class="sub">Anuncie vagas da sua assinatura e divida o custo com outras pessoas.</p>
    <div class="grid grid-2">
      <input id="servico" placeholder="Serviço (ex.: Spotify Premium)" />
      <input id="preco" type="number" step="0.01" placeholder="Preço original (R$)" />
      <input id="max" type="number" placeholder="Máximo de pessoas (ex.: 6)" />
      <input id="nome" placeholder="Seu nome" />
      <input id="whatsapp" inputmode="tel" placeholder="Seu WhatsApp (com DDD)" />
      <button id="btnCriar">Criar grupo</button>
    </div>
    <p class="note">Dica: o preço por pessoa é calculado automaticamente (preço ÷ máximo).</p>
  </section>

  <!-- Lista -->
  <section class="card">
    <div class="row" style="justify-content:space-between;align-items:center;">
      <h2 class="title" style="margin:0">Grupos disponíveis</h2>
      <button class="secondary" id="btnReload" title="Recarregar">Recarregar</button>
    </div>
    <div id="lista"></div>
    <p id="status" class="note"></p>
  </section>
</main>

<script>
/** CONFIGURAÇÃO **/
const API_URL = "https://sheetdb.io/api/v1/u3yce7qsety2q"; // seu endpoint do SheetDB

/** UTILITÁRIOS **/
const fmtBRL = v => new Intl.NumberFormat("pt-BR",{style:"currency",currency:"BRL"}).format(Number(v||0));
const el = id => document.getElementById(id);
const toast = (msg) => { el("status").textContent = msg; setTimeout(()=>el("status").textContent="", 4500); };
const onlyDigits = s => (s||"").replace(/\D+/g,"");

/** ESTADO **/
let grupos = [];

/** CARREGAR LISTA **/
async function carregarGrupos(){
  el("lista").innerHTML = "Carregando...";
  try{
    const res = await fetch(API_URL);
    const data = await res.json();
    grupos = (Array.isArray(data)?data:[]).map(r=>({
      id: r.id,
      servico: r.servico,
      preco: Number(r.preco||0),
      max_pessoas: Number(r.max_pessoas||0),
      pessoas_atual: Number(r.pessoas_atual||0),
      nome: r.nome || "",
      whatsapp: r.whatsapp || ""
    })).sort((a,b)=>a.servico.localeCompare(b.servico));
    renderizar();
  }catch(e){
    el("lista").innerHTML = "";
    toast("Erro ao carregar: verifique seu API_URL no código.");
  }
}

function renderizar(){
  if(!grupos.length){ el("lista").innerHTML = "<p class='note'>Nenhum grupo ainda. Crie o primeiro acima.</p>"; return; }
  const html = grupos.map(g=>{
    const vagas = Math.max(g.max_pessoas - g.pessoas_atual, 0);
    const cheio = vagas === 0;
    const valorPorPessoa = g.max_pessoas>0 ? g.preco / g.max_pessoas : 0;
    const contato = g.whatsapp ? `https://wa.me/55${onlyDigits(g.whatsapp)}` : null;
    return `
      <div class="group">
        <div class="row" style="justify-content:space-between">
          <div><strong>${g.servico}</strong></div>
          <div class="pill ${cheio?'bad':'ok'}">${cheio?'Grupo cheio':'Vagas: '+vagas}</div>
        </div>
        <div class="sep"></div>
        <div class="row">
          <div class="pill">Preço original: ${fmtBRL(g.preco)}</div>
          <div class="pill">Máx.: ${g.max_pessoas}</div>
          <div class="pill">Entraram: ${g.pessoas_atual}</div>
          <div class="pill">Por pessoa: ${fmtBRL(valorPorPessoa)}</div>
        </div>
        <div class="row" style="gap:10px; margin-top:6px">
          <button ${cheio?'disabled':''} onclick="entrarGrupo('${g.id}', ${g.pessoas_atual}, ${g.max_pessoas})">Entrar no grupo</button>
          ${contato ? `<a class="pill" style="text-decoration:none" href="${contato}" target="_blank" rel="noopener">Falar com criador</a>`:''}
        </div>
        <p class="note">Criado por: ${g.nome ? g.nome + (g.whatsapp ? ' • ' + g.whatsapp : '') : 'Não informado'}</p>
        <p class="note">ID do grupo: ${g.id}</p>
      </div>
    `;
  }).join("");
  el("lista").innerHTML = html;
}

/** CRIAR GRUPO **/
async function criarGrupo(){
  const servico = el("servico").value.trim();
  const preco = Number(el("preco").value);
  const max = Number(el("max").value);
  const nome = el("nome").value.trim();
  const whatsapp = onlyDigits(el("whatsapp").value);
  if(!servico || !preco || !max || max<2 || !nome || !whatsapp){ 
    toast("Preencha todos os campos (máximo ≥ 2) e seus dados de contato."); 
    return; 
  }

  const id = Date.now().toString(); // id único simples
  const payload = { data: [ { id, servico, preco, max_pessoas:max, pessoas_atual: 1, nome, whatsapp } ] };

  el("btnCriar").disabled = true;
  try{
    const r = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    });
    if(!r.ok) throw new Error("POST falhou");
    toast("Grupo criado com sucesso!");
    el("servico").value = ""; el("preco").value = ""; el("max").value = "";
    el("nome").value = ""; el("whatsapp").value = "";
    await carregarGrupos();
  }catch(e){
    toast("Erro ao criar. Confira o API_URL e as colunas da planilha (inclua 'nome' e 'whatsapp').");
  }finally{
    el("btnCriar").disabled = false;
  }
}

/** ENTRAR NO GRUPO (incremento pessoas_atual + captura de contato) **/
async function entrarGrupo(id, atual, max){
  if(atual >= max){ toast("Grupo já está cheio."); return; }
  const nome = prompt("Seu nome:");
  const whatsapp = onlyDigits(prompt("Seu WhatsApp (com DDD):"));
  if(!nome || !whatsapp){ toast("Nome e WhatsApp são obrigatórios."); return; }
  const novo = atual + 1;

  try{
    const r = await fetch(`${API_URL}/keys/id/${encodeURIComponent(id)}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ data: { pessoas_atual: novo, nome, whatsapp } })
    });
    if(!r.ok) throw new Error("PATCH falhou");
    toast("Você entrou no grupo!");
    await carregarGrupos();
  }catch(e){
    toast("Erro ao entrar no grupo. Verifique permissões do SheetDB e a coluna 'id'.");
  }
}

/** LISTENERS **/
el("btnCriar").addEventListener("click", criarGrupo);
el("btnReload").addEventListener("click", carregarGrupos);

/** START **/
carregarGrupos();
</script>
</body>
</html>