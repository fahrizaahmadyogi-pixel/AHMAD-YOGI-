<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aplikasi Absensi Mahasiswa</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to right, #4a90e2, #50c9ce);
      margin: 0;
      padding: 0;
    }
    .container {
      width: 90%;
      max-width: 600px;
      background: white;
      margin: 80px auto;
      border-radius: 15px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.2);
      padding: 30px;
      text-align: center;
    }
    h2 {
      color: #333;
    }
    input, button, select {
      width: 90%;
      padding: 10px;
      margin: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 16px;
    }
    button {
      background: #4a90e2;
      color: white;
      border: none;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover {
      background: #357ab8;
    }
    .hidden { display: none; }
    .fingerprint {
      width: 90px;
      height: 90px;
      border: 3px solid #4a90e2;
      border-radius: 50%;
      margin: 20px auto;
      background: url('https://cdn-icons-png.flaticon.com/512/616/616408.png') no-repeat center/50%;
      animation: pulse 2s infinite;
      cursor: pointer;
    }
    @keyframes pulse {
      0% { box-shadow: 0 0 0 0 rgba(74,144,226, 0.6); }
      70% { box-shadow: 0 0 0 20px rgba(74,144,226, 0); }
      100% { box-shadow: 0 0 0 0 rgba(74,144,226, 0); }
    }
    .absen-success { color: green; font-weight: bold; }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    th {
      background: #4a90e2;
      color: white;
    }
    .jam {
      font-weight: bold;
      color: #4a90e2;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>

  <!-- LOGIN PAGE -->
  <div id="loginPage" class="container">
    <h2>🔐 Login Absensi</h2>
    <div class="jam" id="waktu"></div>
    <input type="text" id="username" placeholder="Masukkan Nama">
    <input type="text" id="nim" placeholder="Masukkan NIM">
    <select id="role">
      <option value="mahasiswa">Mahasiswa</option>
      <option value="dosen">Dosen</option>
    </select>
    <button onclick="login()">Masuk</button>
  </div>

  <!-- DOSEN PAGE -->
  <div id="dosenPage" class="container hidden">
    <h2>👨‍🏫 Dosen Mengajar</h2>
    <div class="jam" id="waktuDosen"></div>
    <p id="namaDosen"></p>
    <table>
      <tr><th>No</th><th>NIM</th><th>Nama Mahasiswa</th><th>Status</th></tr>
      <tbody id="daftarMahasiswa"></tbody>
    </table>
    <button onclick="logout()">Keluar</button>
  </div>

  <!-- MAHASISWA PAGE -->
  <div id="mhsPage" class="container hidden">
    <h2>📋 Absensi Mahasiswa</h2>
    <div class="jam" id="waktuMhs"></div>
    <p id="namaMahasiswa"></p>
    <div class="fingerprint" onclick="absenFingerprint()"></div>
    <p id="statusAbsen"></p>
    <button onclick="logout()">Keluar</button>
  </div>

  <script>
    // ======== Data Mahasiswa ===========
    const mahasiswa = [
      {nim: "23001", nama: "Aulia Febriyanti"},
      {nim: "23002", nama: "Budi Santoso"},
      {nim: "23003", nama: "Citra Dewi"},
      {nim: "23004", nama: "Dimas Saputra"},
      {nim: "23005", nama: "Eka Rahmawati"}
    ];

    let absensi = JSON.parse(localStorage.getItem("absensi")) || {};
    let userLogin = {};

    // ======== Update Jam Real-time ========
    function updateJam() {
      const now = new Date();
      const waktuStr = now.toLocaleDateString('id-ID') + " | " + now.toLocaleTimeString('id-ID');
      document.querySelectorAll('.jam').forEach(el => el.textContent = waktuStr);
    }
    setInterval(updateJam, 1000);

    // ======== LOGIN ===========
    function login() {
      const nama = document.getElementById("username").value.trim();
      const nim = document.getElementById("nim").value.trim();
      const role = document.getElementById("role").value;

      if (!nama || !nim) {
        alert("Nama dan NIM wajib diisi!");
        return;
      }

      userLogin = { nama, nim, role };
      document.getElementById("loginPage").classList.add("hidden");

      if (role === "mahasiswa") {
        const ditemukan = mahasiswa.find(m => m.nim === nim && m.nama.toLowerCase() === nama.toLowerCase());
        if (!ditemukan) {
          alert("NIM atau Nama tidak ditemukan!");
          location.reload();
          return;
        }
        document.getElementById("mhsPage").classList.remove("hidden");
        document.getElementById("namaMahasiswa").innerText = Selamat datang, ${nama};
      } else {
        document.getElementById("dosenPage").classList.remove("hidden");
        document.getElementById("namaDosen").innerText = Dosen: ${nama};
        tampilkanMahasiswa();
      }
    }

    // ======== TAMPILKAN DAFTAR MAHASISWA ========
    function tampilkanMahasiswa() {
      const tbody = document.getElementById("daftarMahasiswa");
      tbody.innerHTML = "";
      mahasiswa.forEach((mhs, i) => {
        let status = absensi[mhs.nim] ? absensi[mhs.nim] : "-";
        tbody.innerHTML += <tr><td>${i+1}</td><td>${mhs.nim}</td><td>${mhs.nama}</td><td>${status}</td></tr>;
      });
    }

    // ======== ABSEN DENGAN FINGERPRINT ========
    function absenFingerprint() {
      const status = document.getElementById("statusAbsen");
      status.innerHTML = "🔍 Sidik jari sedang dipindai...";
      setTimeout(() => {
        absensi[userLogin.nim] = "Hadir (" + new Date().toLocaleTimeString('id-ID') + ")";
        localStorage.setItem("absensi", JSON.stringify(absensi));
        status.innerHTML = "✅ Absen Berhasil! Terima kasih.";
        status.classList.add("absen-success");
      }, 2000);
    }

    // ======== LOGOUT ========
    function logout() {
      document.getElementById("loginPage").classList.remove("hidden");
      document.getElementById("mhsPage").classList.add("hidden");
      document.getElementById("dosenPage").classList.add("hidden");
      userLogin = {};
    }
  </script>

</body>
</html>
