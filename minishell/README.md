# minishell

## Оглавление

- [Материалы](#материалы)
- [Структуры и переменные](#cтруктуры-и-переменные)
- [Readline](#readline)
- [Built-in's](#built-ins)
- [Другое](#другое)

<br />

## Материалы

* [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html)
* [SystemsProgrammingBook.Chapter5](https://www.cs.purdue.edu/homes/grr/SystemsProgrammingBook/Book/Chapter5-WritingYourOwnShell.pdf)
* [Shell Command Language](https://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html)
* [Лекция от weambros по minishell](https://www.youtube.com/watch?v=Um3pzuee-4Y)
* [Лекция от rdrizzle по minishell](https://www.youtube.com/watch?v=A7ccmRSn7JY)
* [linenoise](https://github.com/antirez/linenoise)
* [Linux Kernel (OS) Lecture Notes](https://medium.com/pocs/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BB%A4%EB%84%90-%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EA%B0%95%EC%9D%98%EB%85%B8%ED%8A%B8-3-9ed24cf457ce)
* [Minishell 허용 함수 정리](https://42kchoi.tistory.com/259)
* [[minishell] 1. 과제소개 및 선행지식](https://velog.io/@hidaehyunlee/minishell-1.-%EA%B3%BC%EC%A0%9C%EC%86%8C%EA%B0%9C-%EB%B0%8F-%EC%84%A0%ED%96%89%EC%A7%80%EC%8B%9D)
* [[minishell] 2. 프로그램 구조 및 개발 기록들](https://velog.io/@hidaehyunlee/minishell-2.-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EA%B5%AC%EC%A1%B0-%EB%B0%8F-%EA%B0%9C%EB%B0%9C-%EA%B8%B0%EB%A1%9D%EB%93%A4)
* [[minishell] 3. 시그널(Signal) 처리하기](https://velog.io/@hidaehyunlee/minishell-3.-%EC%8B%9C%EA%B7%B8%EB%84%90Signal-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0)
* [[minishell] 4. 종료상태와 에러메세지 처리](https://velog.io/@hidaehyunlee/minishell-4.-%EC%A2%85%EB%A3%8C%EC%83%81%ED%83%9C%EC%99%80-%EC%97%90%EB%9F%AC%EB%A9%94%EC%84%B8%EC%A7%80-%EC%B2%98%EB%A6%AC)
* [[minishell] 5. 파이프(Pipe) 처리](https://velog.io/@hidaehyunlee/minishell-5.-%ED%8C%8C%EC%9D%B4%ED%94%84Pipe-%EC%B2%98%EB%A6%AC)
* [[minishell] 6. 리다이렉 (Redirection) 처리](https://velog.io/@hidaehyunlee/minishell-6.-%EB%A6%AC%EB%8B%A4%EC%9D%B4%EB%A0%89%EC%85%98Redirection-%EC%B2%98%EB%A6%AC)
* [[RU] «Шелл» на С: пишем командную оболочку для Unix](https://tproger.ru/translations/unix-shell-in-c/)
* [[ENG] Tutorial - Write a Shell in C](https://brennan.io/2015/01/16/write-a-shell-in-c/)
* [Minishell. Часть 1. Сущность shell и структура проекта.](https://vk.com/@swquinc-minishell-chast-1-suschnost-shell-i-struktura-proekta)
* [man bash](https://www.opennet.ru/man.shtml?topic=bash&russian=0&category=&submit=%F0%CF%CB%C1%DA%C1%D4%D8+man)
* [Do parentheses really put the command in a subshell?](https://unix.stackexchange.com/questions/138463/do-parentheses-really-put-the-command-in-a-subshell)
* [bash command processing](http://aosabook.org/en/bash.html)
* [bash parser](https://mywiki.wooledge.org/BashParser)
* [Bash Reference Manual](https://www.gnu.org/software/bash/manual/html_node/index.html)

<br />

## Структуры и переменные

**Q**: Обязательно ли включать `char **envp` в общую структуры, если можно просто запихнуть все в список и с этим списком работать: изменять, выписывать из него данные и т. д.?

**TLDR**: Нет, не обязательно

**A**: Можно собирать переменные окружения для `int execve(const char *filename, char *const argv [], char *const envp[])` только при необходимости (т.е. когда окружение изменилось). В остальных случаях (т.е. когда окружение не изменялось) можно ссылаться на данную переменную, в которую окружение уже собрано. В таком случае мы будем экономить время, которое уйдет на сборку окружения в двухмерный массив.

*Comment*: Это верно только в случае, когда Вы храните переменные окружения в связном списке (или другой структуре данных, например в RB-дереве). Если Вы изначально все окружение держите в двухмерном массиве и каждый раз при изменении окружения его пересобираете, то эта переменная у Вас скорее всего уже есть.

---

**Q**: Зачем именно делать массив из built-in функций, их же можно просто так вызывать?

**TLDR**: Чтобы список built-in функций был лего масштабируем

**A**: Если использовать не массив, а `if/else` или `switch/case` (не разрешен в проекте), то код быстро раздуется при добавлении новых функций. Поддерживать такой код крайне трудно (Представьте что у вас поменялась сигнатура функции. Придется изменять вызов во всех местах, если Вы, конечно, не передаете структуру параметров для функции).

Пример:

```c
if (state == 0)
	func_0(params);
else if (state == 1)
	func_1(params);
...
else if (state == n)
	func_n(params);

// Or

switch (state): {
	case 0: {
		func_0(params);
		break ;
	}
	case 1: {
		func_1(params);
		break ;
	}

	...

	case n: {
		func_n(params);
		break ;
	}
}
```

Если же использовать массив, то вызов функции можно будет написать *один* раз, а при добавлении новых функций, просто дописать в определение массива ссылок на функции и увеличить счетчик длины массива на 1

Пример определения:

```c
// 123 для примера, любое беззнаковое целочисленное значение
#define BUILTIN_FUNC_COUNT 123

// RET_TYPE -- возвращаемый тип
// t_func_ptr -- название типа указателя на функцию
// PARAM_TYPE -- тип параметра
typedef RET_TYPE (*t_func_ptr)(PARAM_TYPE_A a, PARAM_TYPE_B b, ...);

t_func_ptr f_ptrs[BUILTIN_FUNC_COUNT];

f_ptrs[0] = func_0;
f_ptrs[1] = func_1;
...
f_ptrs[n] = func_n;
```

Пример вызова:

```c
f_ptrs[state](params);
```

<br />

## Readline

**Q**: Уточнение: Prompt - это то что первое в терминале высвечивается и ждет команды от пользователя? Например, то что на школьном маке: at-f5% или если зайти в bash: bash-3.2$?

**TLDR**: Да

**A**: В большинстве современных (и не очень) оболочек `prompt` берется из переменной окружения `PS1`, а затем перед каждым выводом прогоняется через внутренний механизм оценки переменных.

Пример:

```bash
bash-3.2$ export PS1='$(pwd): '
/Users/User/Documents: cd ..
/Users/User:
```

*Comment*: Данный функционал не является обязательным в рамках проекта

---

**Q**: При возврашение readline-ом NULL указателя можно ли сразу завершать программу без захода в лексер, парсер и т.д?

**A**: Да

---

**Q**: В сабжекте написано: иметь рабочую историю. Что кроме добавления команд и очищения  истории ещё надо делать?

**A**: Ничего, история в библиоте `readline` работает из-коробки. Просто добавьте `add_history(line)` после прочтения непустой строки. И не забудьте очистить историю при выходе из оболочки `rl_clear_history()`

---

**Q**: Не подключается функция `rl_clear_history()` (ругается компилятор)
при подключенных библиотеках
`#include <readline/readline.h>`
`#include <readline/history.h>`

**TLDR**: Ошибка на этапе сопоставления зависимостей

**A**: Добавьте флаги в `C_FLAGS` в Вашем `Makefile`

```bash
# Пояснение:
# $(USER) нужно чтобы у любого юзера искал подобный путь
# 8.1.* любой патч с мажорной версией 8 и минорной 1, вдруг у Вас с тиммейтом разные версии readline

-I /Users/$(USER)/.brew/Cellar/readline/8.1.*/include/ -lreadline -L /Users/$(USER)/.brew/Cellar/readline/8.1.*/lib/
```

<br />

## Built-in's

**Q**: Что значит чтение из `HEREDOC` и как это работает?

**TLDR**: Передача текста в `stdin` процесса, отвечающего за команду

Пример 1:
```bash
bash-3.2$ cat << Cool-Delimiter
> Text 1
> Text 2
> Cool-Delimiter
Text 1
Text 2
```

Пример 2:
```bash
# Да, bash и так умеет, но в сабжекте ничего не написано про раскрытие переменных и тем более вызов сабшеллов

# 15/07/2022 en.subject.pdf
# << should be given a delimiter, then read the input until a line containing the delimiter is seen.
# However, it doesn’t have to update the history!

# Следом за << должен быть разделитель, а после поток ввода должен считываться до указанного разделителя.
# Данный процесс не должен обновлять историю (введенным текстом, команда должна быть в истории)

bash-3.2$ cat << Cool-Delimiter
> echo $HOME $(ls)
> Cool-Delimiter
echo /Users/user file1
folder2
```

**A**: Вам потребуется создать временный файл (где-то), после чего считать туда поток ввода. После всех операций временный файл необходимо удалить. EOF закрывает поток ввода без ошибок.

---

**Q**: Если built-in'ы встроенные команды, то мы их симулируем полностью сами и запускаем как часть нашей программы? В таком случае как понять какой exit code?

**TLDR**: Да, они должны быть реализованны как функции, но есть условности; exit code зависит от того, как завершился билт-ин и как он был вызван :)

**A**: Если подается одна built-in команда, то исполняется она в текущем процессе шелла;

```c
exit_code = built_in_func(params);
```

Если built-in появляется в пайпе, то под него форкается отдельный процесс шелла, для полноценного перенаправления потоков ввода/вывода.

`exit` возвращает код в соответствии с аргументом. Рейндж кодов `0-255` (модуль 256)

`exit` в пайпе просто возвращает код, а не выходит из текущего шелла.

```bash
bash-3.2$ echo Before | exit 213 && echo After
# Здесь пусто
bash-3.2$ echo Before | exit 0 && echo After
After
```

---

**Q**: Что делает команда `export` без аргументов? Что значит `declare -x` при выводе?

**TLDR**: `export` без аргументов выводит непустые переменные окружения (не путать с переменными шелла); `declare` задает переменные шелла (флаг `-x` превращает их в переменные окружения)

**A**: `declare -x` выводится для миграции переменных между окружениями через файл (например)

*Comment*: Сортировать вывод `export`, как это делает bash, не обязательно

<br />

# Другое

**Q**: На каких моментах чаще всего застревают?

**TLDR**: Организационные вопросы; сборка проекта воедино при совместной работе

**A**:

- На взаимодействии с тиммейтом (один все делает, второй не попадает в сроки/ничего не делает)
- На сопоставлении двух частей (один сделал так, а второй думал что будет по-другому)

Обсуждайте сроки, обязанности, структуру проекта или будьте готовы делать все в одиночку

---

**Q**: Возможно ли на школьных маках дебажить дочерние процессы?

**TLDR**: Используйте `gdb`

**A**: Чтобы установить `gdb` используйте `brew`. Как использовать `gdb`? Гуглите 🙂. Не хватает прав? Используйте <img src="https://github.com/devicons/devicon/raw/master/icons/docker/docker-original.svg" width="15" height="15"> Docker / Virtual Box

https://github.com/sickcodes/Docker-OSX

Команды, которые нужно прописать ДО перехода в `fork()`
```
set follow-fork-mode child
set detach-on-fork off
```

---
