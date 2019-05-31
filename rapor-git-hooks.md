# Rapor Edilecek Hooklar

## pre-commit
`git-commit` tarafından çağrılır. Eğer kullanılmaması gerekirse `--no-verify` seçeneği ile komut çalıştırılmadan geçilebilir. Parametre almaz. Yorum yapılmadan ve hatta yorum logları tutulmadan önce çalıştırılır. Başarılı sayılabilmesi için sonuç değerinin `0` olması gerekir.

~~~ruby
  #!/usr/bin/env ruby
  
  # yasaklı klasörününe ekleme yapılmamasını engelleyen betik
  # eklenen dizin isimlerini al
  
  folder = 'yasakli/'
  
  files = %x[git diff --cached --name-status | grep -P "^A\t#{folder}"]
  
  # eşleşme yoksa doğru olarak dön
  exit 0 if files.empty?
  
  # eklenen dosya isimlerini ayıkla
  yasaklilar = files.split("\n").map do |file_name| 
    file_name.split("#{folder}").last
  end
  
  puts "Commit engellendi!!"
  puts "Aşağıdaki dosyaları silip tekrar deneyiniz:"
  yasaklilar.each { |f| puts " * #{folder}#{f}" }
  exit 1
~~~

~~~sh
  #!/bin/sh
  # verilen dizinden dosya silinmesini engelleyen sh betiği
  folder=dont_delete
  
  res=$(git diff --cached --name-status | grep -P "D\t${folder}/" | sed s_^D[[:space:]]\[^/]*\/__)
  
  if test -z "$res"; then exit 0; fi
  
  echo "HATA: $folder dizininden aşağıdaki dosyalar silinmeye çalışılıyor:"
  for i in $res; do; echo " -> $i"; done
  
  echo "Commit engellendi!"
  exit 1
~~~

## post-commit
`git-commit` tarafından çağırılır. Parametre almaz, `commit` yapıldıktan sonra çalıştırılır. Uyarı amaçlıdır ve `commit`'in sonucunu etkilemez.

~~~ruby
#!/usr/bin/env ruby

puts '---------------HATIRLATMA---------------'
puts "Yaptığınız commit'i push etmeyi unutmayın!"
puts '----------------------------------------'
~~~

~~~sh
#!/bin/sh

echo '######## UYARI ########'
echo '-> Bu projede çalışanların dikkatine: '
echo '   * ikinci bir emre kadar izinleriniz iptal edilmiştir'
~~~

## prepare-commit-msg
`pre-commit` `hook`'undan sonra çağrılır. Tetiklendiği zaman bir text editörü içerisinde yorum mesajı üretir. Başarılı sonuç dönmesi için `0` dönmesi gerekir. Eğer sıfırdan farklı bir sonuç dönerse `git commit` komutu iptal edilir. `commit`'lerde otomatik olarak oluşturulması istenen mesajlar için sıkça kullanılır.

`prepate-commit-msg` scripti 1-3 aralığında parametre alır. 
1. Basılacak mesajı içeren geçici dosyanın adı. Bu dosya değiştirilerek onay mesajı değiştirilebilir

2. `commit` türü. `message`, `template`, `merge`, `squash` türlerinde olabilir. `message` türü için -m veya -F parametresiyle, `template` için -T parametresiyle kullanılır. 

3. SHA1 türünde uygun yorum hashi de alabilmektedir. Bu hashi kullanabilmek için `-c`, `-C` veya `--ammend` parametrelerini girmek gerekmektedir.

~~~ruby
  #!/usr/bin/env ruby
  # commit dosyasında yorum satırı hariç satırların md5
  # değerini alarak yorumun başına ekler 
  require 'digest'
  
  commit_msg_file = ARGV[0]
  
  text = File.readlines(commit_msg_file).map { |l| 
    l.gsub!(/^[#].*/, '') 
  }.join
  
  text = "md5: " + Digest::MD5.hexdigest(text) + "\n" + text
  
  File.write(commit_msg_file, text)
~~~

~~~ruby
  #!/usr/bin/env ruby
  # Commit'in sonuna istenilenilen bir sözü ekler
  msg_file = ARGV[0]
  content  = File.read(msg_file)
  QUOTE_OF_DAY = '"Yarını iyileştirmenin tek yolu bugün neyi yanlış yaptığını bilmektir!" ~Robin Sahrman~'.freeze

  content += "\n" + QUOTE_OF_DAY
  
  File.write(msg_file, content)
~~~

## pre-applypatch

`git-am` tarafından çağrılır. Parametre almaz. Düzeltme eki uygulandıktan sonra ancak bir işlem yapılmadan önce çağrılır. Eğer dönen sonuç sıfır değilse, çalışma ağacı yapılan yamalardan sonra `commit` edilmez. Mevcut çalışma ağacını denetlemek ve geçerli testi geçemezse bir commiti reddetmek için kullanılabilir. 

## applypatch-msg

`git-am` tarafından çağrılır. Önerilen işlem günlüğü iletisini tutan dosyanın adı tek bir parametre alır. Sıfır olmayan sonuç dönerse yama uygulamadan önce `git am`'in iptaline sebep olur.

## sendemail-validate

`git-send-email` tarafından çağrılır. Gönderilecek e-postayı tutan dosyanın adı tek bir parametre alır. Sıfır olmayan bir durumdan çıkmak, git gönder e-postasının herhangi bir e-posta göndermeden önce iptal edilmesine neden olur.

## commit-msg

Bu kanca `git-commit` veya `git-merge` tarafından çağrılır. `--no-verify` seçeneği ile geçilebilir. Önerilen `commit` log mesajı tarafından tek parametre ile çalıştırılır. Sıfır harici bir değer ile sonlandırılması durumunda `commit` iptal edilir.

~~~ruby
  #!/usr/bin/env ruby
  
  # yasakli kelimeleri içeren commit mesajlarını engelleyen ruby betiği
  dosya_adi         = ARGV[0]
  dosya             = File.read(dosya_adi)
  yasakli_kelimeler = ["yasakli", "kelime", "at"]
  
  yasakli = nil
  yasakli_kelimeler.any? { |v| yasakli = v if icerik.include? v } 
  
  if yasakli
  	puts "HATA: Herkese açık bir projede \"#{yasakli}\" gibi kelimelerin kullanılması uygun değildir!"
  	puts "Commit iptal ediliyor!"
  	exit 1
  end
~~~

~~~ruby
  #!/usr/bin/env ruby
  
  # committeki yasakli kelimeleri sansürleyen ruby betiği
  file_name  = ARGV[0]
  content    = File.read(file_name)

  forbidden_words = ["yasak1", "yasak2"]
  
  forbidden_words.each { |f_word| content.gsub!(f_word, '*' * f_word.size) }
  
  File.write(file_name, content)
~~~


## fsmonitor-watchman

Bu seçenek, `core.fsmonitor` yapılandırma seçeneği .git / hooks / fsmonitor-watchman olarak ayarlandığında çağrılır. Bir sürüm (şu anda 1) ve 1 Ocak 1970 gece yarısından bu yana geçen nanosaniye cinsinden geçen süre iki argüman alıyor.

Kanca bir sürümden (şu anda 1) ve bir dize olarak biçimlendirilmiş nanosaniye cinsinden bir süreden geçirilir ve verilen zamandan beri değiştirilmiş olan tüm dosyaları saklar. Yollar, çalışan ağacın köküne göre olmalı ve tek bir NUL ile ayrılmalıdır.

An example hook script to integrate Watchman
(https://facebook.github.io/watchman/) with git to speed up detecting
new and modified files.

The hook is passed a version (currently 1) and a time in nanoseconds
formatted as a string and outputs to stdout all files that have been
modified since the given time. Paths must be relative to the root of
the working tree and separated by a single NUL.

To enable this hook, rename this file to "query-watchman" and set
'git config core.fsmonitor .git/hooks/query-watchman'


## pre-push
`git-push` tarafından çağırılır. 

An example hook script to verify what is about to be pushed.  Called by "git
push" after it has checked the remote status, but before anything has been
pushed.  If this script exits with a non-zero status nothing will be pushed.

This hook is called with the following parameters:

$1 -- Name of the remote to which the push is being done
$2 -- URL to which the push is being done

If pushing without using a named remote those arguments will be equal.

Information about the commits which are being pushed is supplied as lines to
the standard input in the form:

  <local ref> <local sha1> <remote ref> <remote sha1>

This sample shows how to prevent push of commits where the log message starts
with "WIP" (work in progress).

## pre-rebase
Copyright (c) 2006, 2008 Junio C Hamano

The "pre-rebase" hook is run just before "git rebase" starts doing
its job, and can prevent the command from running by exiting with
non-zero status.

The hook is called with the following parameters:

$1 -- the upstream the series was forked from.
$2 -- the branch being rebased (or empty when rebasing the current branch).

This sample shows how to prevent topic branches that are already
merged to 'next' branch from getting rebased, because allowing it
would result in rebasing already published history.

## pre-receive
Push seçeneklerinden yararlanmak için örnek bir kanca betiği.
Örnek, `echoback =` ile başlayan tüm itme seçeneklerini yansıtır.
ve `reject` basma seçeneği kullanıldığında tüm basmaları reddeder.

~~~ 
  //TODO: add command
~~~

## update

An example hook script to block unannotated tags from entering.
Called by "git receive-pack" with arguments: refname sha1-old sha1-new

To enable this hook, rename this file to "update".

Config
------
hooks.allowunannotated
  This boolean sets whether unannotated tags will be allowed into the
  repository.  By default they won't be.
hooks.allowdeletetag
  This boolean sets whether deleting tags will be allowed in the
  repository.  By default they won't be.
hooks.allowmodifytag
  This boolean sets whether a tag may be modified after creation. By default
  it won't be.
hooks.allowdeletebranch
  This boolean sets whether deleting branches will be allowed in the
  repository.  By default they won't be.
hooks.denycreatebranch
  This boolean sets whether remotely creating branches will be denied
  in the repository.  By default this is allowed.

## post-update
An example hook script to prepare a packed repository for use over
dumb transports.

To enable this hook, rename this file to "post-update".