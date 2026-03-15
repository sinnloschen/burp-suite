# 🛡️ Burp Suite — Lab Ortamı Olmadan Tanıtım Rehberi

> **Seviye:** Başlangıç  
> **Amaç:** Burp Suite'i kurulum ve lab ortamı gerektirmeden tanımak, temel modülleri anlamak ve PortSwigger Academy üzerinden uygulamalı görmek  
> **Ön koşul:** Burp Suite Community Edition kurulu, internet bağlantısı

---

## 📌 Burp Suite Nedir?

- Web uygulamalarını test etmek için kullanılan en yaygın güvenlik aracı
- Tarayıcı ile sunucu arasına girerek HTTP trafiğini yakalar, değiştirir ve analiz eder
- **MITM (Man-in-the-Middle) proxy** olarak çalışır — tarayıcıdan çıkan her istek önce Burp'ten geçer
- PortSwigger tarafından geliştirilir
- Community (ücretsiz) ve Professional (ücretli) sürümleri var

```
[ Tarayıcı ] → [ Burp Suite ] → [ Sunucu ]
                     ↑
              Tüm trafik burada görünür,
              durdurulabilir, değiştirilebilir
```

---

## 🛠️ Kurulum & İlk Başlatma

```
https://portswigger.net/burp/communitydownload adresinden indir
Java gerektirir — JRE 17+ kurulu olmalı
```

İlk açılışta:
```
"Temporary project" seç → Next
"Use Burp defaults" seç → Start Burp
```

Burp Suite açıldığında 6 ana sekme görünür:
`Dashboard | Target | Proxy | Intruder | Repeater | Decoder`

---

## 🌐 Lab Ortamı Olmadan Nasıl Çalışırız?

### Yöntem 1 — PortSwigger Web Security Academy (Önerilen)

PortSwigger'ın ücretsiz online platformu. Her zafiyet kategorisi için hazır hedef uygulama sunuyor — kurulum gerektirmiyor.

```
1. https://portswigger.net/web-security adresine git
2. Ücretsiz hesap oluştur
3. Herhangi bir konuya gir (örn. SQL Injection)
4. "Access the lab" butonuna tıkla
5. Tarayıcıda canlı bir hedef uygulama açılır
6. Burp Suite ile bu uygulamaya bağlan
```

Her lab için:
- Hazır zafiyetli uygulama
- Ne yapman gerektiğini anlatan görev tanımı
- İpucu sistemi

---

### Yöntem 2 — Ginandjuice.shop (Canlı Demo Hedef)

PortSwigger'ın herkese açık demo uygulaması. Hesap gerektirmez.

```
https://ginandjuice.shop
Burp proxy üzerinden direkt bağlanabilirsin
XSS, SQLi, IDOR gibi zafiyetler içeriyor
```

---

### Yöntem 3 — Burp'ün Dahili Tarayıcısı ile Herhangi Bir Site

Herhangi bir siteye giderek trafiği incelemek için bile lab gerekmez.

```
Proxy → Open Browser
Açılan tarayıcıda herhangi bir siteye git
Proxy → HTTP History sekmesinde tüm istekler görünür
```

> ⚠️ Sadece izin verilen sitelerde test yapılmalı. Traffic incelemek için kendi kullandığın siteler veya PortSwigger labları idealdir.

---

## 🔧 Temel Modüller — Adım Adım

### 1. Proxy — Trafiği Yakala ve İncele

Burp Suite'in kalbi. Tarayıcıdan gelen her HTTP isteğini görür.

**Kurulum:**
```
Proxy → Open Browser ile Burp'ün dahili tarayıcısını aç
Burp otomatik olarak proxy ayarını yapar — ek kurulum gerekmez
```

**HTTP History — Trafik Akışını İzle:**
```
Proxy → HTTP History sekmesi
Tarayıcıda bir siteye git
Her istek ve yanıt burada listelenir

Her satırda şunlar görünür:
# | Method | Host | URL | Status | Length | MIME Type
```

**Bir isteği inceleme:**
```
Herhangi bir satıra tıkla
Alt panelde iki sekme açılır:
  Request  → tarayıcının sunucuya gönderdiği
  Response → sunucunun tarayıcıya döndürdüğü
```

**Örnek Request:**
```http
POST /login HTTP/2
Host: hedef.com
Content-Type: application/x-www-form-urlencoded
Cookie: session=abc123

username=admin&password=test123
```

**Örnek Response:**
```http
HTTP/2 302 Found
Location: /dashboard
Set-Cookie: session=xyz789; HttpOnly; Secure

{"status": "ok", "user": "admin"}
```

**Intercept — İsteği Durdur ve Değiştir:**
```
Proxy → Intercept → "Intercept is on"
Tarayıcıda bir form gönder
İstek Burp'te durur — henüz sunucuya gitmedi
İstediğin parametreyi değiştir
"Forward" → değiştirilmiş istek sunucuya gider
"Drop" → isteği tamamen iptal et
```

---

### 2. Repeater — İsteği Tekrar Gönder

Yakalanan bir isteği defalarca değiştirip göndermeyi sağlar. Manuel test için ideal — her seferinde tarayıcıya gerek yok.

**Kullanım:**
```
HTTP History'de bir isteğe sağ tıkla
"Send to Repeater" seç (veya Ctrl+R)
Repeater sekmesine geç

Sol panel: istek (düzenlenebilir)
Sağ panel: yanıt (otomatik güncellenir)
"Send" butonu → isteği gönder
```

**PortSwigger Academy'de SQL Injection testi:**
```
Lab: "SQL injection vulnerability in WHERE clause"
https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

Bir ürün kategorisine tıkla → isteği HTTP History'de bul
Repeater'a gönder

URL'deki category parametresini değiştir:
  /filter?category=Gifts          → normal
  /filter?category=Gifts'          → hata var mı?
  /filter?category=Gifts'--        → SQL yorumu
  /filter?category='+OR+1=1--     → tüm ürünleri getir

Her değişiklikten sonra "Send" → yanıtı sağ panelde incele
200 OK + farklı içerik = SQL Injection var
```

---

### 3. Intruder — Otomatik Payload Gönderimi

Belirli parametrelere otomatik olarak payload listesi gönderir. Brute force, fuzzing ve enumerate için kullanılır.

**Kullanım:**
```
HTTP History'de bir isteğe sağ tıkla
"Send to Intruder" seç (veya Ctrl+I)
Intruder sekmesine geç
```

**Positions sekmesi — Hedef Parametreyi İşaretle:**
```
İstek içinde test edilecek değerin etrafına § koy
username=§admin§&password=§test§

"Add §" butonu ile otomatik eklenebilir
"Clear §" ile temizlenir
```

**Payloads sekmesi — Payload Listesi Ekle:**
```
Payload Sets → Payload type: Simple list
Payload Options → Add ile tek tek ekle
veya Load ile dosyadan yükle (rockyou.txt vb.)
```

**PortSwigger Academy'de Brute Force testi:**
```
Lab: "Username enumeration via different responses"
https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses

Login isteğini yakala → Intruder'a gönder
username parametresini § ile işaretle
PortSwigger'ın sağladığı username listesini yükle
Attack type: Sniper → Start Attack

Sonuçlar tablosunda:
  Length sütununa göre sırala
  Diğerlerinden farklı uzunluk = geçerli kullanıcı adı bulundu
```

**Attack Türleri:**
| Tür | Ne Zaman Kullanılır |
|---|---|
| Sniper | Tek parametreye tek liste — username enumerate |
| Battering Ram | Tüm parametrelere aynı değer |
| Pitchfork | username listesi + password listesi paralel |
| Cluster Bomb | Tüm kombinasyonlar — yavaş ama kapsamlı |

> ⚠️ Community sürümde Intruder hız sınırlıdır. Pro'da sınır yoktur.

---

### 4. Decoder — Encode / Decode

Veri dönüşümleri için kullanılır. Özellikle token ve cookie analizinde çok işe yarar.

**Kullanım:**
```
Decoder sekmesine geç
Metni yapıştır → format seç

Örnek:
YWRtaW46YWRtaW4= → Decode as Base64 → admin:admin
admin%40mail.com  → Decode as URL    → admin@mail.com
```

**HTTP History'den direkt:**
```
Herhangi bir değere sağ tıkla
"Send to Decoder" seç
Otomatik olarak yapıştırılır
```

**JWT token decode:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.xxx

Noktadan böl → her parça Base64
İlk parça: {"alg":"HS256","typ":"JWT"}   → header
İkinci parça: {"user":"admin"}            → payload
Üçüncü parça: imza
```

---

### 5. Comparer — İki Yanıtı Karşılaştır

İki HTTP yanıtı arasındaki farkları gösterir.

**Kullanım:**
```
İki farklı yanıtı Comparer'a gönder
"Words" veya "Bytes" ile karşılaştır
Farklılıklar renkle işaretlenir
```

**Ne zaman kullanılır:**
- Blind SQL Injection'da true/false yanıtlarını karşılaştırmak
- Geçerli/geçersiz kullanıcı adı yanıtlarının farkını bulmak
- A/B test yanıtlarını analiz etmek

---

## 🔗 Nmap → Burp Suite Geçiş Akışı

```
Nmap ile keşif yap
        │
        ▼
┌───────────────────────────────────────────┐
│ Nmap Bulgusu          │ Burp'te Yapılacak │
├───────────────────────┼───────────────────┤
│ Port 80/443 açık      │ HTTP History ile  │
│                       │ trafik incele     │
├───────────────────────┼───────────────────┤
│ /login endpoint       │ Intercept ile     │
│ bulundu               │ isteği yakala     │
├───────────────────────┼───────────────────┤
│ Eski framework        │ Repeater'da CVE   │
│ versiyonu             │ payload'ı dene    │
├───────────────────────┼───────────────────┤
│ /api/ dizini          │ Intruder ile ID   │
│ keşfedildi            │ enumerate et      │
├───────────────────────┼───────────────────┤
│ Admin panel           │ Intruder ile      │
│ bulundu               │ brute force       │
└───────────────────────┴───────────────────┘
```

---

## 🎯 PortSwigger Academy — Önerilen Başlangıç Labları

Lab ortamı kurmadan, tarayıcıdan direkt denenebilecek lablar:

| Lab | Konu | Seviye | Link |
|---|---|---|---|
| SQL injection WHERE clause | SQLi | Acemi | portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data |
| Username enumeration | Auth | Acemi | portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses |
| Reflected XSS in HTML | XSS | Acemi | portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded |
| IDOR with direct ref | IDOR | Acemi | portswigger.net/web-security/access-control/lab-insecure-direct-object-references |
| Password reset broken | Auth | Acemi | portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic |

---

## 📋 Hızlı Referans — Cheat Sheet

```
Proxy → HTTP History    : Geçmiş tüm istekleri gör
Proxy → Intercept       : İsteği durdur ve değiştir
Repeater (Ctrl+R)       : İsteği manuel değiştirip tekrar gönder
Intruder (Ctrl+I)       : Otomatik payload gönder
Decoder                 : Base64, URL, HTML encode/decode
Comparer                : İki yanıtı karşılaştır

Sık kullanılan kısayollar:
Ctrl+R → Repeater'a gönder
Ctrl+I → Intruder'a gönder
Ctrl+U → URL encode
Ctrl+Z → Değişikliği geri al
```

---

## ⚠️ Yasal Uyarı

> Burp Suite yalnızca **izin verilen** sistemlerde kullanılmalıdır. Bu eğitimde PortSwigger Academy labları veya kendi sistemlerin dışında test yapılmamalıdır.

---

## 📚 Ek Kaynaklar

- [Burp Suite Resmi Dokümantasyon](https://portswigger.net/burp/documentation)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [Ginandjuice Demo Uygulaması](https://ginandjuice.shop)
