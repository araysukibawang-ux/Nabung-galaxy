<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Dompet Galaksi</title>
<style>
:root{--bg:#000010;--card:#0d0d1f;--neon:#00ffff;--neon2:#ff00ff;--text:#e0e0e0;--danger:#ff4d6d}
*{box-sizing:border-box;margin:0;padding:0;font-family:'Segoe UI',sans-serif}
body{background:var(--bg);color:var(--text);overflow-x:hidden}
canvas#bg{position:fixed;top:0;left:0;width:100%;height:100%;z-index:-1}

.login-wrap{display:flex;align-items:center;justify-content:center;height:100vh}
.login-box{background:rgba(13,13,31,.8);backdrop-filter:blur(10px);padding:32px;border-radius:20px;
  border:2px solid var(--neon);box-shadow:0 0 40px var(--neon2);width:90%;max-width:360px}
.login-box h2{text-align:center;margin-bottom:20px;color:var(--neon);text-shadow:0 0 15px var(--neon)}
.login-box input{width:100%;padding:12px;margin:8px 0;border-radius:10px;border:1px solid #333;background:#1a1a2a;color:var(--text)}
.login-box button{width:100%;padding:12px;margin-top:12px;border:none;border-radius:10px;
  background:linear-gradient(90deg,var(--neon),var(--neon2));color:#000;font-weight:800;cursor:pointer;
  animation:kejangBtn 1.5s infinite}
@keyframes kejangBtn{0%,100%{transform:scale(1) rotate(0)}25%{transform:scale(1.05) rotate(-1deg)}75%{transform:scale(1.05) rotate(1deg)}}

.app{display:none;padding:16px;max-width:700px;margin:0 auto}
.header{display:flex;justify-content:space-between;align-items:center;margin-bottom:16px}
.header h1{font-size:20px;color:var(--neon);text-shadow:0 0 10px var(--neon)}
.card{background:rgba(13,13,31,.7);backdrop-filter:blur(8px);padding:16px;border-radius:16px;border:1px solid #222;margin-bottom:16px}
.total{font-size:28px;font-weight:800;color:var(--neon);text-shadow:0 0 8px var(--neon)}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
input,textarea,button{width:100%;padding:10px;border-radius:10px;border:1px solid #333;background:#1a1a2a;color:var(--text)}
button.primary{background:var(--neon);color:#000;font-weight:700;border:none;cursor:pointer}
.log{max-height:250px;overflow-y:auto}
.log-item{display:flex;justify-content:space-between;padding:10px;border-bottom:1px dashed #333;font-size:14px}
.img-preview{width:100%;height:140px;object-fit:cover;border-radius:12px;border:1px solid var(--neon2);margin-top:8px}
.logout{color:var(--danger);cursor:pointer;font-size:12px}
.prog-bar{height:14px;background:#222;border-radius:10px;overflow:hidden;border:1px solid var(--neon);margin-top:10px}
.prog-fill{height:100%;background:linear-gradient(90deg,var(--neon),var(--neon2));width:0%;transition:width.5s;box-shadow:0 0 15px var(--neon)}
.prog-text{display:flex;justify-content:space-between;font-size:12px;margin-top:4px;color:var(--neon2)}
</style>
</head>
<body>
<canvas id="bg"></canvas>
<audio id="notifSound" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_d1712bc588.mp3"></audio>

<div class="login-wrap" id="loginWrap">
  <div class="login-box">
    <h2>🚀 DOMPET GALAKSI</h2>
    <input id="user" placeholder="Commander Name">
    <input id="pass" type="password" placeholder="Kode Akses">
    <button onclick="login()">MASUK KE KAPAL</button>
  </div>
</div>

<div class="app" id="app">
  <div class="header">
    <h1>🌌 Nabung Antariksa</h1>
    <span class="logout" onclick="logout()">Eject</span>
  </div>

  <div class="card">
    <div>Energi Terkumpul</div>
    <div class="total" id="total">0 Kredit</div>
    <div id="dateNow" style="opacity:.7;font-size:12px"></div>
    <div class="prog-bar"><div class="prog-fill" id="progFill"></div></div>
    <div class="prog-text"><span id="progPersen">0%</span><span id="sisa">Sisa: 0</span></div>
  </div>

  <div class="grid">
    <div class="card">
      <h3>Isi Energi</h3>
      <input id="nominal" type="number" placeholder="Jumlah Kredit">
      <textarea id="catatan" rows="2" placeholder="Misi: Patroli, Loot, dll"></textarea>
      <button class="primary" onclick="tambah()">+ CHARGE</button>
    </div>
    <div class="card">
      <h3>Target Misi</h3>
      <input id="namaImpian" placeholder="Nama Kapal: Dreadnought X">
      <input id="target" type="number" placeholder="Butuh Kredit: 7000000">
      <input id="foto" type="file" accept="image/*" onchange="uploadFoto()">
      <img id="preview" class="img-preview" style="display:none">
    </div>
  </div>

  <div class="card">
    <h3>AI Reminder</h3>
    <input id="jamNotif" type="time" value="12:00">
    <button onclick="aktifNotif()">AKTIFKAN ALERT JAM 12</button>
  </div>

  <div class="card">
    <h3>Log Misi</h3>
    <div class="log" id="log"></div>
  </div>
</div>

<script>
const KEY='dompet_galaksi_v3';
let data=JSON.parse(localStorage.getItem(KEY)||'{"user":"","riwayat":[],"foto":"","namaImpian":"","target":0}');
let notifAktif=false;

// ========= ANIMASI ANGKASA PERANG =========
const c=document.getElementById('bg'),ctx=c.getContext('2d');
let W,H,stars=[],ships=[],lasers=[],meteors=[];
function R(){
  W=c.width=innerWidth;H=c.height=innerHeight;
  stars=Array.from({length:150},()=>({x:Math.random()*W,y:Math.random()*H,s:Math.random()*2,v:Math.random()*.5+.2}));
  ships=[{x:W*.2,y:H*.5,vx:1,color:'#00ffff'},{x:W*.8,y:H*.5,vx:-1,color:'#ff00ff'}];
  meteors=Array.from({length:8},()=>({x:Math.random()*W,y:Math.random()*H,vx:(Math.random()-.5)*2,vy:Math.random()*2+1,r:Math.random()*3+2}));
}
R();addEventListener('resize',R);

function tembak(){
  ships.forEach((s,i)=>{
    if(Math.random()<.02)lasers.push({x:s.x,y:s.y,vx:s.vx*8,color:s.color});
  });
}

(function anim(){
  ctx.fillStyle='rgba(0,0,16,.3)';ctx.fillRect(0,0,W,H);

  // BINTANG
  stars.forEach(st=>{st.y+=st.v;if(st.y>H)st.y=0;
    ctx.beginPath();ctx.arc(st.x,st.y,st.s,0,7);ctx.fillStyle='#fff';ctx.fill();});

  // METEOR
  meteors.forEach(m=>{m.x+=m.vx;m.y+=m.vy;if(m.y>H){m.y=-10;m.x=Math.random()*W;}
    ctx.beginPath();ctx.arc(m.x,m.y,m.r,0,7);ctx.fillStyle='#aaa';ctx.shadowBlur=10;ctx.shadowColor='#fff';ctx.fill();});

  // PESAWAT INDUK
  ships.forEach(s=>{
    s.x+=s.vx;if(s.x>W-60||s.x<60)s.vx*=-1;
    ctx.fillStyle=s.color;ctx.shadowBlur=20;ctx.shadowColor=s.color;
    ctx.fillRect(s.x-40,s.y-15,80,30); // badan
    ctx.fillRect(s.x-10,s.y-30,20,60); // tower
  });

  // LASER PERANG
  tembak();
  lasers.forEach((l,i)=>{l.x+=l.vx;
    ctx.fillStyle=l.color;ctx.fillRect(l.x,l.y-2,20,4);
    if(l.x<0||l.x>W)lasers.splice(i,1);
  });

  requestAnimationFrame(anim);
})();

// ========= LOGIC APP =========
function login(){let u=user.value.trim(),p=pass.value.trim();if(!u||!p)return alert('Isi Kode Akses dulu');data.user=u;save();loginWrap.style.display='none';app.style.display='block';render();cekNotif()}
function logout(){app.style.display='none';loginWrap.style.display='flex'}
function save(){localStorage.setItem(KEY,JSON.stringify(data))}
function today(){return new Date().toLocaleDateString('id-ID',{weekday:'long',day:'numeric',month:'long',year:'numeric'})}
function tambah(){let n=Number(nominal.value),c=catatan.value.trim();if(!n||n<=0)return alert('Kredit invalid');data.riwayat.unshift({tgl:today(),n,c});nominal.value='';catatan.value='';save();render();document.querySelector('.total').animate([{transform:'scale(1)'},{transform:'scale(1.2)'},{transform:'scale(1)'}],400)}
function uploadFoto(){let f=foto.files[0];if(!f)return;let rd=new FileReader();rd.onload=e=>{data.foto=e.target.result;save();render()};rd.readAsDataURL(f)}
namaImpian.oninput=e=>{data.namaImpian=e.target.value;save();render()}
target.oninput=e=>{data.target=Number(e.target.value)||0;save();render()}

function render(){
  let total=data.riwayat.reduce((a,b)=>a+b.n,0);
  total.textContent=total.toLocaleString('id-ID')+' Kredit';
  dateNow.textContent=today();
  log.innerHTML=data.riwayat.map(r=>`<div class="log-item"><div><b>${r.tgl}</b><br><span style="opacity:.7">${r.c||'-'}</span></div><div style="color:var(--neon);font-weight:700">+${r.n.toLocaleString('id-ID')}</div></div>`).join('')||'<div style="opacity:.5">Belum ada misi</div>';
  if(data.foto){preview.src=data.foto;preview.style.display='block'}
  namaImpian.value=data.namaImpian||'';target.value=data.target||'';
  let persen=data.target>0?Math.min(100,(total/data.target)*100):0;
  progFill.style.width=persen+'%';progPersen.textContent=persen.toFixed(1)+'%';
  sisa.textContent='Sisa: '+Math.max(0,data.target-total).toLocaleString('id-ID');
  if(data.user){loginWrap.style.display='none';app.style.display='block'}
}

// ========= NOTIF =========
function aktifNotif(){Notification.requestPermission();notifAktif=true;localStorage.setItem('notifJam',jamNotif.value);alert('Alert aktif jam '+jamNotif.value+' ✅')}
function cekNotif(){let jam=localStorage.getItem('notifJam')||'12:00';jamNotif.value=jam;
  setInterval(()=>{let now=new Date();let [h,m]=jam.split(':');
    if(now.getHours()==h && now.getMinutes()==m && now.getSeconds()<10){
      if(!notifAktif)return;
      notifSound.play();if(navigator.vibrate)navigator.vibrate([200,100,200]);
      if(Notification.permission==='granted')new Notification('🚨 ALERT GALAKSI!',{body:'Isi Energi untuk '+ (data.namaImpian||'Kapalmu'),icon:data.foto||''});
    }},1000);
}
render();
</script>
</body>
</html>
