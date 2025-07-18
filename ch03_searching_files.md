# Глава 3. Поиск файлов

Цель этой главы — дать вам представление о том, как быстро искать в Vim. Возможность быстрого поиска — отличный способ повысить производительность Vim. Когда я понял, как быстро искать файлы, я переключился на постоянное использование Vim.

Эта глава разделена на две части: как искать без плагинов и как искать с помощью плагина [fzf.vim](https://github.com/junegunn/fzf.vim). Давайте начнем!

## Открытие и редактирование файлов

Чтобы открыть файл в Vim, вы можете использовать `:edit`.

```
:edit file.txt
```

Если `file.txt` существует, Vim откроет буфер `file.txt`. Если `file.txt` не существует, будет создан новый буфер для `file.txt`.

Автодополнение с помощью `<Tab>` работает и с `:edit`. Например, если ваш файл находится в директории контроллера [Rails](https://rubyonrails.org/) *a*pp *c*ontroller *u*sers controller `./app/controllers/users_controllers.rb`, вы можете использовать `<Tab>` для быстрого дополнения:

```
:edit a<Tab>c<Tab>u<Tab>
```

`:edit` принимает аргументы с универсальными символами. `*` соответствует любому файлу в текущем каталоге. Если вы ищете файлы с расширением `.yml` только в текущей директории:

```
:edit *.yml<Tab>
```

Vim выдаст вам список всех файлов `.yml` в текущей директории для выбора.

Вы можете использовать `**` для рекурсивного поиска. Если вы хотите найти все файлы `*.md` в вашем проекте, но не уверены, в каких каталогах, вы можете сделать это:

```
:edit **/*.md<Tab>
```

`:edit` может быть использован для запуска `netrw`, встроенного файлового менеджера Vim. Для этого передайте `:edit` в аргумент директорию вместо файла:

```
:edit .
:edit test/unit/
```

## Поиск файлов с помощью Find

Вы можете найти файлы с помощью `:find`. Например:

```
:find package.json
:find app/controllers/users_controller.rb
```

Автодополнение также работает с `:find`:

```
:find p<Tab>                " to find package.json
:find a<Tab>c<Tab>u<Tab>    " to find app/controllers/users_controller.rb
```

Вы можете заметить, что `:find` выглядит как `:edit`. В чем разница?

## Find и Path

Разница в том, что `:find` находит файл в `path`, а `:edit` нет. Давайте узнаем немного о `path`. Как только вы научитесь изменять свои path, `:find` может стать мощным инструментом поиска. Чтобы проверить свой path, выполните:

```
:set path?
```

По умолчанию ваш path, вероятно, выглядит так:

```
path=.,/usr/include,,
```

- `.` означает поиск в директории открытого в данный момент файла.
- `,` означает поиск в текущей директории.
- `/usr/include` это типичная директория для заголовочных файлов библиотек C.

Первые два важны в нашем контексте, а третий пока можно проигнорировать. Суть в том, что вы можете изменить свой path, по которому Vim будет искать файлы. Предположим, что структура вашего проекта выглядит следующим образом:

```
app/
  assets/
  controllers/
    application_controller.rb
    comments_controller.rb
    users_controller.rb
    ...
```

Еслы вы хотите перейти к `users_controller.rb` из корневой директории, вам придется перебрать несколько директорий (и много раз нажать Tab). Часто при работе с фреймворком вы проводите 90% своего времени в отдельный директории. В этой ситуации вас интересует только переход в директорию `controllers/` с наименьшим количеством нажатий клавиш. Настройка `path` может помочь вам в этом.

Вам нужно добавить `app/controllers/` в текущий `path`. Вот как вы можете сделать это:

```
:set path+=app/controllers/
```

Теперь, когда ваш path обновлен, когда вы введете `:find u<Tab>`, Vim будет искать в директории `app/controllers/` файлы начинающиеся с "u".

Если директория `controllers/` имеет поддиректории, например `app/controllers/account/users_controller.rb`, Vim не найдет `users_controllers`. Вместо этого вам нужно добавить `:set path+=app/controllers/**` чтобы автодополнение находило `users_controller.rb`. Великолепно! Теперь вы можете найти `user_controller.rb` с помощью одного нажатия Tab вместо трех.

Возможно теперь, вы думаете о том, чтобы добавить директорию проекта целиком, чтобы при нажатии `tab`, Vim искал этот файл везде, например:

```
:set path+=$PWD/**
```

`$PWD` это текущая рабочая директория. Если вы попытаетесь добавить весь свой проект в `path` в надежде сделать все файлы доступными при нажатии `tab` это и может сработать для небольшого проекта, но это значительно замедлит ваш поиск, при большом количестве файлов. Я рекомендую добавлять в `path` наиболее посещаемые файлы / директории.

Вы можете добавить `set path+={ваш-путь-тут}` в вашем vimrc. Обновление `path` занимает всего несколько секунд, и это может сэкономить вам много времени.

## Поиск в файлах с помощью Grep

Если вам нужно найти что-то в файлах (например какую-либо фразу), вы можете использовать grep. У Vim есть два способа сделать это:

- Внутренний grep (`:vim`. Да, это пишется как `:vim`. Это сокращение от `:vimgrep`).
- Внешний grep (`:grep`).

Давайте сначала пройдемся по внутреннему grep. `:vim` имеет следующий синтаксис:

```
:vim /pattern/ file
```

- `/pattern/` - это шаблон регулярного выражения(regex) вашего поискового запроса.
- `file` is the file argument. You can pass multiple arguments. Vim will search for the pattern inside the file argument. Similar to `:find`, you can pass it `*` and `**` wildcards.
- `file` - это название файла. Вы можете указать несколько файлов. Vim будет искать шаблон внутри указаннного файла. Как и в случае с `:find`, вы можете передать ему подстановочные знаки `*` и `**`.

Например, чтобы найти все вхождения строки "breakfast" во всех ruby файлах (`.rb`) в директории `app/controllers/`:

```
:vim /breakfast/ app/controllers/**/*.rb
```

После этого вы будете перенаправлены к первому результату. Команда поиска `vim` в Vim использует операцию `quickfix`. Чтобы увидеть все результаты поиска, выполните команду `:copen`. Откроется окно быстрого исправления `quickfix`. Вот несколько полезных команд быстрого исправления, которые помогут вам немедленно начать работу:

```
:copen        Открыть окно быстрого исправления
:cclose       Закрыть окно быстрого исправления
:cnext        Перейти к следующей ошибке
:cprevious    Перейти к предыдущей ошибке
:colder       Перейти к старому списку ошибок
:cnewer       Перейти к новому списку ошибок
```

Чтобы узнать больше о быстром исправлении, ознакомьтесь с `:h quickfix`.

Вы можете заметить, что выполнение внутреннего grep (`:vim`) может быть медленным, если у вас большое количество совпадений. Это связано с тем, что Vim загружает каждый соответствующий файл в память, как если бы он редактировался. Если Vim найдет большое количество файлов, соответствующих вашему запросу, он загрузит их все и следовательно, будет потреблять большое количество памяти.

Давайте поговорим о внешнем grep. По умолчанию используется команда терминала `grep`. Чтобы найти слово "lunch" в файле ruby в директории `app/controllers/`, вы можете сделать следующее:

```
:grep -R "lunch" app/controllers/
```

Обратите внимание, что вместо использования `/pattern/` применяется терминальный синтаксис grep `"pattern"`. Показать все совпадения можно также с помощью `quickfix`.

Vim имеет переменную `grepprg` для определения того, какую внешнюю программу запускать при выполнении команды Vim `:grep`, чтобы вам не пришлось закрывать Vim и вызывать терминальную команду `grep`. Позже я покажу вам, как изменить программу по умолчанию, вызываемую при использовании команды Vim `:grep`.

## Просмотр файлов с помощью Netrw

`netrw` - это встроенный файловый менеджер Vim. Полезно видеть иерархию проекта. Чтобы запустить `netrw`, вам понадобятся две настройки в вашем `.vimrc`:

```
set nocp
filetype plugin on
```

Поскольку `netrw` - это обширная тема, я расскажу только об основном использовании, но этого должно быть достаточно, чтобы вы могли начать. Вы можете запустить `netrw` при запуске Vim, передав ему директорию в качестве параметра, а не файл. Например:

```
vim .
vim src/client/
vim app/controllers/
```

Чтобы запустить `netrw` из Vim, вы можете использовать команду `:edit` и передать директорию вместо имени файла:

```
:edit .
:edit src/client/
:edit app/controllers/
```

Есть и другие способы запустить окно `netrw` без передачи директории:

```
:Explore     Запуск netrw в текущем файле
:Sexplore    Без шуток. Запуск netrw на разделенной верхней половине экрана
:Vexplore    Запуск netrw на разделенной левой половине экрана
```

Вы можете перемещаться по `netrw` с помощью перемещений в Vim (перемещения будут подробно рассмотрены в следующей главе). Если вам нужно создать, удалить или переименовать файл или директорию, вот список полезных команд `netrw`:

```
%    Создать новый файл
d    Создать новую директорию
R    Переименовать файл или директорию
D    Удалить файл или директорию
```

Команда `:h netrw` очень исчерпывающая. Проверьте её, если у вас есть время.

Если вы находите `netrw` слишком пресным и вам нужно больше вкуса, [vim-vinegar](https://github.com/tpope/vim-vinegar) - хороший плагин для улучшения `netrw`. Если вы ищете другой проводник, [NERDTree](https://github.com/preservim/nerdtree) — хорошая альтернатива. Попробуйте их!

## Fzf

Теперь, когда вы узнали, как искать файлы в Vim с помощью встроенных инструментов, давайте узнаем, как это делать с помощью плагинов.

Одна вещь, которую современные текстовые редакторы понимают правильно, а Vim нет, это то, насколько легко найти файлы, особенно с помощью нечеткого поиска. Во второй половине главы я покажу вам, как использовать [fzf.vim](https://github.com/junegunn/fzf.vim) для простого и мощного поиска в Vim.

## Setup

Во-первых, убедитесь, что у вас загружены [fzf](https://github.com/junegunn/fzf) и [ripgrep](https://github.com/BurntSushi/ripgrep). Следуйте инструкциям в их репозитории на github. Команды `fzf` и `rg` должны быть доступны после успешной установки.

Ripgrep — это инструмент поиска, очень похожий на grep (отсюда и название). В целом он быстрее, чем grep, и имеет много полезных функций. Fzf — это универсальный инструмент командной строки для нечеткого поиска. Вы можете использовать его с любыми командами, включая ripgrep. Вместе они представляют собой мощную комбинацию инструментов поиска.

Fzf не использует ripgrep по умолчанию, поэтому нам нужно сказать fzf использовать ripgrep, определив переменную `FZF_DEFAULT_COMMAND`. В моем `.zshrc` (`.bashrc`, если вы используете bash) я определил вот это:

```
if type rg &> /dev/null; then
  export FZF_DEFAULT_COMMAND='rg --files'
  export FZF_DEFAULT_OPTS='-m'
fi
```

Обратите внимание на `-m` в `FZF_DEFAULT_OPTS`. Эта опция позволяет нам сделать несколько выборов с помощью `<Tab>` или `<Shift-Tab>`. Вам не нужна эта строка, чтобы fzf работал с Vim, но я думаю, что это полезная опция. Она пригодится, когда вы захотите выполнить поиск и замену в нескольких файлах, о которых я расскажу чуть позже. Команда fzf допускает гораздо больше вариантов, но я не буду их здесь рассматривать. Чтобы узнать больше, ознакомьтесь с [репозиторием fzf](https://github.com/junegunn/fzf#usage) или `man fzf`. Как минимум, у вас должен быть `export FZF_DEFAULT_COMMAND='rg'`.

После установки fzf и ripgrep давайте настроим плагин fzf. В этом примере я использую менеджер плагинов [vim-plug](https://github.com/junegunn/vim-plug), но вы можете использовать любые менеджеры плагинов.

Добавьте их в свои плагины `.vimrc`. Вам нужно использовать плагин [fzf.vim](https://github.com/junegunn/fzf.vim) (созданный тем же автором fzf).

```
call plug#begin()
Plug 'junegunn/fzf.vim'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
call plug#end()
```

После добавления этих строк вам нужно будет открыть `vim` и запустить `:PlugInstall`. Это установит все плагины, которые определены в вашем файле `vimrc` и не установлены. В нашем случае это `fzf.vim` и `fzf`.

Для получения дополнительной информации об этом плагине вы можете ознакомиться с [репозиторием fzf.vim](https://github.com/junegunn/fzf/blob/master/README-VIM.md).

## Синтаксис Fzf

Чтобы эффективно использовать fzf, вы должны изучить некоторые основы синтаксиса fzf. К счастью, список небольшой:

- `^` — точное совпадение префиксa. Поиск фразы, начинающейся с "welcome": `^welcome`.
- `$` — точное совпадение суффикса. Поиск фразы, оканчивающейся на "my friends": `friends$`.
- `'` — точное совпадением. Поиск по фразе "welcome my friends": `'welcome my friends`.
- `|` — совпадением "или". Для поиска «friends» или «foes»: `friends | foes`.
- `!` — обратное соответствие. Для поиска фразы, содержащей слово "welcome" и не "friends": `welcome !friends`

Вы можете комбинировать и сочетать эти варианты. Например, `^hello | ^welcome friends$` будет искать фразу, начинающуюся с "welcome" или "hello" и заканчивающуюся на "friends".

## Поиск файлов

Для поиска файлов внутри Vim с помощью плагина fzf.vim вы можете использовать метод `:Files`. Запустите `:Files` из Vim, и вам будет предложено выполнить поиск fzf.

Поскольку вы будете часто использовать эту команду, хорошо подберите для нее сочетани клавиш. Я использую `Ctrl-f`. В vimrc я добавил это:

```
nnoremap <silent> <C-f> :Files<CR>
```

## Поиск в файлах

Для поиска внутри файлов вы можете использовать команду `:Rg`.

Опять же, поскольку вы, вероятно, будете использовать каманду часто, давайте подберем сочетание клавиш. Я использую `<Leader>f`. По умолчанию клавиша `<Leader>` сопоставлена с `\`.

```
nnoremap <silent> <Leader>f :Rg<CR>
```

## Другие поисковые запросы

Fzf.vim предоставляет множество других команд поиска. Я не буду перечислять каждую из них здесь, но вы можете ознакомиться с ними [здесь](https://github.com/junegunn/fzf.vim#commands).

Вот как выглядят мои горячие клавиши для fzf:

```
nnoremap <silent> <Leader>b :Buffers<CR>
nnoremap <silent> <C-f> :Files<CR>
nnoremap <silent> <Leader>f :Rg<CR>
nnoremap <silent> <Leader>/ :BLines<CR>
nnoremap <silent> <Leader>' :Marks<CR>
nnoremap <silent> <Leader>g :Commits<CR>
nnoremap <silent> <Leader>H :Helptags<CR>
nnoremap <silent> <Leader>hh :History<CR>
nnoremap <silent> <Leader>h: :History:<CR>
nnoremap <silent> <Leader>h/ :History/<CR>
```

## Замена grep на rg

Как упоминалось ранее, Vim имеет два способа поиска в файлах: `:vim` и `:grep`. `:grep` использует внешний инструмент поиска, который вы можете переназначить с помощью ключевого слова `grepprg`. Я покажу вам, как настроить Vim на использование ripgrep вместо grep из командной строки при выполнении команды `:grep`.

Теперь давайте настроим `grepprg` так, чтобы команда `:grep` в Vim использовала ripgrep. Добавьте это в свой vimrc:

```
set grepprg=rg\ --vimgrep\ --smart-case\ --follow
```

Не стесняйтесь изменять некоторые из приведенных выше параметров! Для получения дополнительной информации о том, что означают эти параметры, ознакомьтесь с `man rg`.

После того как вы обновили `grepprg`, теперь при запуске `:grep` он запускает `rg --vimgrep --smart-case --follow` вместо `grep`. Если вы хотите искать "donut" с помощью ripgrep, вы теперь можете запустить более лаконичную команду `:grep "donut"` вместо `:grep "donut" . -R`.

Так же, как и старый `:grep`, этот новый `:grep` также использует quickfix для отображения результатов.

Вы можете задаться вопросом: "Ну, это хорошо, но я никогда не использовал `:grep` в Vim, плюс разве я не могу просто использовать `:Rg` для поиска фраз в файлах? Мне вообще когда-нибудь понадобится использовать `:grep`?

Это очень хороший вопрос. Возможно, вам придется использовать `:grep` в Vim для поиска и замены в нескольких файлах, о которых я расскажу далее.

## Поиск и замена в нескольких файлах

Современные текстовые редакторы, такие как VSCode, позволяют очень легко искать и заменять строку в нескольких файлах. В этом разделе я покажу вам два различных метода, как легко сделать это в Vim.

Первый способ, это заменить *все* совпадающие фразы в вашем проекте. Вам нужно будет использовать `:grep`. Если вы хотите заменить все экземпляры слова "pizza" на "donut", вот что вы делаете:

```
:grep "pizza"
:cfdo %s/pizza/donut/g | update
```

Давайте разберем команды:

1. `:grep pizza` использует ripgrep для поиска всех экземпляров слова "pizza" (кстати, это все равно будет работать, даже если вы не переназначили `grepprg` на использование ripgrep. Вам придется сделать `:grep "pizza" . -R` вместо `:grep "pizza"`).
2. `:cfdo` выполняет любую команду, которую вы передаете, всем файлам в вашем списке quickfix. В данном случае вашей командой является команда подстановки `%s/pizza/donut/g`. Канал (`|`) является оператором цепи. Команда `update` сохраняет каждый файл после подстановки. Я расскажу о команде подстановки более подробно в одной из следующих глав.

Второй способ заключается в поиске и замене в выбранных файлах. С помощью этого метода вы можете вручную выбрать, для каких файлов вы хотите выполнить выбор и замену. Вот что вы делаете:

1. Сначала очистите буферы. Крайне важно, чтобы список буферов содержал только те файлы, к которым вы хотите применить замену. Вы можете либо перезапустить Vim, либо запустить `:%bd | e#` (`%bd` удаляет все буферы, а `e#` открывает файл, с которым вы только что работали).
2. Запустите `:Files`.
3. Выберите все файлы, с которыми вы хотите выполнить поиск и замену. Чтобы выбрать несколько файлов, используйте `<Tab>` / `<Shift-Tab>`. Это возможно только в том случае, если у вас есть множественный флаг (`-m`) в `FZF_DEFAULT_OPTS`.
4. Запустите `:bufdo %s/pizza/donut/g | update`. Команда `:bufdo %s/pizza/donut/g | update` выглядит аналогично более ранней `:cfdo %s/pizza/donut/g | update`. Разница в том, что вместо подстановки всех записей quickfix (`:cfdo`), вы подставляете все записи буфера (`:bufdo`).

## Учитесь искать умным способом

Поиск — это хлеб с маслом для редактирования текста. Умение хорошо искать в Vim значительно улучшит ваш рабочий процесс редактирования текста.

Fzf.vim меняет правила игры. Я не могу представить использование Vim без него. Я думаю, что очень важно иметь хороший инструмент поиска при запуске Vim. Я видел людей, которые изо всех сил пытались перейти на Vim, но им не хватало критически важных функций, которые есть в современных текстовых редакторах, таких как простая и мощная функция поиска. Надеюсь, эта глава поможет вам облегчить переход на Vim.

Вы также только что увидели расширяемость Vim в действии - возможность расширения функциональности поиска с помощью плагина и внешней программы. В будущем помните о том, какими еще функциями вы бы хотели дополнить Vim. Скорее всего, они уже есть в Vim, кто-то создал плагин или для этого уже есть программа. Далее вы узнаете об очень важной теме в Vim: Vim грамматика.
