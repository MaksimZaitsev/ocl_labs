# Содержание

* [Тезаурус](#Тезаурус1)
* [Подготовка](#Подготовка)
* [lab0: получение сведений о платформе и устройствах](#lab0-получение-сведений-о-платформе-и-устройствах)
    * [Контрольные вопросы](#control_questions_0)
    * [Задание](#task_0)
* [lab1: выполнение ядра на устройстве](#lab1-выполнение-ядра-на-устройстве)
    * [Задание 1](#task_1_1)
    * [Задание 2](#task_1_2)
    * [Задание 3](#task_1_3)
* [Список источников](#Список-источников)

## Тезаурус<sup>[(1)](#footnote_1)</sup>

**Платформой** называется сочетание из *хоста* и набора *устройств*, управляемых *фреймворком OpenCL*, позволяющим приложению разделять ресурсы и выполнять *ядра* на *устройствах* в этой платформе.

**Устройство** — это набор вычислительных блоков. Используется очередь команд для добавления в порядке FIFO команд для их выполнения устройством. Примерами служат команды для выполнения *ядер*, чтение/запись над *объектами памяти*.

**Ядром** является функция, определенная в *программе* и исполняемая на устройстве. Ядро отличается от других функций спецификатором `__kernel`.

**Программа** — программа, написанная на языке OpenCL C (специальном расширении языка C99), включающая множество ядер. Программа может содержать вспомогательные функции, вызываемые `__kernel` функциями, а также неизменяемые данные.

**Фреймворк** — это система программного обеспечения, содержащая набор компонентов для поддержки разработки ПО и его выполнения. Фреймворк обычно включает библиотеки, программные интерфейсы приложения (APIs), системы времени исполнения, компиляторы и т.д.

**Хост** — это та часть гетерогенной системы, которая взаимодействует с *контекстом* через OpenCL API.

**Контекст** — это окружение, в котором выполняются ядра, и область, внутри которой определены синхронизация и управление памятью. Контекст включает множество устройств, память (доступную этим устройствам), соответствующие свойства памяти и одна или более очередей команд, используемых для планирования выполнения одного или нескольких ядер или операций над *объектами памяти*.

**Объект памяти** — это описатель глобальной области памяти с подсчетом ссылок (после обнуления счетчика память разрешено освободить).

 > <a name="footnote_1"></a><sup>(1)</sup> Определения взяты из [\[1\]](#src_1), раздел «2. Glossary».

## Подготовка

В каталоге D:\Student\ocl_labs расположены предкомпилированные OpenCL ядра и  все необходимые исходные коды. Структура директории:
* CmakeLists.txt — файл для автоматизации сборки;
* toolchain.cmake — файл с указанием сведений о целевой платформе и используемых компиляторах;
* common/ — исходные коды, используемые в разных работах;
* kernels/ — содержит предкомпилированные ядра;
* lab*/ — каталоги одноименных работ.

Потребуется трансляция программ работ, поэтому запустите bash (Win+R -> bash) и выполните команды ниже для инициализации CMake:

```shell
$ cd /d/Student/ocl_labs/
$ mkdir build && cd build # здесь создаем *каталог сборки*
$ cmake .. -G "MSYS Makefiles" -DCMAKE_TOOLCHAIN_FILE="../toolchain.cmake"
```

В директории build сгенерируется Makefile, с помощью которого будут собираться программы работ.

Убедитесь, что на обратной стороне отладочной платы DE1-SOC переключатели MSEL[4:0] выставлены в положении «01010» — это означает, что ПЛИС будет конфигурироваться аппаратной процессорной системой на базе ARM-процессора.

![Настройка переключателей MSEL платы DE1-SOC](https://user-images.githubusercontent.com/31930170/40394454-3907c65c-5e2c-11e8-8c0d-2e7296d86e32.png)

Для переноса  исполняемых файлов на отладочный комплекс понадобится USB флеш накопитель. На нем рекомендуется создать директорию ocl_labs, где будут размещаться исполняемые файлы.

Если в ходе выполнения заданий возникнут затруднения, то можно посмотреть вариант решения. Для примера возьмем [«Задание 1»](#task_1_1), второй вариант, работа lab1. Согласно условию, нужно изменить код файла lab1/host.c. Введите `git status`, находясь в каталоге сборки. Найдите имя подходящего patch-файла, записанного в формате: `lab<номер_работы>/<имя_файла>_t<номер_задачи>[o<номер_варианта>].patch`. В данном случае нас интересует `lab1/host_t1o2`. Введите `git checkout -- lab1/host.c lab1/host_t1o2`, чтобы восстановить редактируемый файл (здесь — host.c) и сам patch-файл, а затем выполните `patch lab1/host.c < lab1/host_t1o2`, чтобы применить изменения.

## lab0: получение сведений о платформе и устройствах

Программа в каталоге lab0 выводит сведения о платформе и всех ее устройствах, доступных для OpenCL. Платформа похожа на драйвер тем, что предоставляет поддерживаемые устройства для использования посредством OpenCL Runtime API. Так, например, в одном ПК может быть платформа от AMD для одной видеокарты и одного процессора, а также платформа от nVIDIA для второй видеокарты.

Первая демонстрационная программа состоит только из хостовой части. Для сборки выполните `make lab0`, находясь в каталоге сборки build. В build/lab0/ будет соответствующий исполняемый файл. Не обращайте внимание на предупреждение компилятора об отсутствии динамической библиотеки.

Для запуска lab0 выполните следующие шаги:

1. Подключите плату к компьютеру через UART-to-USB разъем.
2. Запустите «Диспетчер устройств» (например, с помощью Win+R, введя имя devmgmt.msc). В категории «Порты (COM и LPT) найдите «USB Serial Port (COMn)», где n – номер устройства.

    ![Выбор USB Serial устройства для подключения к плате](https://user-images.githubusercontent.com/31930170/40394456-393e32c8-5e2c-11e8-8c8f-7530a9a43e4c.png)

3. Откройте PuTTY, выберете последовательный тип подключения, укажите название устройства (в данном случае — COM7) и скорость — 115200. Откройте соединение с помощью кнопки «Open» (в появившейся консоли пока ничего нет).

    ![Настройка putty](https://user-images.githubusercontent.com/31930170/40394455-392357c8-5e2c-11e8-8ef2-8a0cc3c4a6cc.png)

4. Подключите к плате питание и нажмите на кнопку включения. В окне с открытой сессией терминального доступа будет выводиться информация о ходе загрузки системы.
5. Когда появится приглашение, выполните вход под пользователем root (пароль не установлен).
6. Выполните `source ./init_opencl.sh`, чтобы загрузить драйвер OpenCL и установить переменные среды окружения для предустановленной библиотеки времени исполнения OpenCL.
7. Загрузите на USB накопитель исполняемый файл lab0. Подключите его к плате и смонтируйте, например, так:

    ```shell
    $ mkdir /mnt/usb
    $ lsblk # узнайте название устройства, пусть оно будет sda1
    $ mount /dev/sda1 /mnt/usb
    $ #umount /mnt/usb #для размонтирования, при этом нельзя находиться на носителе
    ```

    **Замечание**: рекомендуется прописать следующие псевдонимы:

    ```shell
    $ alias mount='mount /dev/sda1 /mnt/usb/'
    $ alias umount='umount /mnt/usb'
    $ alias usb='cd /mnt/usb/ocl_labs'
    ```

8. В директории с расположением исполняемого файла запустите программу,  введя `./lab0`.

Программа использует всего четыре вызова OpenCL API:

* clGetPlatformIDs(…) — извлечение списка платформ;
* clGetPlatfromInfo(…) — получение свойства платформы, соответствующего передаваемой константе из перечисления cl_platform_info;
* clGetDeviceIDs(…) — извлечение списка устройств платформы, описатель которой передается в качестве аргумента функции;
* clGetDeviceInfo(…) — получение свойства устройства, соответствующего передаваемой константе из перечисления cl_device_info.

Потребуется обратиться к спецификации [\[1\]](#src_1) для того, чтобы ответить на контрольные вопросы и выполнить задания. В разделе «4.1. Querying Platform Info» на стр. 29  приводятся сведения о вызове clGetPlatformInfo(…), а в Табл. 4.1 перечислены возможные константы, тип возвращаемого значения и описание. В разделе «4.2 Querying Devices» дано описание вызова clGetDeviceInfo(…), а все возможные константы для получения информации об устройстве представлены в Табл. 4.3.

#### <a name="control_questions_0"></a>Контрольные вопросы

*Изучите вывод программы. Сколько платформ и устройств имеется на отладочной плате? Что такое профили OpenCL, и к какому из них принадлежит DE1-SOC? Как формируется список поддерживаемых расширений платформы: что он в себя включает, когда устройства платформы имеют разные списки расширений?*

Обратите внимание, что устройство содержит только один вычислительный блок — согласно свойству CL_DEVICE_MAX_WORK_GROUP_SIZE. Однако вычислительный блок не реализован в виде фиксированного аппаратного устройства, которым разработчик желает воспользоваться в полном объеме для достижения максимальной производительности. Напротив, вычислительный блок является гибким и формируется с учетом ядра [\[2\]](#src_2).

#### <a name="task_0"></a>Задание

*Измените исходный код программы lab0 (файл host.c, где оставлено место, обозначенное словами «type here…»), чтобы выяснить свойства устройства:*
    
* *вариант 1 — поддерживается ли работа с изображениями?*
* *вариант 2 — какой порядок байт используется устройством?*
* *вариант 3 — какое разрешение у таймера профилирования?*
    
Посмотрите, как выполняются вызовы clGetDeviceInfo(…) в программе. Внимательнее изучите прототип функции в спецификации и поищите<sup>[(2)](#footnote_2)</sup> нужные константы в Табл. 4.3. После исправлений в коде программы достаточно выполнить make lab0 повторно. Проверьте вывод программы после запуска ее на плате.

> <a name="footnote_2"></a><sup>(2)</sup> Список свойств довольно большой, поэтому подсказка: для первых двух вариантов требуемые свойства запросят информацию с типом `cl_bool`, а свойство для третьего варианта соответствует типу `size_t`.

## lab1: выполнение ядра на устройстве

Демонстрационная программа lab1 состоит уже из двух частей: одна будет выполняться на хосте, а другая — работать на устройстве. Такая программа обычно включает в себя 13 шагов (не строго в указанном порядке, причем некоторые могут отсутствовать за ненадобностью):

1. включение заголовочного файла «CL/opencl.h»;
2. выделение памяти для данных хоста;
3. создание контекста — clCreateContext(…);
4. создание очереди команд — clCreateCommandQueue(…);
5. выделение памяти устройства (создание объектов памяти) — clCreateBuffer(…);
6. передача обрабатываемых данных с хоста на устройства — clEnqueueWriteBuffer(…);
7. чтение предкомилированного файла ядра во временный буфер хоста;
8. сборка программы — clCreateProgramWithBinary(…) и  clBuildProgram(…);
9. создание объекта ядра из программы — clCreateKernel(…);
10. задание аргументов, передаваемых ядру — clCreateKernelArg(…);
11. добавление ядра в очередь для выполнения — clEnqueueTask(…) или clEnqueueNDRangeKernel(…);
12. чтение результата из устройства (объекта памяти) — clEnqueueReadBuffer(…);
13. освобождение ресурсов — семейство функций с именем clRelease*(…).

Просмотрите обзорно код хостовой части работы lab1, чтобы пронаблюдать перечисленные выше шаги. В командном интерпретаторе, находясь в build, выполните `make lab1`. Скопируйте соответствующий исполняемый файл на USB носитель, а также предкомпилированное ядро lab1.aocx, размещенное в каталоге kernels. Запустите эту программу на плате.

Код ядра представлен в lab1/core.cl и продублирован ниже:

```opencl
__kernel void
inout(__global int* out, int in)
{
    *out = in; // out[0] = in;
}
```

Функция ядра называется «inout» и имеет спецификатор `__kernel`. Тип возвращаемого значения должен быть `void`. Первый принимаемый аргумент — указатель на глобальную область памяти, второй — простое целочисленное значение, неявно заданное со спецификатором `__private`. Все, что делает это ядро — получает целочисленное значение in и записывает его в глобальную область памяти через указатель, так что хост может считать значение обратно из объекта памяти buf_out, который передавался первым аргументом и создан вызовом clCreateBuffer(…).

При создании очереди команд было включено профилирование (свойство CL_QUEUE_PROFILING_ENABLE). Обратитесь к разделу «5.9 Profiling Operations on Memory Objects and Kernels», чтобы узнать, какую информацию позволяет получить профилирование и выполнить следующее задание.

#### <a name="task_1_1"></a>Задание 1

*Определите время, затраченное на завершение выполнения ядра, начиная с:*
* *вариант 1 — момента постановки ядра в очередь команд;*
* *вариант 2 — момента отправки ядра на устройство;*
* *вариант 3 — момента начала выполнения ядра.*

Для этого найдите в тексте lab1/host.c участки, помеченные как «task1». Описатель события kernel_event типа `cl_event` позволит получить сведения о работе ядра. kernel_event нужно добавить в последний аргумент вызова clEnqueueTask(). Используйте функцию clGetEventProfilingInfo(…), чтобы получить профилировочную информацию о команде, связанной с событием. Выводите результат в микросекундах. Повторите сборку получившейся программы и запустите ее на плате.

#### <a name="task_1_2"></a>Задание 2

*Выведите два числа 0xF00D и 0xBEEF, полученные из глобальной памяти устройства, с помощью двух запусков ядра, передавая соответствующие целочисленные значения в качестве второго аргумента ядра.
Профилирование можно убрать (вернуть код программы в исходное состояние). Достаточно немного модифицировать код для того, чтобы установить второй аргумент для первого запуска ядра 0xF00D, а для второго раза — 0xBEEF. Очередь комманд схематично будет выглядеть примерно так:*

![Предполагаемая схема очереди комманд](https://user-images.githubusercontent.com/31930170/40394453-38ec4ac6-5e2c-11e8-9582-96ce4aab0ae8.png)

*Т.е. не надо дожидаться завершения ядра с одним аргументом и выводить возвращаемый результат, после чего запускать ядро с другим аргументом. При этом первый аргумент — объект памяти — повторно нельзя использовать, иначе результат первого выполнения будет утерян. Таким образом, понадобятся два объекта памяти.*

Ядра NDRange позволяют воспользоваться параллелелизмом данных — когда весь набор входных значений разбивается на более мелкие части, обрабатываемые одновременно путем выполнения одинаковой операции над каждой частью *в отдельном потоке*. Такой поток называют рабочим элементом (work-item), выполняющийся независимо друг от друга, а вся задача целиком называется NDRange (N-dimensional range). Размерность NDRange может быть в диапазоне от 1 до 3. Т.е. можно представить NDRange как одномерный массив рабочих элементов, если *данные* удобно представить в таком виде (сложение векторов), или как двумерный (умножение матриц). У рабочих элементов есть порядковые номера (id), которые уникальны для всех элементов устройства. Получить такой идентификатор элемента можно с помощью get_global_id(N), где N - размерность (число от 0 до 2). Рабочие элементы могут объединяться в рабочую группу (work-group), что позволяет им производить синхронизацию между собой и делить локальную память, которая для каждой рабочей группы своя. В целом, рабочие группы дают возможность декомпозировать задачу. Идентификаторы рабочих элементов внутри рабочих групп можно выяснить с помощью get_local_id(N).

![Идентификаторы рабочих элементов и групп](https://us.fixstars.com/images/openclbook/dtp_462724_USER_CONTENT_0_html_3bb3f126.jpg)

На изображении выше, взятом из [\[7\]](#src_7), приведена взаимосвязь между глобальными и локальными идентификаторами рабочих элементов, а также идентификаторами рабочих групп.

#### <a name="task_1_3"></a>Задание 3

*Передайте в ядро два числа 0xF00D и 0xBEEF и выведите их с помощью одного запуска ядра путем вызова clEnqueueNDRangeKernel(…). Сделайте так, чтобы всего было два рабочих элемента, и каждый бы записал в выходной буфер только одно целочисленное значение, прочитанное из входного буфера.*

В этом задании мы имеем дело с двумя одномерными массивами (объектами памяти) типа `cl_mem` — например, buf_out и buf_in. Первый объект памяти служит для записи результата выполнения ядра. Второй объект — для чтения значения внутри тела функции-ядра. Поскольку всего требуется «обработать» два независимых числа, а результат записать в две непересекающиеся ячейки памяти, то можно воспользоваться параллелизмом по данным. Функция get_global_id(0) вернет индекс 0 для ядра, выполняемого на первом рабочем элементе, и индекс 1 — для второго. Поэтому индексы можно использовать для получения данных из buf_in и записи в buf_out.


Напишите сперва код ядра. Чтобы его скомпилировать, откройте новое окно терминала, перейдите в lab1, и выполните команду
`aoc core.cl --board socde1_sharedonly -v -o ../build/lab1_task3.aocx`, где
 
* core.cl — файл с исходным кодом ядра;
* --board — выбор платы (посмотреть список плат — `aoc --board-list`);
* -v — показывать прогресс выполнения компиляции;
* -o — позволяет указать путь, где будет находиться ядро и другие артефакты. Например, если ядро называется lab1.aocx, то будет создан каталог lab1, внутри которого есть файлы журналирования процесса сборки (с расширением .log).

Пока идет компиляция, исправьте код хостовой части программы. В памяти хоста следует создать два массива: под входные данные и под результат. Необходимо воспользоваться вызовом clEnqueueWriteBuffer(…) над объектом памяти buf_in для записи передаваемых из хоста в ядро значений. Чтобы проверить работу программы на плате, передайте в качестве входного аргумента для lab1 путь к новому ядру.

## Список источников

1. <a name="src_1"></a>[Спецификация OpenCL 1.0](https://www.khronos.org/registry/OpenCL/specs/opencl-1.0.pdf)
2. <a name="src_2"></a>[Accelerating Workloads on FPGAs via OpenCL: A Case Study with OpenDwarfs](https://pdfs.semanticscholar.org/eece/99db75b843c7c24db2ca6b9acd2daa260eca.pdf)
3. <a name="src_3"></a>[DE1-SOC User Manual](http://www.terasic.com.tw/cgi-bin/page/archive_download.pl?Language=English&No=836&FID=3a3708b0790bb9c721f94909c5ac96d6)
4. <a name="src_4"></a>[DE1-SoC OpenCL User Manual](http://www.terasic.com.tw/attachment/archive/836/DE1SOC_OpenCL_v02.pdf)
5. <a name="src_5"></a>[OpenCL (1.2) high level overview](https://www.youtube.com/watch?v=8D6yhpiQVVI)
6. <a name="src_6"></a>[OpenCL (1.2) C](https://www.youtube.com/watch?v=RKyhHonQMbw)
7. <a name="src_7"></a>[The OpenCL Programming Book](https://us.fixstars.com/opencl/book/OpenCLProgrammingBook/calling-the-kernel/)