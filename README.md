<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Judul Lagu & Link ChordTela</title>
  <style>
    body { font-family: Arial; background: #f0f0f0; margin: 20px; }
    #main, #detailPage {
      max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 8px;
      box-shadow: 0 0 10px #aaa; display: none;
    }
    h1, h2 { text-align: center; }
    input[type="text"] { width: 80%; padding: 10px; margin: 10px 0; }
    button {
      padding: 10px 16px; border: none; border-radius: 5px;
      background: #3498db; color: white; cursor: pointer;
    }
    button:hover { background: #2980b9; }
    ul { list-style: none; padding: 0; }
    li { background: #f9f9f9; margin-bottom: 10px; padding: 10px; border-radius: 5px; box-shadow: 0 0 5px #ccc; }
    a { text-decoration: none; color: #2c3e50; font-weight: bold; }
    a:hover { text-decoration: underline; }
    .back-btn { margin-top: 20px; display: inline-block; background: #e74c3c; }
    .back-btn:hover { background: #c0392b; }
    .link-box { margin-top: 15px; }
    .link-box input { width: 100%; padding: 10px; }
    .link-box a { display: inline-block; margin-top: 10px; color: green; }
  </style>
</head>
<body>

<div id="main">
  <h1>Daftar Judul Lagu</h1>
  <input type="text" id="judulInput" placeholder="Masukkan judul lagu..." />
  <button onclick="tambahJudul()">Tambah Judul</button>

  <ul id="daftarJudul"></ul>
</div>

<div id="detailPage">
  <h2 id="judulDetail"></h2>

  <div class="link-box">
    <input type="text" id="linkInput" placeholder="Tempel link dari ChordTela di sini" />
    <button onclick="simpanLink()">Simpan Link</button>
    <div id="linkTersimpan" style="margin-top:10px;"></div>
  </div>

  <button class="back-btn" onclick="kembaliKeUtama()">← Kembali</button>
</div>

<script>
  let daftarLagu = [];
  let indexAktif = null;

  function simpanData() {
    localStorage.setItem('daftarLagu', JSON.stringify(daftarLagu));
  }

  function loadData() {
    const data = localStorage.getItem('daftarLagu');
    if (data) daftarLagu = JSON.parse(data);
  }

  function tambahJudul() {
    const judul = document.getElementById('judulInput').value.trim();
    if (!judul) return alert("Judul lagu tidak boleh kosong!");
    daftarLagu.push({ judul, link: "" });
    document.getElementById('judulInput').value = '';
    simpanData();
    renderDaftar();
  }

  function renderDaftar() {
    const ul = document.getElementById('daftarJudul');
    ul.innerHTML = '';
    daftarLagu.forEach((lagu, index) => {
      const li = document.createElement('li');
      li.innerHTML = `<a href="#" onclick="bukaDetail(${index}); return false;">${lagu.judul}</a>`;
      ul.appendChild(li);
    });
  }

  function bukaDetail(index) {
    indexAktif = index;
    const lagu = daftarLagu[index];

    document.getElementById('main').style.display = 'none';
    document.getElementById('detailPage').style.display = 'block';

    document.getElementById('judulDetail').textContent = lagu.judul;
    document.getElementById('linkInput').value = lagu.link || '';
    tampilkanLink();
  }

  function tampilkanLink() {
    const lagu = daftarLagu[indexAktif];
    const div = document.getElementById('linkTersimpan');
    if (lagu.link) {
      div.innerHTML = `<p>Link tersimpan: <a href="${lagu.link}" target="_blank">${lagu.link}</a></p>`;
    } else {
      div.innerHTML = `<p><em>Belum ada link tersimpan.</em></p>`;
    }
  }

  function simpanLink() {
    const link = document.getElementById('linkInput').value.trim();
    if (!link) return alert("Link tidak boleh kosong!");
    daftarLagu[indexAktif].link = link;
    simpanData();
    tampilkanLink();
  }

  function kembaliKeUtama() {
    document.getElementById('main').style.display = 'block';
    document.getElementById('detailPage').style.display = 'none';
    indexAktif = null;
    renderDaftar();
  }

  window.onload = function() {
    loadData();
    renderDaftar();
    document.getElementById('main').style.display = 'block';
  };
</script>

</body>
</html>



