import React, { useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";

const makananAsal = [
  {
    id: 1,
    nama: "Nasi Ayam",
    harga: 8.5,
    gambar: "https://example.com/nasi-ayam.jpg",
    kandungan: "Nasi, ayam goreng, timun, sup, kicap"
  },
  {
    id: 2,
    nama: "Mee Goreng Mamak",
    harga: 7.0,
    gambar: "https://example.com/mee-goreng.jpg",
    kandungan: "Mee kuning, taugeh, telur, ayam, sos pedas"
  }
];

export default function HalamanPesanan() {
  const [menu, setMenu] = useState(makananAsal);
  const [pesanan, setPesanan] = useState(
    makananAsal.map((item) => ({ ...item, kuantiti: 0, nota: "" }))
  );
  const [namaPelanggan, setNamaPelanggan] = useState("");
  const [nomborTelefon, setNomborTelefon] = useState("");
  const [adminMode, setAdminMode] = useState(false);
  const [adminLogin, setAdminLogin] = useState(false);
  const [password, setPassword] = useState("");
  const [editIndex, setEditIndex] = useState(null);
  const [newItem, setNewItem] = useState({ nama: "", harga: "", gambar: "", kandungan: "" });

  const tambahKuantiti = (id) => {
    setPesanan((prev) =>
      prev.map((item) =>
        item.id === id ? { ...item, kuantiti: item.kuantiti + 1 } : item
      )
    );
  };

  const kurangKuantiti = (id) => {
    setPesanan((prev) =>
      prev.map((item) =>
        item.id === id && item.kuantiti > 0
          ? { ...item, kuantiti: item.kuantiti - 1 }
          : item
      )
    );
  };

  const ubahNota = (id, nota) => {
    setPesanan((prev) =>
      prev.map((item) => (item.id === id ? { ...item, nota } : item))
    );
  };

  const totalHarga = pesanan.reduce(
    (jumlah, item) => jumlah + item.kuantiti * item.harga,
    0
  );

  const hantarPesanan = () => {
    const pesananAktif = pesanan.filter((item) => item.kuantiti > 0);
    if (!namaPelanggan || !nomborTelefon || pesananAktif.length === 0) {
      alert("Sila lengkapkan maklumat pelanggan dan pilih makanan.");
      return;
    }

    const mesej = `Nama: ${namaPelanggan}\nTelefon: ${nomborTelefon}\n\n` +
      pesananAktif
        .map(
          (item) =>
            `- ${item.nama} x${item.kuantiti} (RM${(
              item.kuantiti * item.harga
            ).toFixed(2)})\nNota: ${item.nota}`
        )
        .join("\n\n") +
      `\n\nJumlah: RM${totalHarga.toFixed(2)}`;

    const url = `https://wa.me/60123456789?text=${encodeURIComponent(mesej)}`;
    window.open(url, "_blank");
  };

  const simpanMenu = () => {
    if (editIndex !== null) {
      const dikemaskini = [...menu];
      dikemaskini[editIndex] = {
        ...dikemaskini[editIndex],
        ...newItem,
        harga: parseFloat(newItem.harga)
      };
      setMenu(dikemaskini);
      setPesanan(dikemaskini.map((item) => ({ ...item, kuantiti: 0, nota: "" })));
      setEditIndex(null);
    } else {
      const itemBaru = {
        id: Date.now(),
        nama: newItem.nama,
        harga: parseFloat(newItem.harga),
        gambar: newItem.gambar,
        kandungan: newItem.kandungan,
        kuantiti: 0,
        nota: ""
      };
      const dikemaskini = [...menu, itemBaru];
      setMenu(dikemaskini);
      setPesanan(dikemaskini.map((item) => ({ ...item, kuantiti: 0, nota: "" })));
    }
    setNewItem({ nama: "", harga: "", gambar: "", kandungan: "" });
  };

  const padamItem = (index) => {
    const dikemaskini = [...menu];
    dikemaskini.splice(index, 1);
    setMenu(dikemaskini);
    setPesanan(dikemaskini.map((item) => ({ ...item, kuantiti: 0, nota: "" })));
  };

  const loginAdmin = () => {
    if (password === "admin123") {
      setAdminLogin(true);
      setPassword("");
    } else {
      alert("Kata laluan salah!");
    }
  };

  return (
    <div className="p-6">
      <div className="flex justify-between items-center mb-4">
        <h1 className="text-2xl font-bold">{adminMode ? "Halaman Admin" : "Pesan Makanan Anda"}</h1>
        <Button variant="outline" onClick={() => setAdminMode(!adminMode)}>
          {adminMode ? "Tukar ke Halaman Pelanggan" : "Masuk Halaman Admin"}
        </Button>
      </div>

      {adminMode ? (
        !adminLogin ? (
          <div className="max-w-sm space-y-3">
            <Input
              placeholder="Masukkan kata laluan admin"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
            />
            <Button onClick={loginAdmin}>Log Masuk Admin</Button>
          </div>
        ) : (
          <div className="max-w-xl space-y-4">
            <Input
              placeholder="Nama makanan"
              value={newItem.nama}
              onChange={(e) => setNewItem({ ...newItem, nama: e.target.value })}
            />
            <Input
              placeholder="Harga"
              type="number"
              value={newItem.harga}
              onChange={(e) => setNewItem({ ...newItem, harga: e.target.value })}
            />
            <Input
              placeholder="Pautan gambar"
              value={newItem.gambar}
              onChange={(e) => setNewItem({ ...newItem, gambar: e.target.value })}
            />
            <Textarea
              placeholder="Kandungan makanan"
              value={newItem.kandungan}
              onChange={(e) => setNewItem({ ...newItem, kandungan: e.target.value })}
            />
            <Button onClick={simpanMenu}>{editIndex !== null ? "Simpan Edit" : "Tambah Menu"}</Button>
            <h3 className="text-lg font-semibold mt-4">Senarai Menu</h3>
            {menu.map((item, index) => (
              <div key={item.id} className="flex justify-between items-center border-b py-2">
                <div>
                  <p className="font-medium">{item.nama} (RM{item.harga.toFixed(2)})</p>
                </div>
                <div className="space-x-2">
                  <Button size="sm" variant="outline" onClick={() => {
                    setEditIndex(index);
                    setNewItem({ ...item });
                  }}>Edit</Button>
                  <Button size="sm" variant="destructive" onClick={() => padamItem(index)}>Padam</Button>
                </div>
              </div>
            ))}
          </div>
        )
      ) : (
        <>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
            {pesanan.map((item) => (
              <Card key={item.id} className="rounded-2xl shadow-md">
                <CardContent className="p-4">
                  <img
                    src={item.gambar}
                    alt={item.nama}
                    className="w-full h-48 object-cover rounded-xl mb-4"
                  />
                  <h2 className="text-xl font-semibold mb-1">{item.nama}</h2>
                  <p className="text-sm text-gray-600 mb-2">{item.kandungan}</p>
                  <p className="text-lg font-bold text-orange-600 mb-2">
                    RM {item.harga.toFixed(2)}
                  </p>
                  <div className="flex items-center gap-2 mb-2">
                    <Button onClick={() => kurangKuantiti(item.id)}>-</Button>
                    <span className="text-lg font-medium">{item.kuantiti}</span>
                    <Button onClick={() => tambahKuantiti(item.id)}>+</Button>
                  </div>
                  <Textarea
                    placeholder="Nota atau permintaan khas..."
                    value={item.nota}
                    onChange={(e) => ubahNota(item.id, e.target.value)}
                    className="mb-2"
                  />
                </CardContent>
              </Card>
            ))}
          </div>

          <div className="max-w-xl mx-auto bg-white p-4 rounded-xl shadow">
            <h2 className="text-lg font-semibold mb-2">Maklumat Pelanggan</h2>
            <Input
              placeholder="Nama anda"
              className="mb-2"
              value={namaPelanggan}
              onChange={(e) => setNamaPelanggan(e.target.value)}
            />
            <Input
              placeholder="Nombor telefon (cth: 0123456789)"
              className="mb-2"
              value={nomborTelefon}
              onChange={(e) => setNomborTelefon(e.target.value)}
            />
            <p className="mb-2 text-sm text-gray-700 font-medium">
              Jumlah: RM {totalHarga.toFixed(2)}
            </p>
            <Button className="w-full" onClick={hantarPesanan}>
              Hantar ke WhatsApp
            </Button>
          </div>
        </>
      )}
    </div>
  );
}
