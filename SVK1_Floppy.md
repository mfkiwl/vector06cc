
```
              КОНТРОЛЛЕР НАКОПИТЕЛЯ НА ГИБКИХ МАГНИТНЫХ ДИСКАХ

    Дисковод является наиболее сложным из внешних устройств ПК.  Достаточно
сказать, что не существует единого способа подключения контроллера дисково-
да. Автору известны по крайней мере 4 варианта  интерфейса  контроллера  (в
дальнейшем - КД). Большинство изготовителей  придерживаются  "Кишиневского"
стандарта, рекомендованного разработчиками ПК. Незначительно отличается  от
него вариант Sphere+, распространяемый одноименной фирмой (возможно  созда-
ние универсальных программ). Контроллер дисковода,  выпускаемый  московской
фирмой Coman, отличается от обоих указанных выше вариантов. Существует  еще
вариант Омского центра программирования, способный работать на "Кристе-2" и
совместимый снизу вверх с "Кишиневским" вариантом.
    Все упомянутые контроллеры построены на основе БИС КР1818ВГ93.
    Микросхема КР1818ВГ93 представляет собой однокристальное  программируе-
мое устройство, предназначенное для управления  обменом  информацией  между
ЭВМ и накопителями на гибких магнитных дисках. БИС обеспечивает  программи-
рование номера дорожки, сектора и стороны диска, а также длины сектора, ре-
жимов поиска дорожки и установки магнитной головки  в  исходное  состояние,
режимов записи/чтения, скорости перемещения МГ.  Обмен  информацией  с  ЭВМ
происходит по восьмиразрядной двунаправленной шине данных.  Запись  осущес-
твляется с одинарной (английская аббревиатура - FM) или двойной (MFM) плот-
ностью. Поддерживаются дисководы 5.25" и 8". БИС не содержит схем  управле-
ния двигателем и выбора накопителя. Этим должно заниматься специальное  ус-
тройство управления.
    БИС содержит, как говорилось, 5  внутренних  регистров.  Их  назначение
следующее:
 - Data    (запись, чтение) - служит для обмена данными между ЭВМ и БИС.
 - Track   (запись, чтение) - содержит номер текущей дорожки.
 - Sector  (запись, чтение) - содержит номер сектора.
 - Status  (чтение)         - содержит результат выполнения команды.
 - Command (запись)         - служит для задания команды БИС.
Примечание. Чтение  Status возможно не ранее чем через 30мс после записи ко-
манды.

    Микросхема обеспечивает прием и выполнение 11 команд. Все  команды  ус-
ловно разделены на 4 типа: вспомогательные,  записи  и  чтения  информации,
поиска и чтения индексного поля и принудительного прерывания.

1. Вспомогательные команды.

    Команды этого типа выполняются вне зависимости  от  сигнала  готовности
накопителя.



    Команда RESTORE обеспечивает переход МГ на нулевую дорожку. Если  нуле-
вая дорожка не достигнута после 256 шагов, выполнение команды прекращается.
    Команда SEEK предполагает, что регистр Data  содержит  номер  требуемой
дорожки, а Track - текущей. Перемещение МГ выполняется до тех пор, пока  не
будет достигнута требуемая дорожка. По окончании регистр Track содержит но-
вый номер текущей дорожки.
    Команда STEP обеспечивает перемещение головки на 1 шаг. Направление пе-
ремещения совпадает с направлением предыдущего движения головки.
    Команды STEPF и STEPB обеспечивают перемещение головки на 1  шаг  соот-
ветственно вперед или назад.

Состояние битов регистра Status в процессе и после выполнения этих команд:

R7: 1=накопитель не готов.
R6: 1=дискета защищена от записи.
R5: 1=головка прижата.
R4: 1=ошибка поиска.
R3: 1=ошибка в контрольном коде заголовка сектора
R2: 1=МГ на нулевой дорожке.
R1: 1=индексный импульс.
R0: 1=идет выполнение команды.


2. Команды чтения и записи данных.

    Перед выполнением этих команд необходимо в регистры Track и Sector  за-
писать номер требуемой дорожки и сектора соответственно. Длина сектора  за-
дается кодом в индексной области (заголовке):

00 = 128  байт на сектор;
01 = 256  байт на сектор;
02 = 512  байт на сектор;
03 = 1024 байт на сектор.

    Команда READSCT выполняется, когда прочитан заголовок сектора  с  пара-
метрами, совпадающими с запрошенными в Track и Sector. Если за 10  оборотов
диска запрошенный сектор не найден, вырабатывается признак СЕКТОР НЕ НАЙДЕН
и прекращается выполнение команды. Если сектор найден, он  байт  за  байтом
считывается и передается в регистр Data. В момент передачи  устанавливается
сигнал DRQ и признак ЗАПРОС ДАННЫХ в Status. Регистр Data должен быть  счи-
тан в течение 20мкс после установки признака. Если это не произойдет, то  в
Data записывается следующий байт и вырабатывается признак ПОТЕРЯ ДАННЫХ.  В
конце считывания проверяется контрольная сумма. В случае несовпадения  выс-
тавляется признак ОШИБКА КОНТРОЛЬНОГО КОДА и выполнение команды прекращает-
ся, даже в случае многосекторного режима (m=1).
    Команда WRITESCT выполняется аналогично. Если регистр Data не будет за-
писан в течение 20мс после установки признака ЗАПРОС ДАННЫХ,  то  вырабаты-
вается признак ПОТЕРЯ ДАННЫХ, а на диск записывается байт нулей.

Значение битов регистра Status:

    READSCT                         WRITESECT

R7: 1=Накопитель не готов.          Накопитель не готов.
R6: всегда 0                        Защита записи
R5: 1=стертые данные                Ошибка записи
R4: 1=сектор не найден              Сектор не найден
R3: 1=ошибка в контрольном коде     Ошибка в контрольном коде
R2: 1=потеря данных                 Потеря данных
R1: 1=запрос данных                 Запрос данных
R0: 1=идет выполнение команды.      Идет выполнение команды.

3. Команды поиска и чтения индексного поля.
    Эти команды  предназначены для поиска информации на диске и для формати-
рования.
    Команда READADR предназначена для  определения  положения  головки.  По
этой команде последовательно считываются с диска и передаются в ЭВМ 6  байт
индексной области первого обнаруженного на диске  сектора:  номер  дорожки,
номер стороны, номер сектора, код длины  сектора,  двухбайтная  контрольная
сумма. В процессе выполнения этой  команды  содержимое  Sector  разрушается
(туда копируется содержимое Track). Если за 10 оборотов диска ни один  сек-
тор не будет найден, устанавливается признак СЕКТОР НЕ НАЙДЕН.
    Команда READTRK предназначена для отладочных целей. По этой команде до-
рожка считывается целиком (начало дорожки определяется  по  индексному  им-
пульсу) и передается в ЭВМ. Проверка контрольного кода не производится.
    Команда WRITETRK предназначена для форматирования дорожки. Информация в
ЭВМ для этой процедуры должно содержать все пробелы, индексные метки и т.д.
Байты F5H-FEH - служебные и предназначены для записи контрольных кодов, ме-
ток и т.п. Запись дорожки начинается в момент прихода индексного импульса.

Содержимое регистра Status:

    READADDR              READTRK               WRITETRK
R7: 1=накопитель не готов 1=накопитель не готов 1=накопитель не готов
R6: всегда 0              всегда 0              1=защита записи
R5: всегда 0              всегда 0              1=ошибка записи
R4: 1=сектор не найден    всегда 0              всегда 0
R3: 1=ошибка контроля     всегда 0              всегда 0
R2: 1=потеря данных       1=потеря данных       1=потеря данных
R1: 1=запрос данных       1=запрос данных       1=запрос данных
R0: 1=идет выполнение     1=идет выполнение     1=идет выполнение

4. Команды прерывания.

    По этим командам выполнение текущей команды прекращается и  генерирует-
ся сигнал INTRQ. В зависимости от кода команды, возможны различные  условия
генерации этого сигнала.

Коды команд:

        ┌──────────┬─────────────────────────────────┐
        │ Команда  │             Биты                │
        │          │ 7   6   5   4   3   2   1   0   │
        ├──────────┼─────────────────────────────────┤
        │ RESTORE  │ 0   0   0   0   h   V   Ч1  Ч0  │
        │ SEEK     │ 0   0   0   1   h   V   Ч1  Ч0  │
        │ STEP     │ 0   0   1   И   h   V   Ч1  Ч0  │
        │ STEPF    │ 0   1   0   И   h   V   Ч1  Ч0  │
        │ STEPB    │ 0   1   1   И   h   V   Ч1  Ч0  │
        ├──────────┼─────────────────────────────────┤
        │ READSCT  │ 1   0   0   m   S   E   C   0   │
        │ WRITESCT │ 1   0   1   m   S   E   C   а0  │
        ├──────────┼─────────────────────────────────┤
        │ READADR  │ 1   1   0   0   0   E   0   0   │
        │ READTRK  │ 1   1   1   0   0   E   0   0   │
        │ WRITETRK │ 1   1   1   1   0   E   0   0   │
        ├──────────┼─────────────────────────────────┤
        │ INTERRUPT│ 1   1   0   1   j3  j2  j1  j0  │
        └──────────┴─────────────────────────────────┘

  h=1 - прижимать головку, h=0 - нет.
  V=0 - не проверять положение МГ, V=1 - проверять.
  И=0 - не изменять Track во время выполнения команды; И=1 - изменять.
  m=0 - работать с одним сектором, m=1 - работать до конца дорожки.
  S=0 - нижняя сторона, S=1 - верхняя сторона.
  C=0 - не проверять совпадение стороны, C=1 - проверять.
  Е=1 - задержка 15мс после приема команды (подвод головки), Е=0-без задерж.
  Ч1=0,  Ч0=0 - время на шаг 6 мс
  Ч1=0,  Ч0=1 - время на шаг 12 мс
  Ч1=1,  Ч0=0 - время на шаг 20 мс
  Ч1=1,  Ч0=1 - время на шаг 30 мс.
  а0=0 - обычнаязапись,  а0=1 - запись стертых данных.
  j0=1 - прерывание по готовности накопителя.
  j1=1 - прерывание по неготовности накопителя.
  j2=1 - прерывание по индексному импульсу.
  j3=1 - немедленное прерывание.
      Структура информации для форматирования дорожки (MFM):

         ┌────────┬──────┬───────────────────────────────────────┐
         │ Число  │ Байт │          Назначение                   │
         │ байтов │      │                                       │
         │ десят. │      │                                       │
         ├────────┼──────┼───────────────────────────────────────┤
         │  80    │  4Е  │ Пробел от начала индексного импульса  │
         │  12    │  00  │                                       │
         │   3    │  F6  │ Запись байтов C2                      │
         │   1    │  FC  │ Индексная метка начала дорожки        │
         │  50 ─┐ │  4Е  │ Пробел перед сектором                 │
         │  12  │ │  00  │                                       │
         │   3  │ │  F5  │ Запись байтов A1                      │
         │   1  │ │  FE  │ Метка заголовка сектора               │
         │   1  │ │  ХХ  │ Номер дорожки                         │
         │   1  │ │  ХХ  │ Номер стороны                         │
         │   1  │ │  ХХ  │ Номер сектора                         │
         │   1  │ │  ХХ  │ Код длины сектора                     │
         │   1  │ │  F7  │ Запись контрольной суммы              │
         │  22  │ │  4Е  │ Пробел перед данными                  │
         │  12  │ │  00  │                                       │
         │   3  │ │  F5  │ Запись байтов A1                      │
         │   1  │ │  FB  │ Метка данных                          │
         │ ХХХ  │ │  E5  │ Сектор                                │
         │   1 ─┘ │  F7  │ Запись контрольной суммы              │
         │  ###   │      │ Запись остальных секторов             │
         │  &&&   │  4Е  │ Продолжение записи до конца дорожки.  │
         └────────┴──────┴───────────────────────────────────────┘

Назначение служебных байтов (MFM):

F5: запись байта A1, запуск вычисления контрольного кода (суммы);
F6: запись байта C2;
F7: запись контрольного кода (контрольной суммы).

    В режиме MFM на одну дорожку можно записать:

 - пять секторов размером 1024 байта;
 - девять (на некоторых накопителях - 10) секторов размером 512 байт;
 - шестнадцать секторов размером 256 байт;
 - двадцать шесть секторов размером 128 байт.

    Подробная информация о БИС КР1818ВГ93 опубликована в журнале "Микропро-
цессорные средства и системы", 3-1986, стр. 3-8.

      В ПК "Вектор" адреса регистров БИС (номера портов) следующие:

      ┌──────────┬────────────┬───────────┐
      │ Регистр  │ Порт Coman │ Остальные │
      ├──────────┼────────────┼───────────│
      │ Data     │ 9EH        │ 18H       │
      │ Sector   │ BEH        │ 19H       │
      │ Track    │ DEH        │ 1AH       │
      │ Command  │ FEH        │ 1BH       │
      │ Status   │ FEH        │ 1BH       │
      └──────────┴────────────┴───────────┘

      Кроме того, каждый контроллер содержит еще регистр управления; Sphera
  и Coman имеют второй регистр статуса.
      Контроллеры Кишиневского варианта, Омского варианта и  Кристы-2.  Но-
  мер порта управления - 1CН. Доступен только по записи.
   ┌─────┬────────────────┬──────────────────┬──────────────────────┐
   │     │ Кишиневский    │        Омский    │      Криста-2        │
   ├─────┼────────────────┴──────────────────┴──────────────────────┤
   │ R0: │          выбор накопителя (0=A/C, 1=B/D)                 │
   │ R1: │ Выбор 0=AB,1=CD│       Не задействован; всегда АВ        │
   │ R2: │    1=нижняя сторона, 0=верхняя сторона                   │
   │ R3: │           Не задействован                                │
   │ R4: │ 0=8", 1=5"     │       Не задействован; всегда 5"        │
   │ R5: │ 0=FM, 1=MFM    │       Не задействован; всегда MFM       │
   │ R6: │           Не задействован                                │
   │ R7: │ Не задействован│ Не задействован  │ 0=стандартный,       │
   │     │                │                  │ 1=совмещенный режим  │
   └─────┴────────────────┴──────────────────┴──────────────────────┘

      Для запуска двигателя необходимо записать байт в регистр  управления.
  После этого двигатель работает в течение 2.5 сек. Перезапись регистра во-
  зобновляет отсчет времени с момента последней записи.

      Контроллер фирмы Coman. Номер порта управления - 1ЕН. Доступен только по
  записи.

  R0,R1: выбор накопителя (A-D)
  R2:    сброс БИС ВГ93 (для сброса записать 0; нормально - 1)
  R3:    сигнал готовности головки (записать 1 перед выполнением команд)
  R4:    0=верхняя сторона, 1=нижняя
  R5:    не задействован
  R6:    0=MFM, 1=FM
  R7:    не задействован.
  Второй регистр статуса. Номер порта - 1ЕН. Доступен только по чтению.

  R0-R5: не задействованы
  R6:    сигнал DRQ
  R7:    сигнал INTRQ.

      Для запуска двигателя необходимо выполнить любую вспомогательную  ко-
манду с модификатором прижима головки, а затем  установить  бит  готовности
головки в регистре управления. Время выбега двигателя - 2 сек.  (10  оборо-
тов диска).


    Контроллер фирмы Sphere+.  Номер порта управления - 1СН. Доступен только
по записи.

R0,R1: выбор накопителя (A-D)
R2:    0=верхняя сторона, 1=нижняя сторона
R3:    0=выбор накопителя запрещен, 1=разрешен.
R4-R7: не задействованы.

Второй регистр статуса. Номер порта - 1СН. Доступен только по чтению.

R0,R1: не задействованы
R2:    инверсный DRQ
R3:    INTRQ
R4-R7  не задействованы

    Для запуска двигателя необходимо занести в регистр управления байт, вы-
бирающий нужный накопитель и разрешающий выбор. Двигатель накопителя  будет
работать до тех пор, пока не будет выбран другой накопитель  или  не  будет
сброшен бит разрешения выбора в регистре управления.

    Для  того,  чтобы  что-то  сделать  с  дисководом   (прочитать/записать
сектор), нужно уметь включать мотор, выбирать  накопитель,  позиционировать
головку и обмениваться данными. Однако, как правило, человек, имеющий  дис-
ковод и контроллер, использует его не просто так, а под операционной систе-
мой (напр. CP/M). И система обычно не любит, чтобы  пользовательская  прог-
рамма сама работала с регистрами контроллера - такие действия часто  закан-
чиваются "повисанием". Поэтому рекомендуется в таких  программах  сохранять
состояние всей используемой дисковой аппаратуры - регистров Sector и Track,
положение головок всех используемых накопителей.
    Вот пример программы, определяющей текущую дорожку, на которой находит-
ся головка накопителя, вне зависимости от того, какая  дискета  установлена
(даже неформатированная).

SAFE:   IN TRACK          ; Сохраняем регистр Track
        STA TRKBUF        ; в ячейке TRKBUF,
        IN SECTOR         ; а регистр Sector -
        STA SCTBUF        ; в SCTBUF.
WAIT:   CALL MOTOR        ; Запустим мотор, выберем накопитель.
                          ; Реализация этой процедуры зависит
                          ; от типа контроллера
        IN STATUS         ; Накопитель готов (дверь закрыта)?
        RLC               ; Бит готовности -> флаг переноса
        JC WAIT           ; Нет, еще не готов.
        MVI A,STEPF       ; Делаем шаг вперед
        CALL DO
        MVI B,255         ; B - счетчик шагов
LOOP:   MVI A,STEPB       ; Делаем шаг назад
        CALL DO
        INR B             ; Инкрементируем счетчик шагов.
        IN STATUS         ; Читаем результат.
        ANI 4             ; Нулевая дорожка?
        JZ LOOP           ; Нет еще.
        MOV A,B           ; Иначе B содержит номер дорожки, на которой
        STA TRK0          ; стояла головка
        CALL STOP         ; Остановим мотор (для контроллера Sphera+)
        RET               ; Закончим.
;
DO      PROC              ; Процедура выполнения вспомогательной команды
        OUT COMMAND       ; Пошлем команду в ВГ93
DO_1:   IN STATUS         ; Команда уже выполняется?
        RRC               ; Бит выполнения -> флаг переноса
        JNC DO_1          ; Еще не выполняется.
DO_2:   IN STATUS
        RRC               ; Команда еще выполняется?
        JC DO_2           ; Да.
        RET               ; Команда выполнена.
DO      ENDP

    Для восстановления положения головки можно пользоваться, например,  та-
кой программой:








REST:   CALL MOTOR        ; Запустим мотор
        MVI A,RESTORE     ; Головку - на нулевую дорожку
        CALL DO
        LDA TRK0          ; Номер требуемой дорожки ->
        OUT DATA          ; -> в регистр данных
        MVI A,SEEK
        CALL DO           ; Позиционирование на сохраненную дорожку.
        LDA TRKBUF        ; Восстанавливаем регистры
        OUT TRACK
        LDA SCTBUF
        OUT SECTOR
        RET               ; Вот и все.

    Как было сказано выше, перед началом чтения нужно установить головку на
нужную дорожку. Для этой цели можно использовать программу, аналогичную вы-
шеприведенной. Но необходимо помнить, что в дисковод может быть  установле-
на дискета не того типа,  на  который  рассчитан  накопитель,  например,  в
80-дорожечный накопитель может быть вставлена 40-дорожечная дискета. В этом
случае для перемещения головки на соседнюю дорожку необходимо перемещать ее
на 2 шага, а значение в регистре Track изменять на 1. Чтобы определить, ка-
кая дискета установлена, можно воспользоваться информацией, содержащейся  в
системной области диска. Однако этот способ не универсальный.  Гораздо  на-
дежнее, например, позиционировать головку на дорожку 2  и  прочитать  адрес
(команда READADR). Если на самом деле головка стоит на 1 дорожке -  то  это
ситуация 40дор. диск в 80дор. накопителе.
    Предположим, что Вы установили головку  на  нужную  дорожку  и  регистр
Sector содержит номер требуемого сектора. Процесс  чтения  происходит  так.
Запускается мотор дисковода. Когда накопитель будет готов,  посылается  ко-
манда READSCT. В подтверждение выполнения контроллер устанавливает бит  вы-
полнения. Теперь нужно следить за состоянием бита ЗАПРОС ДАННЫХ. Когда этот
бит равен 1, необходимо прочитать содержимое регистра Data и записать его в
память. Так продолжать до тех пор, пока бит выполнения не очистится  (можно
для проверки окончания процесса чтения проверять состояние сигнала INTRQ (в
контроллерах Coman и Sphere+), который равен 1 в момент завершения выполне-
ния команды.
    Из-за довольно низкой тактовой частоты процессора процесс чтения крити-
чен ко времени(особенно на "Кристе-2"). Во время чтения  прерывания  должны
быть запрещены. Более того, без применения программных ухищрений  невозмож-
но одновременно следить за битами ЗАНЯТО и ЗАПРОС ДАННЫХ. Как эта  ситуация
обходится - читайте ниже.
    При чтении возможно  возникновение  ошибок.  Наиболее  часто  возникают
ошибки "Сектор не найден" и "Ошибка контрольного кода". Разберемся,  почему
они возникают.
    Ошибка "Сектор не найден" возникает, если по каким-то причинам не прои-
зойдет идентификация сектора.  Это  случится,  например,  если  в  дисковод
вставлена неформатированная дискета, неверно  позиционирована  головка  или
заголовок сектора поврежден. При чтении контроллер пытается прочитать заго-
ловок сектора с требуемыми параметрами -  номером  дорожки,  стороны  (если
затребована такая проверка) и сектора. Если он не будет найден за 10 оборо-
тов диска, устанавливается признак СЕКТОР НЕ НАЙДЕН.  Поскольку  диск  вра-
щается со скоростью 5 оборотов в секунду, то для того, чтобы эта ошибка  не
приводила к повисанию, время выбега двигателя должно быть не менее 2.5  сек
(это относится только к контроллеру МикроДОС).
    ОШИБКА КОНТРОЛЬНОГО КОДА происходит, если, например, загрязнена  голов-
ка дисковода, неверная юстировка, некачественная запись, наводки, помехи  и
т.п. Иногда для ее устранения полезно делать несколько попыток чтения  сек-
тора.
    Приведем здесь два варианта программ чтения сектора. Первая из них при-
меняется в почти всех программах, работающих с диском (по-видимому,  она  -
ровесница ДОС). Вторая была разработана в Омском центре программирования  и
применяется в Универсальном загрузчике Spase.
    Будем считать, что мотор запущен и регистры  Track  и  Sector  содержат
верные параметры.

RSECT:  DI              ; Запретим прерывания - критично по времени
        LXI H,BUF       ; HL - адрес буфера, кратный 256
        MVI A,READSCT   ; Команду READSCT
        OUT COMMAND     ; пошлем в регистр команд
RRDY:   IN STATUS       ; Ожидаем начала выполнения:
        RRC             ; Выделим бит выполнения
        JNC RRDY        ; Он сброшен  - ждем.
        MVI B,2         ; Маска бита ЗАПРОС ДАННЫХ
        LDA SCTSIZE     ; Читаем код размера сектора
                        ; (должен быть записан сюда заранее)
        DCR A           ; =1 ?
        JZ R256         ; Да, сектор в 256 байт
        DCR A           ; =2 ?
        JZ R512         ; Да, сектор в 512 байт
                        ; Иначе - 1024 байта
R1024:  IN STATUS       ; Читаем статус
        ANA B           ; Выделяем бит запроса данных.
        JZ R1024        ; Он сброшен - запроса нет.
        IN DATA         ; Читаем порт данных
        MOV M,A         ; Записываем в ОЗУ
        INR L           ; Инкрементируем смещение
        JNZ R1024       ; 256 байт еще не прочитаны
        INR H           ; Читаем следующие 256 байт
R768:   IN STATUS       ; Читаем статус
        ANA B           ; Выделяем бит запроса данных.
        JZ R768         ; Он сброшен - запроса нет.
        IN DATA         ; Читаем порт данных
        MOV M,A         ; Записываем в ОЗУ
        INR L           ; Инкрементируем смещение
        JNZ R768        ; 256 байт еще не прочитаны
        INR H           ; Читаем следующие 256 байт
R512:   IN STATUS       ; Читаем статус
        ANA B           ; Выделяем бит запроса данных.
        JZ R512         ; Он сброшен - запроса нет.
        IN DATA         ; Читаем порт данных
        MOV M,A         ; Записываем в ОЗУ
        INR L           ; Инкрементируем смещение
        JNZ R512        ; 256 байт еще не прочитаны
        INR H           ; Читаем следующие 256 байт
R256:   IN STATUS       ; Читаем статус
        ANA B           ; Выделяем бит запроса данных.
        JZ R256         ; Он сброшен - запроса нет.
        IN DATA         ; Читаем порт данных
        MOV M,A         ; Записываем в ОЗУ
        INR L           ; Инкрементируем смещение
        JNZ R256        ; 256 байт еще не прочитаны
REND:   IN STATUS       ; Ждем завершения
        RRC             ; Завершено?
        JC REND         ; Нет еще. Ждем.
        IN STATUS       ; В аккумуляторе - код завершения.

    Очевидно, что приведенная выше программа обладает недостатками.  Основ-
ной из них - требование заранее знать размер сектора и размещать его по ад-
ресу, кратному 256 байт. Да и размер ее довольно велик.  Ниже  предлагается
программа, работающая по так называемому паритетному алгоритму.

RSECT:  LXI H,BUF       ; HL - адрес буфера
        DI
        MVI A,READSCT
        OUT COMMAND
RRDY:   IN STATUS
        RRC
        JNC RRDY        ; до этого момента - аналогично предыдущему
        MVI B,3         ; Маска ВЫПОЛНЕНИЕ & ЗАПРОС ДАННЫХ
        JMP LOOP2       ; Обходим участок...
LOOP1:  MOV M,A         ; Заносим байт в ОЗУ
        INX H           ; Инкремент адреса
LOOP2:  IN STATUS       ; Читаем статус
        ANA B           ; Выделим нужные биты
; Флаг паритета процессора = 0,  если только один из двух выделенных битов -
; единичный. Поскольку ситуация, при которой ЗАПРОС ДАННЫХ = 1, а ВЫПОЛНЕНИЕ
; =0 - невозможна,  то (PF=0) обозначает, что команда еще выполняется и зап-
; роса данных нет.
        JPO LOOP2
        IN DATA         ; Читаем порт данных
; Флаг паритета будет равен 1,  если либо оба выделенных бита  =1,  и  тогда
; нужно обработать запрос данных,  либо оба равны 0,  и тогда выполнение ко-
; манды закончено. Но в последнем случае дополнительно флаг Z=1. Поэтому
        JNZ LOOP1
        IN STATUS       ; На выходе в аккумуляторе - код завершения.

    Поскольку выполнение команды рано или поздно кончается,  эта  программа
не "зависает", если не найден сектор, как предыдущая. Кроме того, она  зна-
чительно короче.

    Операция записи выполняется на  всех  контроллерах,  кроме  "Криста-2",
аналогично. Приведем здесь паритетный вариант программы записи сектора.

WSECT:  LXI H,BUF
        DI
        MVI A,WRITESCT
        OUT COMMAND
        IN STATUS
        RRC
        JNC $-3
        MVI B,3
LOOP:   IN STATUS
        ANA B
        JPO LOOP
        MOV A,M         ; Эти команды
        OUT DATA        ; на флаги процессора
        INX H           ; не влияют.
        JNZ LOOP
        DCX H           ; Был лишний инкремент
        IN STATUS

```

# See also #
  * [EMULib: C-source a WD1793 emulator](http://svn.akop.org/psp/trunk/fms/EMULib/)