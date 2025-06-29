<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Album & Lagu Band — Sinkronisasi `eleventakustik`</title>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

  <style>
    body { font-family: Arial, margin:20px; background:#f0f0f0; }
    h1, h2 { color:#333; text-align:center; }
    input, textarea, button { font-size:1rem; }
    #formBand, #mainApp, #detailPage { max-width:600px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px #aaa; }
    #formBand { margin-top:100px; text-align:center; }
    button { cursor:pointer; border:none; border-radius:4px; padding:8px 16px; }
    .action-btn { background:#3498db; color:#fff; }
    .hapus-btn { background:#e74c3c; color:#fff; }
    .back-btn { background:#7f8c8d; color:#fff; }
    .action-btn:hover { background:#2980b9; }
    .hapus-btn:hover { background:#c0392b; }
    .back-btn:hover { background:#606b6e; }
    ul#daftarJudul { list-style:none; padding:0; }
    ul#daftarJudul li { padding:10px; border-bottom:1px solid #ddd; cursor:pointer; }
    ul#daftarJudul li:hover { background:#e0e0e0; }
    .link-box input { width:100%; padding:8px; margin-top:10px; font-family:monospace; }
    .link-box button { margin-top:10px; }
    #headerBand { text-align:right; margin-bottom:10px; }
    #headerBand strong { font-size:1.1rem; }
  </style>
</head>
<body>

<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band (contoh: eleventakustik)" />
  <br><br>
  <button class="action-btn" onclick="submitNamaBand()">Mulai</button>
</div>

<div id="mainApp" style="display:none;">
  <div id="headerBand"><strong>Band: <span id="namaBandTampil"></span></strong>
    <button class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>
  <div>
    <input type="text" id="inputJudulLagu" placeholder="Tambah Judul Lagu Baru" style="width:80%;" />
    <button class="action-btn" onclick="tambahJudulLagu()">Tambah</button>
  </div>
  <ul id="daftarJudul"></ul>
</div>

<div id="detailPage" style="display:none;">
  <h2 id="judulDetail"></h2>
  <div class="link-box">
    <input type="text" id="linkInput" placeholder="Tempel link ChordTela di sini" />
    <button id="simpanLinkBtn" class="action-btn" onclick="simpanLink()">Simpan Link</button>
    <button id="editLinkBtn" class="action-btn" onclick="editLink()" style="display:none;">Edit Link</button>
    <div id="linkTersimpan" style="margin-top:10px;"></div>
  </div>
  <button class="back-btn" onclick="kembaliKeUtama()">← Kembali</button>
</div>

<script>
  // ✅ Firebase config: Ganti dengan detail project-mu
  const firebaseConfig = {
    apiKey: "API_KEY_MU",
    authDomain: "PROJECT_ID.firebaseapp.com",
    databaseURL: "https://PROJECT_ID-default-rtdb.firebaseio.com",
    projectId: "PROJECT_ID",
    storageBucket: "PROJECT_ID.appspot.com",
    messagingSenderId: "SENDER_ID",
    appId: "APP_ID"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  let namaBandAktif=null, daftarLagu=[], indexAktif=null;

  function simpanData(){
    if(namaBandAktif==='eleventakustik'){
      db.ref('bands/'+namaBandAktif).set(daftarLagu).catch(console.error);
    } else {
      localStorage.setItem('dataLagu_'+namaBandAktif, JSON.stringify(daftarLagu));
    }
  }

  function loadData(){
    if(!namaBandAktif) return;
    if(namaBandAktif==='eleventakustik'){
      db.ref('bands/'+namaBandAktif).get().then(snap=>{
        daftarLagu = snap.exists() ? snap.val() : [];
        renderDaftarJudul();
      }).catch(e=>{ console.error(e); renderDaftarJudul(); });
    } else {
      const d = localStorage.getItem('dataLagu_'+namaBandAktif);
      daftarLagu = d ? JSON.parse(d) : [];
      renderDaftarJudul();
    }
  }

  function submitNamaBand(){
    const band = document.getElementById('inputNamaBand').value.trim().toLowerCase();
    if(!band) return alert("Nama band kosong!");
    namaBandAktif=band;
    localStorage.setItem('namaBandAktif', band);
    document.getElementById('inputNamaBand').value='';
    document.getElementById('formBand').style.display='none';
    document.getElementById('mainApp').style.display='block';
    document.getElementById('namaBandTampil').textContent=band;
    loadData();
  }

  function gantiBand(){
    if(confirm("Ganti band?")) {
      namaBandAktif=null; daftarLagu=[]; localStorage.removeItem('namaBandAktif');
      document.getElementById('mainApp').style.display='none';
      document.getElementById('detailPage').style.display='none';
      document.getElementById('formBand').style.display='block';
    }
  }

  function tambahJudulLagu(){
    const j = document.getElementById('inputJudulLagu').value.trim();
    if(!j) return alert("Judul kosong!");
    if(daftarLagu.some(l=>l.judul.toLowerCase()===j.toLowerCase())) return alert("Judul sudah ada");
    daftarLagu.push({judul:j,link:""});
    document.getElementById('inputJudulLagu').value='';
    simpanData(); renderDaftarJudul();
  }

  function renderDaftarJudul(){
    const ul=document.getElementById('daftarJudul');
    ul.innerHTML='';
    if(daftarLagu.length===0) ul.innerHTML='<li><em>Belum ada judul.</em></li>';
    else daftarLagu.forEach((l,i)=>{
      const li = document.createElement('li');
      li.textContent = l.judul;
      li.onclick = ()=>bukaDetail(i);
      ul.appendChild(li);
    });
  }

  function bukaDetail(i){
    indexAktif=i;
    const l=daftarLagu[i];
    document.getElementById('mainApp').style.display='none';
    document.getElementById('detailPage').style.display='block';
    document.getElementById('judulDetail').textContent=l.judul;

    const inp=document.getElementById('linkInput'),
          saveBtn=document.getElementById('simpanLinkBtn'),
          editBtn=document.getElementById('editLinkBtn');

    if(l.link){
      inp.value=l.link; inp.readOnly=true;
      saveBtn.style.display='none'; editBtn.style.display='inline-block';
    } else {
      inp.value=''; inp.readOnly=false;
      saveBtn.style.display='inline-block'; editBtn.style.display='none';
    }
    tampilkanLink();
  }

  function editLink(){
    const inp=document.getElementById('linkInput'),
          saveBtn=document.getElementById('simpanLinkBtn'),
          editBtn=document.getElementById('editLinkBtn');
    inp.readOnly=false; saveBtn.style.display='inline-block'; editBtn.style.display='none';
    inp.focus();
  }

  function simpanLink(){
    const val=document.getElementById('linkInput').value.trim();
    if(!val) return alert("Link kosong!");
    daftarLagu[indexAktif].link=val;
    simpanData(); bukaDetail(indexAktif);
  }

  function tampilkanLink(){
    const l=daftarLagu[indexAktif];
    const div=document.getElementById('linkTersimpan');
    div.innerHTML = l.link ? `<p><a href="${l.link}" target="_blank">${l.link}</a></p>` : '<p><em>Belum ada link.</em></p>';
  }

  function kembaliKeUtama(){
    document.getElementById('detailPage').style.display='none';
    document.getElementById('mainApp').style.display='block';
  }

  window.onload=()=>{
    const b=localStorage.getItem('namaBandAktif');
    if(b){ document.getElementById('formBand').style.display='none';
      document.getElementById('mainApp').style.display='block';
      namaBandAktif=b;
      document.getElementById('namaBandTampil').textContent=b;
      loadData();
    }
  };
</script>
</body>
</html>





