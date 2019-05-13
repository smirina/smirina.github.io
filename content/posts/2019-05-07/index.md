---
title: "Почему хорошие разработчики пишут плохие юнит тесты"
cover: "https://mtlynch.io/images/good-developers-bad-tests/cover.jpg"
author: "irina"
date: "2019-05-07"
category: "tech"
tags:
    - translation
    - programming
    - tests
---
*Этот пост - перевод статьи Майкла Линча [Why Good Developers Write Bad Unit Tests](https://mtlynch.io/good-developers-bad-tests/)*

Представьте, что вы наконец написали столько кода, что можете позволить себе домик у моря. Поздравляю! Вы нанимаете Питера Китинга, всемирно известного своими небоскребами архитектора, который оставляет вас в уверенности, что блестяще справится со строительством вашей приморской собственности.

Несколько месяцев спустя вы приезжаете на торжественное открытие. Ваш новый дом - пятиэтажный гигант из хромированной стали, стекла и бетона. Вы проходите через вращающиеся двери, оставляя за собой песчаный след. Видите стойку регистрации и кабинки лифтов. Наверху, в соседних офисных кубиклах - ваша спальня и три гостевые комнаты.

Питер Китинг, настоящий эксперт, не понимает, что вас огорчает. "Я следовал **всем** лучшим практикам", - говорит он в свою защиту. "Толщина стен три фута, потому что структурная целостность жизненно важна. Поэтому ваш дом лучше чем открытые ветру, наполненные светом соседние дома. Может у вас и нет панорамных окон с видом на океан, но я со всей уверенностью заверяю вас, что это не лучшая практика - это энергонеэффективно и отвлекает офисных сотрудников."

Слишком часто разработчики подходят к юнит тестам с тем же ошибочным мышлением. Механически применяют все "правила", на которые опираются в продуктовом коде, не задумываясь, подходят ли они для написания тестов. И в результате строят небоскреб на пляже.

## Код тестов - не такой же, как любой другой код

ОБычно код построен на абстракциях. Хороший код скрывает сложность в чистых (well-scoped) функциях и элегантной иерархии классов. Он позволяет читатющему легко ориентироваться в системе, погружаясь в детали или поднимаясь на более высокий уровень по мере необходимости.

Код тестов - совсем другой зверь. 

Test code is a different beast. Every layer of abstraction in a unit test makes it harder to understand. Tests are a diagnostic tool, so they should be as simple and obvious as possible.



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
