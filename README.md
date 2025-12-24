<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Orkut 3.0</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  font-family:Arial;
  background:#e6f0ff;
}

header{
  background:#3b5998;
  color:#fff;
  padding:15px;
  text-align:center;
  font-size:22px;
  font-weight:bold;
}

#login,#app{display:none;padding:15px}

input,button,textarea{
  width:100%;
  padding:10px;
  margin:5px 0;
  border-radius:6px;
  border:1px solid #ccc;
}

button{
  background:#3b5998;
  color:#fff;
  border:none;
  font-weight:bold;
}

#online{
  background:#fff;
  border-radius:8px;
  padding:10px;
  margin-bottom:10px;
}

.user{
  display:flex;
  align-items:center;
  margin-bottom:6px;
}

.user img{
  width:34px;
  height:34px;
  border-radius:50%;
  object-fit:contain;
  background:#eee;
  margin-right:8px;
}

.user span{
  font-size:14px;
  white-space:nowrap;
}

.dot{
  width:8px;
  height:8px;
  background:green;
  border-radius:50%;
  margin-left:auto;
}

#chat{
  background:#fff;
  border-radius:8px;
  padding:10px;
  height:300px;
  overflow-y:auto;
}

.msg{
  margin-bottom:8px;
}

.msg img,.msg video{
  max-width:100%;
  border-radius:6px;
  margin-top:4px;
}
</style>
</head>

<body>

<header>💙 Orkut 3.0</header>

<div id="login">
  <input id="nome" placeholder="Seu nome">
  <input id="email" placeholder="Email">
  <input id="senha" type="password" placeholder="Senha">
  <input id="foto" type="file" accept="image/*">
  <button onclick="entrar()">Entrar</button>
</div>

<div id="app">
  <div id="online">
    <b>🟢 Online</b>
    <div id="listaOnline"></div>
  </div>

  <div id="chat"></div>

  <textarea id="texto" placeholder="Digite..."></textarea>
  <input id="file" type="file" accept="image/*,video/*">
  <button onclick="enviar()">Enviar</button>
</div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

<script>
const firebaseConfig={
  apiKey:"AIzaSyCUb2VNq9AETlzhE4w0eSwRPA4yBzrVC68",
  authDomain:"meu-50495.firebaseapp.com",
  projectId:"meu-50495",
  storageBucket:"meu-50495.appspot.com",
  messagingSenderId:"941036013080",
  appId:"1:941036013080:web:61281776f379d87867a56d"
};

firebase.initializeApp(firebaseConfig);
const auth=firebase.auth();
const db=firebase.firestore();
const storage=firebase.storage();

let me=null;
let chatUser=null;

auth.onAuthStateChanged(u=>{
  if(u){
    me=u.uid;
    login.style.display="none";
    app.style.display="block";
    setOnline(true);
    loadOnline();
    loadChat();
  }else{
    login.style.display="block";
    app.style.display="none";
  }
});

function entrar(){
  auth.signInWithEmailAndPassword(email.value,senha.value)
  .catch(()=>{
    auth.createUserWithEmailAndPassword(email.value,senha.value)
    .then(c=>{
      const uid=c.user.uid;
      const f=foto.files[0];
      if(f){
        storage.ref("fotos/"+uid).put(f).then(s=>{
          s.ref.getDownloadURL().then(url=>{
            db.collection("users").doc(uid).set({
              nome:nome.value.split(" ")[0],
              foto:url,
              online:true
            });
          });
        });
      }else{
        db.collection("users").doc(uid).set({
          nome:nome.value.split(" ")[0],
          foto:"",
          online:true
        });
      }
    });
  });
}

function setOnline(v){
  db.collection("users").doc(me).update({online:v});
}

window.onbeforeunload=()=>setOnline(false);

function loadOnline(){
  db.collection("users").where("online","==",true)
  .onSnapshot(s=>{
    listaOnline.innerHTML="";
    s.forEach(d=>{
      if(d.id!==me){
        const u=d.data();
        listaOnline.innerHTML+=`
        <div class="user" onclick="chatUser='${d.id}'">
          <img src="${u.foto||'https://via.placeholder.com/40'}">
          <span>${u.nome}</span>
          <div class="dot"></div>
        </div>`;
      }
    });
  });
}

function enviar(){
  if(!chatUser)return alert("Selecione alguém online");
  const f=file.files[0];
  const t=texto.value.trim();
  if(!f && !t)return;

  if(f){
    const ref=storage.ref("midia/"+Date.now()+f.name);
    ref.put(f).then(s=>s.ref.getDownloadURL())
    .then(url=>{
      salvar({media:url,type:f.type,text:t});
    });
  }else{
    salvar({text:t});
  }

  texto.value="";
  file.value="";
}

function salvar(o){
  db.collection("msgs").add({
    from:me,
    to:chatUser,
    ...o,
    t:Date.now()
  });
}

function loadChat(){
  db.collection("msgs").orderBy("t")
  .onSnapshot(s=>{
    chat.innerHTML="";
    s.forEach(d=>{
      const m=d.data();
      if(m.from===me || m.to===me){
        let html=`<div class="msg">`;
        if(m.text)html+=`<div>${m.text}</div>`;
        if(m.media){
          if(m.type.startsWith("video"))
            html+=`<video src="${m.media}" controls></video>`;
          else
            html+=`<img src="${m.media}">`;
        }
        html+=`</div>`;
        chat.innerHTML+=html;
        chat.scrollTop=chat.scrollHeight;
      }
    });
  });
}
</script>

<
 # Orkut.ac
