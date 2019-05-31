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

## pre-receive
`git-receive-pack` tarafından çağrılır.

Bu kanca`git` referensları yolladığı veya güncellediğinde ve deposundaki referansları güncellediğinde `git-pack-pack` tarafından çağrılır.

This hook is invoked by git-receive-pack[1] when it reacts to git push and updates reference(s) in its repository. Just before starting to update refs on the remote repository, the pre-receive hook is invoked. Its exit status determines the success or failure of the update.

This hook executes once for the receive operation. It takes no arguments, but for each ref to be updated it receives on standard input a line of the format:

~~~sh
<old-value> SP <new-value> SP <ref-name> LF
~~~

where <old-value> is the old object name stored in the ref, <new-value> is the new object name to be stored in the ref and <ref-name> is the full name of the ref. When creating a new ref, <old-value> is 40 0.

~~~ruby
  #!/bin/bash
  # check each branch being pushed

  echo "pre-receive HOOK"

  while read old_sha new_sha refname
  do
    if git diff "$old_sha" "$new_sha" | grep -qE '^\+(<<<<<<<|>>>>>>>)'; then
      echo "Conflict'e sebep olacak işaretler bulundu $(basename "$refname")."
      git diff "$old_sha" "$new_sha" | grep -nE '^\+(<<<<<<<|>>>>>>>)'
      exit 1
    fi
  done
~~~

~~~bash
  #!/bin/bash
  # check each branch being pushed
  while read old_sha new_sha refname
  do
    if git diff "$old_sha" "$new_sha" | grep -qE '^\+.*\s+$'; then
      echo "HATA: Satır sonunda boşluk karakteri var!"
      git diff "$old_sha" "$new_sha" | grep -nE '^\+.*\s+$'
      exit 1
    fi
  done
~~~

## update
`pre-receive`'den sonra `git receive-pack` tarafından "ref adı, eski-sha1, yeni-sha1" değerleriyle çağırılır. Uzaktak, depo ref'i güncellemeden önce `update hook` çalıştırılır. Eğer sıfır dönerse ref güncellemesi başarısız sayılır ve iptal edilir.

~~~ruby
  #!/usr/bin/env ruby
  branch      = ARGV[0]
  eski_commit = ARGV[1]
  yeni_commit = ARGV[2]

  if eski_commit == yeni_commit
    puts "HATA: #{branch}, içerisinde son commit isimleri aynı: #{eski_commit}"
    puts "İşlem başarısız.."
    exit 1 
  end
~~~

~~~ruby
  #!/usr/bin/env ruby
  branch      = ARGV[0]
  eski_commit = ARGV[1]
  yeni_commit = ARGV[2]

  puts "#{branch}, #{eski_commit}'ten #{yeni_commit}'e taşınıyor..."
~~~

## post-update

An example hook script to prepare a packed repository for use over
dumb transports.

To enable this hook, rename this file to "post-update".

## fsmonitor-watchman
The hook should output to stdout the list of all files in the working directory that may have changed since the requested time. The logic should be inclusive so that it does not miss any potential changes. The paths should be relative to the root of the working directory and be separated by a single NUL.


The exit status determines whether git will use the data from the hook to limit its search. On error, it will fall back to verifying all files and folders.

-----------------------
`core.fsmonitor` yapılandırma seçeneği .git/hooks/fsmonitor-watchman olarak ayarlandığında çağırılır. İki parametre alır, birinci parametre versiyon bilgisi, ikinci parametre ise zaman bilgisidir. Zaman bilgisi 1 Ocak 1970'den başlayarak şimdiye kadar geçen nanosaniyeler şeklinde tutulur. Çalışma yolları, çalışma dizinin köküne göre olmalı ve tek bir NUL ile ayrılmalıdır.

Çıkış durumu, git'i aramasını sınırlamak için kancadaki verileri kullanıp kullanmayacağını belirler. Hata durumunda, tüm dosya ve klasörleri doğrulamak için geri düşecek.




An example hook script to integrate Watchman
(https://facebook.github.io/watchman/) with git to speed up detecting
new and modified files.

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
