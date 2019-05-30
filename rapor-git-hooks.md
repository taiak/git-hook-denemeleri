# Rapor Edilecek Hooklar

## pre-commit
`git-commit` tarafından çağrılır. Eğer kullanılmaması gerekirse `--no-verify` seçeneği ile komut çalıştırılmadan geçilebilir. Parametre almaz. Yorum yapılmadan ve hatta yorum logları tutulmadan önce çalıştırılır. Başarılı sayılabilmesi için sonuç değerinin `0` olması gerekir.

~~~ruby
#!/usr/bin/env ruby

ARGV
import sys, os
commit_msg_filepath = sys.argv[1]
with open(commit_msg_filepath, 'w') as f:
f.write("# Please include a useful commit message!")
~~~

~~~ruby
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
`pre-commit` tetikleyicisinden sonra çağrılır. Tetiklendiği zaman bir text editörü içerisinde yorum mesajı üretir. Sıkıştırılmış veya birleştirilmiş yorum mesajlarını değiştirmek için bu tetikleyici kullanılır. Başarılı sonuç dönmesi için `0` dönmesi gerekir. Eğer sıfırdan farklı bir değer dönerse yani başarısız sonuç dönerse `git commit` yorumu iptal edilir.

`prepate-commit-msg` scripti 1-3 aralığında parametre alır. 
1. Basılacak mesajı içeren geçici dosyanın adı. Bu dosya değiştirilerek onay mesajı değiştirilebilir

2. Yorum türü. `message`, `template`, `merge`, `squash` türlerinde olabilir. `message` türü için -m veya -F parametresiyle, `template` için -T parametresiyle kullanılır. Yorumun türü `merge` veya `squash` ise bunlar da kendi içeriğinde kullanılmış olur.

3. SHA1 türünde uygun yorum hashi de alabilmektedir. Bu hashi kullaabilmek için `-c`, `-C` veya `--ammend` parametrelerini girmek gerekmektedir.

~~~ruby
#!/usr/bin/env ruby
confy
message = ARGV[0]
branch = %x[git rev-parse --abbrev-ref HEAD]

match = /^feature\/(\w+-\d+)-/.match(branch)

# This makes it easy to extend this hook to provide multiple variables that can then be
# used in the commit message template.
variables = {
	'ISSUE' => (match == nil ? nil : "#{match[1]}: ")
}

text = File.read(message)
 
# Simply replace all the placeholders of the form "$(SOME_NAME)" with the value
# provided in the variables hash.
text.gsub!(/\$\(([^)]+)\)/) do |m|
	name = $1
	if (variables.include?(name))
		variables[name]
	end
end

File.write(message, text)
~~~

İşlem günlüğü iletisini hazırlamak için örnek bir `hook` komut dosyasıdır. `git commit` olarak adlandırılan dosyanın ismiyle commit mesajı, ardından da commitin açıklamasıyla mesajın kaynağını içerir. Bu `hook`'un amacı commiti mesaj dosyasını düzenlemektir. `Hook` sıfır olmayan bir durumla başarısız olursa, commit iptal edildi.


The second includes the output of "git diff --name-status -r"
into the message, just before the "git status" output.  It is
commented because it doesn't cope with --amend or with squashed
commits.

The third example adds a Signed-off-by line to the message, that can
still be edited.  This is rarely a good idea.

## pre-applypatch
This hook is invoked by git-am[1]. It takes no parameter, and is invoked after the patch is applied, but before a commit is made.

If it exits with non-zero status, then the working tree will not be committed after applying the patch.

It can be used to inspect the current working tree and refuse to make a commit if it does not pass certain test.

The default pre-applypatch hook, when enabled, runs the pre-commit hook, if the latter is enabled.

Bu kanca `git-am` tarafından çağrılır. Parametre almaz ve düzeltme eki uygulandıktan sonra ancak bir işlem yapılmadan önce çağrılır. 
Eğer sonuç sıfır dönmezse çalışma ağacı yamayı uyguladıktan sonra yapılmayacaktır. Mevcut çalışma ağacını denetlemek ve belirli bir testi geçemezse bir taahhüt vermeyi reddetmek için kullanılabilir. Etkinleştirildiğinde, varsayılan ön uygulama kancası, ikincil etkinse ön işleme kancasını çalıştırır.

ikinci yazı
An example hook script to verify what is about to be committed
by applypatch from an e-mail message.

The hook should exit with non-zero status after issuing an
appropriate message if it wants to stop the commit.

To enable this hook, rename this file to "pre-applypatch".



## applypatch-msg
This hook is invoked by git-am[1]. It takes a single parameter, the name of the file that holds the proposed commit log message. Exiting with a non-zero status causes git am to abort before applying the patch.

The hook is allowed to edit the message file in place, and can be used to normalize the message into some project standard format. It can also be used to refuse the commit after inspecting the message file.

The default applypatch-msg hook, when enabled, runs the commit-msg hook, if the latter is enabled.

> ikinci yazı
> An example hook script to check the commit log message taken by
> applypatch from an e-mail message.
> The hook should exit with non-zero status after issuing an
> appropriate message if it wants to stop the commit.  The hook is
> allowed to edit the commit message file.
> To enable this hook, rename this file to "applypatch-msg".


## commit-msg
An example hook script to check the commit log message.
Called by "git commit" with one argument, the name of the file
that has the commit message.  The hook should exit with non-zero
status after issuing an appropriate message if it wants to stop the
commit.  The hook is allowed to edit the commit message file.

To enable this hook, rename this file to "commit-msg".
 
Uncomment the below to add a Signed-off-by line to the message.
Doing this in a hook is a bad idea in general, but the prepare-commit-msg
hook is more suited to it.

SOB=$(git var GIT_AUTHOR_IDENT | sed -n 's/^\(.*>\).*$/Signed-off-by: \1/p')
grep -qs "^$SOB" "$1" || echo "$SOB" >> "$1"
 
This example catches duplicate Signed-off-by lines.

> ikinci yazı:
> This hook is invoked by git-commit[1] and git-merge[1], and can be bypassed with the --no-verify option. It takes a single > parameter, the name of the file that holds the proposed commit log message. Exiting with a non-zero status causes the command to > abort.
> 
> The hook is allowed to edit the message file in place, and can be used to normalize the message into some project standard format. > It can also be used to refuse the commit after inspecting the message file.
> 
> The default commit-msg hook, when enabled, detects duplicate "Signed-off-by" lines, and aborts the commit if one is found.

## fsmonitor-watchman
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