# Panduan WebApp “Uploadin gDrive”

# 📂 Aplikasi Upload File ke Google Drive & Pencatatan di Google Sheets

Selamat datang di proyek Aplikasi Upload File! Ini adalah sebuah aplikasi web sederhana namun *powerful* yang memungkinkan kamu untuk mengunggah file langsung ke folder Google Drive pilihanmu, dan secara otomatis mencatat setiap detail upload ke dalam Google Sheet.

Proyek ini dibuat dengan **Google Apps Script**, jadi kamu tidak perlu pusing memikirkan *hosting* atau *server*. Semuanya berjalan di atas infrastruktur Google.

Tujuan dari repositori ini adalah untuk menjadi panduan belajar bagi para pemula yang ingin memahami cara kerja Google Apps Script, integrasi dengan Google Drive dan Google Sheets, serta membuat aplikasi web sederhana dari nol.

*(Tips: Ganti URL di atas dengan screenshot aplikasi buatanmu agar lebih menarik!)*

## ✨ Fitur Utama

- **Upload File Mudah**: Antarmuka *drag-and-drop* yang modern dan intuitif.
- **Pilih File & Kamera**: Bisa memilih file dari komputer, atau langsung mengambil foto/video dari kamera HP.
- **Struktur Folder Dinamis**: Secara otomatis membuat folder utama dan sub-folder di Google Drive jika belum ada.
- **Pencatatan Otomatis**: Setiap file yang diunggah akan tercatat rapi di Google Sheet, lengkap dengan timestamp, nama file, ukuran, tipe, dan URL-nya.
- **Riwayat Upload**: Lihat kembali riwayat file yang sudah diunggah langsung dari aplikasi.
- **Dasbor Sederhana**: Pantau total file dan total ukuran file yang sudah diunggah melalui Google Sheet.
- **Saran Otomatis**: Aplikasi akan memberikan saran nama folder yang pernah digunakan untuk konsistensi data.
- **Sepenuhnya Gratis**: Dibangun di atas Google Apps Script, tidak perlu biaya server!

## 🛠️ Teknologi yang Digunakan

- **Backend**: Google Apps Script (JavaScript di lingkungan Google)
- **Frontend**: HTML, CSS, JavaScript (sisi klien)
- **Database**: Google Sheets
- **Penyimpanan**: Google Drive

## 🚀 Panduan Instalasi dan Deploy (Super Detail untuk Pemula)

Siap untuk membangun aplikasimu sendiri? Tenang, gampang kok! Ikuti langkah-langkah di bawah ini satu per satu. Kita akan bedah proyek ini menjadi 3 bagian utama: **Backend (Logika Server)**, **Frontend (Tampilan)**, dan **Konfigurasi**.

### Bagian 1: Menyiapkan "Otak" Aplikasi (Backend - Google Apps Script)

Otak dari aplikasi ini adalah Google Apps Script yang akan mengatur proses upload ke Drive dan pencatatan ke Sheet.

1. **Buka Google Apps Script Editor**
    - Buka browser, lalu pergi ke [script.google.com](https://script.google.com/).
    - Klik tombol **"+ Proyek baru"** di pojok kiri atas.
2. **Membuat File-File yang Dibutuhkan**
    - Secara default, kamu akan melihat file bernama `Code.gs`. File ini akan kita gunakan untuk menaruh semua logika backend.
    - Sekarang, kita butuh satu file lagi untuk tampilan antarmuka (frontend). Klik ikon **+** (Tambah file) di sebelah kiri, pilih **HTML**, dan beri nama file `index`.
    - Sekarang kamu punya 2 file: `Code.gs` dan `index.html`. Keren!
3. **Mengisi Logika di `Code.gs`**
    - Hapus semua isi `Code.gs` yang ada, lalu salin dan tempel (copy-paste) kode di bawah ini.
    - **Jangan khawatir**, setelah blok kode ini ada penjelasan detail per fungsinya, jadi kamu bisa sambil belajar!
    
    ```jsx
    /**
     * =================================================================
     * PENTING: BAGIAN KONFIGURASI
     * Ganti nilai variabel di bawah ini dengan ID Folder & Sheet kamu.
     * Cara mendapatkannya akan dijelaskan di Bagian 3.
     * =================================================================
     */
    const TARGET_DRIVE_FOLDER_ID = 'GANTI_DENGAN_ID_FOLDER_DRIVE_KAMU';
    const METADATA_SHEET_ID = 'GANTI_DENGAN_ID_GOOGLE_SHEET_KAMU';
    
    /**
     * Fungsi ini adalah pintu gerbang utama aplikasi kita.
     * Saat URL aplikasi diakses, fungsi inilah yang pertama kali dijalankan.
     * Tugasnya adalah menampilkan file 'index.html' sebagai halaman web.
     */
    function doGet() {
      return HtmlService.createHtmlOutputFromFile('index')
        .setTitle('Aplikasi Upload File')
        .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
    }
    
    /**
     * Fungsi ini bertanggung jawab untuk berinteraksi dengan Google Sheet.
     * Dia akan membuka Sheet berdasarkan ID, dan jika Sheet-nya masih kosong,
     * dia akan membuatkan baris header (judul kolom) secara otomatis.
     */
    function getMetadataSheet() {
      try {
        const spreadsheet = SpreadsheetApp.openById(METADATA_SHEET_ID);
        let sheet = spreadsheet.getSheets()[0]; // Ambil sheet pertama
    
        // Cek jika sheet masih benar-benar kosong
        if (sheet.getLastRow() === 0) {
          const headers = [
            'Timestamp Upload', 'Nama Drive/Folder Utama', 'Sub Folder',
            'Nama File', 'Tipe File', 'Ukuran File (bytes)', 'URL File', 'ID File Drive'
          ];
          sheet.appendRow(headers); // Tambahkan header
          // Beri sedikit style agar cantik
          const headerRange = sheet.getRange(1, 1, 1, headers.length);
          headerRange.setFontWeight('bold').setBackground('#4a86e8').setFontColor('#ffffff');
          sheet.setFrozenRows(1); // Bekukan baris header
        }
        return sheet;
      } catch (e) {
        console.error('Error saat akses Sheet: ' + e.toString());
        throw new Error('Gagal mengakses Google Sheet. Pastikan ID sheet benar.');
      }
    }
    
    /**
     * Fungsi ini bertugas menyimpan detail file yang di-upload ke dalam baris baru di Sheet.
     */
    function saveMetadataToSheet(driveName, subFolderName, fileInfo) {
      try {
        const sheet = getMetadataSheet();
        const rowData = [
          new Date(), fileInfo.driveName, fileInfo.subFolderName,
          fileInfo.name, fileInfo.type, fileInfo.size,
          fileInfo.url, fileInfo.id
        ];
        sheet.appendRow(rowData);
      } catch (e) {
        console.error('Error saat simpan metadata: ' + e.toString());
      }
    }
    
    /**
     * Fungsi ini bertugas untuk membuat folder di Google Drive.
     * Dia akan cek dulu, apakah folder dengan nama yang diberikan sudah ada?
     * - Jika sudah ada, dia akan gunakan folder itu.
     * - Jika belum ada, dia akan buatkan folder baru.
     * Cerdas, kan? Jadi tidak ada folder duplikat.
     */
    function createFolderIfNotExist(folderName, parentFolder) {
      const folders = parentFolder.getFoldersByName(folderName);
      return folders.hasNext() ? folders.next() : parentFolder.createFolder(folderName);
    }
    
    /**
     * INI ADALAH FUNGSI INTI UNTUK UPLOAD!
     * Fungsi ini dipanggil dari JavaScript di frontend (file index.html).
     * Menerima nama folder dan data file, lalu memprosesnya satu per satu.
     */
    function processFileUpload(driveName, subFolderName, fileObjects) {
      if (!driveName || !subFolderName || !fileObjects || fileObjects.length === 0) {
        return { success: false, error: "Data tidak lengkap." };
      }
    
      try {
        const targetFolder = DriveApp.getFolderById(TARGET_DRIVE_FOLDER_ID);
        const mainFolder = createFolderIfNotExist(driveName, targetFolder);
        const subFolder = createFolderIfNotExist(subFolderName, mainFolder);
    
        const uploadedFiles = [];
    
        fileObjects.forEach(fileObject => {
          // Data file dari frontend dikirim dalam format base64, kita perlu decode
          const decodedBlob = Utilities.base64Decode(fileObject.content);
          const blob = Utilities.newBlob(decodedBlob, fileObject.type, fileObject.name);
    
          // Buat file di dalam sub-folder yang tepat
          const uploadedFile = subFolder.createFile(blob);
    
          // Kumpulkan informasi file yang berhasil di-upload
          const fileInfo = {
            name: uploadedFile.getName(),
            type: uploadedFile.getMimeType(),
            size: uploadedFile.getSize(),
            url: uploadedFile.getUrl(),
            id: uploadedFile.getId(),
            driveName: driveName,
            subFolderName: subFolderName
          };
    
          uploadedFiles.push(fileInfo);
          // Simpan informasinya ke Google Sheet
          saveMetadataToSheet(driveName, subFolderName, fileInfo);
        });
    
        return { success: true, uploadedFiles: uploadedFiles };
      } catch (e) {
        console.error(`Error saat proses upload: ${e.toString()}`);
        return { success: false, error: e.message };
      }
    }
    
    // --- FUNGSI-FUNGSI TAMBAHAN UNTUK TAB RIWAYAT & DATA SHEET ---
    
    function getFolderSuggestions() {
      try {
        const sheet = getMetadataSheet();
        const values = sheet.getDataRange().getValues();
        if (values.length <= 1) return { driveNames: [], subFolderNames: [] };
    
        const headers = values[0];
        const driveNameIndex = headers.indexOf('Nama Drive/Folder Utama');
        const subFolderNameIndex = headers.indexOf('Sub Folder');
    
        const driveNames = new Set();
        const subFolderNames = new Set();
    
        for (let i = 1; i < values.length; i++) {
          if (values[i][driveNameIndex]) driveNames.add(values[i][driveNameIndex]);
          if (values[i][subFolderNameIndex]) subFolderNames.add(values[i][subFolderNameIndex]);
        }
        return {
          driveNames: Array.from(driveNames).sort(),
          subFolderNames: Array.from(subFolderNames).sort()
        };
      } catch (e) {
        return { driveNames: [], subFolderNames: [] };
      }
    }
    
    function getUploadHistory() {
      try {
        const sheet = getMetadataSheet();
        const lastRow = sheet.getLastRow();
        if (lastRow <= 1) return { success: true, data: [] };
    
        const data = sheet.getRange(2, 1, lastRow - 1, 8).getValues();
        const history = data.map(row => ({
          timestamp: row[0] instanceof Date ? row[0].toISOString() : null,
          driveName: row[1], subFolderName: row[2], fileName: row[3],
          fileSize: formatBytes(row[5] || 0), fileUrl: row[6],
        })).reverse();
    
        return { success: true, data: history };
      } catch (e) {
        return { success: false, error: e.message, data: [] };
      }
    }
    
    function getSheetData() {
        // Mirip dengan getUploadHistory, tapi mengambil semua data untuk statistik
        // ... (Kode lengkap bisa dilihat di source code)
        // Untuk panduan ini, kita akan membuatnya simpel
        const sheet = getMetadataSheet();
        const lastRow = sheet.getLastRow();
        if (lastRow <= 1) return { success: true, data: [], totalFiles: 0, totalSize: '0 Bytes' };
    
        const dataRange = sheet.getRange(2, 1, lastRow - 1, 8);
        const values = dataRange.getValues();
    
        let totalSize = 0;
        values.forEach(row => { totalSize += (Number(row[5]) || 0); });
    
        return {
            success: true,
            totalFiles: values.length,
            totalSize: formatBytes(totalSize)
        };
    }
    
    function getMetadataSheetUrl() {
      return `https://docs.google.com/spreadsheets/d/${METADATA_SHEET_ID}/edit`;
    }
    
    function formatBytes(bytes) {
      if (bytes === 0) return '0 Bytes';
      const k = 1024;
      const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
      const i = Math.floor(Math.log(bytes) / Math.log(k));
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    }
    
    ```
    

### Bagian 2: Membangun "Wajah" Aplikasi (Frontend - HTML, CSS, JS)

Sekarang kita akan membuat tampilan yang dilihat dan digunakan oleh pengguna.

1. **Buka file `index.html`** yang sudah kamu buat tadi.
2. Hapus semua isinya, lalu **salin dan tempel** kode di bawah ini. Kode ini sudah berisi 3 bagian:
    - **HTML**: Struktur atau kerangka halaman web.
    - **CSS**: Aturan untuk mempercantik tampilan (warna, layout, dll).
    - **JavaScript**: Logika di sisi pengguna (menangani klik tombol, drag-drop, memanggil backend).
    
    ```html
    <!DOCTYPE html>
    <html>
      <head>
        <base target="_top">
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Aplikasi Upload File ke Google Drive</title>
        <!-- BAGIAN CSS: UNTUK MEMPERCANTIK TAMPILAN -->
        <style>
          /*
            CSS (Cascading Style Sheets) berfungsi untuk mendesain halaman HTML kita.
            Di sini kita mengatur font, warna, layout, tombol, dan semua elemen visual.
            Kamu bisa bereksperimen dengan mengubah warna (misal: #667eea) atau ukuran font.
            Kode CSS sengaja dibuat panjang agar tampilannya modern dan responsif.
          */
          body {
            font-family: 'Segoe UI', sans-serif;
            margin: 20px;
            background: #f4f7f6;
            color: #333;
          }
          .container {
            max-width: 800px;
            margin: 0 auto;
            background-color: #fff;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
          }
          h1 { text-align: center; color: #2c3e50; }
          .form-group { margin-bottom: 20px; }
          label { display: block; margin-bottom: 8px; font-weight: 600; }
          input[type="text"] { width: 100%; padding: 12px; border: 2px solid #e1e8ed; border-radius: 8px; }
          .file-upload-area { border: 3px dashed #cbd5e0; padding: 40px; text-align: center; border-radius: 12px; transition: all 0.3s ease; }
          .file-upload-area.dragover { border-color: #667eea; background: #f0f4ff; }
          .btn { display: inline-block; padding: 12px 24px; background: #667eea; color: white; border: none; border-radius: 8px; cursor: pointer; font-size: 16px; transition: all 0.3s ease; }
          .btn:disabled { background: #a0aec0; cursor: not-allowed; }
          .file-input-hidden { display: none; }
          .file-list { margin-top: 20px; }
          .file-item { display: flex; justify-content: space-between; align-items: center; padding: 10px; border-bottom: 1px solid #e2e8f0; }
          .file-remove { background: #e53e3e; color: white; border: none; border-radius: 4px; padding: 5px 10px; cursor: pointer; }
          .progress-container { margin-top: 20px; display: none; }
          .progress-bar { height: 8px; background-color: #e2e8f0; border-radius: 4px; overflow: hidden; }
          .progress { height: 100%; background: #667eea; width: 0%; }
          .status { margin-top: 20px; padding: 15px; border-radius: 8px; display: none; }
          .status.success { background-color: #c6f6d5; color: #22543d; }
          .status.error { background-color: #fed7d7; color: #742a2a; }
          /* Style untuk Tab */
          .tab-buttons { display: flex; border-bottom: 2px solid #e2e8f0; }
          .tab-button { padding: 15px 25px; background: none; border: none; cursor: pointer; font-size: 16px; border-bottom: 3px solid transparent; }
          .tab-button.active { color: #667eea; border-bottom-color: #667eea; }
          .tab-panel { display: none; padding: 20px 0; }
          .tab-panel.active { display: block; }
        </style>
      </head>
      <body>
        <!-- BAGIAN HTML: STRUKTUR HALAMAN WEB -->
        <div class="container">
          <h1>📁 Upload File ke Drive</h1>
    
          <div class="tab-container">
            <div class="tab-buttons">
              <button class="tab-button active" onclick="openTab(event, 'uploadTab')">Upload</button>
              <button class="tab-button" onclick="openTab(event, 'historyTab')">Riwayat</button>
            </div>
    
            <!-- Tab untuk Upload -->
            <div id="uploadTab" class="tab-panel active">
              <div class="form-group">
                <label for="driveName">Nama Folder Utama:</label>
                <input type="text" id="driveName" list="driveNameOptions" placeholder="cth: Laporan Keuangan" required>
                <datalist id="driveNameOptions"></datalist>
              </div>
    
              <div class="form-group">
                <label for="subFolderName">Nama Sub-Folder:</label>
                <input type="text" id="subFolderName" list="subFolderNameOptions" placeholder="cth: Bulan Januari" required>
                <datalist id="subFolderNameOptions"></datalist>
              </div>
    
              <div class="file-upload-area" id="fileUploadArea">
                <p>Seret & Lepas file di sini, atau klik tombol di bawah</p>
                <button type="button" class="btn" onclick="document.getElementById('fileInput').click()">Pilih File</button>
              </div>
    
              <input type="file" id="fileInput" class="file-input-hidden" multiple>
    
              <div id="fileList" class="file-list"></div>
    
              <button id="uploadButton" class="btn" style="width:100%; margin-top: 20px;" onclick="uploadFiles()" disabled>Upload</button>
    
              <div id="progressContainer" class="progress-container">
                <div class="progress-bar"><div id="progressBar" class="progress"></div></div>
                <div id="progressText" style="text-align:center;"></div>
              </div>
    
              <div id="statusSuccess" class="status success"></div>
              <div id="statusError" class="status error"></div>
            </div>
    
            <!-- Tab untuk Riwayat -->
            <div id="historyTab" class="tab-panel">
              <h3>Riwayat Upload Terbaru</h3>
              <div id="uploadHistory"><p>Memuat riwayat...</p></div>
            </div>
          </div>
        </div>
    
        <!-- BAGIAN JAVASCRIPT: LOGIKA DI SISI PENGGUNA -->
        <script>
          // Variabel global untuk menyimpan file yang dipilih
          let selectedFiles = [];
    
          // Mengambil elemen-elemen HTML agar bisa dimanipulasi
          const fileUploadArea = document.getElementById('fileUploadArea');
          const fileInput = document.getElementById('fileInput');
          const uploadButton = document.getElementById('uploadButton');
          const driveNameInput = document.getElementById('driveName');
          const subFolderNameInput = document.getElementById('subFolderName');
    
          // --- LOGIKA UNTUK DRAG & DROP ---
          ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
            fileUploadArea.addEventListener(eventName, (e) => { e.preventDefault(); e.stopPropagation(); }, false);
          });
          ['dragenter', 'dragover'].forEach(eventName => {
            fileUploadArea.addEventListener(eventName, () => fileUploadArea.classList.add('dragover'), false);
          });
          ['dragleave', 'drop'].forEach(eventName => {
            fileUploadArea.addEventListener(eventName, () => fileUploadArea.classList.remove('dragover'), false);
          });
          fileUploadArea.addEventListener('drop', (e) => handleFiles(e.dataTransfer.files), false);
          fileInput.addEventListener('change', (e) => handleFiles(e.target.files));
    
          // --- FUNGSI-FUNGSI UTAMA ---
    
          // Fungsi yang dipanggil saat pengguna memilih file
          function handleFiles(files) {
            selectedFiles = [...files]; // Simpan file yang dipilih
            displaySelectedFiles();
            validateForm();
          }
    
          // Fungsi untuk menampilkan daftar file yang dipilih di layar
          function displaySelectedFiles() {
            const fileListElement = document.getElementById('fileList');
            fileListElement.innerHTML = ''; // Kosongkan daftar sebelumnya
            selectedFiles.forEach((file, index) => {
              const fileItem = document.createElement('div');
              fileItem.className = 'file-item';
              fileItem.innerHTML = `
                <span>${file.name} (${(file.size / 1024).toFixed(2)} KB)</span>
                <button class="file-remove" onclick="removeFile(${index})">Hapus</button>
              `;
              fileListElement.appendChild(fileItem);
            });
          }
    
          // Fungsi untuk menghapus file dari daftar pilihan
          function removeFile(index) {
            selectedFiles.splice(index, 1);
            displaySelectedFiles();
            validateForm();
          }
    
          // Fungsi untuk mengecek apakah form sudah siap untuk di-submit
          function validateForm() {
            const isValid = driveNameInput.value.trim() && subFolderNameInput.value.trim() && selectedFiles.length > 0;
            uploadButton.disabled = !isValid;
          }
          driveNameInput.addEventListener('input', validateForm);
          subFolderNameInput.addEventListener('input', validateForm);
    
          // FUNGSI UTAMA UPLOAD!
          function uploadFiles() {
            if (uploadButton.disabled) return;
    
            setLoading(true, 'Mempersiapkan file...');
    
            // Kita perlu membaca setiap file dan mengubahnya menjadi format base64
            // agar bisa dikirim ke Google Apps Script
            const filePromises = selectedFiles.map(file => {
              return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.onload = e => resolve({
                  name: file.name, type: file.type, size: file.size,
                  content: e.target.result.split(',')[1] // Ambil hanya data base64 nya
                });
                reader.onerror = err => reject(err);
                reader.readAsDataURL(file);
              });
            });
    
            // Setelah semua file selesai dibaca...
            Promise.all(filePromises)
              .then(fileObjects => {
                setLoading(true, 'Mengunggah file...');
                // INI BAGIAN PALING PENTING!
                // Kita memanggil fungsi 'processFileUpload' yang ada di Code.gs
                google.script.run
                  .withSuccessHandler(handleUploadSuccess) // Jika berhasil, panggil fungsi ini
                  .withFailureHandler(handleUploadFailure) // Jika gagal, panggil fungsi ini
                  .processFileUpload(driveNameInput.value, subFolderNameInput.value, fileObjects);
              })
              .catch(err => handleUploadFailure({message: 'Gagal membaca file.'}));
          }
    
          function handleUploadSuccess(result) {
            setLoading(false);
            if (result.success) {
              showStatus(`Berhasil! ${result.uploadedFiles.length} file telah diunggah.`, 'success');
              // Kosongkan form setelah berhasil
              selectedFiles = [];
              displaySelectedFiles();
              subFolderNameInput.value = '';
              validateForm();
              loadFolderSuggestions(); // Muat ulang saran folder
            } else {
              showStatus(`Gagal: ${result.error}`, 'error');
            }
          }
    
          function handleUploadFailure(error) {
            setLoading(false);
            showStatus(`Error: ${error.message}`, 'error');
          }
    
          // Fungsi bantuan untuk menampilkan status loading dan pesan
          function setLoading(isLoading, message = '') {
            uploadButton.disabled = isLoading;
            document.getElementById('progressContainer').style.display = isLoading ? 'block' : 'none';
            document.getElementById('progressText').innerText = message;
          }
    
          function showStatus(message, type) {
            const statusEl = document.getElementById(type === 'success' ? 'statusSuccess' : 'statusError');
            statusEl.innerText = message;
            statusEl.style.display = 'block';
            setTimeout(() => { statusEl.style.display = 'none'; }, 8000);
          }
    
          // --- FUNGSI UNTUK TAB ---
          function openTab(evt, tabName) {
            document.querySelectorAll(".tab-panel").forEach(p => p.classList.remove('active'));
            document.querySelectorAll(".tab-button").forEach(b => b.classList.remove('active'));
            document.getElementById(tabName).classList.add('active');
            evt.currentTarget.classList.add('active');
    
            if (tabName === 'historyTab') {
              // Panggil fungsi backend untuk mengambil data riwayat
              document.getElementById('uploadHistory').innerHTML = '<p>Memuat riwayat...</p>';
              google.script.run.withSuccessHandler(data => {
                const historyContainer = document.getElementById('uploadHistory');
                if(data.success && data.data.length > 0) {
                  historyContainer.innerHTML = data.data.map(item =>
                    `<div class="file-item">
                       <span>${new Date(item.timestamp).toLocaleString('id-ID')} - <b>${item.fileName}</b> ke <i>${item.driveName}/${item.subFolderName}</i></span>
                       <a href="${item.fileUrl}" target="_blank">Lihat</a>
                    </div>`
                  ).join('');
                } else {
                  historyContainer.innerHTML = '<p>Belum ada riwayat upload.</p>';
                }
              }).getUploadHistory();
            }
          }
    
          // Fungsi untuk mengambil saran folder dari sheet
          function loadFolderSuggestions() {
            google.script.run.withSuccessHandler(suggestions => {
              const driveDatalist = document.getElementById('driveNameOptions');
              const subFolderDatalist = document.getElementById('subFolderNameOptions');
              driveDatalist.innerHTML = suggestions.driveNames.map(name => `<option value="${name}">`).join('');
              subFolderDatalist.innerHTML = suggestions.subFolderNames.map(name => `<option value="${name}">`).join('');
            }).getFolderSuggestions();
          }
    
          // Jalankan fungsi ini saat halaman pertama kali dimuat
          window.onload = function() {
            loadFolderSuggestions();
          };
        </script>
      </body>
    </html>
    
    ```
    

### Bagian 3: Konfigurasi - Menghubungkan Semuanya

Ini adalah langkah paling krusial. Kita perlu memberitahu skrip kita, "Hei, simpan file ke folder INI dan catat datanya di sheet ITU."

1. **Menyiapkan Folder Google Drive & Mendapatkan ID-nya**
    - Buka [drive.google.com](https://drive.google.com/).
    - Buat sebuah folder baru. Beri nama apa saja, misal "HASIL UPLOAD APLIKASI". Ini akan menjadi folder induk tempat semua file akan masuk.
    - Buka folder yang baru kamu buat itu.
    - Lihat URL di address bar browser kamu. URL-nya akan terlihat seperti ini: `https://drive.google.com/drive/folders/1a2b3c4d5e6f7g8h9i0jABC-XYZ`.
    - Bagian teks acak setelah `/folders/` adalah **ID FOLDER**nya. Dalam contoh di atas, ID-nya adalah `1a2b3c4d5e6f7g8h9i0jABC-XYZ`.
    - Salin ID folder kamu.
2. **Menyiapkan Google Sheet & Mendapatkan ID-nya**
    - Buka [sheets.google.com](https://sheets.google.com/).
    - Buat spreadsheet baru (kosong). Beri nama, misal "DATABASE UPLOAD".
    - Lihat URL di address bar. URL-nya akan terlihat seperti: `https://docs.google.com/spreadsheets/d/1k2l3m4n5o6p7q8r9s0tABC-XYZ/edit#gid=0`.
    - Bagian teks acak di antara `/d/` dan `/edit` adalah **ID SHEET**nya. Dalam contoh di atas, ID-nya adalah `1k2l3m4n5o6p7q8r9s0tABC-XYZ`.
    - Salin ID sheet kamu.
3. **Memasukkan ID ke dalam Skrip**
    - Kembali ke editor Google Apps Script (`script.google.com`).
    - Buka file `Code.gs`.
    - Cari baris paling atas:
        
        ```jsx
        const TARGET_DRIVE_FOLDER_ID = 'GANTI_DENGAN_ID_FOLDER_DRIVE_KAMU';
        const METADATA_SHEET_ID = 'GANTI_DENGAN_ID_GOOGLE_SHEET_KAMU';
        
        ```
        
    - **Ganti** `GANTI_DENGAN_ID_FOLDER_DRIVE_KAMU` dengan ID Folder Drive yang sudah kamu salin.
    - **Ganti** `GANTI_DENGAN_ID_GOOGLE_SHEET_KAMU` dengan ID Google Sheet yang sudah kamu salin.
    - Pastikan tanda kutipnya (`'`) tidak terhapus.
    - Klik ikon **Simpan proyek** (gambar disket).

### Bagian 4: Deploy! (Menerbitkan Aplikasi ke Internet)

Ini adalah langkah terakhir untuk membuat aplikasimu bisa diakses melalui URL.

1. Di editor Apps Script, klik tombol **"Deploy"** berwarna biru di pojok kanan atas, lalu pilih **"Deployment baru"**.
2. Akan muncul sebuah jendela. Klik ikon **roda gigi** di sebelah "Pilih jenis", lalu pilih **"Aplikasi Web"**.
3. Isi konfigurasinya seperti ini:
    - **Deskripsi**: Boleh diisi apa saja, misal "Versi pertama aplikasi upload".
    - **Jalankan sebagai**: Pilih **"Saya"**.
    - **Siapa yang memiliki akses**: Pilih **"Siapa saja"** (Agar bisa diakses oleh siapa pun) atau **"Siapa saja yang memiliki Akun Google"** (Hanya yang login Google bisa akses). Untuk awal, **"Siapa saja"** adalah pilihan termudah.
4. Klik tombol **"Deploy"**.
5. **PENTING**: Google akan meminta **izin (otorisasi)** karena skrip ini akan mengakses Drive dan Sheet kamu. Ini normal.
    - Klik **"Otorisasi akses"**.
    - Pilih akun Google kamu.
    - Kamu mungkin akan melihat peringatan "Google belum memverifikasi aplikasi ini". Tenang saja, ini karena aplikasi ini buatanmu sendiri. Klik **"Lanjutan"** lalu klik **"Buka [Nama Proyek Kamu] (tidak aman)"**.
    - Klik **"Izinkan"**.
6. **SELESAI!** Kamu akan mendapatkan **URL Aplikasi Web**. Salin URL tersebut. Itulah alamat aplikasi keren buatanmu!

Coba buka URL itu di tab browser baru. Seharusnya aplikasi upload file kamu sudah muncul dan siap digunakan!

## 🤝 Berkontribusi

Merasa ada yang bisa ditingkatkan? Punya ide fitur baru? Jangan ragu untuk melakukan *Fork* pada repositori ini, buat perubahanmu, dan kirimkan *Pull Request*. Kontribusi sekecil apapun akan sangat dihargai.
