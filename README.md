<!DOCTYPE html><html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BRUXO SENSI</title>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>
<style>
body{font-family:Arial;background:#0f0f0f;color:#fff;padding:20px;margin:0}
.box{max-width:800px;margin:auto;background:#1b1b1b;padding:20px;border-radius:20px}
input,button{width:100%;padding:12px;margin-top:10px;border:none;border-radius:10px}
button{background:#ff3c00;color:#fff;font-weight:bold;cursor:pointer}
.card{background:#222;padding:15px;margin-top:10px;border-radius:15px}
</style>
</head>
<body>
<div class="box">
<h1>🔥 BRUXO SENSI</h1><h2>Login</h2>
<input id="email" placeholder="Email">
<input id="senha" type="password" placeholder="Senha">
<button onclick="registrar()">Criar Conta</button>
<button onclick="login()">Entrar</button><hr><h2>Gerar Sensi</h2>
<input id="nomeSensi" placeholder="Nome da sensi">
<input id="celular" placeholder="Seu celular">
<button onclick="gerarSensi()">Gerar</button>
<button onclick="publicarSensi()">Publicar</button><div id="resultado"></div><hr>
<h2>📢 Sensis Publicadas</h2>
<div id="lista"></div><hr>
<h2>🔥 Top 10 do Dia</h2>
<div id="top10"></div>
</div><script>
// COLOCA SUA CONFIG DO FIREBASE AQUI
const firebaseConfig = {
  apiKey: "SUA_API_KEY",
  authDomain: "SEU_PROJETO.firebaseapp.com",
  projectId: "SEU_PROJECT_ID",
  storageBucket: "SEU_BUCKET",
  messagingSenderId: "SEU_ID",
  appId: "SEU_APP_ID"
};

firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();

let ultimaSensi = null;

function registrar(){
 const email=document.getElementById('email').value;
 const senha=document.getElementById('senha').value;
 auth.createUserWithEmailAndPassword(email,senha)
 .then(()=>alert('Conta criada!'))
 .catch(e=>alert(e.message));
}

function login(){
 const email=document.getElementById('email').value;
 const senha=document.getElementById('senha').value;
 auth.signInWithEmailAndPassword(email,senha)
 .then(()=>alert('Login feito!'))
 .catch(e=>alert(e.message));
}

function gerarSensi(){
 ultimaSensi = {
   geral: Math.floor(Math.random()*20)+180,
   redDot: Math.floor(Math.random()*20)+170,
   botao: Math.floor(Math.random()*15)+50
 };

 document.getElementById('resultado').innerHTML=`
 <div class='card'>
 Geral: ${ultimaSensi.geral}<br>
 Red Dot: ${ultimaSensi.redDot}<br>
 Botão: ${ultimaSensi.botao}%
 </div>`;
}

async function publicarSensi(){
 const user=auth.currentUser;
 if(!user){
   alert('Faça login primeiro');
   return;
 }

 if(!ultimaSensi){
   alert('Gere uma sensi primeiro');
   return;
 }

 const nome=document.getElementById('nomeSensi').value;
 const celular=document.getElementById('celular').value;

 await db.collection('sensis').add({
   nome,
   celular,
   sensi:ultimaSensi,
   likes:0,
   user:user.email,
   createdAt:new Date()
 });

 alert('Sensi publicada!');
 carregarSensis();
}

async function carregarSensis(){
 const lista=document.getElementById('lista');
 const snap=await db.collection('sensis').get();
 lista.innerHTML='';

 let dados=[];

 snap.forEach(doc=>{
   let s=doc.data();
   dados.push({id:doc.id,...s});

   lista.innerHTML += `
   <div class='card'>
   <b>${s.nome}</b><br>
   📱 ${s.celular}<br>
   👍 ${s.likes}
   <br><br>
   <button onclick="curtir('${doc.id}')">Curtir</button>
   </div>`;
 })

 dados.sort((a,b)=>b.likes-a.likes);
 let top=dados.slice(0,10);

 document.getElementById('top10').innerHTML='';
 top.forEach((s,i)=>{
   document.getElementById('top10').innerHTML += `
   <div class='card'>
   #${i+1} ${s.nome}<br>
   👍 ${s.likes}
   </div>`;
 })
}

async function curtir(id){
 const ref=db.collection('sensis').doc(id);
 await ref.update({
   likes: firebase.firestore.FieldValue.increment(1)
 });
 carregarSensis();
}

carregarSensis();
</script></body>
</html>
