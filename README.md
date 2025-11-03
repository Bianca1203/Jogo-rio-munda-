# Jogo-rio-munda-<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jogo do Rio Mundaú</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: linear-gradient(#87ceeb, #3ca1c3);
    font-family: Arial, sans-serif;
    color: white;
    text-align: center;
  }

  canvas {
    background: linear-gradient(#3ca1c3, #1b5e20);
    display: block;
    margin: 0 auto;
    border: 3px solid #004d40;
  }

  #hud {
    margin: 10px;
    font-size: 20px;
  }

  #telaInicial, #mensagemFinal, #mensagemFase {
    position: absolute;
    top: 20%;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.85);
    padding: 30px;
    border-radius: 20px;
    font-size: 22px;
    width: 80%;
    max-width: 600px;
    animation: aparecer 2s ease-in-out;
  }

  #telaInicial {
    display: block;
  }

  #mensagemFinal, #mensagemFase {
    display: none;
  }

  #telaInicial h1 {
    font-size: 36px;
    color: #ffeb3b;
  }

  #telaInicial button {
    background: #4caf50;
    color: white;
    border: none;
    padding: 12px 30px;
    border-radius: 10px;
    font-size: 20px;
    margin-top: 20px;
    cursor: pointer;
    transition: 0.3s;
  }

  #telaInicial button:hover {
    background: #66bb6a;
  }

  #creditos {
    margin-top: 15px;
    font-size: 16px;
    color: #ddd;
  }

  #mensagemFinal img, #mensagemFase img {
    width: 150px;
    border-radius: 50%;
    margin-bottom: 15px;
    animation: flutuar 3s ease-in-out infinite;
  }

  #botaoReiniciar {
    background: #4caf50;
    color: white;
    border: none;
    padding: 10px 25px;
    border-radius: 10px;
    font-size: 18px;
    margin-top: 15px;
    cursor: pointer;
    transition: 0.3s;
  }

  #botaoReiniciar:hover {
    background: #66bb6a;
  }

  @keyframes aparecer {
    from { opacity: 0; transform: translate(-50%, -20%); }
    to { opacity: 1; transform: translate(-50%, 0); }
  }

  @keyframes flutuar {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-10px); }
  }
</style>
</head>
<body>

<div id="telaInicial">
  <h1>🌊 Jogo do Rio Mundaú 🌿</h1>
  <p>Ajude a limpar o Rio Mundaú removendo todos os lixos.<br>
  A cada lixo coletado, você ganha <b>2 pontos</b>!<br><br>
  Cuidado com o tempo, ele passa rápido ⏳</p>
  <button onclick="iniciarJogo()">Iniciar Jogo</button>
  <div id="creditos">
    Criado por <b>Bianca Alexandre</b> 💚<br>
    Professora: <b>👩‍🏫 Profa. do Rio Mundaú</b>
  </div>
</div>

<div id="hud" style="display:none;">
  <span id="fase">Fase: 1</span> | 
  <span id="pontos">Pontos: 0</span> | 
  <span id="tempo">Tempo: 30</span>s
</div>

<canvas id="jogo" width="800" height="500"></canvas>

<!-- Mensagem entre fases -->
<div id="mensagemFase">
  <img id="professoraImgFase" src="">
  <p id="textoFase"></p>
</div>

<!-- Mensagem final -->
<div id="mensagemFinal">
  <img id="professoraImg" src="">
  <p id="textoFinal"></p>
  <button id="botaoReiniciar" onclick="reiniciarJogo()">Jogar Novamente</button>
</div>

<!-- Sons -->
<audio id="somRio" loop>
  <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_7cf0d45f3e.mp3?filename=small-river-ambient-14594.mp3" type="audio/mpeg">
</audio>

<audio id="somClique">
  <source src="https://cdn.pixabay.com/download/audio/2022/03/02/audio_c62bbf2b88.mp3?filename=water-drop-1-8852.mp3" type="audio/mpeg">
</audio>

<script>
const canvas = document.getElementById("jogo");
const ctx = canvas.getContext("2d");

let fase = 1;
let pontos = 0;
let tempoRestante = 30;
let lixos = [];
let jogoAtivo = false;
let intervaloTempo;
let tempoLimite = 30;
let aguardandoMensagem = false;

const mensagensFases = [
  "",
  "Muito bem! Você limpou o primeiro trecho do rio! Vamos continuar!",
  "Excelente! O rio já está mais limpo, mas ainda há muito o que fazer!",
  "Uau! Você está indo muito bem, continue salvando o Rio Mundaú!",
  "Falta pouco, herói! Continue com esse ótimo trabalho!",
  "Essa é a última fase! Mostre que você é um verdadeiro protetor do meio ambiente!"
];

// imagens reais de lixo
const imagens = {
  banana: "https://cdn.pixabay.com/photo/2016/11/21/15/56/banana-peel-1842012_1280.png",
  jornal: "https://cdn.pixabay.com/photo/2013/07/13/13/41/newspaper-161551_1280.png",
  garrafa: "https://cdn.pixabay.com/photo/2013/07/12/15/34/bottle-150590_1280.png",
  plastico: "https://cdn.pixabay.com/photo/2020/05/12/22/55/plastic-bottle-5163930_1280.png"
};

const professoraImg = document.getElementById("professoraImg");
const professoraImgFase = document.getElementById("professoraImgFase");
professoraImg.src = professoraImgFase.src = "https://cdn.pixabay.com/photo/2016/10/03/20/09/teacher-1713766_1280.png";

const tiposLixo = [
  { nome: "casca de banana", img: new Image(), key: "banana" },
  { nome: "jornal", img: new Image(), key: "jornal" },
  { nome: "garrafa de vidro", img: new Image(), key: "garrafa" },
  { nome: "plástico", img: new Image(), key: "plastico" }
];

tiposLixo.forEach(t => t.img.src = imagens[t.key]);

const somRio = document.getElementById("somRio");
const somClique = document.getElementById("somClique");

function tocarSomRio() {
  somRio.volume = 0.3;
  somRio.play().catch(()=>{});
}

function gerarLixos(qtd) {
  lixos = [];
  for (let i = 0; i < qtd; i++) {
    const tipo = tiposLixo[Math.floor(Math.random() * tiposLixo.length)];
    lixos.push({
      x: Math.random() * (canvas.width - 80),
      y: Math.random() * (canvas.height - 100),
      tipo,
      coletado: false
    });
  }
}

function desenharLixos() {
  lixos.forEach(lixo => {
    if (!lixo.coletado) {
      ctx.drawImage(lixo.tipo.img, lixo.x, lixo.y, 60, 60);
    }
  });
}

function atualizarHUD() {
  document.getElementById("fase").textContent = `Fase: ${fase}`;
  document.getElementById("pontos").textContent = `Pontos: ${pontos}`;
  document.getElementById("tempo").textContent = `Tempo: ${tempoRestante}`;
}

function mostrarMensagemFase(texto, callback) {
  aguardandoMensagem = true;
  const msg = document.getElementById("mensagemFase");
  const textoFase = document.getElementById("textoFase");
  textoFase.innerHTML = `👩‍🏫 <b>Professora:</b><br> "${texto}"`;
  msg.style.display = "block";
  setTimeout(() => {
    msg.style.display = "none";
    aguardandoMensagem = false;
    callback();
  }, 3500);
}

function proximaFase() {
  fase++;
  if (fase > 5) {
    finalizarJogo();
    return;
  }
  tempoLimite -= 3;
  tempoRestante = tempoLimite;
  mostrarMensagemFase(mensagensFases[fase], () => {
    gerarLixos(fase * 5 + 5);
  });
}

function finalizarJogo() {
  jogoAtivo = false;
  clearInterval(intervaloTempo);
  somRio.pause();

  const msg = document.getElementById("mensagemFinal");
  msg.style.display = "block";

  const texto = document.getElementById("textoFinal");
  texto.innerHTML = `
    👩‍🏫 <b>Professora:</b><br>
    "Parabéns, você acaba de descontaminar o rio!<br>
    O seu trabalho será recompensado!"<br><br>
    💰 Você ganhou <b>2000 moedas</b>!<br><br>
    🌊 Pontuação final: ${pontos}
  `;
}

canvas.addEventListener("click", (e) => {
  if (!jogoAtivo || aguardandoMensagem) return;
  tocarSomRio(); 
  const rect = canvas.getBoundingClientRect();
  const xClick = e.clientX - rect.left;
  const yClick = e.clientY - rect.top;

  lixos.forEach(lixo => {
    if (!lixo.coletado &&
        xClick > lixo.x && xClick < lixo.x + 60 &&
        yClick > lixo.y && yClick < lixo.y + 60) {
      lixo.coletado = true;
      somClique.currentTime = 0;
      somClique.play();
      pontos += 2;
      atualizarHUD();
      if (lixos.every(l => l.coletado)) {
        proximaFase();
      }
    }
  });
});

function iniciarTempo() {
  intervaloTempo = setInterval(() => {
    if (tempoRestante > 0) {
      tempoRestante--;
      atualizarHUD();
    } else {
      clearInterval(intervaloTempo);
      alert("⏰ O tempo acabou! Tente novamente!");
      reiniciarFase();
    }
  }, 1000);
}

function reiniciarFase() {
  tempoRestante = tempoLimite;
  gerarLixos(fase * 5 + 5);
  atualizarHUD();
  iniciarTempo();
}

function loopJogo() {
  if (!jogoAtivo) return;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  desenharLixos();
  requestAnimationFrame(loopJogo);
}

function iniciarJogo() {
  document.getElementById("telaInicial").style.display = "none";
  document.getElementById("hud").style.display = "block";
  fase = 1;
  pontos = 0;
  tempoLimite = 30;
  tempoRestante = tempoLimite;
  jogoAtivo = true;
  document.getElementById("mensagemFinal").style.display = "none";
  gerarLixos(10);
  atualizarHUD();
  iniciarTempo();
  tocarSomRio();
  loopJogo();
}

function reiniciarJogo() {
  iniciarJogo();
}
</script>

</body>
</html>
