# Домашнее задание к занятию «2.4. Инструменты Git»

1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.
    * хеш - `aefead2207ef7e2aa5dc81a34aedf0cad4c32545`
    * комментарий - `Update CHANGELOG.md`
1. Какому тегу соответствует коммит `85024d3`?
    * `v0.12.23`
1. Сколько родителей у коммита `b8d720`? Напишите их хеши.
    * два родителя
        * `56cd7859e05c36c06b56d013b55a252d0bb7e158`
        * `9ea88f22fc6269854151c571162c5bcf958bee2b`
1. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами  v0.12.23 и v0.12.24.
    * hash = `b14b74c4939dcab573326f4e3ee2a62e23e12f89` message =  \[Website\] vmc provider links
    * hash = `3f235065b9347a758efadc92295b540ee0a5e26e` message =  Update CHANGELOG.md
    * hash = `6ae64e247b332925b872447e9ce869657281c2bf` message =  registry: Fix panic when server is unreachable
    * hash = `5c619ca1baf2e21a155fcdb4c264cc9e24a2a353` message =  website: Remove links to the getting started guide's old location
    * hash = `06275647e2b53d97d4f0a19a0fec11f6d69820b5` message =  Update CHANGELOG.md
    * hash = `d5f9411f5108260320064349b757f55c09bc4b80` message =  command: Fix bug when using terraform login on Windows
    * hash = `4b6d06cc5dcb78af637bbb19c198faff37a066ed` message =  Update CHANGELOG.md
    * hash = `dd01a35078f040ca984cdd349f18d0b67e486c35` message =  Update CHANGELOG.md
    * hash = `225466bc3e5f35baa5d07197bbc079345b77525e` message =  Cleanup after v0.12.23 release
1. Найдите коммит в котором была создана функция `func providerSource`, ее определение в коде выглядит 
так `func providerSource(...)` (вместо троеточего перечислены аргументы).
    * commit 8c928e83589d90a031f811fae52a81be7153e82f
    * Author: Martin Atkins <mart@degeneration.co.uk>
    * Date:   Thu Apr 2 18:04:39 2020 -0700
1. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.
    * функция была создана в комминте с хешем `8364383c359a6b738a436d1b7745ccdce178df47` и больше не изменялась
1. Кто автор функции `synchronizedWriters`? 
    * Author: Martin Atkins <mart@degeneration.co.uk>