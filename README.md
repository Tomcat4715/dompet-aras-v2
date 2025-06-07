<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Dompet ARAS v2</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <header>
    <h1>Dompet ARAS v2</h1>
  </header>
  <main>
    <section class="login">
      <h2>Masuk</h2>
      <input type="text" id="username" placeholder="Nama Pengguna" />
      <button onclick="login()">Masuk</button>
    </section>

    <section class="dashboard" style="display:none">
      <h2>Halo, <span id="userDisplay"></span>!</h2>
      <p>Saldo: Rp <span id="balance">0</span></p>

      <div class="actions">
        <button onclick="showTopUpModal()">💰 Top-Up</button>
        <button onclick="showTransferModal()">💸 Transfer</button>
        <button onclick="showScanQRModal()">📷 Scan QR Code</button>
        <button onclick="showPulsaModal()">📱 Beli Pulsa</button>
        <button onclick="showListrikModal()">💡 Bayar Listrik</button>
        <button onclick="showAirModal()">🚿 Bayar Air</button>
        <button onclick="showHargaModal()">♻️ Cek Harga</button>
        <button onclick="showHistoryModal()">📊 Riwayat</button>
        <button onclick="logout()">Keluar</button>
      </div>
    </section>

    <!-- Modal umum -->
    <div class="modal" id="modal">
      <div class="modal-content">
        <span class="close" onclick="closeModal()">&times;</span>
        <div id="modal-body"></div>
      </div>
    </div>
  </main>

  <!-- Library QR Code Scanner -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

  <!-- Library jsPDF untuk export PDF -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background: #e8f5e9;
  color: #2e7d32;
}

header {
  background-color: #4caf50;
  color: white;
  padding: 1rem;
  text-align: center;
}

main {
  padding: 1rem;
  max-width: 480px;
  margin: auto;
}

.login, .dashboard {
  background: white;
  padding: 1rem;
  border-radius: 6px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  margin-bottom: 1rem;
}

input[type="text"], input[type="number"], select {
  width: 100%;
  padding: 0.5rem;
  margin-top: 0.3rem;
  margin-bottom: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 0.5rem;
}

button:hover {
  background-color: #388e3c;
}

.actions button {
  margin: 0.2rem 0.2rem 0.2rem 0;
  width: 100%;
  font-size: 1rem;
}

.modal {
  position: fixed;
  z-index: 1000;
  left: 0; top: 0;
  width: 100%; height: 100%;
  overflow: auto;
  background-color: rgba(0,0,0,0.4);
  display: none;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 1rem;
  border-radius: 6px;
  max-width: 400px;
  width: 90%;
  position: relative;
}

.close {
  position: absolute;
  top: 0.3rem;
  right: 0.5rem;
  font-size: 1.5rem;
  font-weight: bold;
  color: #4caf50;
  cursor: pointer;
}
// Data user dan transaksi tersimpan di localStorage
let users = JSON.parse(localStorage.getItem('users')) || {
  'user1': { saldo: 100000 },
  'user2': { saldo: 50000 }
};

let transactions = JSON.parse(localStorage.getItem('transactions')) || [];

let currentUser = null;

function saveUsers() {
  localStorage.setItem('users', JSON.stringify(users));
}

function saveTransactions() {
  localStorage.setItem('transactions', JSON.stringify(transactions));
}

function login() {
  const username = document.getElementById('username').value.trim();
  if (!username) {
    alert('Masukkan nama pengguna');
    return;
  }
  if (!users[username]) {
    users[username] = { saldo: 0 };
    saveUsers();
  }
  currentUser = username;
  document.querySelector('.login').style.display = 'none';
  document.querySelector('.dashboard').style.display = 'block';
  updateDashboard();
}

function logout() {
  currentUser = null;
  document.querySelector('.dashboard').style.display = 'none';
  document.querySelector('.login').style.display = 'block';
  document.getElementById('username').value = '';
}

function updateDashboard() {
  document.getElementById('userDisplay').innerText = currentUser;
  document.getElementById('balance').innerText = users[currentUser].saldo.toLocaleString('id-ID');
}

function showModal(html) {
  document.getElementById('modal-body').innerHTML = html;
  document.getElementById('modal').style.display = 'flex';
}

function closeModal() {
  document.getElementById('modal').style.display = 'none';
}

// TOP UP
function showTopUpModal() {
  const html = `
    <h3>Top-Up Saldo</h3>
    <label for="topupAmount">Jumlah (Rp):</label>
    <input type="number" id="topupAmount" min="1000" placeholder="Minimal Rp 1000" />
    <button onclick="topUp()">Isi Saldo</button>
  `;
  showModal(html);
}

function topUp() {
  const amount = parseInt(document.getElementById('topupAmount').value, 10);
  if (isNaN(amount) || amount < 1000) {
    alert('Masukkan jumlah minimal Rp 1000');
    return;
  }
  users[currentUser].saldo += amount;
  transactions.push({
    type: 'topup',
    user: currentUser,
    amount,
    date: new Date().toISOString()
  });
  saveUsers();
  saveTransactions();
  updateDashboard();
  alert(`Top-up sebesar Rp ${amount.toLocaleString('id-ID')} berhasil!`);
  closeModal();
}

// TRANSFER
function showTransferModal() {
  let userOptions = '';
  for (const user in users) {
    if (user !== currentUser) {
      userOptions += `<option value="${user}">${user}</option>`;
    }
  }
  const html = `
    <h3>Transfer Saldo</h3>
    <label for="recipient">Penerima:</label>
    <select id="recipient">${userOptions}</select>
    <br><br>
    <label for="amount">Jumlah (Rp):</label>
    <input type="number" id="amount" min="1" placeholder="Masukkan jumlah" />
    <br><br>
    <button onclick="transferSaldo()">Kirim</button>
  `;
  showModal(html);
}

function transferSaldo() {
  const recipient = document.getElementById('recipient').value;
  const amount = parseInt(document.getElementById('amount').value, 10);
  if (!recipient) {
    alert('Pilih penerima terlebih dahulu!');
    return;
  }
  if (isNaN(amount) || amount <= 0) {
    alert('Masukkan jumlah yang valid!');
    return;
  }
  if (users[currentUser].saldo < amount) {
    alert('Saldo tidak cukup!');
    return;
  }
  users[currentUser].saldo -= amount;
  users[recipient].saldo += amount;
  transactions.push({
    type: 'transfer',
    from: currentUser,
    to: recipient,
    amount,
    date: new Date().toISOString()
  });
  saveUsers();
  saveTransactions();
  alert(`Berhasil transfer Rp ${amount.toLocaleString('id-ID')} ke ${recipient}`);
  updateDashboard();
  closeModal();
}

// TRANSFER DARI QR SCAN
function showScanQRModal() {
  const html = `
    <h3>Scan QR Code untuk Transfer</h3>
    <div id="qr-reader" style="width: 100%;"></div>
    <button onclick="closeModal()">Batal</button>
  `;
  showModal(html);
  const qrScanner = new Html5Qrcode("qr-reader");
  qrScanner.start(
    { facingMode: "environment" },
    { fps: 10, qrbox: 250 },
    (decodedText) => {
      qrScanner.stop();
      closeModal();
      const recipient = decodedText.trim();
      if (!users[recipient]) {
        alert('Penerima tidak ditemukan');
        return;
      }
      showTransferModalWithRecipient(recipient);
    },
    (error) => {
      // ignore scan failure per frame
    }
  ).catch(err => {
    alert("Gagal memulai scanner QR: " + err);
    closeModal();
  });
}

function showTransferModalWithRecipient(recipient) {
  const html = `
    <h3>Transfer ke ${recipient}</h3>
    <label for="amount">Jumlah (Rp):</label>
    <input type="number" id="amount" min="1" placeholder="Masukkan jumlah" />
    <br><br>
    <button onclick="transferSaldoTo('${recipient}')">Kirim</button>
  `;
  showModal(html);
}

function transferSaldoTo(recipient) {
  const amount = parseInt(document.getElementById('amount').value, 10);
  if (isNaN(amount) || amount <= 0) {
    alert('Masukkan jumlah yang valid!');
    return;
  }
  if (users[currentUser].saldo < amount) {
    alert('Saldo tidak cukup!');
    return;
  }
  users[currentUser].saldo -= amount;
  users[recipient].saldo += amount;
  transactions.push({
    type: 'transfer',
    from: currentUser,
    to: recipient,
    amount,
    date: new Date().toISOString()
  });
  saveUsers();
  saveTransactions();
  alert(`Berhasil transfer Rp ${amount.toLocaleString('id-ID')} ke ${recipient}`);
  updateDashboard();
  closeModal();
}

// PULSA
function showPulsaModal() {
  const html = `
    <h3>Beli Pulsa</h3>
    <label for="pulsaAmount">Nomor HP:</label>
    <input type="text" id="phoneNumber" placeholder="08123456789" />
    <br><br>
    <label for="pulsaAmount">Jumlah Pulsa (Rp):</label>
    <select id="pulsaAmount">
      <option value="10000">10.000</option>
      <option value="20000">20.000</option>
      <option value="50000">50.000</option>
      <option value="100000">100.000</option>
    </select>
    <br><br>
    <button onclick="buyPulsa()">Beli</button>
  `;
  showModal(html);
}

function buyPulsa() {
  const phone = document.getElementById('phoneNumber').value.trim();
  const amount = parseInt(document.getElementById('pulsaAmount').value, 10);
  if (!phone.match(/^08\d{6,12}$/)) {
    alert('Nomor HP tidak valid');
    return;
  }
  if (users[currentUser].saldo < amount) {
    alert('Saldo tidak cukup');
    return;
  }
  users[currentUser].saldo -= amount;
  saveUsers();
  updateDashboard();
  alert(`Pulsa Rp ${amount.toLocaleString('id-ID')} untuk ${phone} berhasil dibeli`);
  closeModal();
}

// LISTRIK
function showListrikModal() {
  const html = `
    <h3>Bayar Listrik</h3>
    <label for="meterNumber">Nomor Meter:</label>
    <input type="text" id="meterNumber" placeholder="1234567890" />
    <br><br>
    <label for="listrikAmount">Jumlah Bayar (Rp):</label>
    <input type="number" id="listrikAmount" min="1000" placeholder="Minimal Rp 1000" />
    <br><br>
    <button onclick="payListrik()">Bayar</button>
  `;
  showModal(html);
}

function payListrik() {
  const meter = document.getElementById('meterNumber').value.trim();
  const amount = parseInt(document.getElementById('listrikAmount').value, 10);
  if (!meter) {
    alert('Masukkan nomor meter listrik');
    return;
  }
  if (isNaN(amount) || amount < 1000) {
    alert('Masukkan jumlah minimal Rp 1000');
    return;
  }
  if (users[currentUser].saldo < amount) {
    alert('Saldo tidak cukup');
    return;
  }
  users[currentUser].saldo -= amount;
  saveUsers();
  updateDashboard();
  alert(`Pembayaran listrik Rp ${amount.toLocaleString('id-ID')} untuk meter ${meter} berhasil`);
  closeModal();
}

// AIR
function showAirModal() {
  const html = `
    <h3>Bayar Iuran Air</h3>
    <label for="airAccount">Nomor Rekening Air:</label>
    <input type="text" id="airAccount" placeholder="1234567890" />
    <br><br>
    <label for="airAmount">Jumlah Bayar (Rp):</label>
    <input type="number" id="airAmount" min="1000" placeholder="Minimal Rp 1000" />
    <br><br>
    <button onclick="payAir()">Bayar</button>
  `;
  showModal(html);
}

function payAir() {
  const account = document.getElementById('airAccount').value.trim();
  const amount = parseInt(document.getElementById('airAmount').value, 10);
  if (!account) {
    alert('Masukkan nomor rekening air');
    return;
  }
  if (isNaN(amount) || amount < 1000) {
    alert('Masukkan jumlah minimal Rp 1000');
    return;
  }
  if (users[currentUser].saldo < amount) {
    alert('Saldo tidak cukup');
    return;
  }
  users[currentUser].saldo -= amount;
  saveUsers();
  updateDashboard();
  alert(`Pembayaran air Rp ${amount.toLocaleString('id-ID')} untuk rekening ${account} berhasil`);
  closeModal();
}

// CEK HARGA SAMPAH TERBARU
function showHargaModal() {
  // Contoh data harga sampah
  const hargaSampah = {
    plastik: 2000,
    kertas: 1500,
    kaleng: 3000,
    kaca: 2500,
  };
  let html = '<h3>Harga Sampah Terbaru (per kg)</h3><ul>';
  for (const [jenis, harga] of Object.entries(hargaSampah)) {
    html += `<li>${jenis.charAt(0).toUpperCase() + jenis.slice(1)}: Rp ${harga.toLocaleString('id-ID')}</li>`;
  }
  html += '</ul>';
  showModal(html);
}

// RIWAYAT TRANSAKSI
function showHistoryModal() {
  let html = '<h3>Riwayat Transaksi</h3>';
  if (transactions.length === 0) {
    html += '<p>Belum ada transaksi.</p>';
  } else {
    html += `<table border="1" cellpadding="5" style="width:100%; border-collapse: collapse;">
      <thead style="background:#4caf50; color:#fff;">
        <tr><th>Tanggal</th><th>Jenis</th><th>Detail</th><th>Jumlah (Rp)</th></tr>
      </thead><tbody>`;
    for (const tx of transactions) {
      let detail = '';
      if (tx.type === 'topup') detail = `Top-Up oleh ${tx.user}`;
      else if (tx.type === 'transfer') detail = `Dari ${tx.from} ke ${tx.to}`;
      html += `<tr>
        <td>${new Date(tx.date).toLocaleString('id-ID')}</td>
        <td>${tx.type}</td>
        <td>${detail}</td>
        <td style="text-align:right;">${tx.amount.toLocaleString('id-ID')}</td>
      </tr>`;
    }
    html += '</tbody></table>';
    html += `<br/><button onclick="exportHistoryPDF()">📄 Ekspor PDF</button> <button onclick="exportHistoryCSV()">📊 Ekspor CSV</button>`;
  }
  showModal(html);
}

// EXPORT RIWAYAT PDF
function exportHistoryPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.setFontSize(14);
  doc.text("Riwayat Transaksi Dompet ARAS", 10, 10);
  doc.setFontSize(10);
  let y = 20;
  for (const tx of transactions) {
    let detail = tx.type === 'topup' ? `Top-Up oleh ${tx.user}` : `Dari ${tx.from} ke ${tx.to}`;
    let line = `${new Date(tx.date).toLocaleString('id-ID')} | ${tx.type} | ${detail} | Rp ${tx.amount.toLocaleString('id-ID')}`;
    doc.text(line, 10, y);
    y += 7;
    if (y > 280) {
      doc.addPage();
      y = 10;
    }
  }
  doc.save('riwayat-transaksi.pdf');
}

// EXPORT RIWAYAT CSV
function exportHistoryCSV() {
  let csv = 'Tanggal,Jenis,Detail,Jumlah (Rp)\n';
  for (const tx of transactions) {
    let detail = tx.type === 'topup' ? `Top-Up oleh ${tx.user}` : `Dari ${tx.from} ke ${tx.to}`;
    csv += `"${new Date(tx.date).toLocaleString('id-ID')}","${tx.type}","${detail}","${tx.amount}"\n`;
  }
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'riwayat-transaksi.csv';
  a.click();
  URL.revokeObjectURL(url);
}

// LOGOUT
function logout() {
  currentUser = null;
  document.getElementById('login-section').style.display = 'block';
  document.getElementById('dashboard').style.display = 'none';
  closeModal();
}

// Simpan data ke LocalStorage
function saveUsers() {
  localStorage.setItem('users', JSON.stringify(users));
}

function saveTransactions() {
  localStorage.setItem('transactions', JSON.stringify(transactions));
}

// Load data dari LocalStorage
function loadUsers() {
  const data = localStorage.getItem('users');
  if (data) users = JSON.parse(data);
}

function loadTransactions() {
  const data = localStorage.getItem('transactions');
  if (data) transactions = JSON.parse(data);
}

// Initialize app
function init() {
  loadUsers();
  loadTransactions();
  updateUserList();
  if (currentUser) {
    updateDashboard();
    document.getElementById('login-section').style.display = 'none';
    document.getElementById('dashboard').style.display = 'block';
  }
}

// Run init on page load
window.onload = init;
