Devam. Şimdi **Fixed Point & Floating Point Gösterim** kısmına geçiyoruz. Bu belge el yazısı/görsel olduğu için kod gibi değil, sayı gösterimi mantığı üzerinden çalışacağız.

**1. Sabit Nokta Gösterimi**

Sabit nokta, binary sayıda virgülün yerinin sabit kabul edilmesidir.

Örnek:

```text
01101011
```

Belgede bu sayı şöyle ayrılmış:

```text
01101 . 011
```

Yani:

```text
integer bits     = 01101
fractional bits  = 011
```

Bu şu anlama gelir:

```text
Virgülden önce 5 bit
Virgülden sonra 3 bit
```

Format:

```text
Qm.n
```

Burada:

```text
m = integer bit sayısı
n = fractional bit sayısı
```

**2. Binary Kesir Nasıl Hesaplanır?**

Virgülden önceki bitler normal ikili sayı:

```text
01101₂ = 13₁₀
```

Virgülden sonraki bitler:

```text
0.011₂
```

Bit ağırlıkları:

```text
1. kesir biti = 2^-1 = 0.5
2. kesir biti = 2^-2 = 0.25
3. kesir biti = 2^-3 = 0.125
```

Yani:

```text
0.011₂ = 0*0.5 + 1*0.25 + 1*0.125
       = 0.375
```

Toplam:

```text
01101.011₂ = 13.375₁₀
```

**3. Sabit Nokta Örnekleri**

Örnek:

```text
000100.01₂
```

Virgülden önce:

```text
000100₂ = 4
```

Virgülden sonra:

```text
.01₂ = 0*2^-1 + 1*2^-2
     = 0 + 0.25
     = 0.25
```

Sonuç:

```text
4.25₁₀
```

Örnek:

```text
11010.1001₂
```

Virgülden önce:

```text
11010₂ = 26
```

Virgülden sonra:

```text
.1001₂ = 1*0.5 + 0*0.25 + 0*0.125 + 1*0.0625
       = 0.5625
```

Sonuç:

```text
26.5625₁₀
```

**4. Decimal Sayıyı Sabit Noktaya Çevirme**

Örnek:

```text
12.25₁₀
```

Önce tam kısmı çevir:

```text
12₁₀ = 1100₂
```

Sonra kesir kısmı:

```text
0.25 * 2 = 0.5   -> bit 0
0.5  * 2 = 1.0   -> bit 1
```

Yani:

```text
0.25₁₀ = .01₂
```

Sonuç:

```text
12.25₁₀ = 1100.01₂
```

Örnek:

```text
5.75₁₀
```

Tam kısım:

```text
5 = 101₂
```

Kesir:

```text
0.75 * 2 = 1.5 -> bit 1
0.5  * 2 = 1.0 -> bit 1
```

Sonuç:

```text
5.75₁₀ = 101.11₂
```

**5. Floating Point Mantığı**

Floating point, virgülün yerini sabit tutmaz. Sayıyı bilimsel gösterime benzer yazar.

Genel mantık:

```text
sayı = (-1)^S x mantissa x 2^üs
```

Belgedeki IEEE-754 tek hassasiyet yapısı:

```text
32 bit toplam
```

Parçalar:

```text
1 bit  -> işaret biti
8 bit  -> exponent / üst
23 bit -> mantissa / kesir
```

Şema:

```text
S | Exponent | Mantissa
1 |    8     |   23
```

**6. İşaret Biti**

```text
S = 0 -> sayı pozitif
S = 1 -> sayı negatif
```

Formülde:

```text
(-1)^S
```

Eğer `S=0`:

```text
(-1)^0 = 1
```

Eğer `S=1`:

```text
(-1)^1 = -1
```

**7. Exponent ve Bias**

IEEE-754 tek hassasiyette exponent 8 bittir.

Bias değeri:

```text
127
```

Gerçek üs:

```text
E = exponent_değeri - 127
```

Örnek:

```text
exponent = 10000010₂
```

Decimal karşılığı:

```text
130
```

Gerçek üs:

```text
130 - 127 = 3
```

**8. Mantissa**

Normalize edilmiş binary sayı genelde şöyle yazılır:

```text
1.xxxxx x 2^E
```

IEEE-754’te baştaki `1` saklanmaz. Buna gizli 1 gibi düşünebilirsin.

Mantissa alanına sadece virgülden sonraki kısım yazılır.

Örnek:

```text
1.1011 x 2^3
```

Mantissa kısmı:

```text
1011
```

23 bite tamamlanır:

```text
10110000000000000000000
```

**9. Decimal Sayıyı IEEE-754’e Çevirme**

Örnek:

```text
13.25₁₀
```

Önce binary yapalım.

Tam kısım:

```text
13 = 1101₂
```

Kesir kısmı:

```text
0.25 = .01₂
```

Yani:

```text
13.25₁₀ = 1101.01₂
```

Normalize et:

```text
1101.01₂ = 1.10101 x 2^3
```

İşaret pozitif:

```text
S = 0
```

Gerçek üs:

```text
E = 3
```

Exponent alanı:

```text
E + bias = 3 + 127 = 130
130₁₀ = 10000010₂
```

Mantissa:

```text
10101
```

23 bite tamamla:

```text
10101000000000000000000
```

Sonuç:

```text
0 | 10000010 | 10101000000000000000000
```

**10. Negatif Sayı Örneği**

Örnek:

```text
-5.25₁₀
```

Binary:

```text
5 = 101₂
0.25 = .01₂
```

```text
5.25 = 101.01₂
```

Normalize:

```text
101.01₂ = 1.0101 x 2^2
```

İşaret:

```text
S = 1
```

Exponent:

```text
2 + 127 = 129
129₁₀ = 10000001₂
```

Mantissa:

```text
01010000000000000000000
```

Sonuç:

```text
1 | 10000001 | 01010000000000000000000
```

**11. IEEE-754’ten Decimal’e Dönme**

Örnek:

```text
0 | 10000010 | 10100000000000000000000
```

İşaret:

```text
S = 0 -> pozitif
```

Exponent:

```text
10000010₂ = 130
E = 130 - 127 = 3
```

Mantissa:

```text
1.101
```

Değer:

```text
1.101₂ x 2^3
```

Virgülü 3 sağa kaydır:

```text
1101.0₂
```

Decimal:

```text
13₁₀
```

**12. En Sınavlık Ezber**

Sabit nokta:

```text
Virgülden önce: 2^0, 2^1, 2^2...
Virgülden sonra: 2^-1, 2^-2, 2^-3...
```

IEEE-754 tek hassasiyet:

```text
Toplam 32 bit
1 bit işaret
8 bit exponent
23 bit mantissa
Bias = 127
Değer = (-1)^S x 1.M x 2^(Exponent-127)
```

Decimalden floating’e:

```text
1. Sayıyı binary yap.
2. 1.xxxx x 2^E şeklinde normalize et.
3. İşaret bitini yaz.
4. Exponent = E + 127.
5. Mantissa = 1’den sonraki bitler.
6. Mantissa’yı 23 bite tamamla.
```

Bu bölümde hocanın sorusu büyük ihtimalle ya sabit noktalı bir binary sayıyı decimal’e çevirme, ya da IEEE-754 formatının alanlarını sorma şeklinde gelir.

Sırada son teorik bölüm kaldı: **SST_sunu.pdf**, yani FPGA/VHDL genel teori.