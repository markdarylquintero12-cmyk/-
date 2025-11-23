<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DALTON — Student Support</title>
<style>
:root{--bg:#f5f7fb;--card:#fff;--primary:#2563eb;--muted:#6b7280;--radius:12px;}
*{box-sizing:border-box;}
body{margin:0;font-family:Inter,system-ui,-apple-system,"Segoe UI",Roboto,Arial;background:var(--bg);}
.wrap{max-width:1000px;margin:20px auto;padding:16px;}
.card{background:var(--card);border-radius:var(--radius);padding:16px;margin-bottom:16px;box-shadow:0 6px 18px rgba(2,6,23,0.06);}
.page{display:none;}.page.active{display:block;}
h1{color:var(--primary);}
label{display:block;margin-top:8px;font-weight:600;}
input[type=text],textarea,select{width:100%;padding:10px;border-radius:8px;border:1px solid #e6eef8;}
.btn{padding:10px 14px;border:none;border-radius:8px;font-weight:700;cursor:pointer;background:var(--primary);color:#fff;}
.btn.ghost{background:transparent;color:var(--muted);border:1px solid rgba(2,6,23,0.06);}
.avatar{width:44px;height:44px;border-radius:10px;display:flex;align-items:center;justify-content:center;font-size:20px;background:#fff;}
.emoji-grid{display:flex;flex-wrap:wrap;gap:8px;}
.emoji{font-size:26px;padding:6px;cursor:pointer;border-radius:8px;border:1px solid transparent;}
.selected{border:1px solid rgba(37,99,235,0.5);box-shadow:0 6px 18px rgba(37,99,235,0.08);}
.post{border-radius:10px;background:#fff;padding:12px;margin-bottom:12px;border:1px solid #eef2ff;}
.comment{margin-top:10px;padding:8px;border-radius:8px;background:#f8fafc;border:1px solid rgba(2,6,23,0.03);}
.comment.teacher-comment{background:#fff8f6;border:1px solid rgba(239,68,68,0.06);}
.comment .small{font-size:12px;color:var(--muted);}
textarea{resize:none;}
</style>
</head>
<body>
<div class="wrap">

<!-- LOGIN PAGE -->
<div id="loginPage" class="page active">
  <div class="card"><h1>DALTON — Student Support</h1><p>Creator: <b>MD_QUIN</b></p></div>
  <div class="card">
    <label>Username</label>
    <input id="usernameInput" type="text" placeholder="Enter your unique username">
    <label>Role</label>
    <div style="display:flex;gap:8px;">
      <button id="roleStudent" class="btn">Student</button>
      <button id="roleTeacher" class="btn ghost">Teacher</button>
    </div>
    <label>Choose Avatar (first time signup only)</label>
    <div id="avatarGrid" class="emoji-grid"></div>
    <div style="height:12px"></div>
    <button id="loginBtn" class="btn">Login / Create</button>
  </div>
</div>

<!-- FEED PAGE -->
<div id="feedPage" class="page">
  <div class="card" style="display:flex;justify-content:space-between;align-items:center;">
    <div style="display:flex;gap:12px;align-items:center;">
      <div id="hdrAvatar" class="avatar">🙂</div>
      <div>
        <div id="hdrName" style="font-weight:800;">User</div>
        <div class="small" id="hdrRole">Role</div>
      </div>
    </div>
    <div>
      <button id="logoutBtn" class="btn ghost">Logout</button>
    </div>
  </div>
  <div class="card">
    <button id="openCompose" class="btn">Compose Complaint</button>
    <div id="postsContainer"></div>
  </div>
</div>

<!-- COMPOSE PAGE -->
<div id="composePage" class="page">
  <div class="card">
    <button id="composeBack" class="btn ghost">Back to Feed</button>
    <label>Emotion</label>
    <div id="composeEmotions" class="emoji-grid"></div>
    <label>Message</label>
    <textarea id="composeText" rows="5" placeholder="Write your complaint here..."></textarea>
    <button id="composePost" class="btn">Post Complaint</button>
  </div>
</div>

<footer style="text-align:center;margin-top:20px;">DALTON — built with ❤️ by MD_QUIN</footer>
</div>

<!-- Firebase SDK -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
import { getFirestore, doc, setDoc, getDoc, collection, addDoc, onSnapshot, query, orderBy, updateDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-firestore.js";

// --- Firebase Config ---
const firebaseConfig = {
  apiKey: "PASTE_YOUR_API_KEY",
  authDomain: "PASTE_YOUR_AUTHDOMAIN",
  projectId: "PASTE_YOUR_PROJECT_ID",
  storageBucket: "PASTE_YOUR_STORAGE_BUCKET",
  messagingSenderId: "PASTE_YOUR_MSG_SENDER_ID",
  appId: "PASTE_YOUR_APP_ID"
};
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// UI Elements
const loginPage=document.getElementById('loginPage');
const feedPage=document.getElementById('feedPage');
const composePage=document.getElementById('composePage');
const usernameInput=document.getElementById('usernameInput');
const roleStudent=document.getElementById('roleStudent');
const roleTeacher=document.getElementById('roleTeacher');
const loginBtn=document.getElementById('loginBtn');
const hdrAvatar=document.getElementById('hdrAvatar');
const hdrName=document.getElementById('hdrName');
const hdrRole=document.getElementById('hdrRole');
const logoutBtn=document.getElementById('logoutBtn');
const postsContainer=document.getElementById('postsContainer');
const openCompose=document.getElementById('openCompose');
const composeBack=document.getElementById('composeBack');
const composeText=document.getElementById('composeText');
const composePost=document.getElementById('composePost');

const AVATARS=['😀','😊','😎','🧑‍🎓','🧑‍🏫','😅','😔','😢','🤍','🤝','🌟','🔥','🦋','🌈'];
const EMOTIONS=['😔','😡','😢','😞','🤯','🙂','😌'];

let selectedRole='', selectedAvatar='', selectedEmotion='', SESSION=null;

// Render avatars
const avatarGrid=document.getElementById('avatarGrid');
AVATARS.forEach(a=>{
  const btn=document.createElement('button'); btn.type='button'; btn.className='emoji'; btn.innerText=a;
  btn.onclick=()=>{ document.querySelectorAll('#avatarGrid .emoji').forEach(x=>x.classList.remove('selected')); btn.classList.add('selected'); selectedAvatar=a; };
  avatarGrid.appendChild(btn);
});

// Render compose emotions
const composeEmotions=document.getElementById('composeEmotions');
EMOTIONS.forEach(e=>{
  const b=document.createElement('button'); b.type='button'; b.className='emoji'; b.innerText=e;
  b.onclick=()=>{ document.querySelectorAll('#composeEmotions .emoji').forEach(x=>x.classList.remove('selected')); b.classList.add('selected'); selectedEmotion=e; };
  composeEmotions.appendChild(b);
});

// Role buttons
roleStudent.onclick=()=>{ selectedRole='Student'; roleStudent.classList.add('btn'); roleTeacher.classList.remove('btn'); roleTeacher.classList.add('btn','ghost'); };
roleTeacher.onclick=()=>{ selectedRole='Teacher'; roleTeacher.classList.add('btn'); roleStudent.classList.remove('btn'); roleStudent.classList.add('btn','ghost'); };

// Show page
function showPage(p){ loginPage.classList.remove('active'); feedPage.classList.remove('active'); composePage.classList.remove('active'); document.getElementById(p).classList.add('active'); }

// Login / Signup
loginBtn.onclick=async()=>{
  const name=usernameInput.value.trim();
  if(!name){alert('Enter username'); return;}
  if(!selectedRole){alert('Select role'); return;}
  const userRef=doc(db,'users',name);
  const snap=await getDoc(userRef);
  if(snap.exists()){
    const data=snap.data();
    if(data.role!==selectedRole){alert('Role mismatch'); return;}
    SESSION={username:data.username,role:data.role,avatar:data.avatar};
  }else{
    if(!selectedAvatar){alert('Select avatar'); return;}
    await setDoc(userRef,{username:name,role:selectedRole,avatar:selectedAvatar,createdAt:serverTimestamp()});
    SESSION={username:name,role:selectedRole,avatar:selectedAvatar};
  }
  localStorage.setItem('dalton_session',JSON.stringify(SESSION));
  afterLogin();
}

// After login
function afterLogin(){ hdrAvatar.innerText=SESSION.avatar; hdrName.innerText=SESSION.username; hdrRole.innerText=SESSION.role; showPage('feed'); listenPosts(); }

// Logout
logoutBtn.onclick=()=>{ localStorage.removeItem('dalton_session'); SESSION=null; showPage('login'); postsContainer.innerHTML=''; usernameInput.value=''; };

// Load session
(function(){ const s=localStorage.getItem('dalton_session'); if(s){ SESSION=JSON.parse(s); afterLogin(); } })();

// Listen posts realtime
let unsubscribe=null;
function listenPosts(){
  const q=query(collection(db,'posts'),orderBy('createdAt','desc'));
  if(unsubscribe) unsubscribe();
  unsubscribe=onSnapshot(q,snap=>{
    postsContainer.innerHTML='';
    snap.forEach(docSnap=>{
      const p=docSnap.data(); const id=docSnap.id;
      const div=document.createElement('div'); div.className='post';
      let commentsHTML='';
      if(p.comments) commentsHTML=p.comments.map(c=>`<div class="comment ${c.role==='Teacher'?'teacher-comment':''}"><b>${c.username}</b>: ${c.text}</div>`).join('');
      div.innerHTML=`<div style="display:flex;align-items:center;gap:8px;">
        <div class="avatar">${p.avatar}</div><b>${p.username}</b> <span>${p.role}</span> ${p.emotion}<br>${p.text}
        <div class="small">AI: ${p.ai}</div>
        </div>${commentsHTML}
        <textarea id="cmt-${id}" rows="2" placeholder="Comment..."></textarea>
        <button onclick="addComment('${id}')">Comment</button>`;
      postsContainer.appendChild(div);
    });
  });
}

// Add comment
window.addComment=async(id)=>{
  const txt=document.getElementById('cmt-'+id).value.trim();
  if(!txt){alert('Write a comment'); return;}
  const postRef=doc(db,'posts',id);
  const postSnap=await getDoc(postRef);
  const cmt={username:SESSION.username,role:SESSION.role,text:txt,createdAt:serverTimestamp()};
  const comments=postSnap.data().comments||[];
  await updateDoc(postRef,{comments:[...comments,cmt]});
  document.getElementById('cmt-'+id).value='';
}

// Open compose
openCompose.onclick=()=>showPage('compose');
composeBack.onclick=()=>showPage('feed');
composePost.onclick=async()=>{
  const txt=composeText.value.trim(); if(!txt){alert('Write message'); return;}
  const newPost={username:SESSION.username,role:SESSION.role,avatar:SESSION.avatar,text:txt,emotion:selectedEmotion||'🙂',ai:'Keep going! You are strong!',comments:[],createdAt:serverTimestamp()};
  await addDoc(collection(db,'posts'),newPost);
  composeText.value=''; selectedEmotion=''; document.querySelectorAll('#composeEmotions .emoji').forEach(x=>x.classList.remove('selected'));
  showPage('feed');
}
</script>
</body>
</html>
