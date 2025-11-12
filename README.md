<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Urutkan Angka 4 Digit (Fleksibel & Fix Index)</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f6f8fa; margin: 20px; }
    h2 { text-align: center; }
    textarea, input { width: 100%; box-sizing: border-box; padding: 8px; margin: 6px 0; }
    textarea { height: 260px; resize: vertical; }
    button { padding: 8px 16px; margin: 5px; cursor: pointer; border: none; border-radius: 6px; }
    .process { background: #007bff; color: white; }
    .copy { background: #28a745; color: white; }
    .reset { background: #dc3545; color: white; }
    #resultBox {
      white-space: pre-wrap;
      background: #fff;
      border: 1px solid #ccc;
      padding: 10px;
      border-radius: 6px;
      margin-top: 10px;
      min-height: 120px;
    }
    #counter { margin-top: 6px; font-weight: bold; color: #333; }
    #infoUrut { color: #555; font-size: 0.9em; margin-bottom: 6px; }
    .note { font-size:0.9em; color:#666; margin-top:6px; }
  </style>
</head>
<body>

  <h2>📅 Urutkan Angka 4 Digit (Fleksibel & Akurat)</h2>
  <div id="infoUrut">Urutan hasil: <b>dari tanggal paling lama ➜ terbaru</b></div>

  <label><b>Tempel Data (bebas, penting: ada tanggal & 4 digit di baris):</b></label>
  <textarea id="dataInput" placeholder="Contoh:&#10;1	11 Nov 2025	Selasa	CAM - 264	6523&#10;2	10 Nov 2025	Senin	CAM - 263	1451"></textarea>

  <div>
    <button class="process" onclick="prosesData()">Proses</button>
    <button class="copy" onclick="salinHasil()">Salin Hasil</button>
    <button class="reset" onclick="resetSemua()">Reset</button>
  </div>

  <div id="counter"></div>
  <div id="resultBox"></div>
  <div class="note">Catatan: Kode melewatkan nomor urut di depan (jika ada) dan mendukung format tanggal seperti 11-11-2025, 11/11/25, 11 Nov 2025, 11 November 2025, 11 Nov '25, dll.</div>

  <script>
    function getMonthFromString(mStr) {
      if (!mStr) return undefined;
      const s = String(mStr).toLowerCase().replace(/\./g,'').trim();
      if (/^\d+$/.test(s)) {
        const n = parseInt(s);
        if (n >= 1 && n <= 12) return n - 1;
        return undefined;
      }
      if (s.startsWith('jan')) return 0;
      if (s.startsWith('feb')) return 1;
      if (s.startsWith('mar')) return 2;
      if (s.startsWith('apr')) return 3;
      if (s.startsWith('mei') || s.startsWith('may')) return 4;
      if (s.startsWith('jun')) return 5;
      if (s.startsWith('jul')) return 6;
      if (s.startsWith('ag') || s.startsWith('agu') || s.startsWith('agt')) return 7;
      if (s.startsWith('sep')) return 8;
      if (s.startsWith('okt') || s.startsWith('oct')) return 9;
      if (s.startsWith('nov')) return 10;
      if (s.startsWith('des') || s.startsWith('dec')) return 11;
      return undefined;
    }

    function prosesData() {
      const raw = document.getElementById('dataInput').value;
      const data = raw ? raw.trim() : '';
      const resultBox = document.getElementById('resultBox');
      const counter = document.getElementById('counter');
      resultBox.textContent = '';
      counter.textContent = '';

      if (!data) {
        alert('Mohon tempelkan data terlebih dahulu.');
        return;
      }

      const lines = data.split(/\r?\n/);
      const records = [];

      // Regex yang melewatkan nomor urut awal (opsional), lalu menangkap:
      // day (1-2 digit), month (angka atau teks), year (2-4 digit) ... akhir baris 4 digit number
      const regex = /^\s*(?:\d+\s+)?(\d{1,2})\s*(?:[-\/\s]+)\s*([A-Za-z]+|\d{1,2})\s*(?:[-\/\s]+)\s*(\d{2,4}).*?(\d{4})\s*$/;

      for (const line of lines) {
        const match = line.match(regex);
        if (!match) continue;

        let [, dStr, mStr, yStr, num] = match;
        const d = parseInt(dStr, 10);
        let y = parseInt(yStr, 10);
        if (y < 100) y += 2000; // '25' -> 2025

        const m = getMonthFromString(mStr);
        if (typeof m === 'undefined') continue;

        // Validasi tanggal: buat object UTC dan cek kebenaran tanggal (misal 31 Feb invalid)
        const timestamp = Date.UTC(y, m, d);
        const check = new Date(timestamp);
        if (check.getUTCFullYear() !== y || check.getUTCMonth() !== m || check.getUTCDate() !== d) {
          continue; // tanggal tidak valid
        }

        records.push({
          timestamp,
          number: num,
          dateStr: `${String(d).padStart(2,'0')}-${String(m+1).padStart(2,'0')}-${y}`
        });
      }

      if (records.length === 0) {
        alert('Tidak ditemukan baris dengan tanggal & angka 4 digit yang valid.');
        return;
      }

      // Urutkan berdasarkan timestamp (pasti akurat)
      records.sort((a, b) => a.timestamp - b.timestamp);

      const angka = records.map(r => r.number);
      const tanggalAwal = records[0].dateStr;
      const tanggalAkhir = records[records.length - 1].dateStr;

      // Format 7 angka per baris
      let formatted = '';
      for (let i = 0; i < angka.length; i++) {
        formatted += angka[i] + ((i + 1) % 7 === 0 ? '\n' : ' ');
      }

      counter.textContent = `Jumlah data valid: ${angka.length} | Rentang tanggal: ${tanggalAwal} → ${tanggalAkhir}`;
      resultBox.textContent = formatted.trim();
    }

    function salinHasil() {
      const text = document.getElementById('resultBox').textContent;
      if (!text) return alert('Belum ada hasil untuk disalin.');
      navigator.clipboard.writeText(text).then(() => {
        alert('Hasil telah disalin ke clipboard!');
      }).catch(()=> {
        alert('Gagal menyalin. Coba salin manual.');
      });
    }

    function resetSemua() {
      document.getElementById('dataInput').value = '';
      document.getElementById('resultBox').textContent = '';
      document.getElementById('counter').textContent = '';
    }
  </script>

</body>
</html>
