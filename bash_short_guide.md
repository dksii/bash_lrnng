•  
----------------------------------  
Bash: Bourne Again Shell  
2 режима: интерактивный и неинтерактивный (скрипты)  
  
bash распознает каждое слово как аргумент, чтобы например не удалить лишнее используйте кавычки: rm "one two three four.mp3" иначе  попытается удалить 4 файла  
также нужны пробелы в следующей ситуации: [ -f file ] вместо [-f file] так как [ - это команда, все остальное аргументы  
  
почти всё - строка (имя команды, каждый аргумент, имена и значения переменных, ...)  
  
в начале скрипта идет шебанг (#!)  
#!/bin/bash или #!/usr/bin/env bash  
  
•  
----------------------------------  
спец символы  
----------------------------------  
символ     описание  
----------------------------------------------  
$          expansion: parameter expansion $var or ${var}, command substitution $(command), or arithmetic expansion $(( expression )).  
''         single quotes: защищает строку от интерпретации, например echo выведет как есть выражение с ''  
""         double quotes: защищает строку от разбиения на несколько аргументов, пример с rm выше  
\          escape: защищает следующий символ от интерпретации, игнорируется в ''  
#          comment  
[[]]       test: вычисляет условное выражение в true или false соотв  
!          negate  
><         redirecton: перенаправляет ввод и вывод команды  
|          pipe: перенаправляет вывод начальной команды в ввод другой (echo "Hello world" | grep -o world)  
;          separator: разделяет команды находящиеся на 1 строке  
{}         inline group: команды внутри воспринимаются как 1 команда (No subshell is created)  
()         subshell group: похоже на предыдущую, но если есть сайд-эффекты такие как изменения переменных - не повлияет на текущий shell  
(())       arithmetic expression: используется для присваивания (( a = 1 + 4 )) или тестов if (( a < b ))  
$(())      arithmetic expansion: как и прошлое, но выражение заменяется рез-татом вычисления echo "Среднее значение = $(( (a+b)/2 ))"  
~          home dir  
  
•  
----------------------------------  
Переменные  
----------------------------------  
varname=vardata  
нет пробелов между =, так как иначе воспримет = и vardata как аргументы к некой команде varname  
для доступа к значению переменной используем parameter expansion:  foo=bar; echo "Foo is $foo"  
  
параметры:  
переменные - это 1 из видов параметров, которые доступны по имени  
остальные это спец параметры:  
имя           использование        описание  
---------------------------------------------  
0             $0                 содержит название или путь скрипта  
1 2 и тд      $1 и тд            positional parameters: аргументы, переданные данному скрипту/функции  
*             $*                 содержит все positional parameters, в двойных кавычках, 1 строка  
@             $@                 содержит все positional parameters, в двойных кавычках, список  
#             $#                 число positional parameters  
?             $?                 exit code предыдущей команды  
$             $$                 PID (process ID) текущей shell  
!             $!                 PID of the most recently executed background pipeline  
_             $_                 последний аргумент последней выполненной команды  
  
  
типы переменных:  
bash не типизирован, но есть разные виды переменных  
--------------------------------------  
Array: declare -a variable: массив строк  
Associative array: declare -A variable: ассоциативный массив строк (bash 4.0 or higher)  
Integer: declare -i variable: присвоение к этому числу автоматически вызывает Arithmetic Evaluation  
Read Only: declare -r variable  
Export: declare -x variable: Будет унаследовано дочерними процессами  
  
примеры  
$ a=5; a+=2; echo "$a"; unset a  
52  
$ a=5; let a+=2; echo "$a"; unset a  
7  
$ declare -i a=5; a+=2; echo "$a"; unset a  
7  
$ a=5+2; echo "$a"; unset a  
5+2  
$ declare -i a=5+2; echo "$a"; unset a  
7  
  
лучше не использовать Integer, а писать явно ((..)) или использовать let, ибо не очевидно  
для массива (не ассоциативного) лучше писать array=(...)  
  
  
parameter expansion:  
  
иногда $variable не хватает и требуется ${variable}  
$ echo "'$USER', '$USERs', '${USER}s'"  
'lhunath', '', 'lhunaths'  
  
синтаксис                   описание  
--------------------------------------  
${parameter:-word}            Use default value. Если 'parameter' unset или null, 'word' (может являться expansion) подставится  
${parameter:=word}            Assign default value. Если 'parameter' unset или null, 'word' (может являться expansion) присвоится 'parameter'  
${parameter:+word}            Use Alternate Value. Если 'parameter' unset или null ничего не подставится, в противном случае 'word'  
${parameter:offset:length}    Substring expansion. Здесь offset - позиция (с нуля), length - число символов, length необязателен, offset бывает отриц  
${#parameter}                 Количество символов 'parameter'  
${parameter#pattern}          Паттерн матчится с начала, результат - выкидывание из строки наиболее короткого совпадения  
${parameter##pattern}         Как и прошлое, только выкидывается длиннейшее совпадение (жадность)  
${parameter%pattern}          Паттерн матчится с конца, кротчайшее  
${parameter%%pattern}         ..., длиннейшее  
${parameter/pat/string}       Первое совпадение 'pat' меняется на 'string'  
${parameter//pat/string}      Каждое совпадение ...  
${parameter/#pat/string}        
${parameter/%pat/string}  
  
нельзя использовать множественные PE, надо разбивать выражения:  
file=$HOME/image.jpg; file=${file##*/}; echo "${file%.*}"  
image  
  
  
•  
----------------------------------  
Паттерны  
----------------------------------  
есть три вида - globs, extended globs, regular expression (в основном в скриптах для проверки ввода юзера)  
для выборки файлов только globs и ext globs  
  
Glob Patterns  
------------------  
важны (can be used to match filenames or other strings)  
состоят из символов и метасимволов:  
• *: Matches any string, including the null string.  
• ?: Matches any single character.  
• [...]: Matches any one of the enclosed characters.  
  
Они неявно содержат якоря с двух сторон, то есть a* не сматчит cat, но ca* сматчит  
$ ls  
a abc b c  
$ echo *  
a abc b c  
$ echo a*  
a abc  
  
Bash видит glob, расширяет (expand) его, превращая в список, то есть echo a* -> echo a abc  
Когда bash матчит filenames, то * и ? не могут в слэш (/), то есть матчят до него  
например */bin сматчится на foo/bin, но не сможет на /usr/local/bin  
  
Для обхода вместо ls лучше использовать glob *, пример:  
$ touch "a b.txt"  
$ ls  
a b.txt  
$ for file in `ls`; do rm "$file"; done  
rm: cannot remove `a': No such file or directory  
rm: cannot remove `b.txt': No such file or directory  
$ for file in *; do rm "$file"; done  
$ ls  
  
glob используется не только для матчинга filenames, пример:  
$ filename="some.jpg"  
$ if [[ $filename = *.jpg ]]; then  
> echo "tru"  
> fi  
tru  
  
Extended Globs  
-----------------  
По умолчанию могут быть выключены, включаются через: $ shopt -s extglob  
  
• ?(list): Matches zero or one occurrence of the given patterns.  
• *(list): Matches zero or more occurrences of the given patterns.  
• +(list): Matches one or more occurrences of the given patterns.  
• @(list): Matches one of the given patterns.  
• !(list): Matches anything but the given patterns.  
  
list внутри - список globs или ext globs, разделенные |  
пример:  
$ ls  
names.txt tokyo.jpg california.bmp  
$ echo !(*jpg|*bmp)  
names.txt  
  
Regular Expressions  
--------------------  
похожи на glob patterns, но не могут использоваться для filenames matching  
с версии 3.0 Bash поддерживает оператор =~ для [[  
этот оператор матчит строку с этим регэкспом и воззвращает 0("true") если матч, 1("false") если нет и 2 если ошибка  
bash использует Extended Regular Expression (ERE) диалект  
http://mywiki.wooledge.org/RegularExpression  
  
Brace Expansion  
--------------------  
по сути это не относится к паттернам, они расширяются ко всевозможным вариантам, это не матчинг, а просто все варианты (прямое произведение)  
  
$ echo th{e,a}n  
then than   
$ echo {/home/*,/root}/.*profile  
/home/axxo/.bash_profile /home/lhunath/.profile /root/.bash_profile /root/.profile  
$ echo {1..9} # тут нельзя variables, только явно числа, умеет в обратную сторону - {9..1}  
1 2 3 4 5 6 7 8 9  
$ echo {0,1}{0..9}  
00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19  
  
похоже на glob, но возвращает не filenames, а все что может, но происходит до filename expansion,  
второй пример показывает это (комбинацию с glob), где уже находятся реальные файлы  
получается: echo /home/*/.*profile /root/.*profile  
  
•  
Exit status/code  
это число от 0 до 255 (0 - успех)  
Для выхода с кодом - команда exit  
  
•  
----------------------------------  
Control Operators (&& and ||)  
----------------------------------  
простейший способ управления последовательностью команд в зависимости от успеха/неуспеха предыдущей  
$ mkdir d && cd d #выполнится cd, если успешно выполн cd  
$ rm file || { echo 'Could not delete file!' >&2; exit 1; } # >&2 это в стандартный вывод для ошибок   
лучше не перебарщивать с || и &&  
  
также лучше группировать:  
$ grep -q goodword "$file" && ! grep -q badword "$file" && { rm "$file" || echo "Couldn't delete: $file" >&2; }  
  
группируем не только для таких простых вещей  
например следующее:  
  
{  
    read firstLine  
    read secondLine  
    while read otherLine; do  
        something  
    done  
} < file  
  
читаем построчно файл  
  
•  
----------------------------------  
Условные блоки (if, test, [[)  
----------------------------------  
if COMMANDS; then  
OTHER COMMANDS  
fi  
  
вместо fi может быть elif, если хотим несколько условий  
  
- Команда test (или [)  
более продвинутая аналогичная команда [[  
  
$ if [ a = b ]  
> then echo "a is the same as b."  
> else echo "a is not the same as b."  
> fi  
a is not the same as b.  
  
В отличии от [, команда [[ поддерживает pattern matching:  
$ [[ $filename = *.png ]] && echo "$filename looks like a PNG file"  
  
Желательно всегда использовать "" при работе с PE, и только в редких случаях избегать, например тут:  
  
$ foo=[a-z]* name=lhunath  
$ [[ $name = $foo ]] && echo "Name $name matches pattern $foo"  
Name lhunath matches pattern [a-z]*  
$ [[ $name = "$foo" ]] || echo "Name $name is not equal to the string $foo"  
Name lhunath is not equal to the string [a-z]*  
  
В этом случае pattern matching работает если правая часть не в кавычках  
  
Дальше следует тесты поддерживаемые [ и [[:  
-----------------------------------------------  
пример                  описание  
-----------------------------------------------  
-e FILE:                True if file exists.  
-f FILE:                True if file is a regular file.  
-d FILE:                True if file is a directory.  
-h FILE:                True if file is a symbolic link.  
-p PIPE:                True if pipe exists.  
-r FILE:                True if file is readable by you.  
-s FILE:                True if file exists and is not empty.  
-t FD :                 True if FD is opened on a terminal.  
-w FILE:                True if the file is writable by you.  
-x FILE:                True if the file is executable by you.  
-O FILE:                True if the file is effectively owned by you.  
-G FILE:                True if the file is effectively owned by your group.  
FILE -nt FILE:          True if the first file is newer than the second.  
FILE -ot FILE:          True if the first file is older than the second.  
  
-z STRING:              True if the string is empty (it’s length is zero).  
-n STRING:              True if the string is not empty (it’s length is not zero).  
STRING = STRING:        True if the first string is identical to the second.  
STRING != STRING:       True if the first string is not identical to the second.  
STRING < STRING:        True if the first string sorts before the second.  
STRING > STRING:        True if the first string sorts after the second.  
  
! EXPR:                 Inverts the result of the expression (logical NOT).  
  
INT -eq INT:            True if both integers are identical.  
INT -ne INT:            True if the integers are not identical.  
INT -lt INT:            True if the first integer is less than the second.  
INT -gt INT:            True if the first integer is greater than the second.  
INT -le INT:            True if the first integer is less than or equal to the second.  
INT -ge INT:            True if the first integer is greater than or equal to the second.  
  
Дальше следует тесты поддерживаемые только [:  
-----------------------------------------------  
пример                  описание  
-----------------------------------------------  
EXPR -a EXPR:           True if both expressions are true (logical AND).  
EXPR -o EXPR:           True if either expression is true (logical OR).  
  
Дальше следует тесты поддерживаемые только [[:  
-----------------------------------------------  
пример                           описание  
-----------------------------------------------  
STRING = (or ==) PATTERN:        Not string comparison like with [ (or test), but pattern matching is performed.  
                                 True if the string matches the glob pattern.  
STRING != PATTERN:               Not string comparison like with [ (or test), but pattern matching is performed. True  
                                 if the string does not match the glob pattern.  
STRING =~ REGEX:                 True if the string matches the regex pattern.  
( EXPR ):                        Parentheses can be used to change the evaluation precedence.  
EXPR && EXPR:                    Much like the ’-a’ operator of test, but does not evaluate the second expression if the first  
                                 already turns out to be false.  
EXPR || EXPR:                    Much like the ’-o’ operator of test, but does not evaluate the second expression if the first  
                                 already turns out to be true.  
------------------------------------------------  
  
Если пишите скрипт на bash, лучше использовать [[  
  
•  
----------------------------------  
Conditional Loops (while, until and for)  
----------------------------------  
• while command:               Повторяет пока команда успешно завершается (exit code is 0).  
• until command:               Повторяет пока команда неуспешно завершается (exit code is not 0).  
• for variable in words:       Цикл по списку  
• for (( expression; expression; expression )): Starts by evaluating the first arithmetic expression; repeats the loop  
so long as the second arithmetic expression is successful; and at the end of each loop evaluates the third arithmetic  
expression.  
  
После каждого идет do и в конце done  
Есть break и continue  
  
Примеры:  
  
$ while true  
> do echo "Infinite loop"  
> done  
  
$ while ! ping -c 1 -W 1 1.1.1.1; do  
> echo "still waiting for 1.1.1.1"  
> sleep 1  
> done  
  
$ for (( i=10; i > 0; i-- ))  
> do echo "$i empty cans of beer."  
> done  
  
$ for i in {10..1}  
> do echo "$i empty cans of beer."  
> done  
  
•  
----------------------------------  
Choices (case and select)  
----------------------------------  
case $LANG in  
    en*) echo 'Hello!' ;;  
    fr*) echo 'Salut!' ;;  
    de*) echo 'Guten Tag!' ;;  
    nl*) echo 'Hallo!' ;;  
    it*) echo 'Ciao!' ;;  
    es*) echo 'Hola!' ;;  
    C|POSIX) echo 'hello world' ;;  
    *) echo 'I do not speak your language.' ;;  
esac  
  
case $var in  
    foo|bar|more) ... ;;  
esac  
  
работает с globs  
также есть select  
  
# Arrays  
> An **array** is a numbered list of strings: It maps integers to strings  

Пример *нерабочего* кода (проблемы с пробелами):  
```bash
$ files=$(ls ~/*.jpg); cp $files /backups/  
```
Адекватный фикс с массивами:
```bash
$ files=(~/*.jpg); cp "${files[@]}" /backups/  
```
Использование массивов для представления множества строк - самый безопасный способ  
  
## Создание массивов   

Способов несколько, зависит от данных, откуда приходят и т.д.  
  
**Простейший способ**:
```bash
$ names=("Bob" "Peter" "$USER" "Big Bad John")  
```  
**Можно задать индексы явно**:
```bash  
$ names=([0]="Bob" [1]="Peter" [20]="$USER" [21]="Big Bad John")  
# or...  
$ names[0]="Bob"
```
  
В первом варианте есть дыра между индексами 1 и 20, такой массив называется **Sparse Array**  
  
**Также для создания можно использовать Globs**:
```bash 
$ photos=(~/"My Photos"/*.jpg)  
```
Как записать результат команды (например find) в массив? Правильный вариант:  
```bash 
$ files=()                     # объявили пустой массив  
$ while read -r -d ''; do  
$     files+=("$REPLY")        # добавляем элемент в конец массива  
$ done < <(find /foo -print0)  # < <(..) это комбинация File Redirection (<) и Process Substitution (<(..))
```  
  
## Использование массивов 

Сперва напечатаем массив, используя команду ```declare -p```, она печатает содержимое переменных и какой у них тип. 

**Пример**:
```bash
$ a=(1 2 3)  
$ declare -p a  
declare -a a=([0]="1" [1]="2" [2]="3")  
```  
**Напечатаем сами**:  
```bash
$ a=(1 10 100)  
$ for n in "${!a[@]}"; do echo "$n: ${a[n]}"; done  
0: 1  
1: 10  
2: 100  
```
  
**Или цикл не по индексам, а по самим значениям**:
```bash
$ for v in "${a[@]}"; do echo "$v"; done  
1  
10  
100
```  
  
> Важно брать в кавычки (```"${a[@]}" и "${!a[@]}"```), иначе профита от массивов мало. Каждый элемент в таком случае заключается в кавычки, защита от word splitting 
  
*Рассмотрим*:
```bash  
$ myfiles=(db.sql home.tbz2 etc.tbz2)  
$ cp "${myfiles[@]}" /backups/
```  
Последнее трансформируется в:  
```bash
$ cp "db.sql" "home.tbz2" "etc.tbz2" /backups/  
```
  
Также есть форма ```"${arrayname[*]}"```, она конвертирует в строку  
  
**Использование IFS** (чтобы указать разделитель для элементов в массиве):
```bash
$ names=("Bob" "Peter" "Big Bad John")  
$ ( IFS=,; echo "Today's contestants are: ${names[*]}" )  
Today's contestants are: Bob,Peter,Big Bad John  
```  
Взяли в скобки ( IFS=... чтобы создалась subshell и не зааффектила переменную в текущей shell  
  
**Доступ по индексам**: 
Можем производить арифметические вычисления без использования **$**, по умолчанию ariphmetic context  
```bash
$ a=(a b c q w x y z)  
$ for ((i=0; i<${#a[@]}; i+=2)); do  
> echo "${a[i]} and ${a[i+1]}"  
> done 
``` 
  
  
## Associative Arrays  
**2 основных отличия от обычных массивов:**
  
1) порядок ключей не гарантирован (при получении через ```"${!array[@]}"```)  
2) для индекса необходим **$**, [...] не интерпретируется в арифметическом контексте по умолчанию как в обычных массивах, что логично, необязательно индексы числа
  
**Примеры:**  
  
```bash  
$ indexedArray=( "one" "two" )  
$ declare -A associativeArray=( ["foo"]="bar" ["alpha"]="omega" )  
$ index=0 key="foo"  
$ echo "${indexedArray[$index]}"  
one  
$ echo "${indexedArray[index]}"  
one  
$ echo "${indexedArray[index + 1]}"  
two  
$ echo "${associativeArray[$key]}"  
bar  
$ echo "${associativeArray[key]}"  
$ echo "${associativeArray[key + 1]}"  
```