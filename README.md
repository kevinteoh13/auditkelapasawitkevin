<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Aplikasi Audit Penghasilan Kelapa Sawit</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 20px;
      background-color: #f4f4f9;
    }
    .container {
      max-width: 1200px;
      margin: 0 auto;
    }
    .form-group {
      margin-bottom: 10px;
    }
    .form-group input, .form-group select {
      width: 100%;
      padding: 8px;
    }
    .btn {
      padding: 10px 20px;
      background-color: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
      margin-top: 10px;
      border-radius: 5px;
    }
    .btn-danger {
      background-color: #f44336;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: center;
    }
    .login-container, .register-container {
      text-align: center;
      margin-top: 50px;
    }
    .login-container input, .register-container input {
      width: 250px;
    }
    .toast {
      background: #333;
      color: #fff;
      padding: 10px 20px;
      position: fixed;
      bottom: 30px;
      right: 30px;
      border-radius: 5px;
      display: none;
      z-index: 1000;
    }
  </style>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
</head>

<body>
<div class="container">
  <h1>Aplikasi Audit Penghasilan Kelapa Sawit</h1>

  <!-- Register -->
  <div id="registerForm" class="register-container">
    <h2>Register</h2>
    <div class="form-group">
      <input type="email" id="regUsername" placeholder="Email">
    </div>
    <div class="form-group">
      <input type="password" id="regPassword" placeholder="Password">
    </div>
    <button class="btn" onclick="register()">Register</button>
    <p id="registerError" style="color: red;"></p>
    <p>Sudah punya akun? <a href="#" onclick="showLogin()">Login</a></p>
  </div>

  <!-- Login -->
  <div id="loginForm" class="login-container" style="display:none;">
    <h2>Login</h2>
    <div class="form-group">
      <input type="email" id="username" placeholder="Email">
    </div>
    <div class="form-group">
      <input type="password" id="password" placeholder="Password">
    </div>
    <button class="btn" onclick="login()">Login</button>
    <p id="loginError" style="color: red;"></p>
    <p>Belum punya akun? <a href="#" onclick="showRegister()">Register</a></p>
  </div>

  <!-- Main App -->
  <div id="app" style="display:none;">
    <h2>Tambah Data Panen</h2>
    <div class="form-group">
      <input type="date" id="tanggal" placeholder="Tanggal Panen">
    </div>
    <div class="form-group">
      <input type="text" id="bpMobil" placeholder="BP Mobil">
    </div>
    <div class="form-group">
      <input type="number" id="berat" placeholder="Berat (Kg)">
    </div>
    <div class="form-group">
      <input type="number" id="harga" placeholder="Harga per Kg">
    </div>
    <button class="btn" onclick="addData()">Tambah Data</button>

    <h2>Data Panen</h2>
    <div class="form-group">
      <select id="filterBulan" onchange="renderData()">
        <option value="all">Tampilkan Semua Bulan</option>
      </select>
    </div>

    <table id="dataTable">
      <thead>
        <tr>
          <th>Tanggal</th>
          <th>BP Mobil</th>
          <th>Berat (Kg)</th>
          <th>Harga</th>
          <th>Penghasilan Kotor</th>
          <th>Setelah PPn</th>
          <th>Biaya Bongkar</th>
          <th>Penghasilan Bersih</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>

    <div id="summary" style="margin-top:20px; font-weight:bold;"></div>

    <button class="btn btn-danger" onclick="logout()">Logout</button>
    <button class="btn" onclick="exportToExcel()">Export Excel</button>
  </div>
</div>

<div class="toast" id="toast"></div>

<!-- Firebase -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
  import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";
  import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc, query, where, orderBy } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

  const firebaseConfig = {
    apiKey: "AIzaSyB-mQdV7yaESX7fIfCWp9vlPKX3ga74xwY",
    authDomain: "kelapa-sawit-audit.firebaseapp.com",
    projectId: "kelapa-sawit-audit",
    storageBucket: "kelapa-sawit-audit.appspot.com",
    messagingSenderId: "909941962760",
    appId: "1:909941962760:web:81c936109aca6a210939bd",
    measurementId: "G-M45XQGFT51"
  };

  const app = initializeApp(firebaseConfig);
  const auth = getAuth(app);
  const db = getFirestore(app);

  const toast = (msg) => {
    const t = document.getElementById('toast');
    t.innerText = msg;
    t.style.display = 'block';
    setTimeout(() => { t.style.display = 'none'; }, 3000);
  };

  window.register = async function () {
    const email = document.getElementById('regUsername').value;
    const password = document.getElementById('regPassword').value;
    try {
      await createUserWithEmailAndPassword(auth, email, password);
      toast("Berhasil register");
      showLogin();
    } catch (e) {
      document.getElementById('registerError').innerText = e.message;
    }
  };

  window.login = async function () {
    const email = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    try {
      await signInWithEmailAndPassword(auth, email, password);
      document.getElementById('loginError').innerText = '';
      toast("Berhasil login");
    } catch (e) {
      document.getElementById('loginError').innerText = e.message;
    }
  };

  window.logout = async function () {
    await signOut(auth);
    toast("Berhasil logout");
  };

  window.addData = async function () {
    const tanggal = document.getElementById('tanggal').value;
    const bpMobil = document.getElementById('bpMobil').value;
    const berat = parseFloat(document.getElementById('berat').value);
    const harga = parseFloat(document.getElementById('harga').value);
    const user = auth.currentUser;

    if (!tanggal || !bpMobil || berat <= 0 || harga <= 0) {
      alert("Mohon isi semua data dengan benar.");
      return;
    }

    const penghasilanKotor = berat * harga;
    const afterPPN = penghasilanKotor * 0.9975;
    const biayaBongkar = berat * 18;
    const bersih = afterPPN - biayaBongkar;

    await addDoc(collection(db, "panen"), {
      uid: user.uid,
      tanggal: new Date(tanggal),
      bpMobil,
      berat,
      harga,
      penghasilanKotor,
      afterPPN,
      biayaBongkar,
      bersih
    });

    toast("Data berhasil ditambahkan");
    renderData();
  };

  window.renderData = async function () {
    const user = auth.currentUser;
    if (!user) return;

    const q = query(collection(db, "panen"), where("uid", "==", user.uid), orderBy("tanggal", "asc"));
    const snapshot = await getDocs(q);
    const tbody = document.querySelector("#dataTable tbody");
    const filter = document.getElementById("filterBulan").value;
    tbody.innerHTML = "";

    let totalKotor = 0, totalBersih = 0;
    let bulanTahunSet = new Set();

    snapshot.forEach(docu => {
      const d = docu.data();
      const tanggalObj = new Date(d.tanggal.seconds * 1000);
      const bulanTahun = tanggalObj.getMonth() + 1 + "-" + tanggalObj.getFullYear();

      bulanTahunSet.add(bulanTahun);

      if (filter === "all" || filter === bulanTahun) {
        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${tanggalObj.toLocaleDateString()}</td>
          <td>${d.bpMobil}</td>
          <td>${d.berat.toLocaleString()}</td>
          <td>${d.harga.toLocaleString()}</td>
          <td>${d.penghasilanKotor.toLocaleString()}</td>
          <td>${d.afterPPN.toLocaleString()}</td>
          <td>${d.biayaBongkar.toLocaleString()}</td>
          <td>${d.bersih.toLocaleString()}</td>
          <td><button class="btn btn-danger" onclick="deleteData('${docu.id}')">Delete</button></td>
        `;
        tbody.appendChild(tr);

        totalKotor += d.penghasilanKotor;
        totalBersih += d.bersih;
      }
    });

    document.getElementById("summary").innerText =
      `Total Kotor: Rp ${totalKotor.toLocaleString()} | Total Bersih: Rp ${totalBersih.toLocaleString()}`;

    // Update Filter Dropdown
    const select = document.getElementById("filterBulan");
    select.innerHTML = `<option value="all">Tampilkan Semua Bulan</option>`;
    bulanTahunSet.forEach(bulan => {
      select.innerHTML += `<option value="${bulan}">${bulan}</option>`;
    });
  };

  window.deleteData = async function (id) {
    if (confirm("Yakin ingin hapus data?")) {
      await deleteDoc(doc(db, "panen", id));
      toast("Data berhasil dihapus");
      renderData();
    }
  };

  window.showLogin = function () {
    document.getElementById('loginForm').style.display = 'block';
    document.getElementById('registerForm').style.display = 'none';
  };

  window.showRegister = function () {
    document.getElementById('registerForm').style.display = 'block';
    document.getElementById('loginForm').style.display = 'none';
  };

  auth.onAuthStateChanged(user => {
    if (user) {
      document.getElementById('loginForm').style.display = 'none';
      document.getElementById('registerForm').style.display = 'none';
      document.getElementById('app').style.display = 'block';
      renderData();
    } else {
      document.getElementById('app').style.display = 'none';
      document.getElementById('loginForm').style.display = 'block';
    }
  });

  window.exportToExcel = function () {
    const table = document.getElementById('dataTable');
    const wb = XLSX.utils.table_to_book(table, { sheet: "Data Panen" });
    XLSX.writeFile(wb, "data_kelapa_sawit.xlsx");
  };
</script>

</body>
</html>
