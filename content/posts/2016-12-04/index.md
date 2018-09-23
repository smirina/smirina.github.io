---
title: "Как писать сообщения коммитов"
cover: "http://imgs.xkcd.com/comics/git_commit.png"
author: "irina"
date: "2016-12-04"
category: "tech"
tags:
    - translation
    - programming
---
*Этот пост - перевод статьи Криса Бимса [How to Write a Git Commit Message](https://github.com/spring-projects/spring-framework/commits/e5f4b49?author=cbeams)*

## Вступление: почему важны сообщения коммитов.

Если посмотреть лог любого случайного репозитория, то в описании коммитов будет, в той или иной степени, беспорядок. Например, эти [шедевральные сообщения](https://github.com/spring-projects/spring-framework/commits/e5f4b49?author=cbeams) я оставил когда начал контрибьютить в Spring.

>     $ git log --oneline -5 --author cbeams --before "Fri Mar 26 2009"
>     
>     e5f4b49 Re-adding ConfigurationPostProcessorTests after its brief removal in r814. @Ignore-ing the testCglibClassesAreLoadedJustInTimeForEnhancement() method as it turns out this was one of the culprits in the recent build breakage. The classloader hacking causes subtle downstream effects, breaking unrelated tests. The test method is still useful, but should only be run on a manual basis to ensure CGLIB is not prematurely classloaded, and should not be run as part of the automated build.
>     2db0f12 fixed two build-breaking issues: + reverted ClassMetadataReadingVisitor to revision 794 + eliminated ConfigurationPostProcessorTests until further investigation determines why it causes downstream tests to fail (such as the seemingly unrelated ClassPathXmlApplicationContextTests)
>     147709f Tweaks to package-info.java files
>     22b25e0 Consolidated Util and MutableAnnotationUtils classes into existing AsmUtils
>     7f96f57 polishing

Упс. И сравните это с [более поздними](https://github.com/spring-projects/spring-framework/commits/5ba3db?author=philwebb) коммитами из того же репозитория:

>     $ git log --oneline -5 --author pwebb --before "Sat Aug 30 2014"
>     
>     5ba3db6 Fix failing CompositePropertySourceTests
>     84564a0 Rework @PropertySource early parsing logic
>     e142fd1 Add tests for ImportSelector meta-data
>     887815f Update docbook dependency and generate epub
>     ac8326d Polish mockito usage

Какие лучше читаются?


Первые совсем разные по длине и содержанию, последние - краткие и последовательные. Первые мы пишем не задумываясь, последние написаны целенаправленно.


Несмотря на то, что большинство репозиториев подтверждают это правило, есть и исключения. [Ядро Linux](https://github.com/torvalds/linux/commits/master) и [сам Git](https://github.com/git/git/commits/master) отличные тому примеры. Еще [Spring Boot](https://github.com/spring-projects/spring-boot/commits/master) или любой репозиторий, которым управлят [Тим Поп](https://github.com/tpope/vim-pathogen/commits/master).


Контрибьюторы этих репозиториев понимают, что хорошо продуманные сообщения коммитов - лучший способ донести контекст изменений до других разработчиков (и будущего себя). С помощью диффа можно выяснить *что* изменилось, но только сообщение коммита может внятно объяснить *почему*. Питер Хаттерер [рассуждает так](http://who-t.blogspot.ru/2009/12/on-commit-messages.html):

> Описывать заново контекст фрагмента кода - это расточительно. Мы не можем полностью избежать этого, поэтому наши усилия направлены на то, чтобы [сократить это](http://www.osnews.com/story/19266/WTFs_m) настолько, насколько это возможно. Сообщения коммитов снижают необходимость переустановления контекста и, в конечном итоге, показывают, *насколько хорошо разработчик умеет работать в команде*.


Если вы еще не задумывались о том, каким должно быть хорошее сообщение комммита - то это, вероятно, потому, что не тратили время на `git log` и похожие инструменты. Это замкнутый круг: история коммитов неструктурирована и непоследовательна, поэтому никто не тратит много времени на работу с ней. А потому, что никто о ней не заботится, она остается неструктурированной и непоследовательной.


Но если она ведется тщательно и аккуратно - это прекрасная и крайне полезная вещь. Настает время `git blame`, `revert`, `rebase`, `log`, `shortlog` и других подкоманд. Внезапно оказывается, что анализ чужих коммитов и пуллреквестов - это стоящая вещь, а причину изменений месячной (или годовой) давности понять не просто возможно, но и имеет практическое значение.


Долгосрочный успех проекта зависит, помимо всего прочего, от того, насколько удобно этот проект поддерживать. А у мейнтейнера едва ли есть инструменты мощнее, чем лог проекта. Стоит потратить время, чтобы разобраться, как за ним ухаживать. Поначалу это может вызвать затруднения, но вскоре станет привычкой, а в конечном итоге - источником продуктивности и поводом для гордости всех участников проекта.


В этой статье я обращаюсь только к самому базовому элементу поддержания порядка в истории коммитов: как писать индивидуальные сообщения коммитов. Есть и другие важные практики, такие как commit squashing - “склеивание” коммитов, про которые я сейчас не говорю. Возможно, сделаю это в будущих статьях.


В большинстве языков программирования есть устоявшиеся соглашения относительно идиоматического стиля, то есть именования, форматирования и тому подобного. Конечно, есть вариации, но большинство разработчиков согласны с тем, что лучше придерживаться одного стандарта.


Чтобы создать полезную историю изменений, подход команды к сообщениям коммитов не должен отличаться - стоит согласовать, как минимум, три вещей:

**Стиль.** Синтаксис разметки, отступы, грамматика, прописные буквы, пунктуация. Пропишите все это, даже то, что кажется очевидным (вам), и сделайте максимально простым. Тогда у вас будет согласованный лог, который *действительно читают*.

**Контент.** Какая информация должна быть в теле коммита (если оно есть)? А какая не должна?

**Метаданные.** Как следует ссылаться на ишьи, пуллреквесты и тому подобное?

К счастью, существуют устоявшиеся соглашения относительно того, что делает сообщение коммита хорошим. Некоторые подкомманды гита предполагают, что ты следуешь части этих правил. Не нужно изобретать велосипед. Просто следуйте этим семи правилам и станьте ближе к тому, чтобы коммитить как профессионал.

> Не забывайте,
[все](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
[это](https://www.git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#Commit-Guidelines)
[уже](https://github.com/torvalds/subsurface/blob/master/README#L82-109)
[было](http://who-t.blogspot.ru/2009/12/on-commit-messages.html)
[сказано](https://github.com/erlang/otp/wiki/writing-good-commit-messages)
[раньше](https://github.com/spring-projects/spring-framework/blob/30bce7/CONTRIBUTING.md#format-commit-messages)

1. Отделяйте заголовок от содержимого коммита пустой строкой
2. Уложите заголовок в 50 символов
3. Пишите заголовок с заглавной буквы
1. Не ставьте точку в конце заголовка
1. Используйте повелительное наклонение в заголовке
1. Ограничьте длину строк в сообщении 72 символами
1. В сообщении пишите что и почему сделано, а не как

Например:

    Обобщите изменения используя меньше 50 символов.

    Более подробные объяснения, если они нужны. Длина строки - примерно 72
    символа. В некоторых случаях первая строчка рассматривается как заголовок
    коммита, а все остальное - как тело. Эта пустая строка имеет решающее
    значение - иначе, если вы напишете их слитно, такие инструменты как log,
    shortlog и rebase могут сработать неправильно.

    Объясните проблему, которую решает коммит. Сфокусируйтесь на том, почему
     вы сделали изменения, а не на том, как (это видно из кода). Может есть
     побочные эффекты или последствия, которые не видно сразу? Здесь самое
     место рассказать о них.

    Следующий параграф идет после пустой строки.

     - Можно добавить список

     - Обычно элемент списка обозначают дефисом или звездочкой после
     пробела в начале строки, отделяя элементы друг от друга пустой
     строкой, но возможны варианты.  

    Если вы используете систему отслеживания задач, поставьте ссылки на нее:

    Resolves: #123
    See also: #456, #789



### 1. Разделяйте заголовок и содержимое коммита пустой строкой
Из документации [git commit](https://www.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion):


> Хотя это и не обязательно, неплохо начинать сообщения коммита с одной короткой (до 50 символов) строки со сжатым изложением изменений, после нее оставлять пустую строку и потом писать более подробное описание. Текст до первой пустой строки рассматривается в качестве заголовка и используется по всему гиту. Например, git-format-patch(1) превращает коммит в письмо и указывает заголовок как тему.


Для начала, не каждому коммиту нужны и заголовок, и описание. Иногда одной строки достаточно, особенно если коммит так прост, что пояснение не требуется. Например:

    Fix typo in introduction to user guide

И ничего больше не нужно. Если кому-то станет интересно, какая именно была опечатка, он сам посмотрит изменения, например, при помощи `git show`, `git diff` или `git log -p`.


Если вы коммитите что-то вроде этого в командной строке, коммит легко сделать с флагом `-m`.

    $ git commit -m"Fix typo in introduction to user guide"

Однако, когда нужны пояснения, стоит написать тело сообщения коммита. Например:

    Derezz the master control program

    MCP turned out to be evil and had become intent on world domination.
    This commit throws Tron's disc into MCP (causing its deresolution)
    and turns it back into a chess game.



Это уже не так-то просто закоммитить с помощью `-m` - тут понадобится подходящий редактор. Если у вас еще нет редактора, настроенного для работы с гитом из консоли, то стоит прочитать [вот эту часть](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration) из Pro Git
В любом случае, отделение заголовка окупается при просмотре полного лога. Он выглядит так:

    $ git log
    commit 42e769bdf4894310333942ffc5a15151222a87be
    Author: Kevin Flynn <kevin@flynnsarcade.com>
    Date:   Fri Jan 01 00:00:00 1982 -0200

     Derezz the master control program

     MCP turned out to be evil and had become intent on world domination.
     This commit throws Tron's disc into MCP (causing its deresolution)
     and turns it back into a chess game.

А теперь `git log --oneline`, который выводит только заголовок:

    $ git log --oneline
    42e769 Derezz the master control program

Или `git shortlog`, который группирует коммиты по авторам и для краткости показывает тоже только заголовок:

    $ git shortlog
    Kevin Flynn (1):
          Derezz the master control program

    Alan Bradley (1):
          Introduce security program "Tron"

    Ed Dillinger (3):
          Rename chess program to "MCP"
          Modify chess program
          Upgrade chess program

    Walter Gibbs (1):
          Introduce prototype chess program



Есть много других случаев в гите, когда используется разделение на заголовок и тело, но ни один из них не работает нормально без этой пустой строки.

### 2. Уложите заголовок в 50 символов

50 символов - это не жесткое ограничение, просто практическая рекомендация. С одной стороны, такая длина делает заголовки читаемыми, с другой - заставляет задуматься, как максимально четко объяснить коммит.

> Совет: Если вам с трудом удается кратко сформулировать изменения, возможно, для одного коммита их было слишком много. Стремитесь к [атомарным](https://www.freshconsulting.com/atomic-commits/) коммитам (а это тема для отдельного поста).

UI гитхаба в курсе этого правила и предупреждает вас, если вы превышаете лимит.

![screenshot](http://i.imgur.com/zyBU2l6.png)

И переносит текст дальше 69 символов за многоточие.

![screenshot](http://i.imgur.com/27n9O8y.png)

Так что ориентируйтесь на 50 символов, но не выходите за 69

### 3. Пишите заголовок с заглавной буквы


Здесь ничего сложного. Начинайте заголовок с прописной (заглавной, большой) буквы.

Например:

<span style="color:green">- Accelerate to 88 miles per hour</span>

А не

<span style="color:red">- accelerate to 88 miles per hour</span>



### 4. Не ставьте точку в конце заголовка


Завершающие знаки препинания не важны в заголовке, а вот место имеет значение, когда вы пытаетесь уложиться в 50 символов.

Например

<span style="color:green">- Open the pod bay doors</span>

Вместо

<span style="color:red">- Open the pod bay doors.</span>



### 5. Используйте повелительное наклонение


*Повелительное наклонение* - это когда “говорящий или пишущий дает команду или инструкцию”.

Например

- Уберись в комнате  
- Закройте дверь  
- Вынеси мусор  

Все семь правил, которые вы сейчас читаете, написаны в повелительном наклонении (“Используйте повелительное наклонение”, и т.п.).

Это может звучать немного грубо, поэтому обычно мы так не говорим. Но это именно то, что нужно для заголовка коммита. Одна из причин - **гит сам использует повелительное наклонение, когда создает коммит от вашего имени**.

Например, по умолчанию при `git merge` сообщение выглядит так

    Merge branch 'myfeature'

а при `git revert`

    Revert "Add the thing with the stuff"
    This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.

или при клике на кнопку “Merge” в пуллреквесте гитхаба

    Merge pull request #123 from someuser/somebranch

Так что когда вы пишите сообщение коммита в повелительном наклонении, вы следуете внутреннему соглашению гита:

<span style="color:green">- Refactor subsystem X for readability</span>
<span style="color:green">- Update getting started documentation</span>
<span style="color:green">- Remove deprecated methods</span>
<span style="color:green">- Release version 1.0.0</span>


Сначала писать так может быть неловко. Обычно мы используем *изъявительное* наклонение (оно сообщает о фактах) и поэтому сообщения к коммитам выглядят так:

<span style="color:red">- Fixed bug with Y</span>

<span style="color:red">- Changing behavior of X</span>


Иногда описывается их содержимое

<span style="color:red">- More fixes for broken stuff</span>

<span style="color:red">- Sweet new API methods</span>


Чтобы избежать недоразумений, есть простое правило:

**Правильно сформулированным заголовком коммита всегда можно продолжить фразу:**
- If applied, this commit will (и здесь заголовок коммита)

Например

- If applied, this commit will <span style="color:green">refactor subsystem X for readability</span>
- If applied, this commit will <span style="color:green">update getting started documentation</span>
- If applied, this commit will <span style="color:green">remove deprecated methods</span>
- If applied, this commit will <span style="color:green">release version 1.0.0</span>
- If applied, this commit will <span style="color:green">merge pull request #123 from user/branch</span>

А с другим наклонением уже не работает:

If applied, this commit will fixed bug with Y
If applied, this commit will changing behavior of X
If applied, this commit will more fixes for broken stuff
If applied, this commit will sweet new API methods

>На заметку: повелительное наклонения важно только в заголовке. В теле пишите так, как удобнее


### 6. Ограничьте длину строки сообщения 72 символами


Git никак не форматирует сообщения автоматически, поэтому не забывайте про правый край и переносите строки вручную. Совет - остановиться на 72 символах, чтобы оставить гиту достаточно места для индентации и не превысить, в конечном счете, 80 знаков.

Тут поможет хороший текстовый редактор. Легко настроить, например, Vim, который будет ограничивать длину строки сообщения. А IDE традицинно плохо справляются с автоматическим форматированием текста (хотя в IntelliJ IDEA в последних версиях, наконец, [стало](https://youtrack.jetbrains.com/issue/IDEA-53615) [получше](https://youtrack.jetbrains.com/issue/IDEA-53615#comment=27-448299) [с этим](https://youtrack.jetbrains.com/issue/IDEA-53615#comment=27-446912)).

### 7. В сообщении пишите что и почему сделано, а не как


Этот [коммит из Bitcoin Core](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6) отличный пример объяснения, что и почему изменилось.

    commit eb0b56b19017ab5c16c745e6da39c53126924ed6
    Author: Pieter Wuille <pieter.wuille@gmail.com>
    Date:   Fri Aug 1 22:57:55 2014 +0200

       Simplify serialize.h's exception handling

       Remove the 'state' and 'exceptmask' from serialize.h's stream
       implementations, as well as related methods.

       As exceptmask always included 'failbit', and setstate was always
       called with bits = failbit, all it did was immediately raise an
       exception. Get rid of those variables, and replace the setstate
       with direct exception throwing (which also removes some dead
       code).

       As a result, good() is never reached after a failure (there are
       only 2 calls, one of which is in tests), and can just be replaced
       by !eof().

       fail(), clear(n) and exceptions() are just never called. Delete
       them.

Посмотрите [дифф целиком](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6) и только подумайте, сколько времени автор сэкономил текущим и будущим разработчикам, потратив время, чтобы объяснить контекст здесь и сейчас. Не сделай он этого, мы могли бы этого никогда и не понять.

В большинстве случаев вы можете объяснить, почему было сделано изменение. Код требует пояснений (а сложный код - комментариев внутри). Сфокусируйтесь на том, почему вы сделали эти изменения, в первую очередь - как все работало до изменений (и что с этим было не так), как работает сейчас и почему вы решили исправить это именно так.

Возможно, будущим мейнтейнером, благодарным за это, станете вы сами!

## Советы

### Полюбите командную строку. Расстаньтесь с IDE.

По многим причинам, среди которых подкоманды гита, подружиться с командной строкой - мудрое решение. Git безумно мощный, IDE - тоже, но каждый по своему. Я использую одну IDE каждый день (IntelliJ IDEA) и использовал другие (Eclipse), но я никогда не видел такой интеграции IDE и гита, которая по простоте и мощности приблизилась бы к командной строке (как только вы с ней подружитесь).

Некоторые связанные с гитом возможности IDE бесценны, вроде вызова `git rm` при удалении файла или правильной работы с гитом при переименовании. Но все разваливается, когда вы начинаете коммитить, мержить, ребейзить или делать сложный анализ истории изменений через IDE.

Когда дело доходит до полноценного использования всех возможностей гита - нужна командная строка.

Помните, что если вы используете bash или zsh, то существуют [скрипты автодополнения](https://git-scm.com/book/en/v1/Git-Basics-Tips-and-Tricks), благодаря которым не надо запоминать все подкомманды и флаги.

### Прочитайте Pro Git

Книгу [Pro Git](https://git-scm.com/book/ru/v2) можно бесплатно прочитать онлайн. Воспользуйтесь этим!
