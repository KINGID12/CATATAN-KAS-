
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Aplikasi Uang Kas</title>
  <!-- QRCodeJS -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
  <!-- XLSX -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
  <style>
    body { 
      font-family: "Segoe UI", sans-serif; 
      margin: 20px; 
      background: #f5f7fa;
      color: #333;
    }
    h2 { text-align: center; margin-bottom: 20px; }
    .hidden { display: none; }
    .card { 
      background: #fff; 
      border-radius: 12px; 
      padding: 15px; 
      margin: 15px auto; 
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      max-width: 600px;
    }
    .summary { 
      display: flex; 
      gap: 10px; 
      margin-bottom: 15px; 
      flex-wrap: wrap;
    }
    .summary div { 
      flex: 1; 
      padding: 10px; 
      border-radius: 8px; 
      text-align: center; 
      font-weight: bold;
    }
    .masuk { background: #d4edda; color: #155724; }
    .keluar { background: #f8d7da; color: #721c24; }
    .saldo { background: #d1ecf1; color: #0c5460; }
    a.bukti-link { 
      display: inline-block; 
      margin-top: 5px; 
      font-size: 13px; 
      color: #007bff; 
      text-decoration: underline; 
      cursor: pointer;
    }
    table { 
      width: 100%; 
      border-collapse: collapse; 
      margin-top: 10px; 
      font-size: 14px;
    }
    table, th, td { 
      border: 1px solid #ccc; 
      padding: 6px; 
      text-align: center; 
    }
    input, select, button {
      margin: 5px 0;
      padding: 8px;
      border-radius: 6px;
      border: 1px solid #ccc;
      width: 100%;
      font-size: 14px;
    }
    button {
      background: #007bff;
      color: white;
      cursor: pointer;
      border: none;
      transition: 0.3s;
    }
    button:hover { background: #0056b3; }
    #qrUmum { 
      margin: 10px auto; 
      width: 150px; 
      height: 150px; 
    }

    /* Toast Notification */
    #toast {
      visibility: hidden;
      min-width: 250px;
      background: #333;
      color: #fff;
      text-align: center;
      border-radius: 6px;
      padding: 12px;
      position: fixed;
      z-index: 1000;
      right: 20px;
      top: 20px;
      font-size: 14px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
      opacity: 0;
      transition: opacity 0.5s, top 0.5s;
    }
    #toast.show {
      visibility: visible;
      opacity: 1;
      top: 40px;
    }
    #toast.success { background: #28a745; }  /* hijau */
    #toast.info { background: #007bff; }     /* biru */
    #toast.error { background: #dc3545; }    /* merah */
  </style>
</head>
<body>
  <h2>📒 Catatan Pembayaran Uang Kas</h2>

  <!-- Login Admin -->
  <div id="loginPanel" class="card">
    <h3>🔑 Login Admin</h3>
    <input type="password" id="adminPassword" placeholder="Password" />
    <button onclick="loginAdmin()">Login</button>
  </div>

  <!-- Dashboard Admin -->
  <div id="adminPanel" class="hidden">
    <div class="summary">
      <div class="masuk">Total Masuk<br>Rp<span id="totalMasuk">0</span></div>
      <div class="keluar">Total Keluar<br>Rp<span id="totalKeluar">0</span></div>
      <div class="saldo">Saldo<br>Rp<span id="saldo">0</span></div>
    </div>

    <div class="card">
      <h4>➕ Tambah Transaksi</h4>
      <input type="text" id="namaPeserta" placeholder="Nama Peserta" />
      <input type="number" id="jumlah" placeholder="Jumlah" />
      <select id="jenis">
        <option value="masuk">Uang Masuk</option>
        <option value="keluar">Uang Keluar</option>
      </select>
      <button onclick="tambahTransaksi()">Tambah</button>
    </div>

    <div class="card">
      <h4>📋 Daftar Transaksi</h4>
      <div id="transaksiList"></div>
      <button onclick="exportExcel()">Export Excel</button>
    </div>

    <div class="card">
      <h4>📆 Rekapan Bulanan</h4>
      <div id="rekapBulanan"></div>
    </div>

    <div class="card">
      <h4>🔒 Ubah Password Admin</h4>
      <input type="password" id="oldPw" placeholder="Password Lama" />
      <input type="password" id="newPw" placeholder="Password Baru" />
      <button onclick="ubahPassword()">Ubah Password</button>
      <br><br>
      <button style="background:#dc3545;" onclick="logoutAdmin()">Logout</button>
    </div>

    <div class="card">
      <h4>📲 QR Umum (Scan untuk Konfirmasi)</h4>
      <p style="font-size:14px;">Scan QR berikut untuk konfirmasi pembayaran uang kas</p>
      <div id="qrUmum"></div>
    </div>
  </div>

  <!-- Form Konfirmasi Peserta -->
  <div id="pesertaForm" class="hidden card">
    <h3>📝 Form Konfirmasi Pembayaran</h3>
    <input type="text" id="konfirmasiNama" placeholder="Nama" />
    <input type="number" id="konfirmasiJumlah" placeholder="Jumlah Bayar" />
    <input type="file" id="buktiUpload" accept="image/*" />
    <button onclick="kirimKonfirmasi()">Kirim</button>
  </div>

  <!-- Toast -->
  <div id="toast"></div>

  <script>
    let passwordAdmin = localStorage.getItem("passwordAdmin") || "MERDEKA321";
    let transaksi = JSON.parse(localStorage.getItem("transaksi")) || [];

    function loginAdmin(){
      const pw = document.getElementById("adminPassword").value;
      if(pw === passwordAdmin){
        document.getElementById("loginPanel").classList.add("hidden");
        document.getElementById("adminPanel").classList.remove("hidden");
        renderTransaksi();
        renderRekap();
      } else {
        alert("Password salah!");
      }
    }

    function logoutAdmin(){
      document.getElementById("adminPanel").classList.add("hidden");
      document.getElementById("loginPanel").classList.remove("hidden");
    }

    function tambahTransaksi(){
      const nama = document.getElementById("namaPeserta").value;
      const jumlah = parseInt(document.getElementById("jumlah").value);
      const jenis = document.getElementById("jenis").value;
      const tgl = new Date().toISOString().slice(0,10);
      if(!nama || !jumlah){ alert("Lengkapi data!"); return; }
      transaksi.push({ nama, jumlah, jenis, tanggal: tgl, status:(jenis==='masuk'?'Menunggu Verifikasi':'-') });
      localStorage.setItem("transaksi", JSON.stringify(transaksi));
      renderTransaksi();
      renderRekap();
      showToast(`Transaksi ${jenis} ditambahkan`, "info");
    }

    function renderTransaksi(){
      const div = document.getElementById("transaksiList");
      div.innerHTML = "";
      let totalMasuk=0, totalKeluar=0;
      transaksi.forEach((t,i)=>{
        if(t.jenis==='masuk') totalMasuk+=t.jumlah;
        else totalKeluar+=t.jumlah;
        let html = `<div style="margin-bottom:8px; text-align:left;">
          <b>${t.nama}</b> - Rp${t.jumlah} (${t.jenis})<br>
          Tanggal: ${t.tanggal}<br>
          Status: <i>${t.status}</i>
          ${t.bukti?`<br><a href="${t.bukti}" target="_blank" class="bukti-link">📎 Lihat Bukti</a>`:''}
          ${t.status!=='Lunas' && t.jenis==='masuk'?`<br><button onclick="verifikasi(${i})">Verifikasi</button>`:''}
          <button style="background:#dc3545; margin-left:5px;" onclick="hapus(${i})">Hapus</button>
        </div><hr>`;
        div.innerHTML += html;
      });
      document.getElementById("totalMasuk").innerText = totalMasuk;
      document.getElementById("totalKeluar").innerText = totalKeluar;
      document.getElementById("saldo").innerText = totalMasuk - totalKeluar;
    }

    function verifikasi(i){
      transaksi[i].status = "Lunas";
      localStorage.setItem("transaksi", JSON.stringify(transaksi));
      renderTransaksi();
      renderRekap();
      showToast(`Transaksi ${transaksi[i].nama} diverifikasi`, "info");
    }

    function hapus(i){
      const nama = transaksi[i].nama;
      transaksi.splice(i,1);
      localStorage.setItem("transaksi", JSON.stringify(transaksi));
      renderTransaksi();
      renderRekap();
      showToast(`Transaksi ${nama} dihapus`, "error");
    }

    function generateQRUmum(){
      const container = document.getElementById("qrUmum");
      container.innerHTML = "";
      new QRCode(container, {
        text: window.location.href.split("#")[0] + "#konfirmasi",
        width: 150,
        height: 150
      });
    }

    function kirimKonfirmasi(){
      const nama = document.getElementById("konfirmasiNama").value;
      const jumlah = parseInt(document.getElementById("konfirmasiJumlah").value);
      const file = document.getElementById("buktiUpload").files[0];
      const tgl = new Date().toISOString().slice(0,10);
      if(!nama || !jumlah || !file){ alert("Lengkapi data!"); return; }
      const reader = new FileReader();
      reader.onload = function(e){
        transaksi.push({ 
          nama, 
          jumlah, 
          jenis:'masuk', 
          tanggal: tgl, 
          status:'Menunggu Verifikasi', 
          bukti:e.target.result 
        });
        localStorage.setItem("transaksi", JSON.stringify(transaksi));

        renderTransaksi();
        renderRekap();
        showToast(`Konfirmasi baru dari ${nama}`, "success");
        alert("Konfirmasi terkirim!");
      }
      reader.readAsDataURL(file);
    }

    function renderRekap(){
      const div = document.getElementById("rekapBulanan");
      let rekap = {};
      transaksi.forEach(t=>{
        const bulan = t.tanggal.slice(0,7);
        if(!rekap[bulan]) rekap[bulan] = { masuk:0, keluar:0 };
        if(t.jenis==='masuk') rekap[bulan].masuk += t.jumlah;
        else rekap[bulan].keluar += t.jumlah;
      });
      let html = "<table><tr><th>Bulan</th><th>Total Masuk</th><th>Total Keluar</th><th>Saldo</th></tr>";
      for(const b in rekap){
        html += `<tr><td>${b}</td><td>Rp${rekap[b].masuk}</td><td>Rp${rekap[b].keluar}</td><td>Rp${rekap[b].masuk - rekap[b].keluar}</td></tr>`;
      }
      html += "</table>";
      div.innerHTML = html;
    }

    function ubahPassword(){
      const oldPw = document.getElementById("oldPw").value;
      const newPw = document.getElementById("newPw").value;
      if(oldPw !== passwordAdmin){ alert("Password lama salah!"); return; }
      if(!newPw){ alert("Password baru tidak boleh kosong!"); return; }
      passwordAdmin = newPw;
      localStorage.setItem("passwordAdmin", newPw);
      alert("Password berhasil diubah!");
    }

    function exportExcel(){
      const ws = XLSX.utils.json_to_sheet(transaksi);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "Transaksi");
      XLSX.writeFile(wb, "transaksi_uangkas.xlsx");
    }

    function showToast(msg, type="info"){
      const toast = document.getElementById("toast");
      toast.className = `show ${type}`;
      toast.innerText = msg;
      setTimeout(()=>{ toast.className = toast.className.replace("show", ""); }, 3000);
    }

    // jalankan saat halaman dibuka
    window.onload = function(){
      generateQRUmum();
      if(window.location.hash==="#konfirmasi"){
        document.getElementById("loginPanel").classList.add("hidden");
        document.getElementById("pesertaForm").classList.remove("hidden");
      }
    }

    // ✅ Auto sync peserta → admin tanpa reload
    window.addEventListener("storage", function(e){
      if(e.key === "transaksi"){
        transaksi = JSON.parse(localStorage.getItem("transaksi")) || [];
        renderTransaksi();
        renderRekap();
        showToast("📢 Ada konfirmasi baru masuk!", "success");
      }
    });
  </script>
</body>
</html>
