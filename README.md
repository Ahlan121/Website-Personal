<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Daftar Album & Lagu</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; background: #f0f0f0; }
  h1 { color: #333; }
  input, textarea, button { font-size: 1rem; }
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

<h1>Daftar Album & Lagu</h1>

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
  let dataAlbum = [];
  let indexAlbumAktif = null;

  function simpanKeLocalStorage() {
    localStorage.setItem("dataAlbum", JSON.stringify(dataAlbum));
  }

  function loadDariLocalStorage() {
    const data = localStorage.getItem("dataAlbum");
    if (data) {
      try {
        dataAlbum = JSON.parse(data);
      } catch(e) {
        dataAlbum = [];
      }
    }
  }

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

  function tambahLagu(indexAlbum) {
    indexAlbumAktif = indexAlbum;
    document.getElementById('formLagu').style.display = 'block';
    document.getElementById('namaAlbumForm').textContent = dataAlbum[indexAlbum].nama;
    document.getElementById('judulLaguInput').value = '';
    document.getElementById('lirikLaguInput').value = '';
    autoResizeTextarea(document.getElementById('lirikLaguInput'));
  }

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

  // Modal edit lirik vars
  const modal = document.getElementById('modalEdit');
  const modalJudul = document.getElementById('modalJudul');
  const modalLirikInput = document.getElementById('modalLirikInput');

  let currentEditAlbumIndex = null;
  let currentEditLaguIndex = null;

  function editLirik(indexAlbum, indexLagu) {
    currentEditAlbumIndex = indexAlbum;
    currentEditLaguIndex = indexLagu;

    const lagu = dataAlbum[indexAlbum].lagu[indexLagu];
    modalJudul.textContent = lagu.judul;
    modalLirikInput.value = lagu.lirik || '';
    autoResizeTextarea(modalLirikInput);

    modal.style.display = 'flex';
  }

  function tutupModalEdit() {
    modal.style.display = 'none';
  }

  function simpanEditLirik() {
    const teksBaru = modalLirikInput.value;
    if (currentEditAlbumIndex !== null && currentEditLaguIndex !== null) {
      dataAlbum[currentEditAlbumIndex].lagu[currentEditLaguIndex].lirik = teksBaru;
      simpanKeLocalStorage();
      renderAlbum();
      tutupModalEdit();
    }
  }

  // Auto-resize textarea (dipakai pada input dan modal)
  function autoResizeTextarea(textarea) {
    textarea.style.height = 'auto';
    textarea.style.height = textarea.scrollHeight + 'px';
  }

  // Pas input lirik baru, auto resize realtime
  document.getElementById('lirikLaguInput').addEventListener('input', function() {
    autoResizeTextarea(this);
  });

  // Pas input lirik modal edit, auto resize realtime
  modalLirikInput.addEventListener('input', function() {
    autoResizeTextarea(this);
  });

  function renderAlbum() {
    const container = document.getElementById('albumContainer');
    container.innerHTML = '';

    if (dataAlbum.length === 0) {
      container.innerHTML = "<p><em>Belum ada album. Silakan buat album dulu.</em></p>";
      return;
    }

    dataAlbum.forEach((album, indexAlbum) => {
      const div = document.createElement('div');
      div.className = 'album';

      let html = `<h2>${album.nama}</h2>`;
      html += `<button class="action-btn" onclick="tambahLagu(${indexAlbum})">+ Tambah Lagu</button>`;

      if (album.lagu.length === 0) {
        html += `<p><em>Belum ada lagu.</em></p>`;
      } else {
        album.lagu.forEach((lagu, indexLagu) => {
          html += `
            <div class="lagu">
              <strong>${lagu.judul}</strong>
              <div class="lirik">${lagu.lirik ? escapeHtml(lagu.lirik).replace(/\n/g,'<br>') : '<em>(Tidak ada lirik)</em>'}</div>
              <button class="action-btn" onclick="editLirik(${indexAlbum}, ${indexLagu})">Edit Lirik</button>
              <button class="hapus-btn" onclick="hapusLagu(${indexAlbum}, ${indexLagu})">Hapus Lagu</button>
            </div>
          `;
        });
      }

      div.innerHTML = html;
      container.appendChild(div);
    });
  }

  // Escape HTML untuk menghindari XSS saat tampil lirik
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

  // Export data album sebagai file JSON
  function exportData() {
    const dataStr = JSON.stringify(dataAlbum, null, 2);
    const blob = new Blob([dataStr], {type: "application/json"});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'data_album_lagu.json';
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
      // Reset input file supaya bisa import ulang kalau perlu
      event.target.value = '';
    };
    reader.readAsText(file);
  }

  // Inisialisasi saat page load
  window.onload = function() {
    loadDariLocalStorage();
    renderAlbum();
  };
</script>

</body>
</html>
