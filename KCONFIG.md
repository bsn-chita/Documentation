Для добавления параметров в menuconfig используется язык Kconfig.

В ESP-IDF дерево меню строится автоматически: система сборки сканирует все папки компонентов и, если находит там файл Kconfig или Kconfig.projbuild, сама добавляет их в общее меню.

# Объявление параметра
Это базовый элемент, который определяет переменную, сохраняемую в итоге в файл .config.
```kconfig
config MY_FEATURE
    bool "Название опции в меню"
    default y
    help
      Здесь можно написать подробное описание опции.
      Оно будет отображаться при нажатии '?' или 'Help'.
```
# Типы данных
1. bool: Логическое значение (y — включено, n — выключено).
2. string: Текстовая строка.
3. int: Целое число.
4. hex: Шестнадцатеричное число
# Зависимости и условия
Для управления видимостью и доступностью опций используются ключевые слова:
- depends on: Опция будет видна, только если выполнено условие.
```kconfig
config OPTION_B
    bool "Option B"
    depends on OPTION_A
```
- select: Принудительно включает другую опцию (обратная зависимость). Используйте осторожно, чтобы не создать конфликтов.
```kconfig
config MY_DRV
    bool "My Driver"
    select I2C  # Автоматически включит поддержку I2C
```
# Группировка: Меню и Подменю
Чтобы логически разделить настройки, используются блоки:
- menu ... endmenu: Создает отдельный раздел в меню.
```kconfig
menu "Network Settings"
    config NET_ENABLE
        bool "Enable Networking"
endmenu
```
- menuconfig: Создает опцию, которая сама является заголовком меню. Если она выключена, все вложенные элементы скрыты.
```kconfig
menuconfig PARENT_OPTION
    bool "Parent option"

if PARENT_OPTION
    config CHILD_OPTION
        bool "Child option"
endif
```
# Выбор из списка (choice)
Позволяет выбрать только один вариант из предложенных (как радиокнопки).
```kconfig
choice
    prompt "Выберите режим работы"
    default MODE_FAST

config MODE_FAST
    bool "Быстрый режим"

config MODE_STABLE
    bool "Стабильный режим"

endchoice
```
# Подключение других файлов
Для структуризации больших проектов используется команда source:
```kconfig
source "path/to/another/Kconfig"
```
Пример структуры:
Допустим, у вас есть компонент my_protocol, и вы хотите вынести настройки в разные файлы:
1. Главный файл components/my_protocol/Kconfig:
```kconfig
menu "My Protocol Settings"

    config MY_PROTOCOL_ENABLE
        bool "Enable My Protocol"
        default y

    # Подключаем настройки физического уровня
    source "components/my_protocol/Kconfig.phy"

    # Подключаем настройки логирования
    source "components/my_protocol/Kconfig.debug"

endmenu
```
2. Дополнительный файл components/my_protocol/Kconfig.phy:
```kconfig
config MY_PROTOCOL_SPEED
    int "Transmission Speed (bps)"
    default 115200
    depends on MY_PROTOCOL_ENABLE
```






```kconfig
```
```kconfig
```
```kconfig
```
```kconfig
```
```kconfig
```
```kconfig
```
```kconfig
```

```kconfig
```
```kconfig
```
```kconfig
```
```kconfig
```
