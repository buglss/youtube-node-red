# Node-RED: Giriş ve Kategori Uygulaması

Bu proje, Node-RED üzerinde çalışan bir web uygulamasıdır. MongoDB'deki kullanıcılarla giriş yapmayı, çıkış yapmayı ve giriş yaptıktan sonra bir destek personeli seçerek kategori oluşturmayı sağlar. Kullanıcı parolaları bcrypt ile özetlenir; oturum için tarayıcıda `token` adlı çerezde saklanan JWT kullanılır.

Uygulamanın kodu `flows.json` dosyasındaki akışlardır. Node-RED, bu akışları tarayıcıdan görüntüleyip düzenleyebileceğiniz bir geliştirme ortamıdır. MongoDB ise kullanıcı ve kategori kayıtlarını bilgisayarınızda saklayan ayrı bir veritabanı servisidir. Uygulamanın çalışması için ikisinin de açık olması gerekir.

Bu rehber MongoDB'nin bilgisayara doğrudan kurulduğu yerel kullanım içindir. Windows, macOS ve Linux için Ubuntu 24.04 LTS adımları verilmiştir.

## 1. Başlamadan önce

Gereken araçlar:

| Araç | Ne için gerekli? | Sürüm bilgisi |
| --- | --- | --- |
| Node.js | Node-RED'i çalıştırır. | Kilit dosyasındaki Node-RED 5.0.9 en az **Node.js 22.9** ister. Aşağıdaki kurulum Node.js 26 LTS kullanır. |
| npm | Projenin yazılım bağımlılıklarını yükler. | Node.js ile birlikte kurulur. Projede ayrıca bir npm sürümü sabitlenmemiştir. |
| Git | Projeyi GitHub'dan bilgisayarınıza indirir. | Projede bir alt sürüm sınırı belirtilmemiştir. |
| MongoDB Community Server | Verileri saklar. | Bu rehber **9.0** serisini temel alır. |
| MongoDB Shell (`mongosh`) | Veritabanı bağlantısını ve kayıtları kontrol etmenizi sağlar. | Windows'ta ayrıca kurulmalıdır. |
| Web tarayıcısı | Uygulamayı ve Node-RED editörünü açar. | JavaScript ve çerezler açık olmalıdır. |

MongoDB 9.0 için bu rehberin kapsadığı platformlar Windows 11 x64, macOS 15 ve üzeri (Intel veya Apple Silicon) ve Ubuntu 24.04 LTS'dir (x86_64 veya arm64). Diğer sistemler için [MongoDB'nin desteklenen platformlarını](https://www.mongodb.com/docs/community-platform-support/) kontrol edin.

İlk indirme ve paket kurulumu için internet bağlantısı gerekir. Sayfaların Bootstrap 5.0.2 ve Bootstrap Icons 1.4.1 dosyaları da `cdn.jsdelivr.net` üzerinden yüklenir; internet olmadan sayfaların görünümü eksik kalabilir.

### Terminal nasıl açılır ve komutlar nasıl kullanılır?

- **Windows:** Başlat menüsüne `cmd` yazıp **Komut İstemi**'ni açın. Aşağıdaki Windows komutlarını burada çalıştırın. Yalnızca servis yönetiminde ayrıca belirtilen komutlar için **Yönetici olarak çalıştır** gerekir.
- **macOS:** Spotlight aramasında `Terminal` yazıp açın.
- **Ubuntu:** `Ctrl + Alt + T` tuşlarına basın. Bu rehberde Ubuntu'nun Bash kabuğu kullanılır.

Kod bloklarının içindeki komutları sırayla çalıştırın. Bir komut bitmeden sonrakine geçmeyin; hata alırsanız önce son bölümdeki sorun giderme adımlarına bakın. `sudo` komutları bilgisayarınızın parolasını isteyebilir; parola yazarken ekranda karakter görünmemesi normaldir.

## 2. Node.js ve Git kurulumu

### Windows

1. [Node.js indirme sayfasını](https://nodejs.org/en/download) açın. **26 LTS**, **Windows** ve **x64** seçerek `.msi` yükleyicisini indirin.
2. Yükleyiciyi açıp kurulumu tamamlayın. npm ve PATH seçeneklerini etkin bırakın. PATH, terminalde `node` ve `npm` komutlarının bulunmasını sağlar.
3. [Git for Windows](https://git-scm.com/install/windows) yükleyicisini indirip kurun. Git'in komut satırından kullanılmasını sağlayan PATH seçeneğini etkin bırakın.
4. Açık Komut İstemi pencerelerini kapatıp yeni bir pencere açın.

### macOS

1. [Node.js indirme sayfasından](https://nodejs.org/en/download) **26 LTS** için macOS `.pkg` yükleyicisini indirip kurun. Apple menüsü → **Bu Mac Hakkında** bölümünden işlemcinizi görebilirsiniz; mimari seçimi istenirse Apple Silicon için `arm64`, Intel için `x64` kullanın.
2. Git ve MongoDB kurulumunda kullanılacak Xcode komut satırı araçlarını yükleyin:

```sh
xcode-select --install
```

Açılan pencerede kurulumu tamamlayın. Araçların zaten yüklü olduğu belirtilirse devam edin. Terminali kapatıp yeniden açın.

### Ubuntu 24.04 LTS

Önce Git ve indirme araçlarını kurun:

```sh
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
```

Node.js sürümünü kullanıcı hesabınız altında yönetmek için [nvm'nin resmî kurulum yöntemini](https://github.com/nvm-sh/nvm#installing-and-updating) kullanın:

```sh
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.41.0/install.sh -o /tmp/node-red-nvm-install.sh
bash /tmp/node-red-nvm-install.sh
```

Terminali kapatıp yeniden açın, ardından:

```sh
nvm install 26
nvm alias default 26
nvm use 26
```

`nvm` bulunamazsa `source ~/.bashrc` komutunu çalıştırıp yeniden deneyin. Bu adımlar sisteminizdeki Node.js sürümüne güvenmeden, belirtilen sürümü kurar.

### Kurulumu kontrol edin — tüm sistemler

Yeni terminal penceresinde:

```sh
node --version
npm --version
git --version
```

Üç komut da sürüm numarası göstermelidir. Yukarıdaki kurulumla Node.js çıktısı `v26.` ile başlar. Kontroller başarılı olmadan proje kurulumuna geçmeyin.

## 3. MongoDB 9.0 kurulumu ve başlatılması

MongoDB Server veritabanını çalıştırır; `mongosh` ona bağlanan komut satırı aracıdır. MongoDB Compass ise isteğe bağlı bir görsel arayüzdür; Server'ın yerini tutmaz.

### Windows 11 x64

1. [MongoDB Community Server indirme sayfasında](https://www.mongodb.com/try/download/community) **Version: 9.0.x**, **Platform: Windows**, **Package: msi** seçin ve yükleyiciyi indirin.
2. Yükleyiciyi açıp **Complete** kurulumunu seçin.
3. **Install MongoD as a Service** seçeneğini etkin bırakın. **Run service as Network Service user** ve servis adı **MongoDB** ile ilerleyin. Böylece veritabanı arka planda çalışır.
4. Compass kurulumunu isteğinize göre seçip kurulumu tamamlayın.
5. [MongoDB Shell indirme sayfasından](https://www.mongodb.com/try/download/shell) Windows için `mongosh` yükleyicisini indirip kurun. Komut satırından erişim için PATH'e ekleme seçeneğini etkin bırakın. Server'ın MSI yükleyicisi `mongosh` içermez.
6. Yeni bir Komut İstemi açın. Servis durumunu kontrol edin:

```bat
sc query MongoDB
```

`STATE` satırında `RUNNING` görmelisiniz. Servis durmuşsa **yönetici olarak açtığınız** Komut İstemi'nde başlatın:

```bat
net start MongoDB
```

Kaynak: [MongoDB'nin Windows kurulum rehberi](https://www.mongodb.com/docs/manual/administration/install-community-windows/?win-method=wizard) ve [Windows servis yönetimi](https://www.mongodb.com/docs/manual/administration/run-mongodb-windows/).

### macOS 15 ve üzeri

Önce `brew --version` komutuyla Homebrew'ün kurulu olup olmadığını kontrol edin. Kurulu değilse [Homebrew'ün resmî kurulum komutunu](https://brew.sh/) Terminal'de çalıştırın:

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Kurulumun gösterdiği adımları izleyin. Sonunda gösterilen **Next steps** komutlarını da çalıştırın; bunlar `brew` komutunu PATH'e ekler. Ardından yeni terminalde `brew --version` komutunun çalıştığını doğrulayın. Homebrew zaten kuruluysa bu kurulum adımını atlayın.

MongoDB'yi kurup servisi başlatın:

```sh
brew update
brew tap mongodb/brew
brew trust mongodb/brew
brew install mongodb-community@9.0
brew services start mongodb-community@9.0
brew services list
```

Listede `mongodb-community@9.0` için `started` durumunu görmelisiniz. Bu kurulum `mongosh` aracını da içerir. `brew trust` tanınmıyorsa Homebrew'ü güncelleyin; komut güncel MongoDB kurulum rehberinde yer alır.

Kaynak: [MongoDB'nin macOS kurulum rehberi](https://www.mongodb.com/docs/manual/administration/install-community-macos/?macos-method=brew).

### Ubuntu 24.04 LTS

Bu komutlar Ubuntu **24.04 / noble** içindir. Sürümünüzü `cat /etc/os-release` ile kontrol edin. Başka bir dağıtım veya sürümde bu depoyu kullanmayın; ilgili [MongoDB Linux kurulum rehberini](https://www.mongodb.com/docs/manual/administration/install-community-linux/?linux-distro=ubuntu&linux-method=pkg) izleyin.

MongoDB'nin resmî paket deposunu ekleyip Server ve Shell'i kurun:

```sh
sudo apt-get update
sudo apt-get install -y gnupg curl
curl -fsSL https://pgp.mongodb.com/server-9.asc -o /tmp/node-red-mongodb-server-9.asc
sudo gpg --dearmor --output /usr/share/keyrings/mongodb-server-9.gpg /tmp/node-red-mongodb-server-9.asc
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-9.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/9.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-9.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod --no-pager
```

Durum çıktısında `active (running)` görmelisiniz. `enable`, veritabanının bilgisayar açıldığında da başlamasını sağlar. `Unit mongod.service not found` hatası alırsanız kurulumun tamamlandığını kontrol edin, `sudo systemctl daemon-reload` çalıştırıp başlatmayı yeniden deneyin.

### Bağlantıyı kontrol edin — tüm sistemler

Terminalde:

```sh
mongosh "mongodb://127.0.0.1:27017/login" --quiet --eval "db.runCommand({ ping: 1 })"
```

Çıktıda `ok: 1` olmalıdır. Bu komut yalnızca bağlantıyı kontrol eder; örnek kullanıcı oluşturmaz. `mongosh --version` Shell'in sürümünü gösterir; Server'ın sürümünü görmek için:

```sh
mongosh "mongodb://127.0.0.1:27017/login" --quiet --eval "db.version()"
```

Sonuç `9.0.x` olmalıdır. Bu rehberdeki bağlantı, aynı bilgisayarda `27017` portunda çalışan ve kullanıcı adı/parola gerektirmeyen yerel MongoDB içindir. Mevcut MongoDB'nizde kimlik doğrulama açıksa aşağıdaki Node-RED bağlantı ayarlarına kendi veritabanı hesabınızı girmeniz gerekir.

## 4. Projeyi indirin ve bağımlılıkları yükleyin

Terminalde projeyi saklamak istediğiniz klasöre geçin. `cd`, terminalin çalıştığı klasörü değiştirir. Örneğin `cd "KLASOR_YOLU"` yazarken `KLASOR_YOLU` yerine bilgisayarınızdaki gerçek klasörün yolunu girin; boşluk içeren yolların tırnaklarını koruyun.

```sh
git clone https://github.com/buglss/youtube-node-red.git
cd youtube-node-red
npm ci
```

Proje zaten bilgisayarınızdaysa tekrar indirmek yerine `package.json`, `settings.js` ve `flows.json` bulunan klasörü terminalde açıp `npm ci` çalıştırın. Örneğin mevcut klasörünüzün adı `node-red` olabilir; komutları çalıştıracağınız yer, bu dosyaların bulunduğu klasördür.

`npm ci`, `package-lock.json` dosyasındaki sürümleri yükler. Mevcut `node_modules` klasörünü yeniden oluşturur. Node-RED ve ek düğümler bu işlemle projenin içine kurulur; ayrıca global Node-RED kurulumu gerekmez. Komut hatasız tamamlanınca devam edin. Depoya erişim hatası alırsanız GitHub adresini ve hesabınızın depoya erişimini kontrol edin.

Kilit dosyasındaki doğrudan bağımlılıklar:

| Paket | Sürüm | Görevi |
| --- | --- | --- |
| `node-red` | 5.0.9 | Akışları ve HTTP sayfalarını çalıştırır. |
| `node-red-contrib-bcrypt` | 0.1.6 | Parola özetleme ve doğrulama düğümlerini sağlar. |
| `node-red-contrib-jwt` | 0.1.0 | JWT oluşturma ve doğrulama düğümlerini sağlar. |
| `node-red-contrib-mongodb4` | 3.4.0 | MongoDB bağlantı ve sorgu düğümlerini sağlar. |

## 5. Yeni kurulumda şifreli ayar dosyasını hazırlayın

**Bu adım yalnızca depodan yeni indirdiğiniz kopya içindir.** Mevcut, çalışan kurulumunuzda kayıtlı bağlantı bilgileri varsa bu adımı uygulamayın.

Depodaki `flows_cred.json`, Node-RED tarafından şifrelenmiştir. Şifreyi açan, bilgisayara özgü `.config.runtime.json` dosyası Git'e dahil değildir. Bu nedenle dosya yeni bir bilgisayarda `Failed to decrypt credentials` uyarısı üretebilir. Depodaki mevcut dosyada MongoDB düğümüne ait boş bir kayıt bulunur; kayıtlı kullanıcı adı veya parola yoktur.

Node-RED henüz çalışmıyorken, proje klasöründe aşağıdaki komutla dosyanın adını değiştirip yedeğini koruyun:

```sh
node -e "require('node:fs').renameSync('flows_cred.json', 'flows_cred.json.kurulum-yedegi')"
```

Bu işlemi **bir kez** yapın. Node-RED, gerektiğinde bu bilgisayara ait yeni kimlik bilgisi dosyasını oluşturur. Yedek dosyayı saklayın. Bu dosyanın şifreleme anahtarı ile JWT imzalama anahtarı farklı şeylerdir.

## 6. Node-RED'i başlatın

MongoDB'nin çalıştığından emin olun. Proje klasöründe:

```sh
node node_modules/node-red/red.js --userDir . -D uiHost=127.0.0.1
```

- `--userDir .`, bu klasördeki `settings.js`, `flows.json` ve kurulan düğümlerin kullanılmasını sağlar.
- `-D uiHost=127.0.0.1`, bu çalıştırmada Node-RED erişimini kendi bilgisayarınızla sınırlar. Depodaki `settings.js` dosyasında `uiHost` etkin değildir.
- Varsayılan port `1880`'dir. `settings.js`, `PORT` ortam değişkeni tanımlıysa onun değerini kullanır.

Terminalde `Started flows` mesajını bekleyin. `Settings file`, `User directory` ve `Flows file` satırlarının bu projeyi gösterdiğini kontrol edin. Terminali açık bırakın; kapatırsanız uygulama durur.

Tarayıcıda [http://localhost:1880/](http://localhost:1880/) adresini açın. Bu adres **Node-RED editörüdür**. İlk açılışta karşınıza çıkan tanıtım veya telemetri tercihi ekranını tamamlayıp editöre geçin. Üstte **Login** ve **Kategori** sekmelerini görmelisiniz; akışları ayrıca içe aktarmanız gerekmez.

Projede `npm start` komutu tanımlı değildir. Çalıştırmak için yukarıdaki `node ...` komutunu kullanın. `npm test` ise henüz bir test takımı çalıştırmaz; bilerek hata koduyla çıkan varsayılan komuttur.

## 7. Node-RED'de MongoDB bağlantısını kontrol edin

Akışların tümü aynı MongoDB bağlantı yapılandırmasını kullanır. Depodaki ayarlar `mongodb`, `localhost` ve veritabanı adı `login` şeklindedir; port alanı boştur ve sürücünün varsayılanı `27017` kullanılır.

1. Node-RED editöründe **Login** sekmesine geçin.
2. Bir `mongodb4` düğümüne çift tıklayın; örneğin `user` koleksiyonundaki `updateOne` işlemini yapan düğüm.
3. Bağlantı alanının yanındaki kalem simgesine tıklayarak `mongodb4-client` ayarlarını açın.
4. Basit bağlantı alanlarında şu değerleri kullanın:

| Alan | Yerel kurulum için değer |
| --- | --- |
| Protocol | `mongodb` |
| Hostname | `127.0.0.1` |
| Port | `27017` |
| Database / DB Name | `login` |
| Username / Password | Kimlik doğrulamasız yeni yerel kurulumda boş |
| Auth Source | Kimlik doğrulamasız yeni yerel kurulumda boş |
| Auth Mechanism | `DEFAULT` |
| TLS | Kapalı |

Mevcut MongoDB'niz farklı port veya kimlik doğrulama kullanıyorsa ilgili alanlara kendi bağlantı bilgilerinizi yazın. MongoDB hesabı, ileride uygulamaya giriş için oluşturacağınız `demo` hesabından ayrıdır.

5. Bağlantı penceresinde **Update**, düğüm penceresinde **Done** düğmesine basın; ardından editörün sağ üstündeki **Deploy** düğmesine basın. Deploy, editörde yaptığınız değişiklikleri kaydedip çalıştırır.

`127.0.0.1` kullanımı, `localhost` bazı bilgisayarlarda IPv6 adresi `::1` olarak çözüldüğünde oluşabilen bağlantı sorununu önler. Veritabanını ve koleksiyonları önceden elle oluşturmanız gerekmez; ilk veri yazıldığında oluşurlar.

## 8. İlk kullanıcıları oluşturun

Yeni veritabanında kullanıcı yoktur. Bir kayıt olma sayfası da bulunmaz. Akıştaki örnek kullanıcı oluşturma adımını elle başlatmanız gerekir:

1. Editörde **Login** sekmesini açın.
2. `1234` değerini gönderen **Inject** düğümünü bulun. Bu düğüm bcrypt düğümüne, ardından `userdata`, Split, `mql` ve MongoDB `user / updateOne` düğümlerine bağlıdır. `flows.json` içindeki düğüm ID'si `88fe7594ee13f276`'dır.
3. Henüz kaydedilmemiş bir değişiklik varsa önce **Deploy** yapın.
4. Inject düğümünün **solundaki küçük kare düğmeye bir kez** tıklayın. Bu işlem beş kullanıcı oluşturur.
5. Sağdaki böcek simgeli **Debug** panelinde hataları kontrol edin. `updateOne` çıkışına bağlı `debug 5` mesajlarında işlem sonuçlarını görebilirsiniz.

| Kullanıcı adı | Görünen ad | Başlangıç parolası |
| --- | --- | --- |
| `demo` | Demo | `1234` |
| `memo` | Memo | `1234` |
| `lolo` | Lolo | `1234` |
| `bibi` | Bibi | `1234` |
| `selo` | Selo | `1234` |

Bu adım otomatik çalışmaz. Aynı kullanıcı adına göre `upsert` yapar: kullanıcı yoksa oluşturur; varsa parolasını, görünen adını ve `createdAt` tarihini yeniden yazar. Bu nedenle sonraki açılışlarda tekrar tıklamayın. Parola veritabanına düz metin olarak değil, bcrypt özeti olarak kaydedilir.

Kontrol için **ayrı bir terminal** açıp şu komutu çalıştırabilirsiniz; Node-RED'in çalıştığı terminali kapatmayın:

```sh
mongosh "mongodb://127.0.0.1:27017/login" --quiet --eval "db.user.find({}, { username: 1, displayName: 1, _id: 0 }).toArray()"
```

Listede yukarıdaki beş kullanıcı görünmelidir.

## 9. Uygulamayı kullanın

### Giriş ve çıkış

1. Tarayıcıda [http://localhost:1880/login](http://localhost:1880/login) adresini açın.
2. **Username** alanına `demo`, **Password** alanına `1234` yazıp **login** düğmesine basın.
3. Başarılı giriş mesajını ve `Hoşgeldin demo` yazısını görmelisiniz. Giriş sayfasında kalmanız normaldir; otomatik kategori sayfasına yönlendirme yoktur.
4. Oturum JWT'sinin süresi akışta **3600 saniye (1 saat)** olarak ayarlanmıştır. Süre dolarsa yeniden giriş yapın.
5. Çıkış yapmak için `/login` sayfasındaki **logout** düğmesini kullanın. Bu işlem tarayıcıdaki `token` çerezini siler.

Sayfalar arasında aynı adresi kullanın: girişte `localhost` kullandıysanız kategori sayfasında da `localhost` kullanın. `127.0.0.1` ve `localhost` tarayıcı çerezleri açısından farklı adreslerdir.

### Kategori oluşturma

1. Giriş yaptığınız **aynı tarayıcıda** [http://localhost:1880/ticket-kategori](http://localhost:1880/ticket-kategori) adresini açın.
2. **Kategori Adı** alanına örneğin `Teknik Destek` yazın.
3. **Destek Personeli** listesinden bir kullanıcı seçin. Liste `user` koleksiyonundaki kullanıcılardan oluşur.
4. **Kaydet** düğmesine basın. Başarı mesajını görmelisiniz.
5. Aynı kategori adıyla tekrar kayıt gönderdiğinizde `Kategori zaten oluşturulmuş.` mesajı gösterilir.

Giriş yapmadan bu sayfayı açarsanız `Yetkisiz erişim.` mesajıyla giriş formu gösterilir; adres çubuğunun otomatik olarak `/login` adresine değişmesi beklenmez.

Kayıtları terminalden kontrol etmek için:

```sh
mongosh "mongodb://127.0.0.1:27017/login" --quiet --eval "db.kategori.find().toArray()"
```

Kaydın `kategoriAdi`, `destekPersoneli`, `stamp.user` ve `stamp.createdAt` alanlarını görebilirsiniz. `stamp.user`, kategoriyi oluşturan giriş yapmış kullanıcının adıdır.

### Kullanılabilir adresler

| Yöntem | Yol | İşlev |
| --- | --- | --- |
| GET | `/` | Node-RED editörü |
| GET | `/login` | Giriş formu |
| POST | `/login` | `username` ve `password` form alanlarıyla giriş |
| POST | `/logout` | Çerez silerek çıkış |
| GET | `/ticket-kategori` | Giriş gerektiren kategori formu |
| POST | `/kategori-olustur` | `kategoriAdi` ve `destekPersoneli` form alanlarıyla kategori kaydı |

Form işlemlerinin yanıtları HTML sayfalarıdır. POST adreslerini adres çubuğuna yazarak çağırmak yerine ilgili form düğmelerini kullanın.

## 10. Durdurma ve sonraki çalıştırma

Node-RED'i durdurmak için çalıştığı terminalde **Ctrl + C** tuşlarına basın. Bu işlem MongoDB verilerini silmez; MongoDB servisi ayrıca çalışmaya devam eder.

Sonraki kullanımda:

1. MongoDB servisinin çalıştığını kontrol edin.
2. Terminalde proje klasörüne geçin.
3. Aşağıdaki komutu çalıştırıp `Started flows` mesajını bekleyin:

```sh
node node_modules/node-red/red.js --userDir . -D uiHost=127.0.0.1
```

4. `/login` adresini açıp giriş yapın. Bağımlılıklar değişmediyse tekrar `npm ci` çalıştırmanız, şifreli dosyanın adını değiştirmeniz veya örnek kullanıcıları yeniden oluşturmanız gerekmez.

MongoDB servisini de durdurmak isterseniz işletim sisteminize uygun **tek** komutu kullanın:

| Sistem | Durdurma | Yeniden başlatma |
| --- | --- | --- |
| Windows, yönetici Komut İstemi | `net stop MongoDB` | `net start MongoDB` |
| macOS | `brew services stop mongodb-community@9.0` | `brew services start mongodb-community@9.0` |
| Ubuntu | `sudo systemctl stop mongod` | `sudo systemctl start mongod` |

## 11. Proje dosyaları

| Dosya | İçeriği |
| --- | --- |
| `flows.json` | Login, Kategori ve ortak Check Token alt akışı; HTML şablonları ve uygulama mantığı |
| `flows_cred.json` | Node-RED'in şifreli bağlantı kimlik bilgileri |
| `settings.js` | Akış dosyası, port ve Node-RED çalışma ayarları |
| `package.json` | Bağımlılık aralıkları ve npm komutları |
| `package-lock.json` | Tekrarlanabilir kurulum için kilitlenmiş paket sürümleri |
| `.vscode/launch.json` | VS Code'da `Node-Red Start` adlı hata ayıklama yapılandırması |
| `.gitignore` | `node_modules`, `.config*` ve `.flows*` dosyalarını Git dışında tutar |

Node-RED yerel `.config*` ayarlarını ve akış yedeklerini çalışırken oluşturabilir. MongoDB kayıtları bu dosyalarda veya Git deposunda saklanmaz; MongoDB'nin veri klasöründe tutulur.

## 12. Sorun giderme

| Belirti | Kontrol / çözüm |
| --- | --- |
| `node`, `npm`, `git` veya `mongosh` bulunamıyor | İlgili aracın kurulu olduğunu kontrol edin. Kurulumdan sonra terminali yeniden açın. Windows'ta PATH seçeneğini kontrol edin; Ubuntu'da nvm için `source ~/.bashrc` çalıştırın. |
| PowerShell'de `npm.ps1` çalıştırılamıyor | Bu rehberdeki Windows adımlarını Komut İstemi'nde çalıştırın veya PowerShell'de `npm.cmd ci` kullanın. |
| `EBADENGINE` / desteklenmeyen Node.js | `node --version` çıktısını kontrol edin. Kilitli Node-RED sürümü en az 22.9 ister; bu rehberdeki 26 LTS kurulumunu kullanın. |
| `package.json` veya `node_modules/node-red/red.js` bulunamıyor | Doğru proje klasöründe olduğunuzu kontrol edin ve `npm ci` işlemini tamamlayın. |
| `npm ci` kilit dosyası uyumsuzluğu bildiriyor | `package.json` ile `package-lock.json` dosyalarını projenin aynı sürümünden aldığınızı kontrol edin. |
| Editör boş; Login/Kategori sekmeleri yok | Başlatma komutunda `--userDir .` kullandığınızı ve logdaki `Flows file` yolunun bu projenin `flows.json` dosyası olduğunu kontrol edin. |
| `unknown` düğümler veya eksik düğüm uyarısı | Node-RED'i durdurun, proje klasöründe `npm ci` çalıştırıp yeniden başlatın. |
| `Failed to decrypt credentials` | Yeni indirdiğiniz kopyada 5. bölümü uygulayın. Mevcut bağlantı bilgileriniz varsa dosyayı yeniden adlandırmayın; o kuruluma ait anahtarı içeren ayarlar ve kimlik bilgisi dosyasının birlikte korunması gerekir. |
| `ECONNREFUSED`, `MongoServerSelectionError` veya MongoDB işlemi zaman aşımı | MongoDB servisini ve `mongosh` bağlantı kontrolünü deneyin. Node-RED bağlantısında Hostname `127.0.0.1`, Port `27017`, DB Name `login` olmalı. Farklı kurulumda kendi değerlerinizi kullanın. |
| `Authentication failed` | MongoDB hesabını ve Auth Source değerini kontrol edin. Uygulamanın `demo` kullanıcısı MongoDB sunucu hesabı değildir. |
| `Kullanıcı adı veya parola hatalı.` | Önce örnek kullanıcı oluşturma adımını tamamlayın; verilerin bağlandığınız `login.user` koleksiyonunda bulunduğunu kontrol edin. |
| `Yetkisiz erişim.`, `jwt expired` veya `invalid signature` | Yeniden giriş yapın. Sayfalarda aynı sunucu adını kullanın. JWT Sign ve Check Token içindeki JWT verify düğümlerinin aynı anahtarı kullandığını kontrol edin. |
| Destek personeli listesi boş | Örnek kullanıcıların oluşturulduğunu ve bağlantının doğru `login` veritabanına gittiğini kontrol edin. |
| Tasarım veya ikonlar eksik | `cdn.jsdelivr.net` erişimini ve internet bağlantısını kontrol edin. |

### 1880 portu kullanımdaysa

Node-RED'i durdurup farklı portla başlatın:

```sh
node node_modules/node-red/red.js --userDir . -D uiHost=127.0.0.1 --port 1881
```

Bu durumda bütün adreslerde `1880` yerine `1881` kullanın; örneğin `http://localhost:1881/login`. Node-RED logunda bildirilen adresi kontrol edin.

### Akış çalışırken hata veriyorsa

Node-RED editöründeki **Debug** paneline ve uygulamayı başlattığınız terminale bakın. Akışlarda `ERROR` adlı Debug düğümleri bulunur. Düğümleri güvenle incelemek için akışları başlatmadan editörü açabilirsiniz:

```sh
node node_modules/node-red/red.js --userDir . -D uiHost=127.0.0.1 --safe
```

Safe modda uygulama akışları çalışmaz; **Deploy** yaptığınızda başlar. Başka bir Node-RED süreci aynı portta çalışıyorsa önce onu durdurun.

## 13. Mevcut kapsam ve doğrulama

Giriş formundaki **Remember me** kutusuna bağlı ek bir işlem, **Forgot Password?** bağlantısına bağlı parola sıfırlama akışı ve kullanıcı kayıt sayfası bulunmaz. Kategori akışında kategori listeleme, düzenleme veya silme sayfası yoktur. Destek personeli listesi ayrı bir rol filtresi uygulamadan kullanıcıları getirir.

Depodaki JWT düğümlerinde ortak imzalama anahtarı bulunur; Node-RED editörü için `adminAuth` etkin değildir. README'deki başlatma komutu bu nedenle erişimi kendi bilgisayarınızla sınırlar. Örnek `1234` parolaları ve mevcut ayarlar yerel deneme içindir.

8 Kasım 2026'da macOS üzerinde **Node.js 26.12.1**, **npm 11.20.0**, **Node-RED 5.0.9** ve **MongoDB 9.0.4** ile geçici bir veritabanında şu işlemler doğrulandı: giriş sayfasını açma, beş örnek kullanıcıyı oluşturma ve bcrypt parolasını doğrulama, giriş, kategori formunu açma, kategori kaydetme, aynı kategori adını tekrar kaydetmeyi engelleme, oturumsuz erişim mesajı ve çıkış. Yeni kopyadaki şifre çözme uyarısı ve yedek dosyayla ilerleme adımı da kontrol edildi.

Windows ve Ubuntu kurulum adımları resmî belgelere göre hazırlanmıştır; bu sistemlerde uygulamalı kurulum testi yapılmamıştır. Doğrulama mevcut kurulu bağımlılıklarla yapılmıştır; temiz bir `npm ci` indirmesi bu kontrolün parçası değildir.

Node-RED komut satırı seçenekleri için [resmî yerel çalıştırma rehberine](https://nodered.org/docs/getting-started/local), `npm ci` davranışı için [npm belgelerine](https://docs.npmjs.com/cli/v11/commands/npm-ci/) bakabilirsiniz.
