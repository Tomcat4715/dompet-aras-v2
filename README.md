<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dompet ARAS v2</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>Dompet ARAS v2</h1>
  </header>
  <main>
    <section class="login">
      <h2>Masuk</h2>
      <input type="text" id="username" placeholder="Nama Pengguna">
      <button type="button" onclick="login()">Masuk</button>
    </section>

    <section class="dashboard" style="display:none">
      <h2>Halo, <span id="userDisplay"></span>!</h2>
      <p>Saldo: Rp <span id="balance">0</span></p>

      <div class="actions">
        <button type="button" onclick="showTopUpModal()">💰 Top-Up</button>
        <button type="button" onclick="showTransferModal()">💸 Transfer</button>
        <button type="button" onclick="showPulsaModal()">📱 Pulsa</button>
        <button type="button" onclick="showListrikModal()">💡 Listrik</button>
        <button type="button" onclick="showAirModal()">🚿 Air</button>
        <button type="button" onclick="showHargaModal()">♻️ Cek Harga</button>
      </div>
    </section>

    <div class="modal" id="modal" style="display:none">
      <div class="modal-content">
        <button type="button" class="close" onclick="closeModal()">&times;</button>
        <div id="modal-body"></div>
      </div>
    </div>
  </main>
  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  background-color: #e8f5e9;
  margin: 0;
  padding: 0;
  color: #2e7d32;
  text-align: center;
}

header {
  background-color: #4caf50;
  padding: 1rem;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.logo {
  width: 40px;
  height: 40px;
}

h1 {
  margin: 0;
}

main {
  padding: 2rem;
}

input[type="text"], input[type="number"] {
  padding: 0.5rem;
  font-size: 1rem;
  width: 200px;
  margin: 0.5rem 0;
  border: 1px solid #4caf50;
  border-radius: 4px;
}

button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 0.6rem 1.2rem;
  font-size: 1rem;
  border-radius: 4px;
  cursor: pointer;
  margin: 0.5rem;
}

button:hover {
  background-color: #388e3c;
}

.hidden {
  display: none;
}
.services {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
}

.modal-box {
  background-color: #ffffff;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 0 10px #aaa;
  width: 90%;
  max-width: 400px;
  margin: 2rem auto;
}

#modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
// Konfigurasi Firebase
const firebaseConfig = {
  apiKey: "AIzaSyDVUHmKIjaHWdoAdB20cmAM06LWdFJBRW8",
  authDomain: "dompet-karangtaruna-aras.firebaseapp.com",
  projectId: "dompet-karangtaruna-aras",
  storageBucket: "dompet-karangtaruna-aras.appspot.com",
  messagingSenderId: "373945571180",
  appId: "1:373945571180:web:28823141c06db4e7d6c549",
  measurementId: "G-PWB8RX7JF0"
};

firebase.initializeApp(firebaseConfig);

const loginSection = document.getElementById('login-section');
const walletSection = document.getElementById('wallet-section');
const usernameInput = document.getElementById('username');
const loginBtn = document.getElementById('login-btn');
const logoutBtn = document.getElementById('logout-btn');
const balanceSpan = document.getElementById('balance');
const topupAmountInput = document.getElementById('topup-amount');
const topupBtn = document.getElementById('topup-btn');

let currentUser = null;

// Fungsi simpan dan ambil saldo lokal menggunakan localStorage
function saveBalance(username, amount) {
  localStorage.setItem(`balance_${username}`, amount);
}

function getBalance(username) {
  const bal = localStorage.getItem(`balance_${username}`);
  return bal ? parseInt(bal) : 0;
}

loginBtn.addEventListener('click', () => {
  const username = usernameInput.value.trim();
  if (!username) {
    alert('Masukkan nama pengguna!');
    return;
  }
  currentUser = username;
  loginSection.classList.add('hidden');
  walletSection.classList.remove('hidden');
  updateBalanceDisplay();
});

logoutBtn.addEventListener('click', () => {
  currentUser = null;
  loginSection.classList.remove('hidden');
  walletSection.classList.add('hidden');
  usernameInput.value = '';
});

topupBtn.addEventListener('click', () => {
  if (!currentUser) return;
  let amount = parseInt(topupAmountInput.value);
  if (isNaN(amount) || amount <= 0) {
    alert('Masukkan jumlah top-up yang valid!');
    return;
  }
  let currentBalance = getBalance(currentUser);
  currentBalance += amount;
  saveBalance(currentUser, currentBalance);
  updateBalanceDisplay();
  topupAmountInput.value = '';
});

function updateBalanceDisplay() {
  if (!currentUser) return;
  const bal = getBalance(currentUser);
  balanceSpan.textContent = bal;
}
function openModal(title, htmlContent) {
  document.getElementById('modal-title').textContent = title;
  document.getElementById('modal-content').innerHTML = htmlContent;
  document.getElementById('modal').classList.remove('hidden');
}

function closeModal() {
  document.getElementById('modal').classList.add('hidden');
}

// Fungsi layanan simulasi
function openPulsa() {
  openModal("📱 Beli Pulsa", `
    <input type="text" placeholder="Nomor HP" id="noHp"><br>
    <input type="number" placeholder="Nominal (e.g. 10000)" id="pulsaNom"><br>
    <button onclick="bayarPulsa()">Beli</button>
  `);
}

function bayarPulsa() {
  const no = document.getElementById('noHp').value;
  const nom = parseInt(document.getElementById('pulsaNom').value);
  if (!no || isNaN(nom)) return alert("Isi semua data!");
  if (getBalance(currentUser) < nom) return alert("Saldo tidak cukup!");
  saveBalance(currentUser, getBalance(currentUser) - nom);
  updateBalanceDisplay();
  alert(`Pulsa ${nom} untuk ${no} berhasil dibeli!`);
  closeModal();
}

function openListrik() {
  openModal("💡 Bayar Listrik", `
    <input type="text" placeholder="ID Pelanggan" id="idListrik"><br>
    <input type="number" placeholder="Jumlah Tagihan" id="listrikNom"><br>
    <button onclick="bayarListrik()">Bayar</button>
  `);
}

function bayarListrik() {
  const id = document.getElementById('idListrik').value;
  const nom = parseInt(document.getElementById('listrikNom').value);
  if (!id || isNaN(nom)) return alert("Lengkapi data!");
  if (getBalance(currentUser) < nom) return alert("Saldo tidak cukup!");
  saveBalance(currentUser, getBalance(currentUser) - nom);
  updateBalanceDisplay();
  alert(`Tagihan listrik ID ${id} berhasil dibayar!`);
  closeModal();
}

function openAir() {
  openModal("🚿 Iuran Air", `
    <input type="text" placeholder="Nama / ID Pelanggan" id="idAir"><br>
    <input type="number" placeholder="Nominal Iuran" id="airNom"><br>
    <button onclick="bayarAir()">Bayar</button>
  `);
}

function bayarAir() {
  const id = document.getElementById('idAir').value;
  const nom = parseInt(document.getElementById('airNom').value);
  if (!id || isNaN(nom)) return alert("Lengkapi data!");
  if (getBalance(currentUser) < nom) return alert("Saldo tidak cukup!");
  saveBalance(currentUser, getBalance(currentUser) - nom);
  updateBalanceDisplay();
  alert(`Iuran air untuk ${id} berhasil dibayar!`);
  closeModal();
}

function openHarga() {
  openModal("♻️ Harga Sampah Hari Ini", `
    <ul style="text-align:left;">
      <li>Plastik: Rp 2.000/kg</li>
      <li>Kertas: Rp 1.500/kg</li>
      <li>Kaleng: Rp 4.000/kg</li>
      <li>Botol Kaca: Rp 1.000/kg</li>
    </ul>
  `);
}
