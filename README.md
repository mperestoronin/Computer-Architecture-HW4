# Computer-Architecture-HW4
### Выполнил Пересторонин Максим БПИ 217 Вариант 22 на оценку 8
Четвертое ДЗ по предмету "Архитектура вычислительных систем"
<br>Surprisingly, no assembler this time :)
## Общие сведения:
### Условье задачи:
```
Задача об инвентаризации по книгам. После нового года в библиотеке университета обнаружилась пропажа каталога.
После поиска и наказания, виноватых ректор дал указание восстановить каталог силами студентов.
Фонд библиотека представляет собой прямоугольное помещение, в котором находится M рядов по N шкафов по K книг в каждом шкафу.
Требуется создать многопоточное приложение, составляющее каталог.
При решении задачи использовать метод «портфель задач», причем в качестве отдельной задачи задается внесение в каталог записи
об отдельной книге.
Примечание.
Каталог — это список книг, упорядоченный по их инвентарному номеру или по алфавиту (на выбор разработчика).
Каждая строка каталога содержит идентифицирующее значение (номер или название), номер ряда, номер
шкафа, номер книги в шкафу.
```
### На чем написана программа?
Я использовал c++ 17, писал в Clion, которая использует g++, операционная система Windows 11, тестировал на Linux
### Как работает ввод данных?
Для выбора режима работы программы необходимо ввести нужное количество аргументов командной строки:
<br>Вот как [я это делал](https://github.com/mperestoronin/Computer-Architecture-HW4/blob/main/commandlineArgs.md) в CLion.
<br>В зависимости от количества аргументов командной строки выбирается способ получения входных данных для работы программы.
* Если не вводить ни одного аргумента, то автоматически включится консольный ввод данных.
* Если предоставить один аргумент, то программа сгенерирует данные сама. Аргумент должен содержать имя файла, куда вы хотите записать результат работы программы.
* При вводе двух аргументов, программа войдет в режим ввода и вывода данных через файл. Первый аргумент - файл с входными данными, содержащий 3 числа, разделяемых пробелом - количество рядов, шкафов с книгами и непосредственно самих книг. Второй файл - файл для вывода данных. 
* Если же вы предоставите три входных параметра, то программа будет получать данные через командную строку. Сами параметры - это числа M N K строго в этом порядке, где M - кол-во рядов, N - кол-во шкафов в ряду и K - количество книг в шкафу.
### Где создавать файлы для ввода/вывода данных?
Файлы для ввода и вывода данных нужно расположить в папке cmake-build-debug.
### Что если ввести некорректные входные данные?
В случае ввода некорректных значений M, N, K или передачи некорректного названия файла/файлов через аргумент(ы) командной строки, программа досрочно завершит свою работу.
### Входные данные
Если не считать имена файлов, описанных выше, входные данные представляют собой три положительных числа:
<br>M - кол-во рядов в библиотеке
<br>N - кол-во шкафов в каждом ряду библиотеки
<br>K - кол-во книг в каждом шкафу
Вне зависимости от режима ввода, данные нужно вводить именно в этом порядке.
Очевидно, что каждое из вышеперечисленных чисел должно быть положительным, иначе задача либо теряет смысл (в библиотеке 0 потерянных книг), либо полностью теряет логику (потерялось отрицательное количество книг). Ограничивать ввод сверху я не стал, так интересней наблюдать за работой программы. Однако на всякий случай, будем считать данные корректными, если они находятся в диапазоне от 1 до 100.
### Вывод данных
Во всех режимах ввода данных результат работы программы будет выводиться в консоль. Существует критерий:
```
Результаты работы программы должны выводиться на экран и записываться в файл.
```
Это легко осуществимо для режимов, где файл для вывода передается через аргументы командной строки. Однако в случае с консольным режимом, никакие аргументы не передаются, поэтому при успешном завершении работы программы пользователя попросят ввести путь к файлу для записи данных.
### Комментарии
В программе большое количество разнообразных комментариев, постарался максимально описать все основные моменты, чтобы повысить читаемость кода.
### Описание работы программы
Пользователь указывает "размеры" библиотеки. Главный поток создаёт портфель, формирует список задач, а также ставит цель - заполнить всю библиотеку (максимальное количество задач). После чего студенты (потоки) начинают возвращать книги в библиотеку и писать об этом в отчете. Поток считывает информацию из портфеля задач и повышает его счетчик:
``` cpp
arguments->briefcase += 1;
```
После чего поток "возвращает книги в библиотеку", вписывая данные о новой книге в каталог по старому номеру портфеля задач используя координаты книги, хранящиеся в списке задач:
``` cpp
arguments->catalog[old_briefcase] = Book{std::to_string(old_briefcase + 1), name, coordinates[0] + 1, coordinates[1] + 1, coordinates[2] + 1};
```
Поток останавливает только тогда, когда портфель задач достигает максимального количества задач (библиотека заполнена)
После чего главный поток выводит необходимую информацию в консоль и файл.
Как и требовалось в условии, для решения данной задачи я использовал портфель задач:
```
Метод портфеля задач заключается в поддержании очереди из задач.
Каждый свободный поток берёт задачу из портфеля, выполняет её, при необходимости генерируя новые подзадачи и помещая их в портфель.
Одним из общих подходов к динамическому распределению задач (данных) по процессам (потокам) является портфель задач.
В этом подходе явно прослеживается единство параллелизма данных и задач.
Вычислительная задача делится на конечное число подзадач, являющихся независимыми единицами работы.
Каждая подзадача может выполнить однотипные действия над разными данными (яркий пример – получение элемента произведения матриц).
Подзадачи нумеруются, т. е. каждому номеру соответствует свой набор данных, точнее над множеством номеров задач определяется функция,
которая однозначно отражает номер задачи на соответствующий ему набор данных.
Создается переменная или, если нумерация сложная, то структура для хранения номера задачи,
которую следует выполнять следующей (например, номер столбца, в котором надо посчитать элементы произведения).
Каждый поток независимо выполняет следующую цепочку действий: 
1) обращается к портфелю задач для выяснения текущего номера задачи;
2) увеличивает портфель задач на единицу;
3) выполняет задачу, используя соответствующие данные;
4) обращается к портфелю задач для получения следующего номера задачи.
Должен быть предусмотрен механизм остановки процессов при исчерпывании всего множества задач.
```
[Источник](https://studfile.net/preview/16404441/page:6/) Карепова Е.Д Основы многопоточного и параллельного программирования (стр 43)
## Тесты:
Тестовое покрытие предоставлено [тут](https://github.com/mperestoronin/Computer-Architecture-HW4/blob/main/Tests.md).
