<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>Urutkan Angka 4 Digit Berdasarkan Tanggal (Pasti Terlama → Terbaru)</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f6f8fa; margin: 20px; }
    h2 { text-align: center; }
    textarea, input { width: 100%; box-sizing: border-box; padding: 8px; margin: 6px 0; }
    textarea { height: 220px; resize: vertical; }
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
    #counter {
      margin-top: 6px;
      font-weight: bold;
      color: #333;
    }
    #infoUrut {
      color: #555;
      font-size: 0.9em;
      margin-bottom: 6px;
    }
  </style>
</head>
<body>

  <h2>📅 Urutkan Angka 4 Digit Berdasarkan Tanggal</h2>
  <div id="infoUrut">Urutan hasil: <b>dari tanggal paling lama ➜ terbaru</b></div>

  <label><b>Tempel Data (format bebas, contoh: 01-10-2025 HNO-277 9103):</b></label>
  <textarea id="dataInput" placeholder="Contoh:&#10;01-10-2025 HNO-277 9103&#10;02-10-2025 X-90 5127&#10;03-10-2025 ABC-123 8490"></textarea>

  <button class="process" onclick="prosesData()">Proses</button>
  <button class="copy" onclick="salinHasil()">Salin Hasil</button>
  <button class="reset" onclick="resetSemua()">Reset</button>

  <div id="counter"></div>
  <div id="resultBox" placeholder="Hasil akan muncul di sini..."></div>

  <script>
    function prosesData() {
      const data = document.getElementById('dataInput').value.trim();
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

      // regex fleksibel: tanggal di depan, angka 4 digit terakhir di akhir baris
      const regex = /^(\d{2})-(\d{2})-(\d{4}).*?(\d{4})$/;

      for (const line of lines) {
        const match = line.match(regex);
        if (match) {
          const [_, d, m, y, num] = match;
          const dateObj = new Date(`${y}-${m}-${d}`);
          records.push({ date: dateObj, number: num, dateStr: `${d}-${m}-${y}` });
        }
      }

      if (records.length === 0) {
        alert('Tidak ditemukan baris dengan format valid (DD-MM-YYYY ... 4digit).');
        return;
      }

      // Pastikan urutan dari terlama ke terbaru
      records.sort((a, b) => a.date - b.date);

      // Ambil hanya angka
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
      navigator.clipboard.writeText(text);
      alert('Hasil telah disalin ke clipboard!');
    }

    function resetSemua() {
      document.getElementById('dataInput').value = '';
      document.getElementById('resultBox').textContent = '';
      document.getElementById('counter').textContent = '';
    }
  </script>

</body>
</html>
