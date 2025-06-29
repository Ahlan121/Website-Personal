<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Album & Lagu ke ChordTela</title>
  <style>
    body { font-family: Arial; margin:20px; background:#f0f0f0; }
    h1,h2 { color:#333; }
    input, textarea, button { font-size:1rem; }
    #formBand, #mainApp { max-width:600px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px #aaa; }
    #albumContainer .album { background:white; padding:15px; margin-bottom:15px; border-radius:8px; box-shadow:0 0 6px #bbb; }
    .lagu { border-top:1px solid #ddd; padding:10px 0; }
    .hapus-btn { color:white; background:#e74c3c; border:none; padding:5px 10px; border-radius:4px; cursor:pointer; }
    .action-btn { background:#3498db; color:white; border:none; padding:5px 10px; border-radius:4px; cursor:pointer; }
    ul { padding-left:20px; }
    li { margin-bottom:6px; }
    a { text-decoration:none; color:#2c3e50; }
    a:hover { text-decoration:underline; }
  </style>
</head>
<body>

<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band" autocomplete="off" /><br/><br/>
  <button class="action-btn" onclick="submitNamaBand()">Mulai</button>
</div>

<div id="mainApp" style="display:none;">
  <div style="text-align:right;"><strong>Band: <span id="namaBandTampil"></span></strong>
    <button class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>

  <div style="margin:15px 0;">
    <input type="text" id="namaAlbum" placeholder="Nama Album" />
    <button class="action-btn" onclick="buatAlbum()">Buat Album</button>
  </div>

  <!-- Daftar Semua Judul Lagu -->
  <div style="margin:20px 0;">
    <h2>Semua Judul Lagu</h2>
    <div id="daftarJudul"></div>
  </div>

  <div id="albumContainer"></div>

  <div id="formLagu" style="display:none; margin-top:15px; background:white; padding:15px; border-radius:8px; box-shadow:0 0 5px #ccc;">
    <h3>Tambah Lagu ke Album: <span id="namaAlbumForm"></span></h3>
    <input type="text" id="judulLaguInput" placeholder="Judul Lagu" style="width:100%; margin-bottom:8px;" />
    <textarea id="lirikLaguInput" placeholder="Lirik dan chord lagu" rows="3" style="width:100%; overflow:hidden;"></textarea><br/>
    <button class="action-btn" onclick="simpanLagu()">Simpan Lagu</button>
    <button class="hapus-btn" onclick="tutupForm()">Batal</button>
  </div>

  <div style="text-align:center; margin-top:20px;">
    <button class="action-btn" onclick="exportData()">Export Data</button>
    <input type="file" id="importFile" accept=".json" onchange="importData(event)" style="display:none" />
    <button class="action-btn" onclick="document.getElementById('importFile').click()">Import Data</button>
  </div>
</div>

<script>
  let dataAlbum = [];
  let indexAlbumAktif = null;
  let namaBandAktif = null;

  function simpanKeLocalStorage(){
    if(!namaBandAktif) return;
    localStorage.setItem("dataAlbum_"+namaBandAktif, JSON.stringify(dataAlbum));
  }
  function loadDariLocalStorage(){
    if(!namaBandAktif) return;
    const d = localStorage.getItem("dataAlbum_"+namaBandAktif);
    dataAlbum = d ? JSON.parse(d) : [];
  }

  function submitNamaBand(){
    const band = document.getElementById('inputNamaBand').value.trim().toLowerCase();
    if(!band) return alert("Nama band tidak boleh kosong!");
    namaBandAktif = band;
    localStorage.setItem("namaBandAktif", band);
    document.getElementById('formBand').style.display='none';
    document.getElementById('mainApp').style.display='block';
    document.getElementById('namaBandTampil').textContent = band;
    loadDariLocalStorage();
    renderAll();
  }

  function gantiBand(){
    if(confirm("Ganti band? Semua data disimpan.")){
      namaBandAktif = null;
      dataAlbum = [];
      localStorage.removeItem("namaBandAktif");
      document.getElementById('mainApp').style.display='none';
      document.getElementById('formBand').style.display='block';
    }
  }

  function buatAlbum(){
    const nama = document.getElementById('namaAlbum').value.trim();
    if(!nama) return alert("Nama album tidak boleh kosong!");
    dataAlbum.push({ nama, lagu: [] });
    document.getElementById('namaAlbum').value = '';
    simpanKeLocalStorage();
    renderAll();
  }

  function tambahLagu(i){
    indexAlbumAktif = i;
    document.getElementById('formLagu').style.display='block';
    document.getElementById('namaAlbumForm').textContent = dataAlbum[i].nama;
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
  }

  function simpanLagu(){
    const j = document.getElementById('judulLaguInput').value.trim();
    const l = document.getElementById('lirikLaguInput').value.trim();
    if(!j) return alert("Judul sayang harus diisi.");
    dataAlbum[indexAlbumAktif].lagu.push({ judul: j, lirik: l });
    simpanKeLocalStorage();
    renderAll();
    tutupForm();
  }

  function tutupForm(){ document.getElementById('formLagu').style.display='none'; }
  function hapusLagu(ai, li){
    if(confirm("Hapus lagu?")){
      dataAlbum[ai].lagu.splice(li, 1);
      simpanKeLocalStorage();
      renderAll();
    }
  }

  function renderAll(){
    renderDaftarJudul();
    renderAlbum();
  }

  function renderDaftarJudul(){
    const el = document.getElementById('daftarJudul');
    let html = '';
    dataAlbum.forEach((alb, ai)=>{
      alb.lagu.forEach((lagu, li)=>{
        html += `<li><a href="https://www.chordtela.com/?s=${encodeURIComponent(lagu.judul)}" target="_blank">${escapeHtml(lagu.judul)}</a></li>`;
      });
    });
    el.innerHTML = html ? `<ul>${html}</ul>` : '<em>Belum ada lagu.</em>';
  }

  function renderAlbum(){
    const cnt = document.getElementById('albumContainer');
    cnt.innerHTML = '';
    dataAlbum.forEach((alb, ai)=>{
      const div = document.createElement('div');
      div.className='album';
      let html = `<h2>${escapeHtml(alb.nama)}</h2><button class="action-btn" onclick="tambahLagu(${ai})">+ Tambah Lagu</button>`;
      if(alb.lagu.length){
        alb.lagu.forEach((lagu, li)=>{
          html += `<div class="lagu"><strong>${escapeHtml(lagu.judul)}</strong>
                   <div class="lirik">${escapeHtml(lagu.lirik)}</div>
                   <button class="action-btn" onclick="tambahLagu(${ai}); document.getElementById('judulLaguInput').value='${escapeHtml(lagu.judul)}'">Edit Judul</button>
                   <button class="hapus-btn" onclick="hapusLagu(${ai},${li})">Hapus</button></div>`;
        });
      }
      div.innerHTML = html;
      cnt.appendChild(div);
    });
  }

  function exportData(){
    const b = new Blob([JSON.stringify(dataAlbum, null,2)], {type:'application/json'});
    const u = URL.createObjectURL(b);
    const a = document.createElement('a');
    a.href = u;
    a.download = 'data_album_'+namaBandAktif+'.json';
    a.click();
    URL.revokeObjectURL(u);
  }

  function importData(e){
    const f = e.target.files[0];
    if(!f) return;
    const r = new FileReader();
    r.onload = ev => {
      try {
        const d = JSON.parse(ev.target.result);
        if(Array.isArray(d)){
          dataAlbum = d;
          simpanKeLocalStorage();
          renderAll();
        } else alert("Format salah");
      } catch {
        alert("Error baca file");
      }
    };
    r.readAsText(f);
    e.target.value='';
  }

  function escapeHtml(t){
    return t.replace(/[&<>"']/g, c => '&#'+c.charCodeAt(0)+';');
  }

  window.onload = ()=>{
    const b = localStorage.getItem('namaBandAktif');
    if(b){
      namaBandAktif = b;
      document.getElementById('formBand').style.display='none';
      document.getElementById('mainApp').style.display='block';
      document.getElementById('namaBandTampil').textContent = b;
      loadDariLocalStorage();
      renderAll();
    }
    document.getElementById('importFile').addEventListener('change', importData);
  };
</script>
</body>
</html>


