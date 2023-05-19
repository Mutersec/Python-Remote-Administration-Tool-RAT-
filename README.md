# Rat

_Terminal uygulamaları oluşturmak için kabuk komutları oluşturun_


## Genel Bakış

Rat, [tig](https://github.com/Mutersec/Python-Remote-Administration-Tool-RAT-) gibi, görüş bildirmeyen bir UI mantığı olmadan, `git log` gibi kabuk komutlarının `--pretty` ve `--graph` seçenekleriyle yeteneklerine dayanarak yapılandırılmış bir uygulama oluşturma çabasının bir parçası olarak geliştirilmiştir.

Kabuk komutları yürütülür ve çıktı yakalanır ve sayfalayıcılarda görüntülenir. Yapılandırılabilir anotasyon oluşturucuları, çıktıyı tarar ve diğer kabuk komutlarını çalıştırmak için harekete geçilebilecek anotasyonlar ekler.

## Başlarken

**UYARI: BU TÜMÜYLE DENEYSEL VE MUHTEMELEN ÇOK DEĞİŞECEK**

### Kurulum

```shell
$ go get github.com/Mutersec/Python-Remote-Administration-Tool-RAT-
$ go build && go install
```

### Yapılandırma

Rat, ev yapılandırma dizininizde (`$XDG_CONFIG_HOME/rat` [standartları](https://specifications.freedesktop.org/basedir-spec/latest)'na göre veya varsayılan olarak `~/.config/rat`) bulunan `ratrc` adlı bir dosya aracılığıyla yapılandırılır.

Rat sayfalayıcıları bir veya daha fazla "mod"da açılabilir. Bir mod, "anotasyon oluşturucuları" ve "tuş atamaları" yapılandırmasından oluşur:

- Anotasyon oluşturucuları, sayfalayıcının içeriğinde özel metin parçalarını arayan anotasyonları ekler. Bu metin parçaları "anotasyonlar" olarak adlandırılır ve her biri bir başlangıç, bir bitiş, bir sınıf ve bir değer içerir.
- Tuş atamaları, anotasyonlar üzerinde gerçekleştirilebilecek işlemleri tanımlar.

#### Tuş Atamaları

Öncelikle bazı tuş atamalarını yapılandırmanız gerekecektir. Aşağıdakileri `ratrc` dosyanıza ekleyin ve isteğe bağlı olarak düzenleyin:

```shell
bindkey C-r reload


bindkey j   cursor-down
bindkey k   cursor-up
bindkey C-e scroll-down
bindkey C-y scroll-up
bindkey C-d page-down
bindkey C-u page-up
bindkey g,g cursor-first-line
bindkey S-g cursor-last-line
bindkey S-j parent-cursor-down
bindkey S-k parent-cursor-up
bindkey q   pop-pager
bindkey S-q quit
bindkey M-1 show-one
bindkey M-2 show-two
bindkey M-3 show-three
```

<kbd>ctrl</kbd>+<kbd>c</kbd> her zaman çıkış yapar.

#### Mod Tanımları

`mode` anahtar kelimesi bir mod tanımını başlatır.

```shell
mode <ad>
  ...
end
```

#### Anotasyon Oluşturucu Tanımları

Bir mod tanımının içinde, `annotate` anahtar kelimesi bir anotasyon tanımını başlatır.

```shell
mode <ad>
  annotate <tip> <sınıf> -- <seçenekler>
end
```

- `tip`: Anotasyon oluşturucu tipi. "match", "regex" veya "external" olabilir.
- `sınıf`: Bu anotasyonu bulan anotasyon oluşturucu tarafından uygulanacak sınıf.
- `seçenekler`:
    - `tip` "match" ise, bu, anotasyon oluşturucunun arayacağı satır satır çıktı üreten bir kabuk komutu olmalıdır.
    - `tip` "regex" ise, bu, aranacak bir düzenli ifadeyi tanımlamalıdır (Golang düzenli ifadeleri desteklenir).
    - `tip` "external" ise, bu, `~/.config/rat/annotators/` konumunda bulunan ve sayfalayıcının içeriğini STDIN aracılığıyla alacak ve çalıştırılacak yürütülebilir bir dosyanın adı olmalıdır. Yürütülebilir dosya, aşağıdaki ikili biçimde STDOUT'a anotasyonları yazmalıdır:
        - Başlangıç: STDIN'in başlangıcından itibaren bayt ofseti (64 bitlik küçük uçlu işaretli tamsayı)
        - Bitiş: STDIN'in başlangıcından itibaren bayt ofseti (64 bitlik küçük uçlu işaretli tamsayı)
        - Değer uzunluğu: Bulunan değer dizisinin bayt cinsinden uzunluğu (64 bitlik küçük uçlu işaretli tamsayı)
        - Değer dizisi: Yukarıda belirtilen uzunlukta bir dizedir (UTF-8 kodlaması)

#### Tuş Atama Tanımları

`bindkey` anahtar kelimesi bir tuş atama tanımını başlatır.

```shell
mode <ad>
  bindkey <tuş> [<anotasyon-sınıfları>] [<yeni-sayfalayıcı-modu>] -- <eylem>
end

bindkey <tuş> <eylem>
bindkey <tuş> <

yeni-sayfalayıcı-modu> -- <komut>
```

- `tuş`: Basıldığında bu eylemi tetikleyecek tuş kombinasyonu. Modifiyerler `C-` ve `S-` ile eklenir. Desteklenen isimli tuşların bir listesi için `lib/key_event.go` dosyasına bakın.
- `eylem`: Tuş basıldığında çalıştırılacak adlandırılmış bir eylem. Kullanılabilir eylemlerin bir listesi için action.go'ya bakın.
- `anotasyon-sınıfları`: Bu eylem, geçerli satırda bu sınıflardan anotasyonlar varsa yalnızca tetiklenir. Belirtilmezse, tuş ataması sayfalayıcıda herhangi bir yerde çalışır. Bunlar virgülle ayrılmalıdır.
- `yeni-sayfalayıcı-modu`: Eylem yeni bir sayfalayıcı oluşturacaksa, bu, o sayfalayıcıyı oluştururken kullanılacak mod(lar)ı tanımlar.
- `komut`: Belirtilen tuş kombinasyonu basıldığında çalıştırılacak bir kabuk komutudur. Anotasyon değerleri, ilgili anotasyon sınıflarına adlandırılmış değişkenler olarak komut işlemine aktarılır. Varsayılan olarak, kabuk komutunun çıktısını gösteren yeni bir sayfalayıcı açar, ancak aşağıdaki özel önekler kullanılarak farklı eylemler belirtmek için bazı özel önekler kullanılabilir:
    - `!`: Yeni bir sayfalayıcı açmayın. Komutu çalıştırın ve geçerli sayfalayıcıyı yeniden yükleyin.
    - `?!`: `!` gibi, ancak önce kullanıcı onayını onaylar (evet için 'y' veya hayır için 'n' tuşuna basmanız gerekecektir).
    - `>`: Varsayılan gibi, kabuk komutunun içeriğiyle yeni bir sayfalayıcı açar, ancak aynı zamanda ebeveyn-çocuk ilişkisi kurar, böylece çocuk sayfalayıcının içinden ebeveyn imleci yukarı ve aşağı hareket ettirebilirsiniz `ParentCursorUp` ve `ParentCursorDown` komutlarıyla.

Not: Mod tanımı içinde olmayan tuş atamaları her zaman kullanılabilir ve yukarıda açıklanan özel önek davranışına sahip değillerdir.

#### Ayrı dosyalardan yapılandırma tanımlarını içe aktarma

`source` anahtar kelimesi başka bir dosyadan yapılandırma kurallarını içe aktarır.

```shell
source <dosya>
```

- `dosya`: Geçerli rat yapılandırma dizinine göre göreli bir yol olmalıdır ve geçerli rat yapılandırma talimatlarını içeren bir dosyayı belirt

melidir.

#### Örnek yapılandırma

Aşağıda örnek bir `ratrc` dosyası verilmiştir:

```shell
mode main
  annotate match Error -- grep -i error
  annotate regex Info -- \\binfo\\b
  bindkey j cursor-down
  bindkey k cursor-up
  bindkey g,g cursor-first-line
  bindkey G cursor-last-line
  bindkey q quit
end

mode diff
  annotate external HighlightLines -- ~/.config/rat/annotators/highlight_lines
  bindkey j cursor-down main
  bindkey k cursor-up main
  bindkey q quit
end

source ~/.config/rat/custom_bindings
```

Bu yapılandırma, `main` modunda "Error" metnini arayan bir anotasyon oluşturucu, "Info" kelimesini düzenli ifadelerle arayan bir anotasyon oluşturucu, ve bazı tuş atamaları tanımlar. Ayrıca `diff` modunda `HighlightLines` adlı harici bir anotasyon oluşturucu ve modlar arasında geçiş yapmak için bazı tuş atamaları içe aktarılır.

### Kullanım

Rat'ı kullanmak için, kabuk komutunuzu `rat` ile başlatın ve komutu çalıştırın:

```shell
$ git log --pretty=oneline --graph | rat
```

Bu, `git log --pretty=oneline --graph` komutunun çıktısını Rat üzerinden görüntüler.

Rat, komutun çıktısını sayfalayıcıda görüntüler. Aşağıdaki tuş kombinasyonlarını kullanarak gezinebilirsiniz:

- <kbd>j</kbd>: Aşağı hareket eder.
- <kbd>k</kbd>: Yukarı hareket eder.
- <kbd>C-e</kbd>: Aşağı kaydırır.
- <kbd>C-y</kbd>: Yukarı kaydırır.
- <kbd>C-d</kbd>: Sayfa aşağı kaydırır.
- <kbd>C-u</kbd>: Sayfa yukarı kaydırır.
- <kbd>g</kbd><kbd>g</kbd>: İlk satıra gider.
- <kbd>G</kbd>: Son satıra gider.
- <kbd>q</kbd>: Sayfalayıcıyı kapatır.

Ayrıca, belirli bir anotasyona atanan bir tuş atamasını tetiklemek için ilgili tuş kombinasyonunu kullanabilirsiniz. Örneğin, yukarıdaki örnekte `j` tuşuna bastığınızda, sayfalayıcı yukarı hareket ederken aynı zamanda imleç de yukarı hareket edecektir.

## Katkıda bulunma

Soru, sorunlar, düşünceler ve pull istekleri için GitHub sorun takipçisini kullanın.