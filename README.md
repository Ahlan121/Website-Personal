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
  
  /* Modal Edit Lirik */
  #modalEdit {
    display:none; 
    position:fixed; 
    top:0; left:0; width:100%; height:100%; 
    background: rgba(0,0,0,0.5); 
    justify-content:center; 
    align-items:center;
    z-index: 9999;
  }
  #modalEdit > div {
    background:white; 
    padding:20px; 
    border-radius:8px; 
    width:90%; 
    max-width:500px; 
    box-shadow:0 0 10px #000;
    display: flex;
    flex-direction: column;
  }
  #modalEdit textarea {
    width: 100%; 
    resize: none; 
    overflow: hidden;
    font-family: monospace;
    font-size: 1rem;
    min-height: 100px;
  }
  #modalEdit h3 {
    margin-top: 0;
  }
  #modalEdit .modal-buttons {
    margin-top: 10px; 
    text-align: right;
  }
</style>
</head>
<body>

<!-- Halaman Input Nama Band -->
<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band" autocomplete="off" />
  <br/>
  <button class="action-btn" onclick="submitNamaBand()">Mulai</button>
</div>

<!-- Halaman Utama Aplikasi -->
<div id="mainApp" style="display:none;">
  <div style="text-align:right; margin-bottom:10px;">
    <strong>Band: <span id="namaBandTampil"></span></strong>
    <button style="margin-left:10px;" class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>
  
  <div>
    <input type="text" id="namaAlbum" placeholder="Nama Album" />
    <button class="action-btn" onclick="buatAlbum()">Buat Album</button>
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

<!-- Modal Edit Lirik -->
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
  // Variabel global
  let dataAlbum = [];
  let indexAlbumAktif = null;
  let namaBandAktif = null;

  // Fungsi simpan data album ke localStorage per band
  function simpanKeLocalStorage() {
    if (!namaBandAktif) return;
    localStorage.setItem("dataAlbum_" + namaBandAktif, JSON.stringify(dataAlbum));
  }

  // Fungsi load data album dari localStorage per band
  function loadDariLocalStorage() {
    if (!namaBandAktif) return;
    const data = localStorage.getItem("dataAlbum_" + namaBandAktif);
    if (data) {
      try {
        dataAlbum = JSON.parse(data);
      } catch(e) {
        dataAlbum = [];
      }
    } else {
      dataAlbum = [];
    }
  }

  // Submit nama band (dari halaman awal)
  function submitNamaBand() {
    const band = document.getElementById('inputNamaBand').value.trim();
    if (!band) {
      alert("Nama band tidak boleh kosong!");
      return;
    }
    namaBandAktif = band;
    localStorage.setItem("namaBandAktif", namaBandAktif);

    // Tampilkan halaman utama dan sembunyikan form band
    document.getElementById('formBand').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';

    // Tampilkan nama band dan load data album
    document.getElementById('namaBandTampil').textContent = namaBandAktif;
    loadDariLocalStorage();
    renderAlbum();
  }

  // Fungsi ganti band (reset aplikasi)
  function gantiBand() {
    if(confirm("Ganti band? Semua data band saat ini akan disimpan. Kamu akan kembali ke halaman input nama band.")) {
      namaBandAktif = null;
      dataAlbum = [];
      localStorage.removeItem("namaBandAktif");

      // Sembunyikan halaman utama, tampilkan halaman input band
      document.getElementById('mainApp').style.display = 'none';
      document.getElementById('formBand').style.display = 'block';
      document.getElementById('inputNamaBand').value = '';
    }
  }

  // Fungsi buat album baru
  function buatAlbum() {
    const nama = document.getElementById('namaAlbum').value.trim();
    if (nama === '') {
      alert("Nama album tidak boleh kosong!");
      return;
    }
    const album = {
      nama,
      lagu: []
    };
    dataAlbum.push(album);
    document.getElementById('namaAlbum').value = '';
    simpanKeLocalStorage();
    renderAlbum();
  }

  // Fungsi tambah lagu ke album
  function tambahLagu(indexAlbum) {
    indexAlbumAktif = indexAlbum;
    document.getElementById('formLagu').style.display = 'block';
    document.getElementById('namaAlbumForm').textContent = dataAlbum[indexAlbum].nama;
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
    autoGrow(document.getElementById('lirikLaguInput'));
  }

  // Fungsi simpan lagu baru
  function simpanLagu() {
    const judul = document.getElementById('judulLaguInput').value.trim();
    const lirik = document.getElementById('lirikLaguInput').value.trim();
    if (!judul) {
      alert("Judul lagu harus diisi.");
      return;
    }
    dataAlbum[indexAlbumAktif].lagu.push({ judul, lirik });
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
    document.getElementById('formLagu').style.display = 'none';
    simpanKeLocalStorage();
    renderAlbum();
  }

  // Fungsi tutup form tambah lagu
  function tutupForm() {
    document.getElementById('formLagu').style.display = 'none';
  }

  // Fungsi hapus lagu
  function hapusLagu(indexAlbum, indexLagu) {
    if (confirm("Hapus lagu ini?")) {
      dataAlbum[indexAlbum].lagu.splice(indexLagu, 1);
      simpanKeLocalStorage();
      renderAlbum();
    }
  }

  // Fungsi escape html untuk aman tampil lirik
  function escapeHtml(text) {
    var map = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#039;'
    };
    return text.replace(/[&<>"']/g, function(m) { return map[m]; });
  }

  // Render daftar album & lagu
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

  // Fungsi auto grow textarea supaya tinggi menyesuaikan isi
  function autoGrow(textarea) {
    textarea.style.height = "5px";
    textarea.style.height = (textarea.scrollHeight) + "px";
  }

  // Event listener auto grow untuk input lirik lagu
  document.getElementById('lirikLaguInput').addEventListener('input', function(){
    autoGrow(this);
  });

  // Modal edit lirik
  let indexAlbumEdit = null;
  let indexLaguEdit = null;
  function bukaModalEdit(idxAlbum, idxLagu) {
    indexAlbumEdit = idxAlbum;
    indexLaguEdit = idxLagu;
    const lagu = dataAlbum[idxAlbum].lagu[idxLagu];
    document.getElementById('modalJudul').textContent = lagu.judul;
    const ta = document.getElementById('modalLirikInput');
    ta.value = lagu.lirik;
    autoGrow(ta);
    document.getElementById('modalEdit').style.display = 'flex';
  }
  function tutupModalEdit() {
    document.getElementById('modalEdit').style.display = 'none';
  }
  function simpanEditLirik() {
    const ta = document.getElementById('modalLirikInput');
    dataAlbum[indexAlbumEdit].lagu[indexLaguEdit].lirik = ta.value.trim();
    simpanKeLocalStorage();
    renderAlbum();
    tutupModalEdit();
  }

  // Export data album sebagai file JSON
  function exportData() {
    if (!namaBandAktif) {
      alert("Tidak ada band aktif.");
      return;
    }
    const dataStr = JSON.stringify(dataAlbum, null, 2);
    const blob = new Blob([dataStr], {type: "application/json"});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'data_album_lagu_' + namaBandAktif + '.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // Import data album dari file JSON
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
          alert('Data berhasil diimport!');
        } else {
          alert('Format data salah.');
        }
      } catch (err) {
        alert('Gagal membaca file JSON.');
      }
      event.target.value = '';
    };
    reader.readAsText(file);
  }

  // Saat halaman pertama kali dimuat, cek apakah ada band aktif sebelumnya
  window.onload = function() {
    const bandTerdahulu = localStorage.getItem("namaBandAktif");
    if (bandTerdahulu) {
      namaBandAktif = bandTerdahulu;
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
