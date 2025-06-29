<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Album & Lagu Berdasarkan Band</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f0f0f0; }
    h1 { color: #333; }
    input, textarea, button { font-size: 1rem; }
    #formBand, #mainApp { max-width: 600px; margin: auto; }
    #formBand { margin-top: 100px; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #aaa; text-align: center; }
    #formBand input { width: 80%; padding: 8px; margin-bottom: 12px; }
    #formBand button { padding: 8px 16px; cursor: pointer; }
    #formLagu { display: none; margin-top: 15px; background: white; padding: 15px; border-radius: 8px; box-shadow: 0 0 5px #ccc; }
    #formLagu textarea { width: 100%; resize: none; overflow: hidden; }
    #albumContainer .album { background: white; padding: 15px; margin-bottom: 15px; border-radius: 8px; box-shadow: 0 0 6px #bbb; }
    #albumContainer .album h2 { margin-top: 0; }
    .lagu { border-top: 1px solid #ddd; padding: 10px 0; }
    .lagu:first-child { border-top: none; }
    .lirik { white-space: pre-wrap; background: #eee; padding: 10px; margin: 8px 0; border-radius: 5px; font-family: monospace; }
    button { margin-right: 6px; cursor: pointer; }
    .hapus-btn { color: white; background: #e74c3c; border: none; padding: 5px 10px; border-radius: 4px; }
    .hapus-btn:hover { background: #c0392b; }
    .action-btn { background: #3498db; color: white; border: none; padding: 5px 10px; border-radius: 4px; }
    .action-btn:hover { background: #2980b9; }
    #exportImportButtons { margin-top: 20px; }

    #modalEdit {
      display:none; position:fixed; top:0; left:0; width:100%; height:100%;
      background: rgba(0,0,0,0.5); justify-content:center; align-items:center; z-index: 9999;
    }
    #modalEdit > div {
      background:white; padding:20px; border-radius:8px; width:90%; max-width:500px;
      box-shadow:0 0 10px #000; display: flex; flex-direction: column;
    }
    #modalEdit textarea {
      width: 100%; resize: none; overflow: hidden; font-family: monospace; font-size: 1rem; min-height: 100px;
    }
    #modalEdit .modal-buttons { margin-top: 10px; text-align: right; }
  </style>
</head>
<body>

<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band" autocomplete="off" />
  <br/>
  <button class="action-btn" onclick="submitNamaBand()">Mulai</button>
</div>

<div id="mainApp" style="display:none;">
  <div style="text-align:right; margin-bottom:10px;">
    <strong>Band: <span id="namaBandTampil"></span></strong>
    <button style="margin-left:10px;" class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>

  <div>
    <input type="text" id="namaAlbum" placeholder="Nama Album" />
    <button class="action-btn" onclick="buatAlbum()">Buat Album</button>
  </div>

  <!-- 🔍 Pencarian Lagu -->
  <div style="margin: 20px 0;">
    <input type="text" id="cariLaguInput" placeholder="Cari judul lagu..." oninput="cariLagu()" style="width: 100%; padding: 8px;" />
    <div id="hasilPencarian" style="margin-top: 10px;"></div>
  </div>

  <div id="albumContainer"></div>

  <div id="formLagu">
    <h3>Tambah Lagu ke Album: <span id="namaAlbumForm"></span></h3>
    <input type="text" id="judulLaguInput" placeholder="Judul Lagu" style="width: 100%; margin-bottom: 8px;" />
    <textarea id="lirikLaguInput" placeholder="Lirik dan chord lagu (bisa pakai spasi dan baris baru)" rows="3" style="overflow:hidden;"></textarea><br/>
    <button class="action-btn" onclick="simpanLagu()">Simpan Lagu</button>
    <button class="hapus-btn" onclick="tutupForm()">Batal</button>
  </div>

  <div id="exportImportButtons">
    <button onclick="exportData()" class="action-btn">Export Data (Backup)</button>
    <input type="file" id="importFile" accept=".json" onchange="importData(event)" style="display:none" />
    <button onclick="document.getElementById('importFile').click()" class="action-btn">Import Data (Restore)</button>
  </div>
</div>

<div id="modalEdit">
  <div>
    <h3>Edit Lirik Lagu: <span id="modalJudul"></span></h3>
    <textarea id="modalLirikInput" rows="5" style="overflow:hidden;"></textarea>
    <div class="modal-buttons">
      <button class="action-btn" onclick="simpanEditLirik()">Simpan</button>
      <button class="hapus-btn" onclick="tutupModalEdit()">Batal</button>
    </div>
  </div>
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
    let band = document.getElementById('inputNamaBand').value.trim();
    if (!band) return alert("Nama band tidak boleh kosong!");
    band = band.toLowerCase();
    namaBandAktif = band;
    localStorage.setItem("namaBandAktif", namaBandAktif);
    document.getElementById('formBand').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';
    document.getElementById('namaBandTampil').textContent = namaBandAktif;
    loadDariLocalStorage();
    renderAlbum();
  }

  function gantiBand() {
    if(confirm("Ganti band? Semua data band saat ini akan disimpan.")) {
      namaBandAktif = null;
      dataAlbum = [];
      localStorage.removeItem("namaBandAktif");
      document.getElementById('mainApp').style.display = 'none';
      document.getElementById('formBand').style.display = 'block';
      document.getElementById('inputNamaBand').value = '';
    }
  }

  function buatAlbum() {
    const nama = document.getElementById('namaAlbum').value.trim();
    if (!nama) return alert("Nama album tidak boleh kosong!");
    dataAlbum.push({ nama, lagu: [] });
    document.getElementById('namaAlbum').value = '';
    simpanKeLocalStorage();
    renderAlbum();
  }

  function tambahLagu(indexAlbum) {
    indexAlbumAktif = indexAlbum;
    document.getElementById('formLagu').style.display = 'block';
    document.getElementById('namaAlbumForm').textContent = dataAlbum[indexAlbum].nama;
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
    autoGrow(document.getElementById('lirikLaguInput'));
  }

  function simpanLagu() {
    const judul = document.getElementById('judulLaguInput').value.trim();
    const lirik = document.getElementById('lirikLaguInput').value.trim();
    if (!judul) return alert("Judul lagu harus diisi.");
    dataAlbum[indexAlbumAktif].lagu.push({ judul, lirik });
    document.getElementById('formLagu').style.display = 'none';
    simpanKeLocalStorage();
    renderAlbum();
  }

  function tutupForm() {
    document.getElementById('formLagu').style.display = 'none';
  }

  function hapusLagu(indexAlbum, indexLagu) {
    if (confirm("Hapus lagu ini?")) {
      dataAlbum[indexAlbum].lagu.splice(indexLagu, 1);
      simpanKeLocalStorage();
      renderAlbum();
    }
  }

  function bukaModalEdit(iA, iL) {
    indexAlbumAktif = iA;
    const lagu = dataAlbum[iA].lagu[iL];
    document.getElementById('modalJudul').textContent = lagu.judul;
    const ta = document.getElementById('modalLirikInput');
    ta.value = lagu.lirik;
    autoGrow(ta);
    document.getElementById('modalEdit').style.display = 'flex';
    indexLaguEdit = iL;
  }

  function simpanEditLirik() {
    const ta = document.getElementById('modalLirikInput');
    dataAlbum[indexAlbumAktif].lagu[indexLaguEdit].lirik = ta.value.trim();
    simpanKeLocalStorage();
    renderAlbum();
    tutupModalEdit();
  }

  function tutupModalEdit() {
    document.getElementById('modalEdit').style.display = 'none';
  }

  function escapeHtml(text) {
    var map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
    return text.replace(/[&<>"']/g, function(m) { return map[m]; });
  }

  function renderAlbum() {
    const container = document.getElementById('albumContainer');
    container.innerHTML = '';
    if (dataAlbum.length === 0) {
      container.innerHTML = '<p><em>Belum ada album. Buat album dulu ya!</em></p>';
      return;
    }
    dataAlbum.forEach((album, indexAlbum) => {
      const div = document.createElement('div');
      div.className = 'album';
      let html = `<h2>${escapeHtml(album.nama)}</h2>`;
      html += `<button class="action-btn" onclick="tambahLagu(${indexAlbum})">+ Tambah Lagu</button>`;
      if (album.lagu.length === 0) {
        html += `<p><em>Belum ada lagu.</em></p>`;
      } else {
        album.lagu.forEach((lagu, indexLagu) => {
          html += `
            <div class="lagu">
              <strong>${escapeHtml(lagu.judul)}</strong>
              <div class="lirik">${lagu.lirik ? escapeHtml(lagu.lirik).replace(/\n/g,'<br>') : '<em>(Tidak ada lirik)</em>'}</div>
              <button class="action-btn" onclick="bukaModalEdit(${indexAlbum}, ${indexLagu})">Edit Lirik</button>
              <button class="hapus-btn" onclick="hapusLagu(${indexAlbum}, ${indexLagu})">Hapus Lagu</button>
            </div>
          `;
        });
      }
      div.innerHTML = html;
      container.appendChild(div);
    });
  }

  function autoGrow(textarea) {
    textarea.style.height = "5px";
    textarea.style.height = (textarea.scrollHeight) + "px";
  }

  document.getElementById('lirikLaguInput').addEventListener('input', function() {
    autoGrow(this);
  });

  function exportData() {
    if (!namaBandAktif) return alert("Tidak ada band aktif.");
    const blob = new Blob([JSON.stringify(dataAlbum, null, 2)], {type: "application/json"});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'data_album_' + namaBandAktif + '.json';
    a.click();
    URL.revokeObjectURL(url);
  }

  function importData(event) {
    const file = event.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = function(e) {
      try {
        const importedData = JSON.parse(e.target.result);
        if (Array.isArray(importedData)) {
          dataAlbum = importedData;
          simpanKeLocalStorage();
          renderAlbum();
        } else {
          alert('Format data salah.');
        }
      } catch {
        alert('Gagal membaca file.');
      }
      event.target.value = '';
    };
    reader.readAsText(file);
  }

  function cariLagu() {
    const keyword = document.getElementById('cariLaguInput').value.toLowerCase().trim();
    const hasil = document.getElementById('hasilPencarian');
    hasil.innerHTML = '';
    if (keyword.length === 0) return;

    let ditemukan = [];

    dataAlbum.forEach((album, indexAlbum) => {
      album.lagu.forEach((lagu, indexLagu) => {
        if (lagu.judul.toLowerCase().includes(keyword)) {
          ditemukan.push({
            judul: lagu.judul,
            indexAlbum,
            indexLagu,
            album: album.nama
          });
        }
      });
    });

    if (ditemukan.length === 0) {
      hasil.innerHTML = '<p><em>Tidak ada lagu ditemukan.</em></p>';
    } else {
      const list = document.createElement('ul');
      list.style.paddingLeft = '20px';
      ditemukan.forEach(item => {
        const li = document.createElement('li');
        li.innerHTML = `<a href="javascript:void(0)" onclick="scrollKeLagu(${item.indexAlbum}, ${item.indexLagu})">${escapeHtml(item.judul)} <small style="color:#777;">(${escapeHtml(item.album)})</small></a>`;
        list.appendChild(li);
      });
      hasil.appendChild(list);
    }
  }

  function scrollKeLagu(indexAlbum, indexLagu) {
    const albumDivs = document.querySelectorAll('#albumContainer .album');
    const targetAlbum = albumDivs[indexAlbum];
    const laguDivs = targetAlbum.querySelectorAll('.lagu');
    const targetLagu = laguDivs[indexLagu];
    if (targetLagu) {
      targetLagu.scrollIntoView({ behavior: 'smooth', block: 'center' });
      targetLagu.style.backgroundColor = '#ffffcc';
      setTimeout(() => { targetLagu.style.backgroundColor = ''; }, 1000);
    }
  }

  window.onload = function() {
    const band = localStorage.getItem("namaBandAktif");
    if (band) {
      namaBandAktif = band;
      document.getElementById('formBand').style.display = 'none';
      document.getElementById('mainApp').style.display = 'block';
      document.getElementById('namaBandTampil').textContent = namaBandAktif;
      loadDariLocalStorage();
      renderAlbum();
    }
  };
</script>
</body>
</html>
