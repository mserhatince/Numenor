
2026-05-18 23:20

Status:

Tags:

# Başlıksız 2

Harika bir soru. Öncelikle bir **"gerçeklik kontrolü" (reality check)** yapalım: 

Şartname; kullanacağınız işlemci çekirdeğini (CV32E40P), çalıştıracağınız modeli (TFLite Micro Speech) ve çevre birimlerini sabitlemiş durumda. Yani **"Yeni bir RISC-V komut seti icat edelim"** veya **"Farklı bir yapay zeka algoritması bulalım"** anlamında bir devrim yapamazsınız. Kuralların dışına çıkmak elenmenize sebep olur.

**ANCAK...**
Mühendislik, mimari yaklaşım, otomasyon ve sistem seviyesinde optimizasyon anlamında jürinin **"Bu öğrenciler değil, kıdemli bir çip tasarımı startup'ı"** diyeceği **"devrimsel" (game-changer)** hamleler yapabileceğiniz devasa boşluklar var. 

Çoğu takım "kodum çalıştı, LED yandı, UART'tan 'Evet' yazısı geldi" seviyesinde kalacaktır. Sizi şampiyon yapacak ve jüriyi şok edecek **5 "Gizli Silah" (Devrimsel Yaklaşım)** şunlardır:

---

### 1. "Memory Wall" (Bellek Duvarı) Problemini Çözmek: Ping-Pong DMA Mimarisi
Yarışmacıların %90'ı YZ Hızlandırıcıyı şöyle tasarlayacak: *CPU veriyi SRAM'e yazar -> Hızlandırıcıyı başlatır -> Hızlandırıcı SRAM'den okur, çarpar, toplar -> SRAM'e yazar -> CPU'ya kesme atar.*
Bu mimaride hızlandırıcı, veriyi bellekten beklerken (Memory Bottleneck) boşta oturur.

*   **Devrimsel Hamle:** Hızlandırıcının içine **Özel bir DMA (Direct Memory Access) Kontrolcüsü** ve **Çift Portlu Ping-Pong (Double Buffer) SRAM** mimarisi koyun.
*   **Nasıl Çalışır?** Hızlandırıcı "A" belleğindeki veriyi işlerken, DMA arka planda AXI otobüsü üzerinden "B" belleğine bir sonraki katmanın verisini (weights & activations) çeker. İşlem bittiğinde bellekler anında yer değiştirir (Ping-Pong).
*   **Jüriye Etkisi:** Raporunuzda *"Standart mimarilere göre bellek bekleme gecikmesini (stall cycle) %90 oranında düşürdük, MAC (Çarpma-Toplama) ünitesinin kullanım oranını (utilization) %95'e çıkardık"* dediğinizde, jüri size ayakta alkış tutar. Bu, gerçek bir AI çipi (NPU) tasarımının temelidir.

### 2. Donanım Telemetrisi ve "Profiling" (Gözlemlenebilirlik)
Öğrenci takımları çipi bir "kara kutu" olarak tasarlar. Çalışıyorsa sorun yoktur. Ancak endüstride çipin *nasıl* çalıştığını ölçmek, çalışması kadar önemlidir.

*   **Devrimsel Hamle:** YZ Hızlandırıcısının içine **Performans Sayaçları (Performance Counters)** ekleyin ve bunları AXI-Lite üzerinden okunabilir yazmaçlara (CSR) bağlayın.
*   **Örnek Yazmaçlar:**
    *   `CYCLE_COUNT`: İşlem kaç saat döngüsünde bitti?
    *   `MAC_UTILIZATION`: Çarpma-toplama üniteleri kaç döngü aktif çalıştı?
    *   `BUS_STALL`: AXI otobüsü veriyi getiremediği için hızlandırıcı kaç döngü bekledi?
*   **Jüriye Etkisi:** C kodunuzda inference (çıkarım) bittikten sonra UART'tan sadece "Sonuç: Evet" yazdırmayın. Şöyle bir rapor bastırın:
    ```text
    [AI INFERENCE DONE]
    Result: YES (Confidence: 94%)
    Total Cycles: 14,500
    MAC Utilization: 88%
    Bus Stall Cycles: 120
    ```
    Bu, **HW/SW Co-Design (Donanım/Yazılım Ortak Tasarımı)** konusunda rakiplerinizin fersah fersah önüne geçmenizi sağlar.

### 3. Donanım CI/CD (Sürekli Entegrasyon) Kültürü
Şartname "otomasyon kodları (Makefile, Python)" istiyor. Çoğu takım `make run_test` yazan basit bir bash script koyacaktır.

*   **Devrimsel Hamle:** **GitHub Actions** kullanarak bir **Donanım CI/CD Boru Hattı (Pipeline)** kurun.
*   **Nasıl Çalışır?** GitHub reponuza her kod_push_ ettiğinizde veya Pull Request açtığınızda, arka planda GitHub Actions (veya kendi sunucunuz) otomatik olarak:
    1. Verilator veya DSim ile tüm UVM testlerini koşar.
    2. Lint (kod kalitesi) kontrolü yapar.
    3. Bir Python scripti ile Coverage (kapsam) raporunu çeker.
    4. GitHub README.md dosyanızın içine dinamik olarak "✅ Build Passing | 📊 Coverage: 92%" rozetini (badge) ekler.
*   **Jüriye Etkisi:** Jüri üyesi reponuza girdiğinde, profesyonel bir çip şirketinin (örn. SiFive, Tenstorrent) mühendislik altyapısını görecektir. Bu, "Automation is King" felsefesinin zirvesidir.

### 4. Sparsity (Sıfır Atlama) ve Ağırlık Sıkıştırma
TFLite Micro Speech modeli (Tiny Conv) küçüktür ancak sinir ağlarında ağırlıkların (weights) büyük bir kısmı "0" veya "0'a çok yakın" değerlerdir.

*   **Devrimsel Hamle:** MAC (Multiply-Accumulate) dizinizi tasarlarken **"Zero-Skipping" (Sıfır Atlama)** mantığı ekleyin. Eğer ağırlık veya giriş verisi sıfırsa, çarpma işlemini yapma, saat döngüsünü (veya gücünü) harcamadan bir sonraki adıma geç.
*   **Jüriye Etkisi:** Raporunuzda *"Modeldeki %40'lık sıfır ağırlık oranını donanım seviyesinde tespit edip, dinamik güç tüketimini ve işlem süresini %30 oranında düşüren Sparsity-Aware (Seyreklik Farkında) bir MAC mimarisi kullandık"* demek, yapay zeka donanımı literatüründeki en güncel ve seksi konulardan birine dokunduğunuzu gösterir.

### 5. Hata Yönetimi ve "Graceful Degradation" (Zarif Çöküş)
Öğrenci tasarımlarında AXI otobüsünde bir hata olursa (örn. olmayan bir adrese okuma atıldı) sistem kilitlenir (hang). 

*   **Devrimsel Hamle:** AXI Interconnect'inize ve çevre birimlerinize **AXI Slave Error Response (SLVERR / DECERR)** mekanizması ekleyin.
*   **Nasıl Çalışır?** Yazılım yanlışlıkla YZ hızlandırıcının yasaklı bir adresine yazmaya çalışırsa, donanım sistemi kilitlemek yerine AXI üzerinden RISC-V çekirdeğine "Bus Error" döner. Çekirdek bir **Exception (İstisna)** üretir ve C kodundaki `trap_handler` devreye girerek hatayı loglar ve sistemi resetlemeden o periferali kurtarır.
*   **Jüriye Etkisi:** Endüstriyel otomotiv veya savunma sanayii çiplerinde (ISO 26262 vb.) bu bir zorunluluktur. Jüri, sizin sadece "mutlu yolu" (happy path) değil, "kötü senaryoları" da düşünen bir **Senior Mimarı** zihniyetine sahip olduğunuzu görecektir.

---

### ⚠️ ALTIN KURAL: "Yürümeden Koşmaya Çalışmayın"

Bu devrimsel fikirler, **ancak ve ancak** şartnamenin temel gereksinimleri (Boot akışı, temel çevre birimlerinin çalışması, GDSII çıktısının hatasız alınması) **%100 kusursuz** çalışıyorsa işe yarar. 

Eğer UART'ınız bazen veri kaçırıyorsa veya STA (Zamanlama) raporunuzda "Setup Time Violation" hataları varsa, Ping-Pong DMA yazmanızın hiçbir anlamı yoktur. Jüri önce **"Güvenilirlik ve Standartlara Uyum"** arar, sonra **"Yenilikçilik"** arar.

**Stratejiniz şu olmalı:**
1. **İlk 2 Ay:** Şartnamedeki her şeyi "sıkıcı ama kusursuz" şekilde çalıştırın. (Boot, UART, I2C, QSPI, Temel MAC).
2. **Sonraki 2 Ay:** Yukarıdaki "Devrimsel" fikirlerden **en az birini** (Örn: Telemetri ve Ping-Pong DMA) sisteme entegre edin.
3. **Son 1 Ay:** Backend (Fiziksel Tasarım) optimizasyonları, Clock Gating ve CI/CD otomasyonunu parlatın.

Eğer bu vizyonla ilerlerseniz, sadece bir yarışma kazanmazsınız; aynı zamanda Yongatek veya benzeri üst düzey çip tasarımı şirketlerinden **iş teklifi** alacak bir portföy oluşturursunuz. Hangi fikir size daha yakın geldi? Onun üzerinden teknik detaylara (RTL mimarisine) inebiliriz.





# References

