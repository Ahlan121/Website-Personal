<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Album & Lagu Berdasarkan Band</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f0f0f0; }
    h1, h2, h3 { color: #333; }
    input, textarea, button { font-size: 1rem; }
    #formBand, #mainApp, #songView { max-width: 600px; margin: auto; }
    #formBand, #mainApp { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #aaa; }
    #songView { display: none; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #aaa; margin-top: 50px; }
    #albumContainer .album { background: white; padding: 15px; margin-bottom: 15px; border-radius: 8px; box-shadow: 0 0 6px #bbb; }
    .lagu { border-top: 1px solid #ddd; padding: 10px 0; }
    .lirik { white-space: pre-wrap; background: #eee; padding: 10px; margin: 8px 0; border-radius: 5px; font-family: monospace; }
    .hapus-btn { color: white; background: #e74c3c; border: none; padding: 5px 10px; border-radius: 4px; }
    .action-btn { background: #3498db; color: white; border: none; padding: 5px 10px; border-radius: 4px; }
    button:hover { opacity: 0.9; cursor: pointer; }
    ul { padding-left: 20px; }
  </style>
</head>
<body>

<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band" autocomplete="off" />
  <br/><br/>
  <button class="action-btn" onclick="submitNamaBand()">Mulai</button>
</div>

<div id="mainApp" style="display:none;">
  <div style="text-align:right; margin-bottom:10px;">
    <strong>Band: <span id="namaBandTampil"></span></strong>
    <button class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>

  <div>
    <input type="text" id="namaAlbum" placeholder="Nama Album" />
    <button class="action-btn" onclick="buatAlbum()">Buat Album</button>
  </div>

  <!-- Daftar Semua Judul Lagu -->
  <div style="margin: 20px 0;">
    <h3>Semua Judul Lagu</h3>
    <div id="daftarJudul"></div>
  </div>

  <div id="albumContainer"></div>

  <div id="formLagu">
    <h3>Tambah Lagu ke Album: <span id="namaAlbumForm"></span></h3>
    <input type="text" id="judulLaguInput" placeholder="Judul Lagu" style="width: 100%; margin-bottom: 8px;" />
    <textarea id="lirikLaguInput" placeholder="Lirik dan chord lagu" rows="3" style="overflow:hidden; width:100%;"></textarea><br/>
    <button class="action-btn" onclick="simpanLagu()">Simpan Lagu</button>
    <button class="hapus-btn" onclick="tutupForm()">Batal</button>
  </div>

  <div id="exportImportButtons" style="margin-top:20px; text-align:center;">
    <button onclick="exportData()" class="action-btn">Export Data</button>
    <input type="file" id="importFile" accept=".json" onchange="importData(event)" style="display:none" />
    <button class="action-btn" onclick="document.getElementById('importFile').click()">Import Data</button>
  </div>
</div>

<!-- Tampilan Lagu Tunggal -->
<div id="songView">
  <h2 id="svJudul"></h2>
  <div id="svLirik" class="lirik"></div>
  <button class="action-btn" onclick="closeSongView()">Kembali</button>
</div>

<script>
  let dataAlbum = [];
  let indexAlbumAktif = null;
  let namaBandAktif = null;
  let indexLaguEdit = null;

  function simpanKeLocalStorage() {
    if (!namaBandAktif) return;
    localStorage.setItem("dataAlbum_" + namaBandAktif, JSON.stringify(dataAlbum));
  }
  function loadDariLocalStorage() {
    if (!namaBandAktif) return;
    const data = localStorage.getItem("dataAlbum_" + namaBandAktif);
    dataAlbum = data ? JSON.parse(data) : [];
  }

  function submitNamaBand() {
    const band = document.getElementById('inputNamaBand').value.trim().toLowerCase();
    if (!band) return alert("Nama band tidak boleh kosong!");
    namaBandAktif = band;
    localStorage.setItem("namaBandAktif", band);
    document.getElementById('formBand').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';
    document.getElementById('namaBandTampil').textContent = band;
    loadDariLocalStorage();
    renderAll();
  }
  function gantiBand() {
    if (confirm("Ganti band? Semua data disimpan.")) {
      namaBandAktif = null;
      dataAlbum = [];
      localStorage.removeItem("namaBandAktif");
      document.getElementById('mainApp').style.display = 'none';
      document.getElementById('formBand').style.display = 'block';
    }
  }
  function buatAlbum() {
    const nama = document.getElementById('namaAlbum').value.trim();
    if (!nama) return alert("Nama album tidak boleh kosong!");
    dataAlbum.push({ nama, lagu: [] });
    document.getElementById('namaAlbum').value = '';
    simpanKeLocalStorage();
    renderAll();
  }
  function tambahLagu(i) {
    indexAlbumAktif = i;
    document.getElementById('formLagu').style.display = 'block';
    document.getElementById('namaAlbumForm').textContent = dataAlbum[i].nama;
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
  }
  function simpanLagu() {
    const judul = document.getElementById('judulLaguInput').value.trim();
    const lirik = document.getElementById('lirikLaguInput').value.trim();
    if (!judul) return alert("Judul lagu harus diisi.");
    dataAlbum[indexAlbumAktif].lagu.push({ judul, lirik });
    simpanKeLocalStorage();
    renderAll();
    document.getElementById('formLagu').style.display = 'none';
  }
  function tutupForm() { document.getElementById('formLagu').style.display = 'none'; }
  function hapusLagu(a, l) {
    if (confirm("Hapus lagu?")) {
      dataAlbum[a].lagu.splice(l,1);
      simpanKeLocalStorage();
      renderAll();
    }
  }

  function renderAll() {
    renderDaftarJudul();
    renderAlbum();
  }

  function renderDaftarJudul() {
    const div = document.getElementById('daftarJudul');
    div.innerHTML = '';
    let list = '';
    dataAlbum.forEach((alb, ai) => {
      alb.lagu.forEach((lagu, li) => {
        list += `<li><a href="javascript:void(0)" onclick="viewSong(${ai},${li})">${escapeHtml(lagu.judul)}</a></li>`;
      });
    });
    div.innerHTML = list ? `<ul>${list}</ul>` : '<em>Belum ada lagu.</em>';
  }

  function renderAlbum() {
    const cnt = document.getElementById('albumContainer');
    cnt.innerHTML = '';
    dataAlbum.forEach((alb, ai) => {
      const d = document.createElement('div');
      d.className = 'album';
      let html = `<h2>${escapeHtml(alb.nama)}</h2><button class="action-btn" onclick="tambahLagu(${ai})">+ Tambah Lagu</button>`;
      if (alb.lagu.length) {
        alb.lagu.forEach((lagu, li) => {
          html += `<div class="lagu"><strong>${escapeHtml(lagu.judul)}</strong>
                   <div class="lirik">${escapeHtml(lagu.lirik)}</div>
                   <button class="action-btn" onclick="bukaModalEdit(${ai},${li})">Edit</button>
                   <button class="hapus-btn" onclick="hapusLagu(${ai},${li})">Hapus</button></div>`;
        });
      } else html += '<p><em>Belum ada lagu.</em></p>';
      d.innerHTML = html;
      cnt.appendChild(d);
    });
  }

  function viewSong(ai, li) {
    const lagu = dataAlbum[ai].lagu[li];
    document.getElementById('svJudul').textContent = lagu.judul;
    document.getElementById('svLirik').textContent = lagu.lirik;
    document.getElementById('mainApp').style.display = 'none';
    document.getElementById('songView').style.display = 'block';
  }

  function closeSongView() {
    document.getElementById('songView').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';
  }

  function bukaModalEdit(ai, li) {
    indexAlbumAktif = ai;
    indexLaguEdit = li;
    const l = dataAlbum[ai].lagu[li];
    const modal = document.createElement('div');
    // Modal edit implementation disini (bisa adaptasi modal sebelumnya).
  }

  function escapeHtml(t){return t.replace(/[&<>"']/g,c=>'&#'+c.charCodeAt(0)+';');}

  function exportData(){
    const b=new Blob([JSON.stringify(dataAlbum,null,2)],{type:"application/json"});
    const u=URL.createObjectURL(b);
    const a=document.createElement('a');a.href=u;a.download='data_album_'+namaBandAktif+'.json';
    a.click();URL.revokeObjectURL(u);
  }
  function importData(e){
    const f=e.target.files[0]; if(!f) return;
    const r=new FileReader();r.onload=function(ev){
      try {
        const d=JSON.parse(ev.target.result);
        if(Array.isArray(d)){dataAlbum=d;simpanKeLocalStorage();renderAll();}
        else alert('Format salah');
      } catch { alert('Error baca file');}
    };
    r.readAsText(f);
    e.target.value='';
  }

  window.onload = () => {
    const b = localStorage.getItem("namaBandAktif");
    if(b){namaBandAktif=b;
      document.getElementById('formBand').style.display = 'none';
      document.getElementById('mainApp').style.display = 'block';
      document.getElementById('namaBandTampil').textContent = b;
      loadDariLocalStorage(); renderAll();
    }
  };
</script>
</body>
</html>

