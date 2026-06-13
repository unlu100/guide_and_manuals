# Repo Kopyalama & Index.html / Supabase Sorun Giderme Kılavuzu (finalnot_mat2_26)

Bu döküman, unlu100/finalnot_mat2_26 reposu için yaptığımız hata tespitleri, düzeltmeler ve repoyu başka bir GitHub reposuna kopyalama adımlarının özetini içerir. Ayrıca Windows/PowerShell ile ilgili notlar ve sık görülen hatalar için çözümler yer almaktadır.

---

## 1) Repoyu başka bir GitHub reposuna kopyalama (2 yöntem)

A) Mirror (tüm ref'leri, geçmişi, tag'leri korur — tam kopya)
- Dezavantaj: geçici "bare" klasör oluşturur.

Komutlar (Unix / Git Bash önerilir):
```
# 1) Mirror olarak klonla (bare repo oluşturur)
git clone --mirror https://github.com/unlu100/finalnot_mat2_26.git

# 2) Yeni GitHub'da boş bir repo oluştur: USERNAME/new-repo (boş, README olmasın)
# 3) Push et (tüm branch'ler, tag'ler vs)
cd finalnot_mat2_26.git
git push --mirror https://github.com/USERNAME/new-repo.git

# 4) Geçici bare klasörü sil (Unix/Git Bash)
cd ..
rm -rf finalnot_mat2_26.git
```

Windows PowerShell için silme:
```
# PowerShell'de
Remove-Item -Recurse -Force .\finalnot_mat2_26.git
```
Not: PowerShell'de `rm -rf` çalışmaz — `Remove-Item -Recurse -Force` veya Git Bash kullanın.

B) Normal klon + push (sade, çoğu zaman yeterli)
```
git clone https://github.com/unlu100/finalnot_mat2_26.git
cd finalnot_mat2_26

# Yeni repo'yu origin olarak ayarla (veya mevcut origin URL'ini değiştir)
git remote remove origin
git remote add origin https://github.com/USERNAME/new-repo.git

# Tüm branch'leri ve tag'leri gönder
git push -u origin --all
git push -u origin --tags
```

---

## 2) Karşılaşılan hata mesajları ve çözümleri (özeti)

- fatal: not a git repository (or any of the parent directories): .git  
  -> Sebep: `git push` çalıştırdığınız dizinde `.git` yok. Mirror klon yapılmamış/yanlış dizindesiniz. Çözüm: Doğru dizine geçin (`cd finalnot_mat2_26.git`) veya önce `git clone --mirror` çalıştırın.

- remote: Repository not found. fatal: repository 'https://...git/' not found  
  -> Sebep: Hedef repo GitHub'da yok veya URL/user yanlış; repo özelse auth yok.  
  Çözüm: Yeni repo'yu GitHub'da oluşturun (boş repo), URL doğru mu kontrol edin, özel repo ise kimlik doğrulaması (PAT) kullanın.

- Remove-Item : A parameter cannot be found that matches parameter name 'rf'  
  -> Sebep: PowerShell `rm -rf` Unix tarzı.  
  Çözüm: PowerShell için `Remove-Item -Recurse -Force` veya Git Bash'de `rm -rf`.

---

## 3) index.html içinde sorgulama çalışmama (bulunan sorunlar ve düzeltmeler)

Sorunlar:
- Yanlış/çift `<script>` etiketleri nedeniyle HTML parse hatası.
- `config.js` (window.SUPABASE_URL / window.SUPABASE_KEY) yüklenmeden Supabase client oluşturulması => `SUPABASE_KEY` undefined veya placeholder kalması.
- config.js içindeki anahtarın kesik/paylaşılan olmaması veya yanlış türde (service_role yerine anon/public olmalı).

Örnek hatalı bölge:
```html
<script> 
//-- config.js'i buraya ekleyin -->
<script src="./config.js"></script>
  
<script>
...
```
Bu, açılan `<script>` etiketinin kapanmaması / iç içe etiket hatası oluşturur.

Düzeltme (özet):
- config.js'i önce yükle:
```html
<script src="./config.js"></script>
<script>
  // burada window.SUPABASE_URL ve window.SUPABASE_KEY güvenle kullanılabilir
</script>
```
- Supabase client'ın config yüklendikten sonra oluşturulmasını sağla. Güvenli yol: client'ı `sorgula()` fonksiyonu içinde veya `DOMContentLoaded` sonrası oluştur.

Güvenli kontrol örneği (JS):
```javascript
if(!window.SUPABASE_URL || !window.SUPABASE_KEY){
  // kullanıcıya bilgi ver, konsola yaz
  sonucDiv.innerHTML = '<div class="error">Supabase yapılandırması yüklenmedi. Lütfen config.js dosyasını kontrol edin.</div>';
  return;
}
const supabase = supabase.createClient(window.SUPABASE_URL, window.SUPABASE_KEY);
```

Ayrıca, index.html içinde kullanılan placeholder anahtar:
- config.js içindeki anahtarın '<ANON_PUBLIC_KEY>' gibi placeholder olmaması gerekir.
- Kullanıcıya gösterilen hata mesajlarını açıklayıcı yap.

---

## 4) config.js örneği ve güvenlik notu

config.js basitçe şöyle olmalı:
```javascript
// config.js — bu dosya .gitignore'a eklenmeli (commit etmeyin)
window.SUPABASE_URL = 'https://<PROJECT>.supabase.co';
window.SUPABASE_KEY = 'eyJhbGci...ANON_PUBLIC_KEY...';
```
Güvenlik:
- Asla service_role (admin) anahtarını istemci tarafında (tarayıcıda) kullanmayın.
- config.js'i versiyon kontrolüne commit etmeyin; `.gitignore`'a ekleyin.
- Public/anonsuz erişim yetkileri ve tablolar için Row-Level Security (RLS) kurallarını yapılandırın.

---

## 5) Test & Debug adımları (tarayıcıda)

1. Sayfayı aç, F12 (DevTools) -> Console sekmesi:
   - Script hataları var mı? (SyntaxError, ReferenceError)
2. Network sekmesi:
   - config.js yükleniyor mu?
   - Supabase istekleri gidiyor mu? (CORS veya 401/403 hataları olabilir)
3. Sunucu tarafı (Supabase) hataları:
   - Hata mesajı `error` objesinde dönüyor mu? Console.log() ile kontrol et.
4. SQL tablo/kolon isimleri index.html ile birebir uyumlu mu? (ör. tablonun adı `final_haziran26_mat2`, sütun `no_duzgun` vs.)

---

## 6) Kısa özet: Hızlı kontrol listesi

- [ ] Yeni repo GitHub'da boş şekilde oluşturuldu mu? (mirror için)
- [ ] `git clone --mirror ...` yaptın mı? (mirror yöntemi için)
- [ ] Doğru dizindesin? (`cd finalnot_mat2_26.git`)
- [ ] `git push --mirror ...` çalıştırdın mı?
- [ ] PowerShell kullanıyorsan silme için `Remove-Item -Recurse -Force` kullan.
- [ ] index.html içinde `config.js` <script> çağrısı, sonra uygulama scripti var mı?
- [ ] Supabase anahtarı config.js içinde eksiksiz ve anon/public mı?
- [ ] Tarayıcı konsolunda hata var mı? (F12 → Console)

---

Bu rehberi `unlu100/guide_and_manuals` reposu altına `REPO_COPY_AND_FIX_GUIDE_finalnot_mat2_26.md` adıyla ekledim.
