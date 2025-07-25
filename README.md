<!DOCTYPE html>
<html lang="ms">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Sistem Pesanan Makanan</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 text-gray-800">
  <div class="max-w-4xl mx-auto p-4">
    <div class="flex justify-between items-center mb-4">
      <h1 class="text-2xl font-bold">Pesanan Makanan</h1>
      <button id="toggleAdmin" class="bg-blue-500 text-white px-4 py-1 rounded">Admin</button>
    </div>

    <!-- Log Masuk Admin -->
    <div id="loginSection" class="mb-4 hidden">
      <input type="password" id="adminPass" placeholder="Kata Laluan Admin" class="border p-2 mr-2" />
      <button onclick="loginAdmin()" class="bg-green-600 text-white px-4 py-1 rounded">Log Masuk</button>
    </div>

    <!-- Seksyen Tambah/Edit Admin -->
    <div id="adminSection" class="hidden">
      <h2 class="text-xl font-semibold mb-2">Tambah / Edit Makanan</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
        <input id="nama" class="p-2 border" placeholder="Nama Makanan" />
        <input id="harga" type="number" class="p-2 border" placeholder="Harga" />
        <input id="gambar" class="p-2 border" placeholder="URL Gambar" />
        <input id="kandungan" class="p-2 border" placeholder="Kandungan Makanan" />
        <input id="kategori" class="p-2 border" placeholder="Kategori (cth: Minuman)" />
      </div>
      <button onclick="simpanMenu()" class="bg-blue-600 text-white px-4 py-2 rounded">Simpan</button>
    </div>

    <!-- Paparan Menu Makanan -->
    <div id="menuList" class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6"></div>

    <!-- Ringkasan Pesanan -->
    <div id="summary" class="bg-white p-4 rounded shadow">
      <h2 class="text-xl font-semibold mb-2">Ringkasan Pesanan</h2>
      <textarea id="notaPembeli" placeholder="Nota untuk pesanan..." class="border w-full p-2 mb-2"></textarea>
      <p id="jumlahHarga">Jumlah: RM 0.00</p>
      <button onclick="hantarWhatsApp()" class="bg-green-500 text-white px-4 py-2 mt-2 rounded">Hantar ke WhatsApp</button>
    </div>
  </div>

  <script>
    let adminMasuk = false;
    let editIndex = null;
    let menu = [];

    document.getElementById('toggleAdmin').onclick = () => {
      document.getElementById('loginSection').classList.toggle('hidden');
    };

    function loginAdmin() {
      const pass = document.getElementById('adminPass').value;
      if (pass === 'admin123') {
        adminMasuk = true;
        document.getElementById('adminSection').classList.remove('hidden');
        alert('Log masuk berjaya!');
      } else {
        alert('Kata laluan salah');
      }
    }

    function simpanMenu() {
      const nama = document.getElementById('nama').value;
      const harga = parseFloat(document.getElementById('harga').value);
      const gambar = document.getElementById('gambar').value;
      const kandungan = document.getElementById('kandungan').value;
      const kategori = document.getElementById('kategori').value;
      const item = { nama, harga, gambar, kandungan, kategori, kuantiti: 0 };

      if (editIndex !== null) {
        menu[editIndex] = item;
        editIndex = null;
      } else {
        menu.push(item);
      }
      resetForm();
      paparMenu();
    }

    function resetForm() {
      document.getElementById('nama').value = '';
      document.getElementById('harga').value = '';
      document.getElementById('gambar').value = '';
      document.getElementById('kandungan').value = '';
      document.getElementById('kategori').value = '';
    }

    function paparMenu() {
      const container = document.getElementById('menuList');
      container.innerHTML = '';
      let jumlah = 0;

      menu.forEach((makanan, index) => {
        jumlah += makanan.kuantiti * makanan.harga;
        container.innerHTML += `
          <div class="bg-white rounded shadow p-4">
            <img src="${makanan.gambar}" alt="${makanan.nama}" class="w-full h-40 object-cover rounded mb-2" />
            <h3 class="text-lg font-bold">${makanan.nama}</h3>
            <p class="text-sm text-gray-500">${makanan.kandungan}</p>
            <p class="mt-1 font-semibold">RM ${makanan.harga.toFixed(2)}</p>
            <div class="flex items-center mt-2">
              <button onclick="ubahKuantiti(${index}, -1)" class="px-2 bg-gray-300">-</button>
              <span class="px-4">${makanan.kuantiti}</span>
              <button onclick="ubahKuantiti(${index}, 1)" class="px-2 bg-gray-300">+</button>
            </div>
            ${adminMasuk ? `<div class="mt-2">
              <button onclick="editMenu(${index})" class="text-blue-500 mr-2">Edit</button>
              <button onclick="padamMenu(${index})" class="text-red-500">Padam</button>
            </div>` : ''}
          </div>`;
      });
      document.getElementById('jumlahHarga').textContent = `Jumlah: RM ${jumlah.toFixed(2)}`;
    }

    function ubahKuantiti(index, perubahan) {
      menu[index].kuantiti += perubahan;
      if (menu[index].kuantiti < 0) menu[index].kuantiti = 0;
      paparMenu();
    }

    function editMenu(index) {
      const item = menu[index];
      document.getElementById('nama').value = item.nama;
      document.getElementById('harga').value = item.harga;
      document.getElementById('gambar').value = item.gambar;
      document.getElementById('kandungan').value = item.kandungan;
      document.getElementById('kategori').value = item.kategori;
      editIndex = index;
    }

    function padamMenu(index) {
      if (confirm('Padam makanan ini?')) {
        menu.splice(index, 1);
        paparMenu();
      }
    }

    function hantarWhatsApp() {
      const nota = document.getElementById('notaPembeli').value;
      const pesanan = menu
        .filter(m => m.kuantiti > 0)
        .map(m => `${m.nama} x${m.kuantiti} = RM${(m.harga * m.kuantiti).toFixed(2)}`)
        .join('\n');
      const jumlah = menu.reduce((sum, m) => sum + m.kuantiti * m.harga, 0);
      const mesej = `Pesanan:\n${pesanan}\n\nNota: ${nota}\nJumlah: RM${jumlah.toFixed(2)}`;
      const noWhatsApp = '60123456789'; // ganti dengan nombor sebenar anda
      window.open(`https://wa.me/${noWhatsApp}?text=${encodeURIComponent(mesej)}`, '_blank');
    }
  </script>
</body>
</html>
