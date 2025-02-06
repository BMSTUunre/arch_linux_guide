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

10. Монтирование системы
   ```shell
   mount /dev/nvme0n1p3 /mnt
   mount --mkdir /dev/nvme0n1p1 /mnt/boot
   ```

11. Пакстрапим систему
   ```shell
   pacstrap -K /mnt base base-devel pacman-contrib linux linux-headers linux-firmware intel-ucode e2fsprogs nano vim terminus-font
   ```
   `vim` - отсебятина ([Formak](https://github.com/Formak21) советует только nano)

12. Генерируем фстаб (таблицу монтирования)
   ```shell
   genfstab -U /mnt >> /mnt/etc/fstab
   ```

13. Aрчшрутимся (далее команды будут из под арч шрута)
   ```shell
   arch-chroot /mnt
   ```

14. Время
   ```shell
   ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
   hwclock --systohc
   ```

15. Шрифты и кодировка в tty
   ```shell
   touch /etc/vconsole.conf
   echo -e "FONT=ter-v32n\nFONT_MAP=8859-5\nKEYMAP=ruwin_cplk-UTF-8\n" > /etc/vconsole.conf
   ```

16. Тут все понятно
   ```shell
   echo -e 'LANG="en_US.UTF-8"' > /etc/locale.conf
   ```
   ```shell
   vim /etc/locale.gen
   ```
   Раскомментировать `en_US.UTF-8` и `ru_RU.UTF-8`

17. Cгенерировать локаль
   ```shell
   locale-gen
   ```

18. Задаем имя нашему прекрасному устройству
   ```shell
   echo "HOSTNAME" > /etc/hostname
   ```
   - где `HOSTNAME`- произвольное слово

19. Записываем хосты
   ```shell
   vim /etc/hosts
   ```
   ВНИМАНИЕ ТУТ ТАБЫ, А НЕ ПРОБЕЛЫ
   (Дописываете это в конец файла, не забудьте хостнейм поменять)
   ```
   127.0.0.1  localhost
   ::1  localhost
   127.0.1.1  HOSTNAME.localdomain  HOSTNAME
   ```

20. Пароль рута
   ```shell
   passwd
   ```
   Вводите запоминающийся, но сложный пароль для root`a 2 раза

21. Создаем типичного юзера
   ```shell
   useradd -m USER
   passwd USER
   ```
   - где `USER`- произвольное слово
   Вводите запоминающийся, но сложный пароль для юзера 2 раза

22. Даем юзеру права
   ```shell
   usermod -aG wheel,audio,video,optical,storage USER
   ```
   Проверить успешность команды можно через
   ```shell
   userdbctl groups-of-user USER
   ```

23. Снова рефлектор будь он неладен
   ```shell
   reflector --threads 8 --country Russia --sort score --age 3 --save /etc/pacman.d/mirrorlist
   ```

24. Обновим системку
   ```shell
      pacman -Syu sudo
   ```

25. Задаем дефолтный редактор текстовиков
   ```shell
   EDITOR=vim
   ```

26. Раскомментировать wheel (я скажу какой)
   ```shell
   visudo
   ```
   раскоментить %wheel, который через строку после root

27. Grub часть1
   ```shell
   pacman -S grub efibootmgr dosfstools mtools os-prober
   ```

28. Добавляем поддержку второй ос
   ```shell
   vim /etc/default/grub
   ```

В него после первого абзаца дописать ```GRUB_DISABLE_OS_PROBER=false```

29. Grub часть2
   ```shell
   grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck --efi-directory=/boot/EFI
   ```

30. Генерим конфиг для граба
   ```shell
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

31. Установка всяких пакетов
    Дисклеймер, они могут вам не подойти, ставлю под ноут Thinkpad T490
    - Для проца
       ```shell
       pacman -Syu mesa vulkan-intel intel-media-driver libva-utils libva-intel-driver
      ```
    - Блютуз
       ```shell
       pacman -Syu bluez bluez-utils
       systemctl enable bluetooth.service
       ```
    - Аудио
       ```shell
       pacman -Syu pipewire pipewire-audio pipewire-alsa pipewire-jack pipewire-pulse sof-firmware alsa-firmware
       ```
    - Нетворки и мелочи
       ```shell
       pacman -Syu dosfstools inetutils net-tools networkmanager traceroute usbutils xdg-user-dirs
       systemctl enable NetworkManager.service
       ```

### Момент Истины

32. Выходим из арччрута
   ```shell
   exit
   ```

33. Размонтируем диски
   ```shell
   umount -a
   ```

34. Перезагружаемя и выбираем при загрузке граб
   ```shell
   reboot
   ```

## Вуаля вы АрчЛинуксоид, теперь можешь вые"%№!?&* в интернете: `i use arch btw`
