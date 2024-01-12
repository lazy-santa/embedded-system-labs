# **Embedded Systems Labs**
Лабораторные работы по дисциплине встраиваемые системы

## Лабораторная работа №1 работа с SPI и P10 Led Board

В ходе данной работы необходимо написать программу для управления положения круга, нарисованного на светодиодной панели, по средством наклонна панели по осям Ox и Oy
на базе микроконтроллера серии STMicroelectronics. Измерение наклона по осям происходит при помощи МЭМС гироскопа/акселерометра. 

Для выполнения задания необходимо изучить устройство МЭМС гироскопа и его подключения, принцип работы интерфейса SPI и связи по нему 
со светодиодной панелью P10 с размерностью 16x32 пикселей.

### Serial Peripheral Interface - SPI
**Serial Peripheral Interface, SPI** — последовательный синхронный стандарт передачи данных,
предназначенный для связи между микроконтроллерами и периферийными устройствами.

В SPI используются четыре цифровых сигнала:

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/35cccc39-d990-4291-b7f2-cea0e3460775)

* **MOSI** (Master Out, Slave In) — линия передачи данных от мастера к слейву.

* **MISO** (Master In, Slave Out) — линия передачи данных от слейва к мастеру.

* **SCLK** или **SCK** (Serial Clock) — линия передачи тактовой частоты, используемой для синхронизации обмена данными между устройствами.

* **CS** или **SS** (Slave Select) — линия, используемая для выбора активного слейва в многослейвовой конфигурации.

  #### Прием и передача данных в SPI
Передача осуществляется пакетами. Длина пакета, как правило, составляет 1 байт (8 бит),
при этом интерфейс SPI независит от длины пакета, возможны реализации и с 4 битами.
Ведущее устройство инициирует цикл связи установкой
низкого уровня на выводе выбора подчиненного устройства (SS)
того устройства, с которым необходимо установить соединение.

Общий принцип передачи данных изображён на рисунке ниже.

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/42189e49-70de-4a58-9abf-ae964270bc53)

Процесс передачи данных в SPI обычно выглядит следующим образом:

1) Мастер генерирует тактовые импульсы на линии SCLK, чтобы синхронизировать данные.
2) Мастер выбирает нужное Slave устройство путем установки линии SS в активное состояние (обычно низкое состояние).
3) Мастер отправляет данные на линию MOSI в нужный момент тактового импульса (обычно по переднему фронту).
4) Slave устройство принимает данные от мастера на линии MISO, также в нужный момент тактового импульса.
5) Slave устройство может отправлять данные обратно мастеру на линию MISO в нужный момент тактового импульса.
6) Мастер снимает активное состояние с линии SS, позволяя другим Slave устройствам быть выбранными.

SPI позволяет передавать данные одновременно в обоих направлениях (дуплексный режим) и может работать на разных частотах передачи данных. 

#### Преимущества и недостатки интерфейса SPI
##### Преимущества:
* Более высокая пропускная способность по сравнению с I²C или SMBus.
* Возможность произвольного выбора длины пакета.
* более низкие требования к энергопотреблению по сравнению с I²C и SMBus;
* ведомым устройствам не нужен уникальный адрес, в отличие от таких интерфейсов, как I²C, GPIB или SCSI.
* Используется только четыре вывода, что гораздо меньше, чем для параллельных интерфейсов.
* Максимальная тактовая частота ограничена только быстродействием устройств, участвующих в обмене данными.

##### Недостатки
* Необходимо больше выводов, чем для интерфейса **I²C**.
* Ведомое устройство не может управлять потоком данных.
* Нет подтверждения приема данных со стороны ведомого устройства.
* По дальности передачи данных интерфейс **SPI** уступает таким стандартам, как **UART** и **CAN**.

### МЭМС Гироскоп L3GD20
Гироскоп — это устройство, которые способно реагировать на изменение углов ориентации объекта,
относительно инерциальной системы отсчета и определять его положение в пространстве.

МЭМС Гироскоп состоит из трёх независимых одноосных вибрационных датчиков угловой скорости (MEMS гироскопов), которые реагируют на вращение вокруг X-, Y-, Z- осей. Две подвешенные массы совершают колебания по противоположным осям. С появлением угловой скорости эффект Кориолиса вызывает изменение направления вибрации ($\vec{F}_K = -2m[\vec{\omega} \times \vec{v}_r]$), которое фиксируется емкостным датчиком. Измеряемая дифференциальная емкостная составляющая пропорциональна углу перемещения [Время Электроники]. Получившийся сигнал усиливается, демодулируется и фильтруется, давая в итоге напряжение, пропорциональное угловой скорости вращения.

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/089884a5-ce34-428d-aed7-4ac430d0a289)

На отладочной плате имеется интегрированный МЭМС гироскоп/акселерометр L3GD20
![L3GD20](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/8e5172b1-98c9-40e5-89ee-e945507bf404)

Принципиальная схема подключения L3GD20
![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/df0e6b31-d13d-4a31-8cca-d3f6872540b1)

Так микросхема L3GD20 имеет интерфейсы передачи SPI и I2C. Для лабораторной работы выбран интерфейс SPI.

### Устройство LEDP10 Светодиодной панели
**P10 Светодиодная панель** или **P10 Led Board** представляет собой
панель состоящую из корпуса, светодиодов и управляющих микросхем.
Размерность светодиодной панели в пикселях равна 16 по высоте и
32 по ширине. Управление светодиодной панелью осуществляется
через интерфейс SPI, что позволяет подключать несколько панелей
последовательно и масштабиовать размерность панели.

Основные параметры LEDP10: 
- **Разрешение:** 32 (длина) на 16 (высота) в светодиодах
- **Питающее напряжение**: 5 вольт постоянного тока
- **Максимальное потребление тока**: 4 ампера
- **Максимальная мощность**: 20 Ватт

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/95f56691-4c29-4266-853a-ebf2ee979a48)
![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/3e1f7ce6-cf64-497d-8d7d-6c2fee578411)

Ниже приведена принципиальная схема светодиодной панели:

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/05c747d5-1d83-46ad-99e3-8f58f7dfd0e6)

Подключение к матрице будет осуществляться по средству подключения сигнальных проводов к STM32 по интерфейсу передачи SPI.

Распиновка входного разъёма

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/556ca805-ae99-445d-a3a6-6541f3a6ded5)

На разъёме:
* Пин 1 (nOE) — разрешает работу матрицы (лог. 0 гасит все матрицы в цепочке).
* Пины 2 и 4 (A и B) — задают, какая из 4 групп светодиодов экрана работает в данный момент.
Матрицы используют динамическую индикацию, поочерёдно переключая 4 группы светодиодов в зависимости от логических
уровней на ножках A и B.
На плате эти сигналы приходят на дешифратор D18, который открывает 1 из 4 групп P-канальных полевиков,
тем самым подавая +5В на аноды светодиодов выбранной группы. Логика работы этого пина реализована на элементах НЕ в D19 и дешифраторе D18.
* Пины 8 и 12 (CLK и R) — линии клока и данных синхронного последовательного интерфейса.
Их подключаем к SCK и MOSI интерфейса SPI микроконтроллера.
* Пин 10 (SCLK) — канал подтверждения конца загрузки байта данных. По переднему фронту защёлкивает переданные в сдвиговые регистры данные на их выходы.
Так как сдвиговые регистры подключены к катодам светодиодов матрицы передаваемые данные нужно инвертировать (светодиод будет гореть при лог. 0)

Распиновка микроконтроллера выглядит следующим образом

![image](https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/b296ad1b-7f14-4fbf-8658-b4811749e36a)


#### Логика обновления экрана (точнее четверти экрана) выглядит следующим образом:
1. Выдаём по SPI данные для сдвиговых регистров.
Для одной матрицы 32x16 это 16 байт (16 8-битных регистров).
2. Устанавливаем лог. 0 на ножке **nOE**.
3. Устанавливаем лог. уровни на ножках A и B в соответствии
с обновляемой группой светодиодов (одной из четырёх).
Это подаёт +5В на аноды светодиодов выбранной группы.
4. Выдаём на ножку **SCLK** короткий положительный импульс.
Это подаёт землю на катоды светодиодов в соответствии
с загруженными в регистры байтами.
5. Устанавливаем лог. 1 на ножке **nOE**.
При этом четверть экрана (одна группа светодиодов)
загорается и горит до следующего обновления следующей группы светодиодов.

Повторяем пункты **1-5** с постоянным периодом.

**Результат работы:**
https://github.com/lazy-santa/Embedded-System-Labs/assets/84825600/ca1bf0a2-ff5e-4ea2-8913-32cd5d8449d6

