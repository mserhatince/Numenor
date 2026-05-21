Devam. Şimdi hocanın **RAM_Bellek.pdf** dosyasındaki RAM yapısına geçiyoruz.

ROM’da sadece okuma vardı. RAM’de ise:

```text
Okuma var.
Yazma var.
Reset var.
Enable var.
Write Enable var.
```

**1. RAM Nedir?**

Hocanın tanımıyla:

> RAM, verilerin hem okunmasına hem de yazılmasına izin veren bellek elemanıdır.

ROM:

```text
Data sabittir, sadece okunur.
```

RAM:

```text
Data değişebilir, hem okunur hem yazılır.
```

**2. RAM Entity Bölümü**

Hocanın RAM kodundaki giriş-çıkışlar:

```vhdl
entity RAM_bellek is
Generic (
  DATA_W : integer := 8;
  ADRES_W : integer := 6
);
Port (
  Clk : in std_logic;
  Reset : in std_logic;
  Veri_giris : in std_logic_vector(DATA_W - 1 downto 0);
  Adres : in std_logic_vector(ADRES_W - 1 downto 0);
  W_En : in std_logic;
  Enable : in std_logic;
  Veri_cikis : out std_logic_vector(DATA_W - 1 downto 0)
);
end RAM_bellek;
```

Parça parça:

```vhdl
DATA_W : integer := 8;
```

Veri genişliği 8 bit.

```vhdl
ADRES_W : integer := 6;
```

Adres genişliği 6 bit.

6 bit adres kaç farklı adres verir?

```text
2^6 = 64
```

Yani RAM 64 satırlı.

Her satır kaç bit?

```text
8 bit
```

Toplam kapasite:

```text
64 x 8 bit
```

**3. Portların Anlamı**

```text
Clk        -> clock
Reset      -> reset
Veri_giris -> RAM'e yazılacak veri
Adres      -> okunacak/yazılacak adres
W_En       -> write enable, yazma izni
Enable     -> RAM aktif mi?
Veri_cikis -> RAM'den okunan veri
```

`W_En` için hocanın kodunda mantık:

```text
W_En = 1 -> yazma
W_En = 0 -> okuma
```

**4. Bellek Dizisi Tanımlama**

Hocanın kodu:

```vhdl
type RAM_dizi is array ((2 ** ADRES_W) - 1 downto 0)
of std_logic_vector(DATA_W - 1 downto 0);

signal bellek: RAM_dizi;
```

Bunu Türkçeye çevirelim.

```vhdl
2 ** ADRES_W
```

Bu, 2 üzeri adres genişliği demek.

Adres genişliği 6 ise:

```text
2 ** 6 = 64
```

Dolayısıyla:

```vhdl
((2 ** ADRES_W) - 1 downto 0)
```

şuna dönüşür:

```vhdl
63 downto 0
```

Tam anlamı:

```text
RAM_dizi adında bir dizi tipi tanımla.
Bu dizi 64 elemanlı olsun.
Her eleman DATA_W bitlik std_logic_vector olsun.
bellek adında bu tipten bir signal oluştur.
```

Yani:

```text
bellek(0)  = 8 bit
bellek(1)  = 8 bit
...
bellek(63) = 8 bit
```

**5. Okuma Process’i**

Hocanın kodunda okuma işlemi ayrı process:

```vhdl
process (Clk)
begin
  if (Clk'event and Clk='1') then
    if Reset = '1' then
      Veri_cikis <= (others => '0');
    elsif Enable = '1' then
      if W_En = '0' then
        Veri_cikis <= bellek(conv_integer(Adres));
      else
        Veri_cikis <= Veri_giris;
      end if;
    end if;
  end if;
end process;
```

Türkçesi:

```text
Clock yükselen kenar geldiyse:
    Reset 1 ise:
        Veri_cikis'i 0 yap.
    Reset değilse ve Enable 1 ise:
        W_En 0 ise:
            Adres'teki belleği oku ve Veri_cikis'e ver.
        W_En 1 ise:
            Veri_cikis'e Veri_giris'i ver.
```

Buradaki önemli satır:

```vhdl
Veri_cikis <= bellek(conv_integer(Adres));
```

ROM’dakiyle aynı mantık:

```text
Adres vektörünü integer'a çevir, bellek dizisinin o elemanını oku.
```

Bu satır:

```vhdl
Veri_cikis <= (others => '0');
```

şu demek:

```text
Veri_cikis kaç bit ise hepsini 0 yap.
```

Eğer `DATA_W=8` ise:

```text
Veri_cikis = "00000000"
```

**6. Yazma Process’i**

Hocanın kodunda yazma işlemi ikinci process:

```vhdl
process (Clk)
begin
  if (Clk'event and Clk='1') then
    if Reset = '1' then
      for i in bellek'range loop
        bellek(i) <= (others => '0');
      end loop;
    elsif Enable = '1' then
      if W_En = '1' then
        bellek(conv_integer(Adres)) <= Veri_giris;
      end if;
    end if;
  end if;
end process;
```

Türkçesi:

```text
Clock yükselen kenar geldiyse:
    Reset 1 ise:
        bellekteki tüm adresleri 0 yap.
    Reset değilse ve Enable 1 ise:
        W_En 1 ise:
            Veri_giris değerini, Adres'in gösterdiği bellek hücresine yaz.
```

Buradaki en önemli satır:

```vhdl
bellek(conv_integer(Adres)) <= Veri_giris;
```

Anlamı:

```text
Adres'i integer'a çevir.
bellek dizisindeki o adrese Veri_giris'i yaz.
```

Örnek:

```text
Adres = "000011"
conv_integer(Adres) = 3
Veri_giris = "10101010"

bellek(3) <= "10101010"
```

**7. Reset’te Tüm Belleği Temizleme**

```vhdl
for i in bellek'range loop
  bellek(i) <= (others => '0');
end loop;
```

Burada:

```vhdl
bellek'range
```

belleğin tüm indeks aralığı demek.

Eğer bellek:

```vhdl
63 downto 0
```

ise loop bütün adreslerde döner.

Türkçesi:

```text
Bellekteki tüm adresler için:
    o adresteki veriyi sıfırla.
```

**8. RAM’in Tam Kalıbı**

Hocanın stiline uygun hali:

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;

entity RAM_bellek is
Generic (
  DATA_W : integer := 8;
  ADRES_W : integer := 6
);
Port (
  Clk : in std_logic;
  Reset : in std_logic;
  Veri_giris : in std_logic_vector(DATA_W - 1 downto 0);
  Adres : in std_logic_vector(ADRES_W - 1 downto 0);
  W_En : in std_logic;
  Enable : in std_logic;
  Veri_cikis : out std_logic_vector(DATA_W - 1 downto 0)
);
end RAM_bellek;

architecture Behavioral of RAM_bellek is

type RAM_dizi is array ((2 ** ADRES_W) - 1 downto 0)
of std_logic_vector(DATA_W - 1 downto 0);

signal bellek: RAM_dizi;

begin

process (Clk)
begin
  if (Clk'event and Clk='1') then
    if Reset = '1' then
      Veri_cikis <= (others => '0');
    elsif Enable = '1' then
      if W_En = '0' then
        Veri_cikis <= bellek(conv_integer(Adres));
      else
        Veri_cikis <= Veri_giris;
      end if;
    end if;
  end if;
end process;

process (Clk)
begin
  if (Clk'event and Clk='1') then
    if Reset = '1' then
      for i in bellek'range loop
        bellek(i) <= (others => '0');
      end loop;
    elsif Enable = '1' then
      if W_En = '1' then
        bellek(conv_integer(Adres)) <= Veri_giris;
      end if;
    end if;
  end if;
end process;

end Behavioral;
```

**ROM ve RAM Farkı**

```text
ROM:
constant Data vardır.
Sadece okuma yapılır.
Cikis <= Data(adres);

RAM:
signal bellek vardır.
Okuma ve yazma yapılır.
W_En = 0 -> oku
W_En = 1 -> yaz
```

**Mini Kontrol**

Soru:

```vhdl
type RAM_dizi is array ((2 ** ADRES_W) - 1 downto 0)
of std_logic_vector(DATA_W - 1 downto 0);
```

`ADRES_W=6`, `DATA_W=8` ise RAM kaç elemanlıdır ve her eleman kaç bittir?

Cevap:

```text
2^6 = 64 elemanlıdır.
Her eleman 8 bittir.
```

Soru:

```vhdl
bellek(conv_integer(Adres)) <= Veri_giris;
```

ne demek?

Cevap:

```text
Adres'in gösterdiği RAM hücresine Veri_giris değerini yaz.
```

Sıradaki bölüm: **Test bench + component tekrar + function/generic**. Bu kısım kısa ama sınavda çıkarsa direkt puan getirir.