<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kuis Pengetahuan Arduino Dasar</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f0f0;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }

        .quiz-container {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
            width: 100%;
            max-width: 600px;
            text-align: center;
        }

        h1 {
            color: #333;
            margin-bottom: 10px;
        }

        #completion-message {
            color: #28a745;
            font-size: 1.2em;
            font-weight: bold;
            margin-top: 5px;
            margin-bottom: 20px;
        }

        .question-counter-text {
            font-size: 0.9em;
            color: #666;
            margin-bottom: 20px;
        }

        #question-container {
            margin-bottom: 20px;
        }

        #question {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 25px;
            color: #444;
        }

        .btn-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
            margin-bottom: 20px;
        }

        .btn {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 12px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.2s ease, box-shadow 0.2s ease;
            word-wrap: break-word;
            min-height: 50px;
            display: flex;
            align-items: center;
            justify-content: center;
            outline: none;
            font-weight: bold;
        }

        .btn:not(.correct):not(.wrong):not(.skip-btn) { background-color: #007bff; }
        .btn:not(.correct):not(.wrong):not(.skip-btn):focus {
            background-color: #007bff;
            box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
        }
        .btn:not([disabled]):not(.correct):not(.wrong):not(.skip-btn):hover {}
        .btn:not([disabled]):not(.correct):not(.wrong):not(.skip-btn):focus:hover {
            background-color: #007bff;
            box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
        }

        .btn.correct { background-color: #28a745 !important; box-shadow: none; }
        .btn.correct:hover { background-color: #218838 !important; }
        .btn.correct:focus {
            background-color: #28a745 !important;
            box-shadow: 0 0 0 3px rgba(40, 167, 69, 0.6) !important;
        }

        .btn.wrong { background-color: #dc3545 !important; box-shadow: none; }
        .btn.wrong:hover { background-color: #c82333 !important; }
        .btn.wrong:focus {
            background-color: #dc3545 !important;
            box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.6) !important;
        }

        .btn:disabled {
            cursor: not-allowed;
            opacity: 0.65;
        }
        .btn:disabled:not(.correct):not(.wrong):not(.skip-btn) {
            background-color: #6c757d !important;
            color: #ccc !important;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 10px;
        }

        #skip-navigation-controls {
            justify-content: space-between;
            margin-top: 40px;
            margin-bottom: 10px;
        }

        .skip-btn {
            background-color: #28a745;
            color: white;
            padding: 8px 12px;
            font-size: 0.9em;
            min-width: 80px;
        }
        .skip-btn:hover {
            background-color: #218838;
            color: white;
        }
        .skip-btn:disabled {
            background-color: #a3d8b0 !important;
            color: #e9f5ec !important;
            cursor: not-allowed;
        }

        .hide { display: none !important; }
    </style>
</head>
<body>
    <div class="quiz-container">
        <h1>Pengetahuan Arduino Dasar</h1>
        <p id="completion-message" class="hide">Selamat Kuis Sudah Selesai 🎉</p>
        <div id="initial-controls" class="controls">
            <button id="start-btn" class="btn">Mulai</button>
            <button id="continue-btn" class="btn hide">Lanjutkan</button>
        </div>
        <div id="question-counter" class="question-counter-text hide">0/0</div>
        <div id="question-container" class="hide">
            <div id="question">Kata Bahasa Inggris</div>
            <div id="answer-buttons" class="btn-grid">
            </div>
            <div id="skip-navigation-controls" class="controls hide">
                <button id="prev-50-btn" class="btn skip-btn">&laquo; 50</button>
                <button id="next-50-btn" class="btn skip-btn">50 &raquo;</button>
            </div>
        </div>
    </div>

    <script>
        const startButton = document.getElementById('start-btn');
        const continueButton = document.getElementById('continue-btn');
        const initialControls = document.getElementById('initial-controls');
        const completionMessageElement = document.getElementById('completion-message');
        const questionContainerElement = document.getElementById('question-container');
        const questionElement = document.getElementById('question');
        const answerButtonsElement = document.getElementById('answer-buttons');
        const questionCounterElement = document.getElementById('question-counter');

        const skipNavigationControls = document.getElementById('skip-navigation-controls');
        const prev50Button = document.getElementById('prev-50-btn');
        const next50Button = document.getElementById('next-50-btn');
        const JUMP_AMOUNT = 50;

        let orderedQuestions, currentQuestionIndex;
        let score = 0;
        let questionTimeout;

        // Daftar kata mentah dari PDF (Inggris: Indonesia) - Total 1580 kata
        const rawVocabularyList = [
            // Kosakata 1-308 (dari PDF pertama)disini yahhhhh 
{ en: 'Apa judul ebook ini?', id: 'JAGOAN ARDUINO' },
  { en: 'Siapa penulis ebook "Jagoan Arduino"?', id: 'Zamisyak Oby' },
  { en: 'Apa profesi Zamisyak Oby?', id: 'Konsultan Project Robotic' },
  { en: 'Kapan ebook ini diterbitkan?', id: 'Tahun 2018' },
  { en: 'Siapa penerbit ebook ini?', id: 'Indobot Robotic Center' },
  { en: 'Apa pesan yang ingin disampaikan penulis melalui kutipan di halaman 3?', id: '"Hargailah karya orang lain maka suatu saat karyamu akan lebih dihargai orang lain"' },
  { en: 'Apa tujuan dari penulisan buku "Jagoan Arduino" menurut kata pengantar?', id: 'Semoga tulisan ini dapat menjadi pedoman tambahan dalam belajar pemrogaman Arduino.' },
  { en: 'Mengapa pemrogaman dianggap penting di era teknologi informasi menurut buku ini?', id: 'Pemrogaman sudah menjadi kebutuhan penting di era teknologi informasi ini.' },
  { en: 'Meskipun banyak alat elektronik otomatis, mengapa membuat program sendiri tetap diperlukan?', id: 'Membuat progam sendiri untuk kebutuhan yang lebih spesifik akan tetap diperlukan.' },
  { en: 'Apakah program komersial selalu sesuai dengan semua permasalahan?', id: 'Progam komesil belum tentu sesuai dengan permasalahan yang akan dihadapi.' },
  { en: 'Apakah belajar elektronika dan coding membutuhkan waktu lama menurut buku ini?', id: 'Belajar elektronika dan coding tidak butuh waktu yang lama.' },
  { en: 'Mengapa pemrogaman bukan pekerjaan yang sulit menurut buku ini?', id: 'Karena dengan Arduino semua bisa.' },
  { en: 'Di mana belajar pemrogaman Arduino bisa diakses secara global?', id: 'Melalui internet, karena Arduino sudah memiliki wadah komunitas secara global.' },
  { en: 'Apa yang menjadi alasan orang semakin membutuhkan kegiatan memprogram?', id: 'Bukan hanya karena hobi, tapi karena tuntutan permasalahn yang sekarang ini harus diselesaikan dengan otomatisasi.' },
  { en: 'Bagaimana sebagian besar orang belajar memprogram menurut buku ini?', id: 'Sebagian besar orang belajar memprogam dari Learning by doing.' },
  { en: 'Apa yang dibutuhkan dalam memprogram agar hasilnya maksimal?', id: 'Memprogam membutuhkan metode yang baik supaya hasil progam bisa maksimal.' },
  { en: 'Apa yang akan diajarkan oleh buku Jagoan Arduino?', id: 'Buku Jagoan Arduino akan mengajarkan tentang pemrogaman yang baik dan mengupas tentang dunia Arduino.' },
  { en: 'Kepada siapa penulis mengucapkan terima kasih an utama dalam buku ini?', id: 'Alloh SWT.' },
  { en: 'Siapa Rasulullah SAW bagi penulis?', id: 'Suri tauladan dan panutan hingga akhir hayat nanti.' },
  { en: 'Siapa nama ibu penulis yang selalu mendukungnya?', id: 'Darwiyah.' },
  { en: 'Siapa nama ayah penulis yang selalu mendukungnya?', id: 'Sukardi.' },
  { en: 'Siapa saja kakak penulis yang disebutkan?', id: 'Iken Jhonatra dan Laode Kurnia Sandi.' },
  { en: 'Kepada tim mana penulis mengucapkan terima kasih karena selalu membersamai dalam mewujudkan mimpi besar?', id: 'Tim Indobot Robotic Center.' },
  { en: 'Siapa mentor bisnis penulis?', id: 'Coach Mugihardi.' },
  { en: 'Siapa guru kehidupan penulis yang mengajarinya banyak hal?', id: 'Pak Alfan.' },
  { en: 'Siapa saja guru imajiner penulis yang disebutkan?', id: 'Dewa Eka Prayoga, Saptuari Sugiharto, Jaya Setiabudi.' },
  { en: 'Apa yang telah Arduino rubah dalam hidup penulis?', id: 'Arduino sudah merubah hidup saya menjadi lebih mudah.' },
  { en: 'Kepada siapa lagi penulis mengucapkan terima kasih yang dianggap luar biasa?', id: 'Anda, para pembaca buku ini.' },
  { en: 'Di halaman berapa daftar isi buku ini dimulai?', id: 'Halaman 6.' },
  { en: 'Apa nama website yang sering disebutkan di bagian bawah halaman buku?', id: 'WWW.INDOBOTSTORE.COM.' },
  { en: 'Apa definisi Arduino menurut buku ini?', id: 'Arduino merupakan platform prototyping open-source hardware yang mudah digunakan dalam membuat suatu projek berbasis pemrogaman.' },
  { en: 'Apa kemampuan Arduino Board terkait input dan output?', id: 'Arduino Board mampu membaca inputan berupa sensor, tombol dan mengolah menjadi outputan seperti mengaktifkan motor, menyalakan LED dan sebagainya.' },
  { en: 'Bagaimana cara memprogram Arduino Board?', id: 'Dengan memberikan set instruksi tertentu dengan menggunakan Arduino programming language, dan Software Arduino (IDE).' },
  { en: 'Di sistem operasi apa saja Arduino dapat bekerja?', id: 'Arduino dapat bekerja di Mac, Windows, dan Linux.' },
  { en: 'Untuk apa Arduino bisa digunakan dalam bidang ilmiah?', id: 'Semua orang bisa membuat instrumen ilmiah menggunakan Arduino untuk membuktikan prinsip-prinsip kimia dan fisika, atau untuk memulai dengan pemrogaman robotika.' },
  { en: 'Siapa saja yang dapat memulai menggunakan Arduino?', id: 'Siapapun seperti anak anak, seniman, progamer dan penghobi elektronika.' },
  { en: 'Di mana pengguna Arduino dapat berdiskusi secara online?', id: 'Di Facebook, Twitter maupun web arduino.cc dan komunitas di daerah.' },
  { en: 'Apa salah satu keuntungan Arduino dari segi harga?', id: 'Arduino Board relatif murah, di Indonesia dari harga Rp 100.000 - Rp 400.000.' },
  { en: 'Apa yang dimaksud dengan cross-platform pada Software Arduino (IDE)?', id: 'Software Arduino (IDE) lebih fleksibel karena dapat digunakan di Windows, Macintosh OSX, dan sistem operasi Linux.' },
  { en: 'Bagaimana kemudahan penggunaan Software Arduino (IDE)?', id: 'Software Arduino (IDE) mudah digunakan untuk pemula dan tingkat lanjut.' },
  { en: 'Apakah software Arduino bersifat open source?', id: 'Ya, perangkat lunak Arduino diterbitkan sebagai alat Open Source.' },
  { en: 'Bahasa pemrograman apa yang digunakan Arduino dan dapat dikembangkan?', id: 'Bahasa yang digunakan ialah bahasa C untuk AVR dan dapat dikembangkan lagi untuk membuat library melalui C++.' },
  { en: 'Di bawah lisensi apa hardware Arduino Board diterbitkan?', id: 'Di bawah lisensi Creative Commons.' },
  { en: 'Apa yang dapat dilakukan desainer sirkuit berpengalaman dengan adanya lisensi Creative Commons pada Arduino?', id: 'Desainer sirkuit yang berpengalaman dapat membuat versi mereka sendiri, dan mengembangkan sendiri.' },
  { en: 'Apa manfaat bagi pengguna yang relatif tidak berpengalaman dalam membangun versi papan Arduino sendiri?', id: 'Dapat membangun versi papan arduino untuk memahami cara kerjanya dan menghemat uang.' },
  { en: 'Sebutkan tiga jenis Arduino Board yang ditampilkan di halaman 10.', id: 'Arduino Uno, Arduino 101, Arduino Pro (dan lainnya).' },
  { en: 'Papan Arduino mana yang dianggap terbaik untuk memulai belajar elektronik dan coding?', id: 'Arduino Uno.' },
  { en: 'Mengapa Arduino Uno sering digunakan untuk memulai belajar?', id: 'Karena lebih kuat dan banyak digunakan untuk memulai belajar elektronik dan coding.' },
  { en: 'Komponen utama apa yang terdapat di tengah papan Arduino Uno (mikrokontroler)?', id: 'Atmega328.' },
  { en: 'Jenis konektor USB apa yang digunakan pada Arduino Uno?', id: 'USB tipe B.' },
  { en: 'Berapa rentang tegangan input DC yang direkomendasikan untuk Arduino Uno?', id: '7,4V - 9V.' },
  { en: 'Ada berapa pin input analog pada Arduino Uno?', id: 'Uno memiliki 6 input analog, berlabel AO melalui A5.' },
  { en: 'Berapa resolusi bit yang disediakan oleh masing-masing pin input analog Arduino Uno?', id: 'Masing-masing menyediakan 10 bit resolusi (yaitu 1024 nilai yang berbeda).' },
  { en: 'Secara default, dari mana pin input analog Arduino Uno mengukur tegangan?', id: 'Secara default mereka mengukur dari ground ke 5 volt.' },
  { en: 'Pin apa yang digunakan untuk mengubah batas atas rentang pengukuran input analog Arduino Uno?', id: 'Pin AREF.' },
  { en: 'Fungsi apa yang digunakan bersama pin AREF untuk mengubah referensi tegangan input analog?', id: 'Fungsi analogReference().' },
  { en: 'Apa mikrokontroler yang digunakan pada Arduino Uno?', id: 'ATmega328P.' },
  { en: 'Berapa tegangan operasi Arduino Uno?', id: '5V.' },
  { en: 'Berapa batas tegangan input untuk Arduino Uno?', id: '6-20V.' },
  { en: 'Berapa jumlah pin Digital I/O pada Arduino Uno?', id: '14 (dimana 6 memberikan output PWM).' },
  { en: 'Berapa jumlah pin PWM Digital I/O pada Arduino Uno?', id: '6.' },
  { en: 'Berapa arus DC per pin I/O pada Arduino Uno?', id: '20 mA.' },
  { en: 'Berapa arus DC untuk pin 3.3V pada Arduino Uno?', id: '50 mA.' },
  { en: 'Berapa kapasitas Flash Memory pada Arduino Uno?', id: '32 KB (ATmega328P), yang 0,5 KB digunakan oleh bootloader.' },
  { en: 'Berapa kapasitas SRAM pada Arduino Uno?', id: '2 KB (ATmega328P).' },
  { en: 'Berapa kapasitas EEPROM pada Arduino Uno?', id: '1 KB (ATmega328P).' },
  { en: 'Berapa kecepatan clock Arduino Uno?', id: '16 MHz.' },
  { en: 'Bagaimana cara menyalakan Arduino Uno?', id: 'Dengan menghubungkan port USB pada USB tipe B arduino dengan PC/Laptop atau bisa menggunakan tegangan eksternal melalui DC IN.' },
  { en: 'Berapa tegangan eksternal yang dianjurkan untuk Arduino Uno melalui DC IN?', id: '7 sampai 9V.' },
  { en: 'Apa fungsi lain dari pin 0 pada Arduino Uno?', id: 'Rx (Serial Receiver).' },
  { en: 'Apa fungsi lain dari pin 1 pada Arduino Uno?', id: 'Tx (Serial Transmiter).' },
  { en: 'Pin digital mana pada Arduino Uno yang dapat digunakan untuk interupsi eksternal?', id: 'Pin 2 dan Pin 3.' },
  { en: 'Pin digital mana pada Arduino Uno yang berfungsi sebagai output PWM Timer 2?', id: 'Pin 3.' },
  { en: 'Pin digital mana pada Arduino Uno yang berfungsi sebagai output PWM Timer 0?', id: 'Pin 5 dan Pin 6.' },
  { en: 'Pin digital mana pada Arduino Uno yang berfungsi sebagai output PWM Timer 1?', id: 'Pin 9, Pin 10, dan Pin 11.' },
  { en: 'Pin digital mana pada Arduino Uno yang memiliki fungsi SPI-SS?', id: 'Pin 10.' },
  { en: 'Pin digital mana pada Arduino Uno yang memiliki fungsi SPI-MOSI?', id: 'Pin 11.' },
  { en: 'Pin digital mana pada Arduino Uno yang memiliki fungsi SPI-MISO?', id: 'Pin 12.' },
  { en: 'Pin digital mana pada Arduino Uno yang memiliki fungsi SPI-SCK dan terhubung ke LED internal?', id: 'Pin 13.' },
  { en: 'Pin analog mana pada Arduino Uno yang memiliki fungsi lain sebagai TWI-SDA?', id: 'A4.' },
  { en: 'Pin analog mana pada Arduino Uno yang memiliki fungsi lain sebagai TWI-SCK?', id: 'A5.' },
  { en: 'Di mana pengguna dapat mengunduh software Arduino IDE?', id: 'Pengguna dapat mengunduh dari https://www.arduino.cc/en/Main/Software.' },
  { en: 'Apa langkah pertama setelah membuka file instalasi Arduino IDE (arduino.exe)?', id: 'Klik "Yes" pada jendela User Account Control.' },
  { en: 'Apa yang harus dilakukan setelah jendela "Arduino Setup: License Agreement" muncul?', id: 'Klik "I Agree".' },
  { en: 'Apa yang sebaiknya dilakukan pada jendela "Arduino Setup: Installation Options"?', id: 'Centang semua komponen dan klik "Next".' },
  { en: 'Apa yang dilakukan pada jendela "Arduino Setup: Installation Folder"?', id: 'Tentukan Destination Folder (bisa default di C) lalu klik "Install".' },
  { en: 'Apa yang menandakan proses instalasi Arduino IDE telah selesai?', id: 'Muncul tulisan "Completed".' },
  { en: 'Apa yang harus diklik setelah proses instalasi Arduino IDE selesai?', id: 'Klik "Close".' },
  { en: 'Apa kepanjangan dari Arduino IDE?', id: 'Arduino Integrated Development Environment.' },
  { en: 'Sebutkan lima komponen utama dari Arduino Software (IDE).', id: 'Editor teks, area pesan, konsol teks, toolbar dengan tombol untuk fungsi umum, dan serangkaian menu.' },
  { en: 'Apa fungsi tombol "Verify" pada Arduino IDE?', id: 'Memeriksa kode untuk kesalahan kompilasi.' },
  { en: 'Apa fungsi tombol "Upload" pada Arduino IDE?', id: 'Mengkompilasi kode dan meng-upload ke papan yang dikonfigurasi.' },
  { en: 'Apa yang terjadi jika tombol "Shift" ditekan saat mengklik ikon "Upload" jika menggunakan programmer eksternal?', id: 'Teks akan berubah menjadi "Upload Using Programmer".' },
  { en: 'Apa fungsi tombol "New" pada Arduino IDE?', id: 'Membuat sketsa baru.' },
  { en: 'Apa fungsi tombol "Open" pada Arduino IDE?', id: 'Membuka file yang sudah ada.' },
  { en: 'Apa fungsi tombol "Save" pada Arduino IDE?', id: 'Mengamankan sketsa Anda.' },
  { en: 'Apa fungsi tombol "Serial Monitor" pada Arduino IDE?', id: 'Membuka Monitor serial.' },
  { en: 'Struktur bahasa apa yang digunakan dalam pemrograman Arduino?', id: 'Struktur Bahasa C.' },
  { en: 'Sebutkan tiga tahapan mekanisme pemrograman Arduino.', id: 'Membuat sket progam, meng-compile, selanjutnya proses upload pada papan arduino.' },
  { en: 'Apa yang dimaksud dengan metode upload dalam pengisian program ke Arduino?', id: 'Mengisi papan arduino dengan progam yang sudah berbentuk Hex atau hasil compile dari bahasa C ke bahasa mesin.' },
  { en: 'Program Arduino dapat dibagi menjadi tiga bagian utama, sebutkan!', id: 'Struktur, nilai-nilai (variabel dan konstanta), dan fungsi.' },
  { en: 'Kapan fungsi `setup()` dipanggil?', id: 'Fungsi `setup()` dipanggil ketika sketsa progam dimulai.' },
  { en: 'Untuk apa fungsi `setup()` digunakan?', id: 'Digunakan untuk menginisialisasi variabel, mode pin, penggunaan librari, dll.' },
  { en: 'Berapa kali fungsi `setup()` akan berjalan?', id: 'Hanya akan berjalan sekali, setelah power arduino dinyalakan atau saat mereset papan Arduino.' },
  { en: 'Apa fungsi dari `pinMode(ledPin, OUTPUT);` dalam contoh Program 1.1?', id: 'Mengatur pin yang tersimpan di variabel `ledPin` sebagai output.' },
  { en: 'Apa fungsi dari `digitalWrite(ledPin, HIGH);` dalam contoh Program 1.1?', id: 'Menyalakan LED pada pin yang tersimpan di variabel `ledPin`.' },
  { en: 'Apa yang dilakukan Program 1.1?', id: 'Menyalakan LED pada pin 13 selama 5 detik lalu mati, dan eksekusi ini dilakukan hanya sekali.' },
  { en: 'Fungsi apa yang dijalankan setelah `setup()`?', id: 'Fungsi `loop()`.' },
  { en: 'Bagaimana cara kerja fungsi `loop()`?', id: 'Fungsi `loop()` akan melakukan loop berturut-turut dimana program akan dijalankan terus menerus secara berurutan dan loop untuk mengontrol papan Arduino.' },
  { en: 'Apa yang dilakukan Program 1.2 jika tombol pada pin 3 ditekan?', id: 'Pada serial monitor akan menampilkan huruf H.' },
  { en: 'Apa yang dilakukan Program 1.2 jika tombol pada pin 3 dilepaskan?', id: 'Pada serial monitor akan menampilkan huruf L.' },
  { en: 'Untuk apa komentar (// atau /* */) digunakan dalam program?', id: 'Komentar digunakan untuk memberikan keterangan pada progam yang dibuat.' },
  { en: 'Apakah komentar dieksekusi oleh program?', id: 'Komentar tidak dieksekusi.' },
  { en: 'Bagaimana cara menulis komentar untuk satu baris?', id: 'Diawali dengan dua garis miring (//).' },
  { en: 'Bagaimana cara menulis komentar untuk lebih dari satu baris?', id: 'Diawali dengan garis miring lalu tanda bintang (/*) serta diakhiri dengan bintang lalu garis miring (*/).' },
  { en: "Dengan awalan apa bilangan biner ditulis dalam pemrograman Arduino?", id: "Ditulis dengan awalan huruf '0b'." },
  { en: "Bagaimana bilangan desimal ditulis dalam pemrograman Arduino?", id: "Ditulis biasa tanpa awalan." },
  { en: "Dengan awalan apa bilangan oktal ditulis dalam pemrograman Arduino?", id: "Ditulis dengan awalan angka '0'." },
  { en: "Dengan awalan apa bilangan heksadesimal ditulis dalam pemrograman Arduino?", id: "Diawali dengan '0x'." },
  { en: 'Struktur kontrol `if` digunakan untuk apa?', id: 'Digunakan untuk mengecek suatu kondisi. Jika benar maka perintah didalam `if` akan dikerjakan.' },
  { en: 'Kapan perintah di dalam blok `else` pada struktur `if-else` akan dikerjakan?', id: 'Jika kondisinya salah.' },
  { en: 'Kapan struktur `if-else if` digunakan?', id: 'Untuk melakukan pengecekan suatu kondisi lebih dari satu.' },
  { en: 'Apa perbedaan utama antara `switch-case` dengan `if-else if`?', id: 'Kondisi yang diuji pada `switch-case` berupa sebuah nilai variabel.' },
  { en: 'Apa yang terjadi jika nilai variabel dalam `switch-case` tidak memenuhi salah satu `case`?', id: 'Dia akan mengerjakan `default`.' },
  { en: 'Kapan perulangan `while` akan terus berjalan?', id: 'Selama kondisi dalam `while` benar.' },
  { en: 'Bagaimana cara kerja perulangan `do-while`?', id: 'Perulangan ini akan melakukan pernyataan/perintah lalu akan melihat kondisi dalam `while`. Jika benar maka pernyataan/perintah akan dieksekusi kembali.' },
  { en: 'Kapan perulangan `for` digunakan?', id: 'Digunakan untuk perulangan yang sifatnya terbatas.' },
  { en: 'Apa saja tiga bagian dalam deklarasi perulangan `for`?', id: 'Inisialisasi, kondisi, dan step.' },
  { en: 'Apa fungsi dari `inisialisasi` dalam perulangan `for`?', id: 'Nilai awal suatu variable untuk proses perulangan.' },
  { en: 'Apa fungsi dari `kondisi` dalam perulangan `for`?', id: 'Kondisi yang menentukan proses perulangan, jika benar perulangan dikerjakan.' },
  { en: 'Apa fungsi dari `step` dalam perulangan `for`?', id: 'Tahap perulangan bisa dalam bentuk perkalian, pertambahan, pengurangan dan pembagian.' },
  { en: 'Apa fungsi perintah `goto`?', id: 'Perintah ini digunakan untuk melompat/menuju perintah yang telah diberi label.' },
  { en: 'Apa fungsi perintah `return`?', id: 'Digunakan untuk memberikan nilai balik dari sebuah fungsi.' },
  { en: 'Apa fungsi perintah `continue`?', id: 'Untuk melewati perulangan yang tersisa dari struktur looping (do, for, atau while).' },
  { en: 'Apa fungsi perintah `break`?', id: 'Perintah \'keluar\' dari pernyataan perulangan do, for, atau while. Juga digunakan untuk mengakhiri pernyataan dalam switch case.' },
  { en: 'Apa fungsi dari tanda `;` (semicolon) dalam syntax Arduino?', id: 'Digunakan untuk mengakhiri sebuah pernyataan.' },
  { en: 'Apa fungsi dari `{}` (curly braces) dalam syntax Arduino?', id: 'Bagian utama dari bahasa pemrograman C yang digunakan dalam beberapa konstruksi yang berbeda dalam beberapa fungsi.' },
  { en: 'Apa fungsi dari `#define`?', id: 'Komponen C yang berguna yang memungkinkan programmer untuk memberi nama untuk nilai konstan sebelum program dikompilasi.' },
  { en: 'Apa fungsi dari `#include`?', id: 'Digunakan untuk memasukkan perpustakaan atau library di luar di sketsa progam.' },
  { en: 'Sebutkan operator aritmatika untuk penjumlahan.', id: '+' },
  { en: 'Sebutkan operator aritmatika untuk sisa bagi.', id: '%' },
  { en: 'Apa arti dari operator perbandingan `==`?', id: "Persamaan. Jika kedua nilai yang dibandingkan sama maka hasilnya 'true'." },
  { en: 'Apa arti dari operator perbandingan `!=`?', id: "Pertidaksamaan. Jika kedua nilai yang dibandingkan tidak sama hasilnya 'true'." },
  { en: 'Apa arti dari operator boolean `&&`?', id: 'AND.' },
  { en: 'Apa arti dari operator boolean `||`?', id: 'OR.' },
  { en: 'Apa arti dari operator boolean `!`?', id: 'NOT.' },
  { en: 'Untuk apa operator bitwise digunakan?', id: 'Digunakan untuk operasi bit per bit pada nilai integer.' },
  { en: 'Sebutkan salah satu operator bitwise untuk geser kiri.', id: '<<.' },
  { en: 'Apa arti dari operator `++`?', id: 'Pertambahan 1 / increment.' },
  { en: 'Jika `a++`, maka sama dengan ekspresi apa?', id: '$a = a + 1$.' },
  { en: 'Apa itu variabel?', id: 'Variabel adalah suatu wadah untuk menyimpan atau menampung data.' },
  { en: 'Sebutkan salah satu peraturan dalam penamaan variabel.', id: 'Tidak boleh ada spasi, maksimal 32 karakter dan tidak boleh menggunakan istilah baku dalam bahasa C arduino.' },
  { en: 'Bagaimana format umum untuk mendeklarasikan variabel dengan nilai awal?', id: '`[tipe data][spasi] [nama variable][=][nilai]`.' },
  { en: 'Berapa lebar data untuk tipe data `char`?', id: '1 byte.' },
  { en: 'Apa jangkauan nilai untuk tipe data `int`?', id: '-32768 s/d 32767.' },
  { en: 'Apa jangkauan nilai untuk tipe data `unsigned long`?', id: '0 s/d 4294967295.' },
  { en: 'Berapa jumlah pin I/O pada papan Arduino Uno?', id: '20 pin I/O yaitu 14 pin digital dan 6 pin analog.' },
  { en: 'Di fungsi mana inisialisasi fungsi pin I/O dilakukan?', id: 'Di fungsi `setup()`.' },
  { en: 'Bagaimana syntax untuk menginisialisasi fungsi pin I/O?', id: '`pinMode(pin, mode)`.' },
  { en: 'Sebutkan tiga mode yang dapat digunakan dalam `pinMode()`.', id: 'INPUT, INPUT_PULLUP, OUTPUT.' },
  { en: 'Bagaimana cara menuliskan jika pin nomor 3 akan dibuat menjadi input?', id: '`pinMode(3, INPUT);`.' },
  { en: 'Apakah penulisan besar kecilnya huruf berpengaruh dalam pemrograman Arduino?', id: 'Ya, penulisan besar dan kecilnya huruf sangat berpengaruh.' },
  { en: 'Perintah apa yang digunakan untuk menulis data digital ke pin output?', id: '`digitalWrite(pin, value);`.' },
  { en: 'Apa saja nilai `value` yang dapat digunakan dalam `digitalWrite()`?', id: 'HIGH atau LOW.' },
  { en: 'Apa yang perlu dilakukan jika sebuah pin dibuat sebagai input aktif HIGH?', id: 'Dibutuhkan resistor pulldown.' },
  { en: 'Bagaimana cara menggunakan resistor internal pullup pada pin input?', id: 'Dengan memanggil resistor internal dengan pullup pada setiap pin arduino, menggunakan `pinMode(pin, INPUT_PULLUP);`.' },
  { en: 'Perintah apa yang digunakan untuk membaca data digital dari pin input?', id: '`digitalRead(pin);`.' },
  { en: 'Apakah perlu menggunakan `pinMode()` untuk mengatur pin sebagai output saat menggunakan `analogWrite()`?', id: 'Tidak perlu.' },
  { en: 'Bagaimana syntax untuk menulis data analog (PWM) ke pin output?', id: '`analogWrite(pin, value);`.' },
  { en: 'Berapa rentang nilai `value` untuk `analogWrite()`?', id: 'Nilai pwm mulai dari 0-255.' },
  { en: 'Apakah perlu menggunakan `pinMode()` untuk mengatur pin sebagai input saat menggunakan `analogRead()`?', id: 'Tidak perlu.' },
  { en: 'Bagaimana syntax untuk membaca data analog dari pin input?', id: '`analogRead(analogPin);`.' },
  { en: 'Sebutkan pin-pin yang dapat digunakan sebagai input analog pada Arduino Uno.', id: 'A0, A1, A2, A3, A4, A5.' },
  { en: 'Apa fungsi dari `millis()`?', id: 'Menghitung dengan satuan miliseconds sejak papan Arduino mulai menjalankan program.' },
  { en: 'Setelah berapa lama `millis()` akan kembali ke nol?', id: 'Hingga 50 hari setelah itu akan kembali ke nol.' },
  { en: 'Apa fungsi dari `micros()`?', id: 'Menghitung dengan satuan microseconds sejak papan Arduino mulai menjalankan program.' },
  { en: 'Setelah berapa lama `micros()` akan kembali ke nol?', id: 'Hingga 70 menit setelah itu akan kembali ke nol.' },
  { en: 'Apa fungsi dari `delay()`?', id: 'Jeda program untuk jumlah waktu (dalam milidetik).' },
  { en: 'Ada berapa milidetik dalam satu detik?', id: '1000 milidetik.' },
  { en: 'Apa fungsi dari `delayMicroseconds()`?', id: 'Jeda program untuk jumlah waktu (dalam mikrodetik).' },
  { en: 'Fungsi apa yang harus digunakan untuk menerjemahkan pin digital sebenarnya ke nomor interrupt tertentu untuk `attachInterrupt()`?', id: '`digitalPinToInterrupt(pin)`.' },
  { en: 'Pin digital berapa saja pada Arduino Uno yang dapat digunakan untuk interrupt?', id: '2, 3.' },
  { en: 'Sebutkan salah satu syntax yang disarankan untuk `attachInterrupt()`.', id: '`attachInterrupt(digitalPinToInterrupt(pin), ISR, mode);`.' },
  { en: 'Apa itu ISR dalam parameter `attachInterrupt()`?', id: 'ISR untuk panggilan ketika interupsi terjadi; fungsi ini harus ada parameter dan kembali apa-apa. Fungsi ini kadang-kadang disebut sebagai rutin layanan interupsi.' },
  { en: 'Sebutkan salah satu mode pemicu interrupt.', id: 'LOW, CHANGE, RISING, FALLING.' },
  { en: 'Apa fungsi dari `detachInterrupt()`?', id: 'Mematikan interupsi yang diberikan.' },
  { en: 'Apa fungsi dari `interrupts()`?', id: 'Mengaktifkan kembali interupsi (setelah dinonaktifkan oleh `noInterrupts()`).' },
  { en: 'Apakah beberapa fungsi akan bekerja saat interupsi dinonaktifkan?', id: 'Beberapa fungsi tidak akan bekerja saat interupsi dinonaktifkan.' },
  { en: 'Apa fungsi dari `noInterrupts()`?', id: 'Menonaktifkan interupsi.' },
  { en: 'Pin mana pada Arduino yang dapat dimanfaatkan untuk komunikasi serial?', id: 'Pin Rx dan Tx pada arduino maupun pada USB.' },
  { en: 'Perintah apa yang digunakan untuk memulai komunikasi serial dan mengatur baud rate?', id: '`Serial.begin(baudrate);`.' },
  { en: 'Perintah apa yang digunakan untuk mengirim data melalui serial dan menambahkan baris baru?', id: '`Serial.println(data);`.' },
  { en: 'Apa kekurangan dari project Jam Digital Menggunakan LCD 1602 yang dijelaskan?', id: 'Tidak dilengkapi RTC sebagai time keeper, sehingga ketika daya pada sistem putus, maka waktu yang ditampilkan akan ter-reset.' },
  { en: 'Papan Arduino apa yang digunakan dalam project Jam Digital?', id: 'Arduino Uno R3.' },
  { en: 'Sebutkan salah satu hardware yang dibutuhkan untuk project Jam Digital selain Arduino Uno dan kabel USB.', id: 'Breadboard, Kabel male female, LCD 1602, Trimpot ukuran 5K ohm, Pin deret, Soldir + timah.' },
  { en: 'Software apa saja yang dibutuhkan untuk project Jam Digital?', id: 'Arduino IDE, Library time.h, File time.ino.' },
  { en: 'Input apa yang digunakan pada project Kalkulator Sederhana?', id: 'Keypad.' },
  { en: 'Output apa yang digunakan pada project Kalkulator Sederhana?', id: 'LCD.' },
  { en: 'Operasi matematika apa saja yang dapat dilakukan oleh Kalkulator Sederhana ini?', id: 'Penjumlahan, pengurangan, perkalian, dan pembagian.' },
  { en: 'Tombol apa pada keypad yang digunakan untuk fungsi sama dengan (=) pada Kalkulator Sederhana?', id: 'Tombol pagar (#).' }
        ];

        let questions = [];

        rawVocabularyList.sort((a, b) => {
            const enA = a.en.toLowerCase();
            const enB = b.en.toLowerCase();
            if (enA < enB) return -1;
            if (enA > enB) return 1;
            return 0;
        });

        function generateQuestions() {
            const allIndonesianTranslations = rawVocabularyList.map(item => item.id);
            questions = [];
            rawVocabularyList.forEach(vocabItem => {
                const correctAnswer = vocabItem.id;
                const distractors = [];
                let attempts = 0;
                while (distractors.length < 3 && attempts < allIndonesianTranslations.length * 2) {
                    const randomIndex = Math.floor(Math.random() * allIndonesianTranslations.length);
                    const potentialDistractor = allIndonesianTranslations[randomIndex];
                    if (potentialDistractor !== correctAnswer && !distractors.includes(potentialDistractor)) {
                        distractors.push(potentialDistractor);
                    }
                    attempts++;
                }
                while (distractors.length < 3) {
                    const fallbackOptions = ["opsi lain A", "opsi lain B", "opsi lain C", "opsi lain D", "opsi lain E", "opsi lain F"];
                    let fallbackIndex = 0;
                    let safetyNet = 0;
                    while(distractors.length < 3 && safetyNet < fallbackOptions.length * 3) {
                        const fbOption = fallbackOptions[fallbackIndex % fallbackOptions.length] + `_${distractors.length}${Math.floor(Math.random()*100)}`;
                        if (fbOption !== correctAnswer && !distractors.includes(fbOption)) {
                             distractors.push(fbOption);
                        }
                        fallbackIndex++;
                        safetyNet++;
                    }
                     if(distractors.length < 3) {
                        for(let i=0; i < (3-distractors.length); i++){
                            distractors.push("pilihan default " + (i+1+distractors.length) + Math.random().toString(36).substring(7));
                        }
                     }
                }
                const answerOptions = [
                    { text: correctAnswer, correct: true },
                    { text: distractors[0], correct: false },
                    { text: distractors[1], correct: false },
                    { text: distractors[2], correct: false }
                ];
                questions.push({
                    question: vocabItem.en,
                    answers: answerOptions
                });
            });
        }

        generateQuestions();

        function saveProgress() {
            if (!questionContainerElement.classList.contains('hide') && orderedQuestions && currentQuestionIndex < orderedQuestions.length) {
                 const progress = {
                    currentQuestionIndex: currentQuestionIndex,
                    score: score,
                    orderedQuestions: orderedQuestions
                };
                localStorage.setItem('quizProgress', JSON.stringify(progress));
            }
        }

        function loadProgress() {
            const savedProgress = localStorage.getItem('quizProgress');
            if (savedProgress) {
                try {
                    const progressData = JSON.parse(savedProgress);
                    if (progressData && typeof progressData.currentQuestionIndex === 'number' &&
                        typeof progressData.score === 'number' && Array.isArray(progressData.orderedQuestions) &&
                        progressData.orderedQuestions.length > 0 &&
                        progressData.currentQuestionIndex < progressData.orderedQuestions.length &&
                        progressData.orderedQuestions.length === questions.length) { // Validasi tambahan: jumlah soal harus sama
                        return progressData;
                    } else {
                        clearProgress();
                        return null;
                    }
                } catch (e) {
                    console.error("Error parsing saved progress:", e);
                    clearProgress();
                    return null;
                }
            }
            return null;
        }

        function clearProgress() {
            localStorage.removeItem('quizProgress');
        }

        prev50Button.addEventListener('click', () => navigateQuestions(-JUMP_AMOUNT));
        next50Button.addEventListener('click', () => navigateQuestions(JUMP_AMOUNT));

        function navigateQuestions(amount) {
            clearTimeout(questionTimeout);
            if (!orderedQuestions || orderedQuestions.length === 0) return;

            let newIndex = currentQuestionIndex + amount;
            if (newIndex < 0) newIndex = 0;
            else if (newIndex >= orderedQuestions.length) newIndex = orderedQuestions.length - 1;

            if (newIndex !== currentQuestionIndex) {
                currentQuestionIndex = newIndex;
                setNextQuestion();
            } else {
                updateSkipButtonStates();
            }
        }

        function updateSkipButtonStates() {
            if (!orderedQuestions || orderedQuestions.length === 0 || questionContainerElement.classList.contains('hide')) {
                skipNavigationControls.classList.add('hide');
                if(prev50Button) prev50Button.disabled = true;
                if(next50Button) next50Button.disabled = true;
                return;
            }
            skipNavigationControls.classList.remove('hide');
            prev50Button.disabled = currentQuestionIndex === 0;
            next50Button.disabled = currentQuestionIndex === (orderedQuestions.length - 1);

            if (orderedQuestions.length <= 1) {
                prev50Button.disabled = true;
                next50Button.disabled = true;
            }
        }

        window.addEventListener('load', () => {
            const savedData = loadProgress();
            startButton.innerText = 'Mulai';
            completionMessageElement.classList.add('hide');
            if (savedData) {
                continueButton.classList.remove('hide');
            } else {
                continueButton.classList.add('hide');
            }
            if (questionContainerElement.classList.contains('hide')) {
                initialControls.classList.remove('hide');
                skipNavigationControls.classList.add('hide');
            } else {
                 initialControls.classList.add('hide');
            }
        });

        startButton.addEventListener('click', () => startGame(false));
        continueButton.addEventListener('click', () => startGame(true));

        function startGame(isContinuing = false) {
            clearTimeout(questionTimeout);
            completionMessageElement.classList.add('hide');
            if (!isContinuing) {
                startButton.innerText = 'Mulai';
            }
            initialControls.classList.add('hide');
            questionContainerElement.classList.remove('hide');
            questionCounterElement.classList.remove('hide');

            const savedData = loadProgress();
            if (isContinuing && savedData && savedData.orderedQuestions && savedData.orderedQuestions.length === questions.length) {
                orderedQuestions = savedData.orderedQuestions; // Gunakan urutan yang disimpan (yang seharusnya sudah urut)
                currentQuestionIndex = savedData.currentQuestionIndex;
                score = savedData.score;
            } else {
                clearProgress();
                // Gunakan urutan soal langsung dari 'questions' yang sudah diurutkan secara abjad
                // Tidak ada pengacakan di sini.
                orderedQuestions = [...questions]; // Membuat salinan array questions yang sudah terurut
                currentQuestionIndex = 0;
                score = 0;
            }

            if (!orderedQuestions || orderedQuestions.length === 0) {
                showResults();
                completionMessageElement.innerText = "Tidak ada soal untuk ditampilkan.";
                completionMessageElement.style.color = "#dc3545";
                completionMessageElement.classList.remove('hide');
                startButton.innerText = 'Mulai';
                return;
            }
            setNextQuestion();
        }

        function setNextQuestion() {
            resetState();
            if (orderedQuestions && currentQuestionIndex < orderedQuestions.length) {
                questionCounterElement.innerText = `${currentQuestionIndex + 1} / ${orderedQuestions.length}`;
                showQuestion(orderedQuestions[currentQuestionIndex]);
                saveProgress();
                if (document.activeElement && typeof document.activeElement.blur === 'function') {
                    document.activeElement.blur();
                }
            } else {
                showResults();
            }
            updateSkipButtonStates();
        }

        function showQuestion(questionData) {
            questionElement.innerText = questionData.question;
            answerButtonsElement.innerHTML = '';
            const shuffledAnswers = [...questionData.answers].sort(() => Math.random() - 0.5); // Pilihan jawaban tetap diacak
            shuffledAnswers.forEach(answer => {
                const button = document.createElement('button');
                button.innerText = answer.text;
                button.classList.add('btn');
                if (answer.correct) {
                    button.dataset.correct = answer.correct;
                }
                button.addEventListener('click', selectAnswer);
                answerButtonsElement.appendChild(button);
            });
        }

        function resetState() {
            clearTimeout(questionTimeout);
            while (answerButtonsElement.firstChild) {
                answerButtonsElement.removeChild(answerButtonsElement.firstChild);
            }
        }

        function selectAnswer(e) {
            const selectedButton = e.target;
            const correct = selectedButton.dataset.correct === 'true';
            if (correct) { score++; }
            Array.from(answerButtonsElement.children).forEach(button => {
                setStatusClass(button, button.dataset.correct === 'true');
                button.disabled = true;
            });
            saveProgress();
            questionTimeout = setTimeout(() => {
                if (orderedQuestions && currentQuestionIndex < orderedQuestions.length -1) {
                    currentQuestionIndex++;
                    setNextQuestion();
                } else if (orderedQuestions && currentQuestionIndex === orderedQuestions.length -1) {
                    showResults();
                }
            }, 1500);
        }

        function setStatusClass(element, correct) {
            clearStatusClass(element);
            if (correct) { element.classList.add('correct'); }
            else { element.classList.add('wrong'); }
        }

        function clearStatusClass(element) {
            element.classList.remove('correct');
            element.classList.remove('wrong');
        }

        function showResults() {
            clearTimeout(questionTimeout);
            questionContainerElement.classList.add('hide');
            questionCounterElement.classList.add('hide');
            skipNavigationControls.classList.add('hide');
            clearProgress();
            completionMessageElement.innerText = "Selamat Kuis Sudah Selesai 🎉";
            completionMessageElement.style.color = "#28a745";
            completionMessageElement.classList.remove('hide');
            startButton.innerText = 'Ulangi Kuis';
            initialControls.classList.remove('hide');
            continueButton.classList.add('hide');
        }
    </script>
</body>
</html>
