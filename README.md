<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Daftar Lagu & Link ChordTela</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f0f0f0; margin: 20px; }
    #main { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px #aaa; }
    h1 { text-align: center; }
    input[type="text"] { width: 80%; padding: 10px; margin: 10px 0; }
    button { padding: 10px 16px; border: none; border-radius: 5px; background: #3498db; color: white; cursor: pointer; }
    button:hover { background: #2980b9; }
    ul { list-style: none; padding: 0; }
    li { background: #f9f9f9; margin-bottom: 10px; padding: 10px; border-radius: 5px; box-shadow: 0 0 5px #ccc; }
    a { text-decoration: none; color: #2c3e50; font-weight: bold; }
    a:hover { text-decoration: underline; }
    .link-form { margin-top: 10px; }
    .link-form input { width: 100%; padding: 8px; }
    .link-area { margin-top: 10px; }
  </style>
</head>
<body>

<div id="main">
  <h1>Daftar Judul Lagu</h1>

  <input type="text" id="judulInput" placeholder="Masukkan judul lagu..." />
  <button onclick="tambahJudul()">Tambah Judul</button>

  <ul id="daftarJudul"></ul>
</div>

<script>
  let daftarLagu = [];

  function tambahJudul() {
    const judul = document.getElementById('judulInput').value.trim();
    if (!judul) return alert("Judul lagu tidak boleh kosong!");
    daftarLagu.push({ judul: judul, link: "" });
    document.getElementById('judulInput').value = '';
    simpanData();
    renderDaftar();
  }

  function renderDaftar() {
    const ul = document.getElementById('daftarJudul');
    ul.innerHTML = '';
    daftarLagu.forEach((lagu, index) => {
      const li = document.createElement('li');
      li.innerHTML = `<a href="#" onclick="bukaLink(${index}); return false;">${lagu.judul}</a>`;
      ul.appendChild(li);
    });
  }

  function bukaLink(index) {
    const lagu = daftarLagu[index];
    const link = prompt(`Masukkan/paste link ChordTela untuk: "${lagu.judul}"`, lagu.link || `https://www.chordtela.com/?s=${encodeURIComponent(lagu.judul)}`);
    if (link) {
      daftarLagu[index].link = link;
      simpanData();
      window.open(link, '_blank');
    }
  }

  function simpanData() {
    localStorage.setItem('daftarLagu', JSON.stringify(daftarLagu));
  }

  function loadData() {
    const data = localStorage.getItem('daftarLagu');
    if (data) daftarLagu = JSON.parse(data);
  }

  window.onload = function() {
    loadData();
    renderDaftar();
  };
</script>

</body>
</html>


