<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Dompet ARAS v2</title>
  <link rel="stylesheet" href="style.css" />
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
</head>
<body>
  <header>
    <img src="logo.png" alt="Logo Dompet ARAS" class="logo" />
    <h1>Dompet ARAS v2</h1>
  </header>

  <main>
    <section id="login-section">
      <h2>Login Sederhana</h2>
      <input type="text" id="username" placeholder="Masukkan nama pengguna" />
      <button id="login-btn">Login</button>
    </section>

    <section id="wallet-section" class="hidden">
      <h2>Saldo Anda: <span id="balance">0</span> ARAS</h2>
      <input type="number" id="topup-amount" placeholder="Jumlah Top-Up" min="1" />
      <button id="topup-btn">Top-Up</button>
      <button id="logout-btn">Logout</button>
    </section>
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
