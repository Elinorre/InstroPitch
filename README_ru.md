# InstroPitch

> Для запуска приложения введите в консоли ```python app.py```

**Цель:**
использовать нейронную сеть для того чтобы предсказать инструмент и звучащую ноту.

**Гипотеза:**
может ли нейронная сеть предсказать, на каком инструменте и какая нота играется, с точностью свыше 80%?


Музыкальные инструменты имеют широкий диапазон форм и размеров, а характеристики звучания могут быть как отличными, так и похожими на другие инструменты. Автоматическое распознавание музыкальных инструментов не только позволит различать различные типы инструментов и их ноты, но и позволит осуществлять музыкальный поиск по инструментам, поможет распознавать музыкальные жанры, обучает и оценивает системы поиска музыкальной информации (MIR), а также сделает транскрипцию музыки более простой и точной.


**Описание источника данных:**

[Источник данных](https://zenodo.org/record/3685367#.Xo1NVi2ZOuU)

Изначально этот набор звуковых данных записывался для проекта Studio On Line (SOL) в Ircam в Париже (1996-1999). Я использую  версию 6.0, обновленную в феврале 2020 года. Набор данных содержит набор инструментов, играющих определённую ноту. Всего имеются 14 различных инструментов, которые перечислены ниже. 

1. Басовая труба
2. Французский рог
3. Тромбон
4. Труба в С
5. Аккордеон
6. Контрабас
7. Скрипка
8. Альт
9. Виолончель
10. Фагот
11. Кларнет в С-бемоль
12. Флейта
13. Гобой
14. Саксофон-альт

### TinySOL данные:

Набор данных TinySOL содержит 2913 аудио WAV-файлов на частоте 44.1 кГц. TinySOL разделен на 5 частей. В каждом аудиофайле есть инструмент, играющий одну ноту. Информацию о семействе инструментов и звуке, технике воспроизведения радужной оболочки, высоте тона, динамике, дополнительную информацию можно найти в пути к файлу. Кроме того, эту информацию можно найти в метаданных в виде текста, который можно дальше очищать, организовывать и анализировать. 

### Метаданные:

TinySOL_metadata.csv содержит текстовые данные и информацию каждого файла пути в TinySOL. Файл содержит путь, идентификатор папки, семейство инструментов, аббревиатуру, имя, аббревиатуру техники и имя, высоту, динамический идентификатор, идентификатор экземпляра, идентификатор строки, а также необходимость цифровой перестройки. 


### Технологии

[Technology.md](support_info/technology.md)


## Цели и задачи
Цель и задача состоят в том, чтобы создать модель, способную определять инструмент и музыкальную ноту, и имеющую точность выше 80%. В конечном счёте пользователь должен иметь возможность предоставить .WAV-файл и получить ноту и инструмент, на котором она сыграна.


## ETL
**Извлечение:** 
аудиофайлы .wav извлекаются из AWS S3, где набор аудиофайлов хранится с помощью библиотеки python boto3.

**Трансформация:**
каждый .WAV-файл преобразуется в спектрограмму с использованием библиотеки librosa. Список спектрограмм затем сохраняется в датафрейме с именем *notes_df*. Также метаданные TinySOL metadata.csv сохраняются в датафрейм с именем *notes_soldf_sample*. Данные о шаге в этом датафрейме трансформируются, где я использовала функцию разделения для создания столбца нот и столбца октавы, которые впоследствии будут использоваться в качестве входных данных в модели.

Затем я объединяю фрейм данных спектрограмм с фреймом данных метаданных в фрейм данных, называемый *notesDf_merged*. Я создаю два новых фрейма данных из него.  Первый датафрейм - это тот, который содержит данные о пути к аудиофайлу, спектрограмме, высоте, ноте и октаве *(все ненужные столбцы опускаются)*, и я назвала его *notesDF_Final*.  Второй датафрейм -  содержащий путь к аудиофайлу, спектрограмму и название инструмента *(все ненужные колонки опускаются)*, и я назвала его *Instrument_DF_Final*.

**Загрузка:**
Созданные два датафрейма *notesDF_Final* и *Instrument_DF_Final* загружаются и сохраняются в виде таблиц в PostgresDB с помощью питоновой библиотеки SqLAlchemy.


## Модель

**Тип модели:**
Свёрточная нейронная сеть (CNN)

**Почему именно свёрточная сеть?** 
Несмотря на то, что существуют более простые способы определения высоты тона (простой полосовой фильтр), я хочу построить модель, которую можно легко модифицировать для определения различных типов звука (лай собак, набор текста на клавиатуре, машины и т.д.).

**Как тренируется модель?**
Введенные аудиофайлы будут преобразованы в спектрограммы (массивы). Эти спектрограммы вводятся только в CNN. Я буду использовать две различные спектрограммы для обучения двух CNN. CNN, определяющий высоту, будет обучаться коротким спектрограммам из середины аудиофайла. CNN, определяющий высоту, будет тренироваться с помощью более длинных спектрограмм, которые начинаются в начале аудиофайла.

**Какова точность модели?**
В настоящее время есть три рабочих сценария. Все они имеют точность теста более 90% для предсказания высоты тона, и более 85% для предсказания инструмента.

**Как эта модель работает?**
- Вход: одноканальная спектрограмма 22х128

**✓Предобработка данных**
Вход: Загружается спектограмма и преобразуется в нумерованный массив.
Выход: Полученные из PostgressDB данные преобразованы в категориальные сто и Взять данные из пост-гресса и из категориальных столбцов преобразованы в числовые столбцы датафреймов, такие как нота и инструмент.

**✓Описание предварительного проектирования и предварительного выбора объектов, включая процесс принятия решений**
Я решила тренироваться со спектрограммами, потому что они деконструируют звуковые сигналы на составляющие их частоты. Спектрограммы можно интерпретировать как массивы, которые могут быть легко поданы в нейронную сеть.

**✓Как данные были разделены на обучающие и тестовые наборы**
Тестовые и обучающие выборки делятся в отношении 25 и 75 соответственно. Для разделения данных использовался sklearn.model_selection.train_test_split.  Я использую стратификацию <output> для обеспечения равномерного разделения данных.

**✓Объяснение выбора модели**
- Преимущества: можно применять модель к другим звуковым файлам/сигналам и делать другие виды прогнозов. Эта модель хорошо работает со спектрограммами. 

- Ограничения: 
 - - Меньшая точность при более низких частотах дискретизации.
 - - Одновременное воспроизведение нот, например, аккордов, усложняет задачу, так как частоты могут деконструировать друг друга.  
 - - Часто можно предположить, что сходные соседние пиксели принадлежат одному и тому же визуальному объекту. Однако в звуке различные частоты могут быть соседями по спектрограмме.
 
**Выход модели:** 
- Инструмент: Один из 14 перечисленных ранее инструментов
- Шаг (нота и октава): Буква ноты и номер октавы


## База данных 

AWS S3 используется для хранения набора данных аудиофайлов (.wav). Файлы .wav извлекаются из S3 и преобразуются в спектрограммы. Создаются два датафрейма, они загружаются и сохраняются в виде таблиц в PostgresDB *(подробнее см. раздел ETL)*. Эти таблицы называются *Notes_Spectrogram_Table* и *Instruments_Spectrogram_Table*.  Затем они объединяются с помощью sql-запроса для создания третьей таблицы, содержащей спектрограммы, инструменты и данные нот.  

PostGresDB также используется для создания таблиц из исходных файлов метаданных. 

**Схематическая диаграмма**

[Schema_Diagram.PNG](поддержка_info/Schema_Diagram.PNG)