<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Album & Lagu Berdasarkan Band - Versi Judul & Link ChordTela</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; background: #f0f0f0; }
    h1, h2 { color: #333; }
    input, button { font-size: 1rem; }
    #formBand, #mainApp, #detailPage { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #aaa; }
    #formBand { margin-top: 100px; text-align: center; }
    #formBand input { width: 80%; padding: 8px; margin-bottom: 12px; }
    #formBand button, button { padding: 8px 16px; cursor: pointer; border-radius: 4px; border:none; }
    #formBand button { background: #3498db; color: white; }
    #mainApp button.action-btn, #detailPage button.action-btn { background: #3498db; color: white; }
    #mainApp button.hapus-btn, #detailPage button.hapus-btn { background: #e74c3c; color: white; }
    #mainApp button.action-btn:hover, #detailPage button.action-btn:hover { background: #2980b9; }
    #mainApp button.hapus-btn:hover, #detailPage button.hapus-btn:hover { background: #c0392b; }
    ul#daftarJudul { list-style: none; padding: 0; }
    ul#daftarJudul li { padding: 10px; border-bottom: 1px solid #ddd; cursor: pointer; }
    ul#daftarJudul li:hover { background: #e0e0e0; }
    #detailPage { display: none; }
    .back-btn { background: #7f8c8d; color: white; margin-top: 15px; }
    .back-btn:hover { background: #606b6e; }
    .link-box input[type=text] { width: 100%; padding: 8px; margin-top: 10px; font-family: monospace; }
    .link-box button { margin-top: 10px; }
    #headerBand { text-align: right; margin-bottom: 10px; }
    #headerBand strong { font-size: 1.1rem; }
  </style>
</head>
<body>

<div id="formBand">
  <h1>Masukkan Nama Band Kamu</h1>
  <input type="text" id="inputNamaBand" placeholder="Nama Band" autocomplete="off" />
  <br/>
  <button onclick="submitNamaBand()">Mulai</button>
</div>

<div id="mainApp" style="display:none;">
  <div id="headerBand">
    <strong>Band: <span id="namaBandTampil"></span></strong>
    <button class="hapus-btn" onclick="gantiBand()">Ganti Band</button>
  </div>

  <div>
    <input type="text" id="inputJudulLagu" placeholder="Tambah Judul Lagu Baru" style="width: 80%;" />
    <button class="action-btn" onclick="tambahJudulLagu()">Tambah</button>
  </div>

  <ul id="daftarJudul">
    <!-- Daftar judul lagu akan muncul di sini -->
  </ul>
</div>

<div id="detailPage">
  <h2 id="judulDetail"></h2>
  <div class="link-box">
    <input type="text" id="linkInput" placeholder="Tempel link dari ChordTela di sini" />
    <button id="simpanLinkBtn" class="action-btn" onclick="simpanLink()">Simpan Link</button>
    <button id="editLinkBtn" class="action-btn" onclick="editLink()" style="display:none; margin-left:10px;">Edit Link</button>
    <div id="linkTersimpan" style="margin-top:10px;"></div>
  </div>
  <button class="back-btn" onclick="kembaliKeUtama()">← Kembali</button>
</div>

<script>
  let namaBandAktif = null;
  let daftarLagu = [];
  let indexAktif = null;

  // Nama band default yang sudah punya data tersimpan
  const bandSimpan = "eleventakustik";

  // Data lagu default untuk eleventakustik (contoh bisa kamu isi)
  const dataEleventakustik = [
    { judul: "Cinta Ini Membunuhku", link: "https://chordtela.com/cinta-ini-membunuhku-eleventakustik" },
    { judul: "Menghapus Jejakmu", link: "https://chordtela.com/menghapus-jejakmu-eleventakustik" }
  ];

  // Simpan data ke localStorage berdasarkan nama band
  function simpanData() {
    if (!namaBandAktif) return;
    localStorage.setItem('dataLagu_' + namaBandAktif, JSON.stringify(daftarLagu));
  }

  // Load data dari localStorage berdasarkan nama band
  function loadData() {
    if (!namaBandAktif) return;
    const data = localStorage.getItem('dataLagu_' + namaBandAktif);
    if (data) {
      daftarLagu = JSON.parse(data);
    } else {
      // Jika band eleventakustik dan belum ada data di localStorage, isi data default
      if (namaBandAktif.toLowerCase() === bandSimpan.toLowerCase()) {
        daftarLagu = dataEleventakustik.slice(); // clone array default
        simpanData();
      } else {
        daftarLagu = [];
      }
    }
  }

  // Submit nama band, load data, tampilkan halaman utama
  function submitNamaBand() {
    const bandInput = document.getElementById('inputNamaBand').value.trim();
    if (!bandInput) return alert("Nama band tidak boleh kosong!");
    namaBandAktif = bandInput.toLowerCase();

    localStorage.setItem('namaBandAktif', namaBandAktif);
    document.getElementById('formBand').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';
    document.getElementById('namaBandTampil').textContent = bandInput;

    loadData();
    renderDaftarJudul();
  }

  // Ganti band, kembali ke halaman input nama band
  function gantiBand() {
    if(confirm("Ganti band? Semua data band saat ini akan disimpan. Kamu akan kembali ke halaman input nama band.")) {
      namaBandAktif = null;
      daftarLagu = [];
      localStorage.removeItem('namaBandAktif');
      document.getElementById('mainApp').style.display = 'none';
      document.getElementById('detailPage').style.display = 'none';
      document.getElementById('formBand').style.display = 'block';
      document.getElementById('inputNamaBand').value = '';
      document.getElementById('inputJudulLagu').value = '';
    }
  }

  // Tambah judul lagu baru (tanpa link)
  function tambahJudulLagu() {
    const judul = document.getElementById('inputJudulLagu').value.trim();
    if (!judul) return alert("Judul lagu tidak boleh kosong!");
    // Cek apakah judul sudah ada (case insensitive)
    const sudahAda = daftarLagu.some(l => l.judul.toLowerCase() === judul.toLowerCase());
    if (sudahAda) return alert("Judul lagu sudah ada.");
    daftarLagu.push({ judul: judul, link: "" });
    simpanData();
    document.getElementById('inputJudulLagu').value = '';
    renderDaftarJudul();
  }

  // Render daftar judul lagu (tanpa lirik)
  function renderDaftarJudul() {
    const ul = document.getElementById('daftarJudul');
    ul.innerHTML = '';
    if (daftarLagu.length === 0) {
      ul.innerHTML = '<li><em>Belum ada judul lagu, silakan tambah.</em></li>';
      return;
    }
    daftarLagu.forEach((lagu, index) => {
      const li = document.createElement('li');
      li.textContent = lagu.judul;
      li.title = "Klik untuk lihat dan edit link ChordTela";
      li.onclick = () => bukaDetail(index);
      ul.appendChild(li);
    });
  }

  // Buka halaman detail untuk judul lagu yang diklik
  function bukaDetail(index) {
    indexAktif = index;
    const lagu = daftarLagu[index];

    document.getElementById('mainApp').style.display = 'none';
    document.getElementById('detailPage').style.display = 'block';

    document.getElementById('judulDetail').textContent = lagu.judul;

    const linkInput = document.getElementById('linkInput');
    const simpanBtn = document.getElementById('simpanLinkBtn');
    const editBtn = document.getElementById('editLinkBtn');

    if (lagu.link && lagu.link.trim() !== "") {
      linkInput.value = lagu.link;
      linkInput.setAttribute('readonly', true);
      simpanBtn.style.display = 'none';
      editBtn.style.display = 'inline-block';
    } else {
      linkInput.value = '';
      linkInput.removeAttribute('readonly');
      simpanBtn.style.display = 'inline-block';
      editBtn.style.display = 'none';
    }

    tampilkanLink();
  }

  // Edit link: aktifkan input untuk diedit
  function editLink() {
    const linkInput = document.getElementById('linkInput');
    const simpanBtn = document.getElementById('simpanLinkBtn');
    const editBtn = document.getElementById('editLinkBtn');

    linkInput.removeAttribute('readonly');
    simpanBtn.style.display = 'inline-block';
    editBtn.style.display = 'none';

    linkInput.focus();
  }

  // Simpan link dan set input jadi readonly lagi
  function simpanLink() {
    const linkInput = document.getElementById('linkInput');
    const link = linkInput.value.trim();
    if (!link) return alert("Link tidak boleh kosong!");
    // Opsional: validasi sederhana URL (boleh dikembangkan)
    if (!link.startsWith('http://') && !link.startsWith('https://')) {
      return alert("Link harus diawali dengan http:// atau https://");
    }
    daftarLagu[indexAktif].link = link;
    simpanData();
    bukaDetail(indexAktif); // Refresh halaman detail supaya input readonly & tombol muncul sesuai status
  }

  // Tampilkan link tersimpan di bawah input
  function tampilkanLink() {
    const lagu = daftarLagu[indexAktif];
    const div = document.getElementById('linkTersimpan');
    if (lagu.link && lagu.link.trim() !== "") {
      div.innerHTML = `<p>Link tersimpan: <a href="${lagu.link}" target="_blank" rel="noopener noreferrer">${lagu.link}</a></p>`;
    } else {
      div.innerHTML = `<p><em>Belum ada link tersimpan.</em></p>`;
    }
  }

  // Kembali ke halaman utama daftar judul
  function kembaliKeUtama() {
    document.getElementById('detailPage').style.display = 'none';
    document.getElementById('mainApp').style.display = 'block';
  }

  // Jika sudah pernah masuk sebelumnya, otomatis load band & data
  window.onload = function() {
    const band = localStorage.getItem('namaBandAktif');
    if (band) {
      namaBandAktif = band;
      document.getElementById('formBand').style.display = 'none';
      document.getElementById('mainApp').style.display = 'block';
      // Tampilkan nama band persis seperti input terakhir user (optional)
      document.getElementById('namaBandTampil').textContent = band;
      loadData();
      renderDaftarJudul();
    }
  };
</script>

</body>
</html>




