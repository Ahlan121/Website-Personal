<!DOCTYPE html>
<html lang="id">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Daftar Lagu per Album</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f7fa;
            padding: 20px;
        }
        
        h1,
        h2 {
            color: #2c3e50;
        }
        
        input,
        textarea,
        button {
            padding: 10px;
            margin: 5px 0;
            font-size: 16px;
        }
        
        button {
            background-color: #3498db;
            color: white;
            border: none;
            cursor: pointer;
            margin-right: 5px;
        }
        
        button:hover {
            background-color: #2980b9;
        }
        
        .hapus-btn {
            background-color: #e74c3c;
        }
        
        .hapus-btn:hover {
            background-color: #c0392b;
        }
        
        .album {
            background: white;
            border-radius: 8px;
            padding: 15px;
            margin-top: 20px;
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
        }
        
        .lagu {
            margin-bottom: 10px;
            border-bottom: 1px solid #ccc;
            padding-bottom: 8px;
        }
        
        .lirik {
            white-space: pre-line;
            background-color: #ecf0f1;
            padding: 8px;
            border-radius: 5px;
            margin-top: 5px;
        }
        
        #formLagu {
            display: none;
            margin-top: 20px;
            background: #fff;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 1px 5px rgba(0, 0, 0, 0.2);
        }
        
        #formLagu input,
        #formLagu textarea {
            width: 100%;
            margin-bottom: 10px;
            box-sizing: border-box;
        }
        
        textarea {
            resize: vertical;
        }
    </style>
</head>

<body>

    <h1>🎵 Daftar Lagu & Album</h1>

    <h2>Buat Album Baru</h2>
    <input type="text" id="namaAlbum" placeholder="Nama album">
    <button onclick="buatAlbum()">Tambah Album</button>

    <div id="albumContainer"></div>

    <!-- Form Tambah Lagu -->
    <div id="formLagu">
        <h3>Tambah Lagu ke Album: <span id="namaAlbumForm"></span></h3>
        <input type="text" id="judulLaguInput" placeholder="Judul lagu">
        <textarea id="lirikLaguInput" placeholder="Lirik lagu" rows="10"></textarea>
        <button onclick="simpanLagu()">Simpan Lagu</button>
        <button class="hapus-btn" onclick="tutupForm()">Batal</button>
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
                dataAlbum = JSON.parse(data);
            }
        }

        function buatAlbum() {
            const nama = document.getElementById('namaAlbum').value.trim();
            if (nama === '') return alert("Nama album tidak boleh kosong!");
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
        }

        function simpanLagu() {
            const judul = document.getElementById('judulLaguInput').value.trim();
            const lirik = document.getElementById('lirikLaguInput').value.trim();
            if (!judul) {
                alert("Judul lagu harus diisi.");
                return;
            }

            dataAlbum[indexAlbumAktif].lagu.push({
                judul,
                lirik
            });
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

        function editLirik(indexAlbum, indexLagu) {
            const lagu = dataAlbum[indexAlbum].lagu[indexLagu];
            const lirikBaru = prompt("Edit lirik:", lagu.lirik);
            if (lirikBaru !== null) {
                lagu.lirik = lirikBaru;
                simpanKeLocalStorage();
                renderAlbum();
            }
        }

        function renderAlbum() {
            const container = document.getElementById('albumContainer');
            container.innerHTML = '';

            dataAlbum.forEach((album, indexAlbum) => {
                const div = document.createElement('div');
                div.className = 'album';

                let html = `<h2>${album.nama}</h2>`;
                html += `<button onclick="tambahLagu(${indexAlbum})">+ Tambah Lagu</button>`;

                if (album.lagu.length === 0) {
                    html += `<p><em>Belum ada lagu.</em></p>`;
                } else {
                    album.lagu.forEach((lagu, indexLagu) => {
                        html += `
            <div class="lagu">
              <strong>${lagu.judul}</strong>
              <div class="lirik">${lagu.lirik || '<em>(Tidak ada lirik)</em>'}</div>
              <button onclick="editLirik(${indexAlbum}, ${indexLagu})">Edit Lirik</button>
              <button class="hapus-btn" onclick="hapusLagu(${indexAlbum}, ${indexLagu})">Hapus Lagu</button>
            </div>
          `;
                    });
                }

                div.innerHTML = html;
                container.appendChild(div);
            });
        }

        // Jalankan saat pertama kali halaman dibuka
        window.onload = function() {
            loadDariLocalStorage();
            renderAlbum();
        };
    </script>


</body>

</html>
