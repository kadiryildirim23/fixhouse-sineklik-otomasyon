<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fix House - Sineklik Otomasyonu & Canlı Sipariş Havuzu</title>
    <!-- Firebase SDK Bağlantıları (Canlı Havuz İçin) -->
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
    <style>
        :root {
            --primary-color: #e65100;
            --secondary-color: #263238;
            --bg-color: #f5f7f8;
            --surface-color: #ffffff;
            --text-color: #37474f;
            --border-color: #cfd8dc;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background-color: var(--bg-color); color: var(--text-color); padding: 20px; }
        .container { max-width: 1200px; margin: 0 auto; display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 900px) { .container { grid-template-columns: 1fr; } }
        header { grid-column: 1 / -1; background-color: var(--secondary-color); color: white; padding: 20px; border-radius: 8px; display: flex; justify-content: space-between; align-items: center; border-bottom: 4px solid var(--primary-color); }
        h1 { font-size: 24px; }
        .card { background: var(--surface-color); padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); border: 1px solid var(--border-color); }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: 600; font-size: 14px; }
        input, select { width: 100%; padding: 10px; border: 1px solid var(--border-color); border-radius: 4px; font-size: 16px; outline: none; transition: border 0.2s; }
        input:focus, select:focus { border-color: var(--primary-color); }
        button { width: 100%; padding: 12px; background-color: var(--primary-color); color: white; border: none; border-radius: 4px; font-size: 16px; font-weight: bold; cursor: pointer; transition: background 0.2s; margin-bottom: 10px; }
        button:hover { background-color: #bf360c; }
        button.secondary { background-color: #455a64; }
        button.secondary:hover { background-color: #37474f; }
        .result-box { background-color: #fff3e0; border-left: 4px solid var(--primary-color); padding: 15px; border-radius: 4px; margin-top: 15px; }
        .order-list { list-style: none; }
        .order-item { background: #fafafa; border: 1px solid var(--border-color); padding: 15px; border-radius: 4px; margin-bottom: 10px; border-left: 4px solid var(--secondary-color); position: relative; }
        .order-item.new { border-left-color: var(--primary-color); background-color: #fff8e1; }
        .order-item h4 { margin-bottom: 5px; color: var(--secondary-color); }
        .order-item p { font-size: 14px; margin-bottom: 3px; }
        .delete-btn { position: absolute; right: 15px; top: 15px; background: #c62828; color: white; padding: 5px 10px; font-size: 12px; border-radius: 4px; width: auto; margin: 0; }
        .delete-btn:hover { background: #b71c1c; }
        .status { font-weight: bold; color: var(--primary-color); }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Fix House - Atölye Sipariş Otomasyonu</h1>
        <div id="connection-status"><span class="status">● Buluta Bağlanıyor...</span></div>
    </header>

    <!-- HESAPLAMA VE SİPARİŞ GİRİŞ ALANI -->
    <div class="card">
        <h2>🛠️ Ölçü ve Sipariş Girişi</h2>
        <br>
        <div class="form-group">
            <label for="customerName">Müşteri Adı / Sipariş Başlığı</label>
            <input type="text" id="customerName" placeholder="Örn: Ahmet Yılmaz - Sürsürü">
        </div>
        <div class="form-group">
            <label for="productType">Sineklik Türü</label>
            <select id="productType">
                <option value="Plise Sineklik">Plise Sineklik</option>
                <option value="Kedi Sinekliği">Kedi Sinekliği</option>
                <option value="Sürme Sineklik">Sürme Sineklik</option>
                <option value="Stor Sineklik">Stor Sineklik</option>
            </select>
        </div>
        <div class="form-group">
            <label for="width">Genişlik (cm)</label>
            <input type="number" id="width" step="0.1" placeholder="Örn: 75.5">
        </div>
        <div class="form-group">
            <label for="height">Yükseklik (cm)</label>
            <input type="number" id="height" step="0.1" placeholder="Örn: 120">
        </div>
        <div class="form-group">
            <label for="color">Profil Rengi</label>
            <select id="color">
                <option value="Beyaz">Beyaz</option>
                <option value="Antrasit">Antrasit Gri</option>
                <option value="Altın Meşe">Altın Meşe</option>
                <option value="Kahverengi">Kahverengi</option>
            </select>
        </div>
        <div class="form-group">
            <label for="note">Sipariş Notu</label>
            <input type="text" id="note" placeholder="Örn: Yaylı menteşe olacak">
        </div>

        <button onclick="calculateAndPreview()">Hesapla ve Önizle</button>
        
        <div id="resultPreview" class="result-box" style="display: none;">
            <h3>📐 Hesaplama Önizleme</h3>
            <p id="previewText" style="margin: 10px 0; font-size: 15px; line-height: 1.5;"></p>
            <button class="secondary" onclick="sendOrderToFirebase()">🛒 Siparişi Canlı Havuza Gönder</button>
        </div>
    </div>

    <!-- CANLI SİPARİŞ HAVUZU VE SEPET -->
    <div class="card">
        <h2>📋 Ortak Sipariş Havuzu (Canlı)</h2>
        <br>
        <ul id="orderPool" class="order-list">
            <!-- Siparişler Firebase'den canlı olarak buraya akacak -->
        </ul>
    </div>
</div>

<script>
    // 1. Firebase Bağlantı Ayarları (image_ce7edf.png ve image_ce76e4.png verileriyle tam senkronize)
    const firebaseConfig = {
        apiKey: "AIzaSyB4jTIAo0pAcNVXZEc7bzBHi4wRKBdZWs0",
        authDomain: "fixhouse-f2f29.firebaseapp.com",
        projectId: "fixhouse-f2f29",
        storageBucket: "fixhouse-f2f29.firebasestorage.app",
        messagingSenderId: "21162414048",
        appId: "1:21162414048:web:b9ce3c4341de63fbf46585",
        measurementId: "G-T4PJC735X2",
        databaseURL: "https://fixhouse-f2f29-default-rtdb.firebaseio.com"
    };

    // Firebase'i Başlat
    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();

    // Bağlantı Durumunu Kontrol Et
    const statusDiv = document.getElementById('connection-status');
    database.ref('.info/connected').on('value', function(snapshot) {
        if (snapshot.val() === true) {
            statusDiv.innerHTML = '<span class="status" style="color: #2e7d32;">● Atölye Havuzuna Bağlı (Canlı)</span>';
        } else {
            statusDiv.innerHTML = '<span class="status" style="color: #c62828;">● Bağlantı Kesildi (Çevrimdışı)</span>';
        }
    });

    let sonHesaplama = null;

    // Hesaplama ve Önizleme Fonksiyonu
    function calculateAndPreview() {
        const customerName = document.getElementById('customerName').value || "İsimsiz Sipariş";
        const productType = document.getElementById('productType').value;
        const width = parseFloat(document.getElementById('width').value);
        const height = parseFloat(document.getElementById('height').value);
        const color = document.getElementById('color').value;
        const note = document.getElementById('note').value || "-";

        if (!width || !height) {
            alert("Lütfen genişlik ve yükseklik ölçülerini giriniz usta.");
            return;
        }

        // Atölye Hesaplama Algoritması (Metrekare ve Profil Kesim Hesabı)
        const sqm = (width * height) / 10000;
        const hesaplananSqm = sqm < 0.5 ? 0.5 : sqm; // Minimum 0.5 metrekare hesabı
        
        // Örnek Kesim Ölçüleri (Tül ve Profil toleransları)
        const tulEn = width - 4.5;
        const tulBoy = height - 5;

        sonHesaplama = {
            customerName: customerName,
            productType: productType,
            width: width,
            height: height,
            color: color,
            note: note,
            sqm: hesaplananSqm.toFixed(2),
            tulEn: tulEn.toFixed(1),
            tulBoy: tulBoy.toFixed(1),
            tarih: new Date().toLocaleString('tr-TR')
        };

        // Önizleme Ekranını Doldur
        document.getElementById('previewText').innerHTML = `
            <strong>Müşteri:</strong> ${sonHesaplama.customerName}<br>
            <strong>Tür:</strong> ${sonHesaplama.productType} (${sonHesaplama.color})<br>
            <strong>Ölçü:</strong> ${sonHesaplama.width} cm x ${sonHesaplama.height} cm<br>
            <strong>Hesaplanan Alan:</strong> ${sonHesaplama.sqm} m²<br>
            <strong>İmalat Kesim Ölçüleri:</strong> Tül En: ${sonHesaplama.tulEn} cm / Tül Boy: ${sonHesaplama.tulBoy} cm<br>
            <strong>Not:</strong> ${sonHesaplama.note}
        `;
        document.getElementById('resultPreview').style.display = 'block';
    }

    // Siparişi Firebase Realtime Database Havuzuna Gönder
    function sendOrderToFirebase() {
        if (!sonHesaplama) return;

        // Firebase havuzuna yeni bir kayıt ekle
        database.ref('siparisler').push(sonHesaplama)
            .then(() => {
                alert("Sipariş ortak havuza gönderildi usta!");
                // Formu Temizle
                document.getElementById('customerName').value = '';
                document.getElementById('width').value = '';
                document.getElementById('height').value = '';
                document.getElementById('note').value = '';
                document.getElementById('resultPreview').style.display = 'none';
                sonHesaplama = null;
            })
            .catch((error) => {
                alert("Hata oluştu, buluta gönderilemedi: " + error.message);
            });
    }

    // Firebase'den Verileri Canlı Dinle ve Listeyi Anlık Güncelle
    database.ref('siparisler').on('value', function(snapshot) {
        const orderPool = document.getElementById('orderPool');
        orderPool.innerHTML = ''; // Mevcut listeyi temizle

        const data = snapshot.val();
        if (!data) {
            orderPool.innerHTML = '<li style="text-align: center; color: #90a4ae; padding: 20px;">Havuzda henüz bekleyen sipariş yok usta.</li>';
            return;
        }

        // Siparişleri tersten (en yeni en üstte olacak şekilde) listele
        const keys = Object.keys(data).reverse();
        
        keys.forEach(key => {
            const order = data[key];
            const li = document.createElement('li');
            li.className = 'order-item';
            li.innerHTML = `
                <h4>📋 ${order.customerName}</h4>
                <p><strong>Ürün:</strong> ${order.productType} (${order.color})</p>
                <p><strong>Net Ölçü:</strong> ${order.width} x ${order.height} cm | <strong>Alan:</strong> ${order.sqm} m²</p>
                <p style="color: #e65100; font-weight: 600;"><strong>Kesim:</strong> Tül En: ${order.tulEn} cm / Tül Boy: ${order.tulBoy} cm</p>
                <p><strong>Not:</strong> ${order.note}</p>
                <p style="font-size: 11px; color: #90a4ae; margin-top: 5px;">⏱️ Giriş: ${order.tarih}</p>
                <button class="delete-btn" onclick="deleteOrder('${key}')">Tamamlandı / Sil</button>
            `;
            orderPool.appendChild(li);
        });
    });

    // Havuzdan Sipariş Silme (Tamamlanan Siparişi Kapatma)
    function deleteOrder(key) {
        if (confirm("Bu sipariş imalattan çıktı mı? Havuzdan silinecek.")) {
            database.ref('siparisler/' + key).remove();
        }
    }
</script>

</body>
</html>
