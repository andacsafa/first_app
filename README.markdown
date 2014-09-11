
Bu belge sayesinde baştan sona ayrıntıları ile bir Ruby Kodunun nasıl gem'e dönüştüreleceğini öğreneceksiniz.

#Başlangıç :

RubyGems'deki araçlar sayesinde gem oluşturmak ve onu kullanıma sunmak kolay olmaktadır. Genelde başlangıç seviyesi yazılan "Hello World" u bir geme çevirip kullanalım.

Gem için yazılan kod Github da bulunmaktadır.

#İlk gem'iniz :

Kendi 'hola' gem ve gemspec'i için bir Ruby dosyası ile başladım. Siz kendi gem'inizi yayınlamak istiyorsanız başka bir isim koymanız gerekmektedir (hola_kullanıcıadınız gibi...). Bir gem'i isimlendirirken
temel kullanımları görmek için [Model Rehberine](http://guides.rubygems.org/patterns/#consistent-naming) bakabilirsiniz. -> 

Gem için olan kodlar lib dosyası altında bulunmaktadır. Genelde lib dosyasının altında gem'iniz ile aynı ada sahip bir Ruby dosyası kullanılır çünkü require 'hola' çalıştırıldığında bu Ruby dosyası açılır. Bu dosya sizin gem kodunuzu ve API'nizi düzenleyip çalışmasını sağlamaktadır.

lib/hola.rb kodu herşeyin temelidir. Bu kod sadece gem'den bir çıktı görmenizi sağlamaktadır:

'''
% cat lib/hola.rb
class Hola
  def self.hi
    puts "Hello world!"
  end
end
'''

Gemspec dosyası gem'de neler olduğunu, kimin yaptığını, sürümünü tutmaktadır. Ayrıca RubyGems.org ile aranızdaki bir arayüz vazifesindedir. Gördüğünüz bilgiler gemspec dosyasında bulunmaktadır.

'''
% cat hola.gemspec
Gem::Specification.new do |s|
  s.name        = 'hola'
  s.version     = '0.0.0'
  s.date        = '2010-04-28'
  s.summary     = "Hola!"
  s.description = "A simple hello world gem"
  s.authors     = ["Nick Quaranto"]
  s.email       = 'nick@quaran.to'
  s.files       = ["lib/hola.rb"]
  s.homepage    =
    'http://rubygems.org/gems/hola'
  s.license       = 'MIT'
end
'''

*Açıklama kısmı bu örnekte görüldüğünden daha uzun olabilir. Eğer /^== [A-Z]/ formatına uygunsa açıklama RDoc biçimlendirici üzerinden çalıştırılıp RubyGems'de gösterilir. Bu açıklamanın kullanıcılar tarafından anlaşılması hususunda bir dikkat göstermek gerekmektedir.

Gemspec dosyası da Ruby dilinde yazılmaktadır. Bu sayede betiklerinizi yazıp dosya isimleri oluşturabilir, sürümü değiştirebilirsiniz. Gemspec dosyası birçok alanı içermektedir. Hepsini görebilmek için [tam kaynağa](http://guides.rubygems.org/specification-reference/) bakabilirsiniz. 

Gemspec oluşturulduktan sonra gem'inizi bunun üzerinden oluşturabilirsiniz. Bu oluşturulan gem'i denemek için kendi makinanıza kurabilirsiniz.

'''
% gem build hola.gemspec
Successfully built RubyGem
Name: hola
Version: 0.0.0
File: hola-0.0.0.gem

% gem install ./hola-0.0.0.gem
Successfully installed hola-0.0.0
1 gem installed
'''

Denemek için gem konsola eklenir ve kullanılır.

'''
% irb
>> require 'hola'
=> true
>> Hola.hi
Hello world!
'''

Bu aşamalardan sonra 'hola' gem'ini Ruby kullanıcılarına sunabilirsiniz. Eğer RubyGems hesabınız var ise, Gem'inizi bir satırda paylaşabilirsiniz. Hesabınızı bilgisayarda kullanmak için :

'''
$ curl -u qrush https://rubygems.org/api/v1/api_key.yaml >
~/.gem/credentials; chmod 0600 ~/.gem/credentials

Enter host password for user 'qrush':
'''

*Curl, OpenSSl veya sertifika ile alakalı bir problem yaşarsanız, yukarıda gördüğünüz adresi tarayıcınıza yazıp sizden istenilen bilgileri girdikten sonra api_key.yaml dosyası indirilecektir. İnen dosyayı ~/.gem dosyasnın altında 'credentials' adında kaydediniz.

Bu işlemden sonra gem'inizi RubyGems'e koyabilirsiniz.

'''
% gem push hola-0.0.0.gem
Pushing gem to RubyGems.org...
Successfully registered gem: hola (0.0.0)
'''

Kısa bir süre içinde, gem'iniz kurulum için herkese açık olacaktır. RubyGems.org sitesinde gem'i görebilir ve oradan herhangi bir bilgisayara indirilebilirsiniz.

'''
% gem list -r hola

*** REMOTE GEMS ***

hola (0.0.0)

% gem install hola
Successfully installed hola-0.0.0
1 gem installed
'''

Görüldüğü üzere, Ruby ve RubyGems ile kod paylaşımı çok kolay bir şekilde gerçekleştirilmektedir.


#Birden fazla dosya gereksinimi :

Aslında herşeyin bir dosya içinde bulunması pek iyi olmamaktadır. Bunu aşağıdaki örnek ile açıklayalım :

'''
% cat lib/hola.rb
class Hola
  def self.hi(language = "english")
    translator = Translator.new(language)
    translator.hi
  end
end

class Hola::Translator
  def initialize(language)
    @language = language
  end

  def hi
    case @language
    when "spanish"
      "hola mundo"
    else
      "hello world"
    end
  end
end
'''

Bu dosya giderek kabarık bir hal almaktadır. Bunun yerine 'Translator' sınıfını ayrı bir dosyaya koymak görünüm ve okunabilirlik açısından yarar sağlamaktadır. Ayrıca hola.rb dosyası herşeyi kontrol etmektedir. Gem için olan diğer kodlar genelde lib dosyanın altında gem ile aynı ada sahip bir dizinin altında toplanmaktadır. 

'''
% tree
.
├── hola.gemspec
└── lib
    ├── hola
    │   └── translator.rb
    └── hola.rb
'''

Translator dosyası lib/hola.rb ye require ifadesi ile eklenerek kullanılabilir. Translator için yazılan kodda fazla bir değişiklik yapılmaz.

'''
% cat lib/hola/translator.rb
class Translator
  def initialize(language)
    @language = language
  end

  def hi
    case @language
    when "spanish"
      "hola mundo"
    else
      "hello world"
    end
  end
end
'''

'''
% cat lib/hola.rb
class Hola
  def self.hi(language = "english")
    translator = Translator.new(language)
    translator.hi
  end
end

require 'hola/translator'
'''

*Yeni bir dosya oluşumu gerçekleştiği için bunu gemspec dosyasına da bildirim niteliğinde eklemek gerekmektedir.

'''
% cat hola.gemspec
Gem::Specification.new do |s|
...
s.files       = ["lib/hola.rb", "lib/hola/translator.rb"]
...
end
----

*Bu yapılmazsa gem kurulumu bu dosyaları görmeden gerçekleşir.

Irb konsolunda oluşturulan gem'i deneyebilirsiniz.

'''
% irb -Ilib -rhola
irb(main):001:0> Hola.hi("english")
=> "hello world"
irb(main):002:0> Hola.hi("spanish")
=> "hola mundo"
'''

Gem'i çalıştırır iken '-Ilib' ifadesini kullanmak zorundasınız. Normalde RubyGems lib dosyasını kendisi kullanıma sunmaktadır ve bu sayede son kullanıcılar böyle bir problem ile karşı karşıya gelmemektedir. Kodu RubyGems dışından çalıştırıldığı zaman bu ayarlamaları siz yapmalısınız. Kodun içinden $LOAD_PATH ile bu işlemi gerçekleştirebilirsiniz ancak uygunluk açısından pek tavsiye edilmemiştir. 

Gem'e çok fazla dosya eklediyseniz, gemspec'teki 'files' dizisine bu dosyaların yollarını eklemeti unutmayınız. Bundan dolayı bazı Ruby geliştiricileri bu işlemleri bazı programlar ile otomatik hale getirmişlerdir. (  Hoe, Jeweler, Rake, Bundler)

Birden fazla dosya ekleme gösterilen yol ile birebir aynıdır. Gerekli olduğu zamanlar bu işlemi yapmanız tercih edilir. Bu sayede ileri zamanlarda gem'iniz üstünde yapılan değişiklerde fazla sorun yaşamazsınız.


#Çalıştırılabilir özelliği ekleme :

Gemler Ruby kütüphaneleri sağlamaktan başka kendileri çalışma yeteneğine sahiptir. rake ve prettify_json.rb bilinen ve kullanılanlardan bazılarıdır. Örnek olarak :

'''
% curl -s http://jsonip.com/ | \
  prettify_json.rb
{
  "ip": "24.60.248.134"
}
'''

Bir gem'in çalıştırılabilir olması için yapılan işlemler bir kaç basit adımdan ibarettir. Dosyayı 'bin' dosyasına ve gemspec listesine eklemek yeterli olacaktır. Örnek :


'''
% mkdir bin
% touch bin/hola
% chmod a+x bin/hola
'''

Çalıştırılabilir dosya sadece hangi dosya ile birlikte işlem göreceğini bilmesi için #! karakterlerini dosyanın bulunduğu dizin ile birlikte yazılması gerekmektedir. Örnek :

'''
% cat bin/hola
#!/usr/bin/env ruby

require 'hola'
puts Hola.hi(ARGV[0])
'''

Bu kod gem'i aktif hale getirip ilk satırı dil seçeneği ile beraber alıp gem'de çalışmasını sağlamaktır. Nasıl çalıştırıldığı aşağıda gösterilmektedir :

'''
% ruby -Ilib ./bin/hola
hello world

% ruby -Ilib ./bin/hola spanish
hola mundo
'''

Hola çalıştırılabilir dosyasını gemspec'e yazdıktan sonra gem'i sürüm değişikliğini de yaptıktan sonra yayınlayabilirsiniz.

'''
% head -4 hola.gemspec
Gem::Specification.new do |s|
  s.name        = 'hola'
  s.version     = '0.0.1'
  s.executables << 'hola'
'''

#Test Yazmak :


Gem test etmek çok önemlidir. Sadece size değil başkalarına da yazdığınız gem'in çalıştığını gösterir. Bir gem değerlendirilirken, Ruby geliştiricileri küçük bir testi bile kodun çalıştığına güvenmek için bir sebep olarak görmektedir.

Gemler test dosyalarının kendi bulunduğu konuma eklenmesini desteklemektedir. Bu sayede gem indirildiği zaman testler denenebilir.

İlk başta önemsiz gibi görünse de test için GemTesters adı verilen bir grup kurulup gemlerin hangi ortamlarda nasıl çalıştığını incelemektedirler. Kısacası, Gem'inizi test etmeyi unutmayın!

Test::Unit Ruby'nin test sistemidir. Ayrıca başka test sistemleri de bulunmaktadır. RSpec en uygunlarından biridir. Hangisi ile test yaptığınızın bir önemi yoktur, önemli olan kodunuzu test etmektir.

Test için Rakefile dosyası ve test dosyası eklemek gerekmektedir.

'''
% tree
.
├── Rakefile
├── bin
│   └── hola
├── hola.gemspec
├── lib
│   ├── hola
│   │   └── translator.rb
│   └── hola.rb
└── test
    └── test_hola.rb
'''

Rakefile dosyası test için basit otomasyonlar vermektedir.

'''
% cat Rakefile
require 'rake/testtask'

Rake::TestTask.new do |t|
  t.libs << 'test'
end

desc "Run tests"
task :default => :test
'''

Bu sayede 'rake test' i veya 'rake' kodu ile testleri çalıştırabiliriz. Aşağıda hola için basit bir test bulunmaktadır.

'''
% cat test/test_hola.rb
require 'test/unit'
require 'hola'

class HolaTest < Test::Unit::TestCase
  def test_english_hello
    assert_equal "hello world",
      Hola.hi("english")
  end

  def test_any_hello
    assert_equal "hello world",
      Hola.hi("ruby")
  end

  def test_spanish_hello
    assert_equal "hola mundo",
      Hola.hi("spanish")
  end
end
'''

Testleri çalıştırmak için;

'''
% rake test
(in /Users/qrush/Dev/ruby/hola)
Loaded suite
Started
...
Finished in 0.000736 seconds.

3 tests, 3 assertions, 0 failures, 0 errors, 0 skips

Test run options: --seed 15331
'''

#Kod dökümantasyonu :

Yazılan gemlerin çoğu dökümantasyon için RDoc kullanmaktadır. Nasıl RDoc yazılacağı ile alakalı fazlası ile kaynak bulunmaktadır. RDoc ile alakalı bir örnek aşağıda verilmiştir.

'''
# The main Hola driver
class Hola
  # Say hi to the world!
  #
  # Example:
  #   >> Hola.hi("spanish")
  #   => hola mundo
  #
  # Arguments:
  #   language: (String)

  def self.hi(language = "english")
    translator = Translator.new(language)
    puts translator.hi
  end
end
'''


Dökümantasyon için diğer güzel bir seçenek YARD'dır. Bir gem konulduğu zaman, RubyDoc.info dosyası otomatik olarak YARDocs dosyasını gemden üretmektedir.


