###  OTUS Linux Professional Lesson #8 | Subject: Практические навыки работы с ZFS
#### Домашнее задание

####  Цель:
Отработать навыки работы с созданием томов export/import и установкой параметров;

определить алгоритм с наилучшим сжатием;
определить настройки pool’a;
найти сообщение от преподавателей.
составить список команд, которыми получен результат с их выводами.

Описание/Пошаговая инструкция выполнения домашнего задания:
Для выполнения домашнего задания используйте методичку

### Практические навыки работы с ZFS

####  Что нужно сделать?

1. Определить алгоритм с наилучшим сжатием:
2. Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
создать 4 файловых системы на каждой применить свой алгоритм сжатия;
для сжатия использовать либо текстовый файл, либо группу файлов.
Определить настройки пула.
С помощью команды zfs import собрать pool ZFS.
Командами zfs определить настройки:
   
- размер хранилища;
    
- тип pool;
    
- значение recordsize;
   
- какое сжатие используется;
   
- какая контрольная сумма используется.
3. Работа со снапшотами:
скопировать файл из удаленной директории;
восстановить файл локально. zfs receive;
найти зашифрованное сообщение в файле secret_message.

Если возникнут вопросы, обращайтесь к студентам, преподавателям и наставникам в канал группы в Telegram.

Удачи при выполнении!

#### Критерии оценки:
Статус «Принято» ставится при выполнении следующих условий:

Сcылка на репозиторий GitHub.
Vagrantfile с Bash-скриптом, который будет конфигурировать сервер.
Документация по каждому заданию:
Создайте файл README.md и снабдите его следующей информацией:
название выполняемого задания;
текст задания;
описание команд и их вывод;
особенности проектирования и реализации решения;
заметки, если считаете, что имеет смысл их зафиксировать в репозитории.


#### Выполнение
