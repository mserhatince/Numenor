Harika. Şimdi hocanın belgeleri üzerinden **component + test bench + function + generic** kısmını toparlayalım.

**1. Component Tekrarı**

Hocanın `YarımToplayıcı_TamToplayıcı_Component Tanımlama_Test Bench Yazma.pdf` dosyasındaki ana fikir:

> Daha önce yazılan bir devre, başka bir devrede component olarak tekrar kullanılabilir.

Örnek: iki tane yarım toplayıcı kullanarak tam toplayıcı yapmak.

Önce yarım toplayıcı:

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity yarim_toplayici is
port (
  x,y : in std_logic;
  S,C : out std_logic
);
end yarim_toplayici;

architecture Behavioral of yarim_toplayici is
begin
  S <= x xor y;
  C <= x and y;
end Behavioral;
```

Sonra tam toplayıcı:

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity Tam_Toplayici is
port (
  x,y,Ci : in std_logic;
  Co,S: out std_logic
);
end Tam_Toplayici;

architecture Behavioral of Tam_Toplayici is

signal A1,A2,A3 : std_logic;

component yarim_toplayici
port (
  x,y : in std_logic;
  S,C : out std_logic
);
end component;

begin

YT_1: yarim_toplayici port map (x,y,A1,A2);
YT_2: yarim_toplayici port map (A1,Ci,S,A3);

Co <= A3 or A2;

end Behavioral;
```

**2. Buradaki Port Map’i Okuma**

Component port sırası:

```vhdl
x, y, S, C
```

Bu satır:

```vhdl
YT_1: yarim_toplayici port map (x,y,A1,A2);
```

şu demek:

```text
Component x girişine ana devredeki x bağlandı.
Component y girişine ana devredeki y bağlandı.
Component S çıkışına A1 bağlandı.
Component C çıkışına A2 bağlandı.
```

Yani:

```text
x + y -> A1 toplam, A2 elde
```

İkinci yarım toplayıcı:

```vhdl
YT_2: yarim_toplayici port map (A1,Ci,S,A3);
```

şu demek:

```text
A1 + Ci -> S toplam, A3 elde
```

Sonra:

```vhdl
Co <= A3 or A2;
```

İki elde ihtimalinden biri varsa tam toplayıcının elde çıkışı 1 olur.

**3. Test Bench Nedir?**

Hocanın tanımı:

> Test bench, gerçeklemek istediğimiz devrenin simülasyonunu yapmak için hazırlanan test kodudur.

Test bench’in en önemli farkı:

```text
Entity kısmında giriş/çıkış portu olmaz.
```

Çünkü test bench dış dünyaya bağlanan devre değil, devreyi test eden ortamdır.

Hocanın yarım toplayıcı test bench örneği:

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity YT_Test_Bench is
end YT_Test_Bench;

architecture Behavioral of YT_Test_Bench is

component yarim_toplayici
port (
  x,y : in std_logic;
  S,C : out std_logic
);
end component;

signal x: std_logic:= '0';
signal y: std_logic:= '0';
signal C : std_logic;
signal S : std_logic;

begin

U1: yarim_toplayici port map(x=>x,y=>y,C=>C,S=>S);

process
begin
  wait for 2us;
  x<='1';
  y<='0';

  wait for 2us;
  x<='0';
  y<='1';

  wait for 2us;
  x<='1';
  y<='1';
end process;

end Behavioral;
```

**4. Test Bench’i Türkçeye Çevirelim**

```vhdl
entity YT_Test_Bench is
end YT_Test_Bench;
```

Port yok. Çünkü test bench dışarıdan giriş almaz, çıkış vermez.

```vhdl
signal x: std_logic:= '0';
signal y: std_logic:= '0';
```

Test edilecek giriş sinyalleri başlangıçta 0.

```vhdl
U1: yarim_toplayici port map(x=>x,y=>y,C=>C,S=>S);
```

Test edilecek yarım toplayıcı devresi oluşturuldu.

```vhdl
wait for 2us;
x<='1';
y<='0';
```

2 mikrosaniye bekle, sonra girişleri değiştir.

Devamında farklı giriş kombinasyonları veriliyor.

**5. Test Bench Sınav Kalıbı**

Eğer senden bir devre için test bench istenirse iskelet:

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity TB_adi is
end TB_adi;

architecture Behavioral of TB_adi is

component test_edilecek_devre
port(
  -- devrenin portları
);
end component;

signal giris1 : std_logic := '0';
signal giris2 : std_logic := '0';
signal cikis1 : std_logic;

begin

U1: test_edilecek_devre port map(...);

process
begin
  -- girişleri değiştir
  wait for 2us;
  giris1 <= '1';

  wait for 2us;
  giris2 <= '1';

  wait;
end process;

end Behavioral;
```

Hocanın örneğinde en sonda `wait;` yok ama pratikte process’in sonsuz dönmesini engellemek için konabilir. Sınavda hocanın örneğine benzeteceksen koymasan da olur.

**6. Function Nedir?**

Hocanın `function_ornek1.vhd` dosyasında iki tane fonksiyon var.

İlk fonksiyon:

```vhdl
function my_function(A,B,C: std_logic) return std_logic is
variable ara_deger:std_logic;
begin
  ara_deger:=A and B and C;
  return ara_deger;
end my_function;
```

Bu fonksiyonun Türkçesi:

```text
A, B, C isimli üç std_logic giriş al.
std_logic tipinde sonuç döndür.
A and B and C işlemini yap.
Sonucu geri döndür.
```

Kullanımı:

```vhdl
F1<=my_function(i1,i2,i3);
```

Yani:

```text
F1 = i1 and i2 and i3
```

İkinci fonksiyon:

```vhdl
function OR_gate_3giris(A,B,C: std_logic) return std_logic is
variable ara_deger:std_logic;
begin
  ara_deger:=A or B or C;
  return ara_deger;
end OR_gate_3giris;
```

Kullanımı:

```vhdl
F2<=OR_gate_3giris(i1,i2,i3);
```

Yani:

```text
F2 = i1 or i2 or i3
```

**7. Function Genel Kalıbı**

```vhdl
function fonksiyon_adi(girisler) return cikis_tipi is
variable ara_deger:cikis_tipi;
begin
  ara_deger := işlem;
  return ara_deger;
end fonksiyon_adi;
```

Dikkat:

```text
Signal ataması: <=
Variable ataması: :=
```

Function içinde genelde `variable` kullanılıyor.

**8. Generic Nedir?**

Hocanın örneğinde:

```vhdl
generic(n: natural:=4);
```

Bu şu demek:

```text
n isimli genel bir parametre tanımla.
Varsayılan değeri 4 olsun.
```

Sonra portlarda kullanılıyor:

```vhdl
A,B: in std_logic_vector(n-1 downto 0);
```

Eğer `n=4` ise:

```vhdl
std_logic_vector(3 downto 0)
```

Yani 4 bit.

Generic sayesinde aynı devre 4 bit, 8 bit, 16 bit gibi farklı genişliklerde kullanılabilir.

**9. N Bit Toplama/Çıkarma Mantığı**

Hocanın `n_bitlik_toplama_cikarma.vhd` dosyası:

```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;

entity n_bitlik_toplama_cikarma is
generic(n: natural:=4);
port(
  A,B: in std_logic_vector(n-1 downto 0);
  Secme: in std_logic;
  elde: out std_logic;
  Sonuc: out std_logic_vector(n-1 downto 0)
);
end n_bitlik_toplama_cikarma;

architecture Behavioral of n_bitlik_toplama_cikarma is
signal ara_deger: std_logic_vector(n downto 0);
begin

process(Secme,A,B)
begin
  if Secme ='1' then
    ara_deger<=( '0'&A)+('0'&B);
  else
    ara_deger<=('0' & A)-('0' & B);
  end if;
end process;

Sonuc<=ara_deger(n-1 downto 0);
elde<=ara_deger(n);

end Behavioral;
```

**10. Bunu Türkçeye Çevirelim**

```vhdl
signal ara_deger: std_logic_vector(n downto 0);
```

A ve B `n` bit. Ama sonuç `n+1` bit tutuluyor.

Neden?

```text
Elde/taşma biti için fazladan 1 bit gerekiyor.
```

```vhdl
'0' & A
```

A’nın başına 0 ekliyor.

Eğer:

```text
A = "1010"
```

ise:

```text
'0' & A = "01010"
```

Yani 4 bitlik sayı 5 bite çıkarılıyor.

```vhdl
if Secme ='1' then
  ara_deger<=( '0'&A)+('0'&B);
else
  ara_deger<=('0' & A)-('0' & B);
end if;
```

Türkçesi:

```text
Secme 1 ise A+B yap.
Secme 0 ise A-B yap.
```

Sonuç ayırma:

```vhdl
Sonuc<=ara_deger(n-1 downto 0);
elde<=ara_deger(n);
```

Türkçesi:

```text
Alt n biti Sonuc'a ver.
En üst biti elde çıkışına ver.
```

**Bu Bölümün Ezberi**

Component:

```vhdl
component devre_adi
port(...);
end component;

U1: devre_adi port map(...);
```

Test bench:

```vhdl
entity TB is
end TB;
```

Function:

```vhdl
function ad(...) return tip is
variable ara:tip;
begin
  ara := işlem;
  return ara;
end ad;
```

Generic:

```vhdl
generic(n: natural:=4);
```

Başına 0 ekleme:

```vhdl
'0' & A
```

Alt bitleri alma:

```vhdl
Sonuc<=ara_deger(n-1 downto 0);
```

Üst biti alma:

```vhdl
elde<=ara_deger(n);
```

Sırada iki bölüm kaldı: **fixed/floating point** ve **SST teorisi**. Önce sayı gösterimlerini yapalım, sonra teoriyi kısa ama sınavlık şekilde kapatırız.