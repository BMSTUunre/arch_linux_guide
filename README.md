# arch_linux_guide
### Made by [Formak](https://github.com/Formak21) published by [unre](https://github.com/BMSTUunre)
0. Образ качается с [офф. сайта ArchLinux](https://archlinux.org/download/), советую воспользоваться torrent ссылкой
1. Вот вы в арчисо, необходимо подключить интернеты (если вы по проводу - скорее всего все уже работает), иначе
   ```shell
    iwctl --passphrase PASS station wlan0 connect SSID
   ```
   - где
     - PASS- Пароль от сети
     - wlan0- название устройства бесп. подключения (посмотреть можно через `device list`)
     - SSID- Имя сети (посмотреть доступные можно через `station wlan0 get-networks`)
2. Работа русской раскладки (переключение на caps)
   ```shell
   loadkeys ruwin_cplk-UTF-8
   ```
3. Установка шрифта ter-v32n (требует пакет `terminus-font`)
  ```shell
  setfont ter-v32n
  ```
4. Время
  ```shell
  timedatectl set-ntp true
  timedatectl set-timezone Europe/Moscow
  ```
5. Настройка зеркал
  ```shell
  reflector --threads 16 --country Russia --sort score --age 3 --save /etc/pacman.d/mirrorlist
  ```
  - где  `--threads 16`- кол-во потоков в вашем процессоре
6. Синхронизация репозиториев
  ```shell
  pacman -Syy
  ```
### Время поговрить о дисках
Посмотреть накопители можно через команду
  ```shell
  fdisk -l
  ```
Я ставлю Арч на диск с меткой `/dev/nvme0n1` объемом 1TB (954GB), ваш может называться по-другому имейте это ввиду.
7. Очистка файловой системы [ВНИМАНИЕ] (удаляет только начало, то бишь файловую систему)
  ```shell
  wipefs --all --force /dev/nvme0n1
  ```
8. Удаление данных [ВНИМАНИЕ] (обнуляет ссд через TRIM) (сотрёт вообще все на%@#)
  ```shell
  blkdiscard --force /dev/nvme0n1
  ```
### Разметка дисков
#### Размечать диск буду так
  ```
  +----------------+---------+-------+------------+-------------------+--------+
  |  код раздела   | [mount] | объем | усл. метка | Тип файл. системы | Формат |
  +----------------+---------+-------+------------+-------------------+--------+
  | /dev/nvme0n1p1 |  /boot  |  2GB  |    EFI     |     EFI System    | FAT32  |
  | /dev/nvme0n1p2 |  /swap  | 32GB  |   Linux    |     Linux Swap    |  swap  |
  | /dev/nvme0n1p3 |  /root  | 620GB |   Linux    | Linux x86-64 root |  ext4  |
  | +Пустое пространство в 300GB (под винду)      |                   |        |
  +----------------+---------+-------+------------+-------------------+--------+
  ```
Переход в режим форматирования с Таблицей разделов gpt
  ```shell
  gdisk /dev/nvme0n1
  ```
Каждый новый раздел добавляется отдельно, алгоритм таков:
- `n` (новый раздел)
- Вводим номер (или `enter`, чтобы согласиться с предложенным)
- Номер первого сектора (советую соглашаться с предложенным через `enter`, чтобы ничего не сломать)
- Номер последнего сектора (здесь указывается объем раздела, записывается через `[0-9]+[GB|MB|TB]`)
- Тип файловой системы, вот пример используемых мною:
  - `ef00` - EFI System
  - `8200` - Linux Swap
  - `8304` - Linux x86-64 root
Проверить размеченную таблицу можно через `p`, а записать через `w`
9. Форматирование
  EFI System в FAT32
  ```shell
  mkfs.fat -F 32 /dev/nvme0n1p1
  ```
  Linux Swap в Swap
  ```shell 
  mkswap /dev/nvme0n1p2
  swapon /dev/nvme0n1p2
  ```
  Linux x86-64 root в ext4
  ```shell
  mkfs.ext4 /dev/nvme0n1p3
  ```


