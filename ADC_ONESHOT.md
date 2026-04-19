Заголовочный файл: #include "esp_adc/adc_oneshot.h"

Основные шаги:
1. Создание модуля: Настройка adc_oneshot_unit_init_cfg_t и вызов adc_oneshot_new_unit() для получения дескриптора adc_oneshot_unit_handle_t.
2. Настройка канала: Вызов adc_oneshot_config_channel() для установки разрядности и затухания (attenuation).
3. Чтение: Использование функции adc_oneshot_read().
4. Калибровка: Для получения точных значений в милливольтах теперь используется esp_adc/adc_cali.h вместо старых функций adc_char.

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_adc/adc_oneshot.h"
#include "esp_adc/adc_cali.h"
#include "esp_adc/adc_cali_scheme.h"

void app_main(void) {
    // 1. Инициализация модуля ADC1
    adc_oneshot_unit_handle_t adc1_handle;
    adc_oneshot_unit_init_cfg_t init_config1 = {
        .unit_id = ADC_UNIT_1,
        .ulp_mode = ADC_ULP_MODE_DISABLE,
    };
    ESP_ERROR_CHECK(adc_oneshot_new_unit(&init_config1, &adc1_handle));

    // 2. Настройка канала (ADC_CHANNEL_0 обычно GPIO 1 или 36 в зависимости от чипа)
    adc_oneshot_chan_cfg_t config = {
        .bitwidth = ADC_BITWIDTH_DEFAULT,
        .atten = ADC_ATTEN_DB_12, // Позволяет мерить до ~3.1-3.3В
    };
    ESP_ERROR_CHECK(adc_oneshot_config_channel(adc1_handle, ADC_CHANNEL_0, &config));

    // 3. Настройка калибровки (чтобы видеть реальные мВ)
    adc_cali_handle_t cali_handle = NULL;
    adc_cali_line_fitting_config_t cali_config = {
        .unit_id = ADC_UNIT_1,
        .atten = ADC_ATTEN_DB_12,
        .bitwidth = ADC_BITWIDTH_DEFAULT,
    };
    bool do_calibration = (adc_cali_create_scheme_line_fitting(&cali_config, &cali_handle) == ESP_OK);

    while (1) {
        int raw_reading;
        int voltage_mv;

        // Читаем сырые данные
        ESP_ERROR_CHECK(adc_oneshot_read(adc1_handle, ADC_CHANNEL_0, &raw_reading));

        if (do_calibration) {
            // Переводим в милливольты
            adc_cali_raw_to_voltage(cali_handle, raw_reading, &voltage_mv);
            printf("Raw: %d, Voltage: %d mV\n", raw_reading, voltage_mv);
        } else {
            printf("Raw: %d\n", raw_reading);
        }

        vTaskDelay(pdMS_TO_TICKS(500));
    }
}
```

1. Attenuation (Затухание): Для 3.3В используйте ADC_ATTEN_DB_12. Если использовать ADC_ATTEN_DB_0, вы сможете измерить только до ~1.1В, и при повороте резистора выше этой отметки значения "замрут" на максимуме.

2. Шумы: Если значения "прыгают", можно добавить программное усреднение (прочитать 10 раз и взять среднее) или поставить конденсатор 0.1 мкФ между сигнальным пином и GND.

3. Пин: Проверьте распиновку вашего конкретного чипа (ESP32, S3, C3), так как ADC_CHANNEL_0 на разных модулях привязан к разным GPIO.


Схема подключения
- Крайний вывод 1: к 3.3V на ESP32.
- Крайний вывод 2: к GND.
- Средний вывод (ползунок): к пину АЦП (например, GPIO 1 для ADC1_CH0 на ESP32-S3/C3).

Рекомендуется использовать ADC1, так как ADC2 имеет ограничения при работе с Wi-Fi.


|Канал ADC1|GPIO   |Примечание             |
|----------|-------|-----------------------|
|ADC1_CH0  |GPIO 36|Только вход (Sensor VP)|
|ADC1_CH3  |GPIO 39|Только вход (Sensor VN)|
|ADC1_CH4  |GPIO 32|—                      |
|ADC1_CH5  |GPIO 33|—                      |
|ADC1_CH6  |GPIO 34|Только вход            |
|ADC1_CH7  |GPIO 35|Только вход            |



