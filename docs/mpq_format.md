
## 1 Введение в формат MoPaQ

MoPaQ(MPQ) - архивный формат разработанный Mike O'Brien. Формат был использован во всех играх Blizzard включая Diablo. Он хорошо оптимизирован, предназначен только для чтения.
mpyq - open-source python библиотека для чтения MPQ формата.


## 2 Формат MPQ
Все цифры в формате MPQ находятся в прямом порядке следования байтов (little endian byte order). Типы данных представленны в integer или char(система ASCII). Все размеры  и отступы указаны в двоичном виде, если специальный тип не указан. Элементы структуры перечислены в селдующем общем порядке: смещение от начала структуры, тип данных(длина масива), имя элемента и описание элемента
### Общая Компоновка Архива
 - Заголовок Архива(Archive Header)
 - Данные файла(File Data)
 - Данные Файла - Специальные Файлы(File Data - Special Files)
 - Хэш-Таблица(Hash Table)
 - Таблица Блоков(Block Table)
 - Расширенная Таблица Блоков(Extended Block Table)
 - Сложная цифровая подпись(Strong Digital signature)
 

Это обычная компоновка архива, но она может не соблюдаться. Некоторые архивы содержат таблицу хэша и таблицу файла после заголовка архива, и перед данными файла.
Однако, начиная со Starcraft 2, блочная Таблица должна немедленно следовать за хэш-таблицей. 


### Заголовок Архива(Archive Header)

Заголовок архива является первым элеметтом в архиве, поэтому его смещение равно 0. Заголовок имеет следующую структуру:

<details><summary>Структура заголовка архива</summary>
```
00h: char(4) Magic             Indicates that the file is a MoPaQ archive. Must be ASCII "MPQ" 1Ah.
04h: int32 HeaderSize          Size of the archive header.
08h: int32 ArchiveSize         Size of the whole archive, including the header. Does not include the strong digital signature, 
                           if present. This size is used, among other things, for determining the region to hash in computing 
                           the digital signature. This field is deprecated in the Burning Crusade MoPaQ format, and the size 
                           of the archive is calculated as the size from the beginning of the archive to the end of the 
                           hash table, block table, or extended block table (whichever is largest).
0Ch: int16 FormatVersion       MoPaQ format version. MPQAPI will not open archives where this is negative. Known versions:
0000h                  Original format. HeaderSize should be 20h, and large archives are not supported.
0001h                  Burning Crusade format. Header size should be 2Ch, and large archives are supported.
0Eh: int8 SectorSizeShift      Power of two exponent specifying the number of 512-byte disk sectors in each logical sector 
                           in the archive. The size of each logical sector in the archive is 512 * 2^SectorSizeShift. 
                           Bugs in the Storm library dictate that this should always be 3 (4096 byte sectors).
10h: int32 HashTableOffset     Offset to the beginning of the hash table, relative to the beginning of the archive.
14h: int32 BlockTableOffset    Offset to the beginning of the block table, relative to the beginning of the archive.
18h: int32 HashTableEntries    Number of entries in the hash table. Must be a power of two, and must be less than 2^16 
                           for the original MoPaQ format, or less than 2^20 for the Burning Crusade format.
1Ch: int32 BlockTableEntries   Number of entries in the block table.

Fields only present in the Burning Crusade format and later:

20h: int64 ExtendedBlockTableOffset   Offset to the beginning of the extended block table, relative to the beginning of the archive.
28h: int16 HashTableOffsetHigh        High 16 bits of the hash table offset for large archives.
2Ah: int16 BlockTableOffsetHigh       High 16 bits of the block table offset for large archives.
```
</details>
    


### Данные файла(File Data)
Данные для каждого файла составлены в следующую структуру:
```
int32(SectorsInFile* + 1) SectorOffsetTable  Offsets to the start of each sector, 
                    relative to the beginning of the file data. The last entry contains
                    the total compressed file size, making it possible to easily
                    calculate the size of any given sector by simple subtraction. This
                    table is not present or necessary if the file is not compressed.
SECTOR Sectors(SectorsInFile) Data of each sector in the file, packed end to end (see details below).
```
Как правило, файловые данные разделяются на сектора, для простой потоковой передачи. Все сектор будут содержать только столько  байтов данных файла, сколько указано в Заголовке Архива(свойство SectorSizeShift); последний сектор может содержать меньшее количество байт. Файлы обычно сохраняются в сжатом виде, но если сжатие избыточно, то файл сохраняется в исходном виде. 

Формат каждого сектора зависит от его вида. Несжатые сектора - двоичный файл, содижащийся в секторе. Поврежденные сектора - необработанные данные после сжатия с помощью алгоритма implode (эти секторы могут содержать только поврежденные файлы). Сжатые секторы(только сжатые файлы) сжимаются с помощью одного или нескольких алгоритмов сжатия и имеют структуру:
```    
byte CompressionMask : Mask of the compression types applied to this sector.
byte(SectorSize - 1) SectorData : The compressed data for the sector.
``` 
CompressionMask указывает, какой алгоритм(ы) сжатия применяется к сжатому сектору. Этот байт учитывается в общем размере сектора, и сектор сохранится несжатый если данные не смогут быть сжаты по крайней мере на два байта; другими словами должен быть  прирост по крайней мере в один байт через сжатие. Также, этот байт шифруется с данными с данными сектора если это возможно. Определены следующие алгоритмы сжатия (реализации этих алгоритмов см. StormLib):
<details><summary>Доступные алгоритмы сжатия</summary>
```
20h: Sparse compressed. Added in Starcraft 2.
40h: IMA ADPCM mono
80h: IMA ADPCM stereo
01h: Huffman encoded
02h: Deflated (see ZLib). Added in Warcraft 3.
08h: Imploded (see PKWare Data Compression Library)
10h: BZip2 compressed (see BZip2). Added in World of Warcraft: The Burning Crusade.
```
</details> 

### Хэш-Таблица(Hash Table)

Вместо того чтобы хранить имена файлов, для быстрого доступа MPQ использует постаянную двумерную хэш-таблицу файлов в архиве. Файл однозначно идентифицируется по пути к файлу, языку и платформе. Точка входа файла в хэш-таблице вычисляется как хэш пути к файлу. В случае колизии (точка входа занята другим файлом) и файл помещается в другую в следующую доступную запись хэш-таблицы. Поиск нужного файла в хэш-таблице выполняется от точки входа и до тех пор, пока не будет найден файл или не будет найдена пустая запись хэш-таблицы(FileBlockIndex это FFFFFFFFh)

До Starcraft 2 хэш-таблица хранилась без сжатия. Однако в Starcraft 2 таблица может быть дополнительно сжата. Если смещение блочной таблицы не равно смещению хэш-таблицы плюс несжатый размер, Starcraft 2 интерпретирует хэш-таблицу как сжатую (не распакованную). Это вычисление предполагает, что Таблица Блока немедленно следует за хэш-таблицей и аварийно завершит работу в противном случае.

Хэш-Таблица всегда шифруется, используя хэш "(хэш-таблица)" в качестве ключа. Каждая запись имеет следующую структуру:
<details><summary>Структура записи хэш-таблицы</summary>
```
00h: int32 FilePathHashA    The hash of the file path, using method A.
04h: int32 FilePathHashB    The hash of the file path, using method B.
08h: int16 Language         The language of the file. This is a Windows LANGID data type, and uses the same values. 
                        0 indicates the default language (American English), or that the file is language-neutral.
0Ah: int8 Platform          The platform the file is used for. 0 indicates the default platform. No other values 
                        have been observed.
0Ch: int32 FileBlockIndex   If the hash table entry is valid, this is the index into the block table of the file. 
                    Otherwise, one of the following two values:
FFFFFFFFh           Hash table entry is empty, and has always been empty. Terminates searches for a given file.
FFFFFFFEh           Hash table entry is empty, but was valid at some point (in other words, the file was deleted). 
                    Does not terminate searches for a given file.
```
</details>

### Таблица Блоков(Block Table)

Таблица Блоков содержит записи для каждого участка в архиве. Участком могут быть либо файлы, либо пустое пространство, которое может быть перезаписано новыми файлами (обычно это пространство из удаленных файловых данных), либо неиспользуемые записи таблицы. Запись о пустом пространстве должна иметь BlockOffset и BlockSize больше нуля, FileSize Flags равные нолю; запись о неиспользуемом участке должна иметь BlockSize, FileSize и Flags равные нолю. 

Таблица Блоков шифруется, используя в качестве ключа хэш "(Таблица блоков)". Каждая запись имеет следующую структуру:
<details><summary>Структура записи таблицы блоков</summary>
```
00h: int32 BlockOffset   Offset of the beginning of the block, relative to the beginning of the archive.
04h: int32 BlockSize     Size of the block in the archive.
08h: int32 FileSize      Size of the file data stored in the block. Only valid if the block is a file; otherwise 
                     meaningless, and should be 0. If the file is compressed, this is the size of the uncompressed 
                     file data.
0Ch: int32 Flags         Bit mask of the flags for the block. The following values are conclusively identified:
80000000h        Block is a file, and follows the file data format; otherwise, block is free space or unused. 
                     If the block is not a file, all other flags should be cleared, and FileSize should be 0.
04000000h	 File has checksums for each sector (explained in the File Data section). Ignored if file is not
                     compressed or imploded.
    02000000h        File is a deletion marker, indicating that the file no longer exists. This is used to allow
                     patch archives to delete files present in lower-priority archives in the search chain.
01000000h        File is stored as a single unit, rather than split into sectors.
00020000h        The file's encryption key is adjusted by the block offset and file size (explained in detail in the 
                     File Data section). File must be encrypted.
00010000h        File is encrypted.
00000200h        File is compressed. File cannot be imploded.
00000100h        File is imploded. File cannot be compressed.
```
</details>

### Расширенная Таблица Блоков

Расширенная Таблица Блоков была добавлена поддержка архивов размером более 4 Гб (2^32 байт). Таблица содержит верхние биты смещений архива для каждого блока в таблице блоков. Отдельные блоки в архиве по-прежнему ограничены размером 4 гигабайта. Эта Таблица присутствует только в архивах формата Burning Crusade, размер которых превышает 4 гигабайта.

В отличие от хэш- и блочных таблиц, Расширенная блочная Таблица не шифруется и не сжимается.

### Дополнительные Атрибуты(Extended Attributes)

Расширенные атрибуты - необязательные атрибуты файлов в Таблице Блоков. Если архив содержит данный атрибут, то для каждого блока в Таблице Блоков будет создан экземпляр этого атрибута, хотя этот атрибут будет бессмысленным, если блок не является файлом. Этот файл структурирован следующим образом:
<details><summary>Структура дополнительных атрибутов</summary>
```
00h: int32 Version :           Specifies the extended attributes format version. For now, must be 100.
04h: int32 AttributesPresent : Bit mask of the extended attributes present in the archive:
	00000001h: File CRC32s.
	00000002h: File timestamps.
	00000004h: File MD5s.
08h: int32(BlockTableEntries) CRC32s :   CRC32s of the (uncompressed) file data for each block in the archive. 
                                         Omitted if the archive does not have CRC32s.
FILETIME(BlockTableEntries) Timestamps : Timestamps for each block in the archive. The format is that of the 
                                         Windows FILETIME structure. Omitted if the archive does not have timestamps.
MD5(BlockTableEntries) MD5s :            MD5s of the (uncompressed) file data for each block in the archive. 
                                         Omitted if the archive does not have MD5s.
```
</details>


### Сложная цифровая подпись(Strong Digital signature)
Сильная цифровая подпись состоит из SHA-1. Все известные ключи Blizzard являются 2048-битными (сильными) ключами RSA; ключ по умолчанию хранится в Storm. Очевидно, любой ключ RSA может быть использован; на самом деле, архив, подписанный ключом по умолчанию, никогда не был замечен в wild.


### Дополнительные данные(User Data)

Некоторые архивы, основанные на еще не известных критериях, не могут содержать шунтирующий блок. В этом случае данные пользователя хранятся в обычном файле внутри архива с именем " (user data)".

## mpyq

[github.com/eagleflo/mpyq](https://github.com/eagleflo/mpyq) - Репозиторий библиотеки

### Использование скриптов

Команду mpyq можно запустить с консоли, при этом можно указать следующие аргументы:
```
 - h, --help           Отображает меню помощи
 - I, --headers        Печатает загаловок архива
 - H, --hash-table     Печатает хэш таблицу
 - b, --block-table    Печатает таблицу блоков
 - s, --skip-listfile  Пропуск чтения списка файлов
 - t, --list-files     Печатает список файлов архива
 - x, --extract        Распаковывает архив
```

Пример использования скрипта для отобрадения списка файлов в архиве
```python 
$ mpyq -t game.SC2Replay

Files
-----
replay.attributes.events            580 bytes
replay.details                      443 bytes
replay.game.events                42859 bytes
replay.initData                    1082 bytes
replay.load.info                     96 bytes
replay.message.events                94 bytes
replay.smartcam.events             1444 bytes
replay.sync.events                  765 bytes
```