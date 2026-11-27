<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Calculadora</title>
<style>
  :root { --fundo:#f2f0eb; --caixa:#fff; --tecla:#e6e3dc; --op:#c64b82; --texto:#1d1d1b; }
  @media (prefers-color-scheme: dark) {
    :root { --fundo:#16191a; --caixa:#c9879a; --tecla:#140f24; --op:#321c56; --texto:#eceae4; }
  }
  * { box-sizing: border-box; }
  body { margin:0; min-height:100vh; display:grid; place-items:center;
         background:var(--fundo); color:var(--texto); font-family: system-ui, sans-serif; }
  .calc { width:min(320px, 92vw); background:var(--caixa); padding:16px; border-radius:16px; }
  .visor { text-align:right; padding:12px 8px 16px; min-height:84px; }
  .historico { font-size:14px; opacity:.6; min-height:20px; }
  .atual { font-size:40px; font-weight:600; overflow-x:auto; white-space:nowrap; }
  .teclas { display:grid; grid-template-columns:repeat(4,1fr); gap:8px; }
  button { font-size:20px; padding:16px 0; border:0; border-radius:10px; cursor:pointer;
           background:var(--tecla); color:var(--texto); }
  button:focus-visible { outline:3px solid var(--op); }
  button.op { background:var(--op); color:#fff; }
  button.largo { grid-column: span 2; }
</style>
</head>
<body>
<div class="calc">
  <div class="visor">
    <div class="historico" id="historico"></div>
    <div class="atual" id="atual">0</div>
  </div>
  <div class="teclas" id="teclas">
    <button data-acao="limpar">C</button>
    <button data-acao="apagar">⌫</button>
    <button class="op" data-op="%">%</button>
    <button class="op" data-op="/">÷</button>

    <button data-num="7">7</button>
    <button data-num="8">8</button>
    <button data-num="9">9</button>
    <button class="op" data-op="*">×</button>

    <button data-num="4">4</button>
    <button data-num="5">5</button>
    <button data-num="6">6</button>
    <button class="op" data-op="-">−</button>

    <button data-num="1">1</button>
    <button data-num="2">2</button>
    <button data-num="3">3</button>
    <button class="op" data-op="+">+</button>

    <button class="largo" data-num="0">0</button>
    <button data-num=".">,</button>
    <button class="op" data-acao="igual">=</button>
  </div>
</div>

<script>
  // ESTADO da calculadora: três variáveis resolvem tudo
  let atual = "0";        // número que está sendo digitado (como texto)
  let anterior = null;    // primeiro número da conta
  let operador = null;    // + - * / %
  let recomecar = false;  // true logo após apertar "=": o próximo número começa do zero

  const visorAtual = document.getElementById("atual");
  const visorHist = document.getElementById("historico");

  // Função que faz a conta de fato kkkkk
  function calcular(a, b, op) {
    switch (op) {
      case "+": return a + b;
      case "-": return a - b;
      case "*": return a * b;
      case "/": return b === 0 ? NaN : a / b;  // divisão por zero vira erro!!!
      case "%": return a % b;
    }
  }

  // Formata para exibir (vírgula decimal, evita 0.1 + 0.2 = 0.30000000000000004)
  function formatar(texto) {
    const n = Number(texto);
    if (!Number.isFinite(n)) return "Erro";
    return String(parseFloat(n.toFixed(10))).replace(".", ",");
  }

  function atualizarTela() {
    visorAtual.textContent = atual === "Erro" ? "Erro" : atual.replace(".", ",");
    const simbolo = { "*": "×", "/": "÷", "-": "−" }[operador] || operador;
    visorHist.textContent = operador ? `${formatar(anterior)} ${simbolo}` : "";
  }

  function digitar(valor) {
    if (atual === "Erro" || recomecar) { atual = "0"; recomecar = false; }
    if (valor === "." && atual.includes(".")) return;      // só um ponto decimal
    atual = (atual === "0" && valor !== ".") ? valor : atual + valor;
  }

  function escolherOperador(op) {
    if (atual === "Erro") return;
    // Se já existe uma conta pendente, resolve ela antes (ex.: 2 + 3 + ...)
    if (operador && !recomecar) resolver();
    anterior = atual;
    operador = op;
    recomecar = true;
  }

  function resolver() {
    if (!operador) return;
    const resultado = calcular(Number(anterior), Number(atual), operador);
    atual = Number.isFinite(resultado) ? String(parseFloat(resultado.toFixed(10))) : "Erro";
    operador = null;
    anterior = null;
    recomecar = true;
  }

  function limpar() { atual = "0"; anterior = null; operador = null; recomecar = false; }
  function apagar() {
    if (atual === "Erro" || recomecar) return limpar();
    atual = atual.length > 1 ? atual.slice(0, -1) : "0";
  }

  // Um único listener para todos os botões (event delegation)
  document.getElementById("teclas").addEventListener("click", (e) => {
    const b = e.target.closest("button");
    if (!b) return;
    if (b.dataset.num !== undefined) digitar(b.dataset.num);
    else if (b.dataset.op) escolherOperador(b.dataset.op);
    else if (b.dataset.acao === "igual") resolver();
    else if (b.dataset.acao === "limpar") limpar();
    else if (b.dataset.acao === "apagar") apagar();
    atualizarTela();
  });

  // Bônus: usar o teclado do computador
  document.addEventListener("keydown", (e) => {
    if (/^[0-9.]$/.test(e.key)) digitar(e.key);
    else if (e.key === ",") digitar(".");
    else if ("+-*/%".includes(e.key)) escolherOperador(e.key);
    else if (e.key === "Enter" || e.key === "=") { e.preventDefault(); resolver(); }
    else if (e.key === "Backspace") apagar();
    else if (e.key === "Escape") limpar();
    else return;
    atualizarTela();
  });

  atualizarTela();
</script>
</body>
</html>
