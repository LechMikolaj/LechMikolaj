- 👋 Hi, I’m @LechMikolaj
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
LechMikolaj/LechMikolaj is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/ledc.h"
#include "esp_log.h"

#define PWM_GPIO    18
#define PWM_FREQ    40000   // 40 kHz
#define PWM_RES     LEDC_TIMER_8_BIT  // try 8-bit first
#define PWM_CHANNEL LEDC_CHANNEL_0
#define PWM_SPEED_MODE LEDC_HIGH_SPEED_MODE
#define PWM_TIMER LEDC_TIMER_0

void app_main(void)
{
    esp_log_level_set("*", ESP_LOG_INFO);
    ESP_LOGI("PWM", "Configuring LEDC...");

    ledc_timer_config_t timer_conf = {
        .speed_mode = PWM_SPEED_MODE,
        .duty_resolution = PWM_RES,
        .timer_num = PWM_TIMER,
        .freq_hz = PWM_FREQ,
        .clk_cfg = LEDC_AUTO_CLK
    };
    ledc_timer_config(&timer_conf);

    ledc_channel_config_t ch_conf = {
        .gpio_num = PWM_GPIO,
        .speed_mode = PWM_SPEED_MODE,
        .channel = PWM_CHANNEL,
        .intr_type = LEDC_INTR_DISABLE,
        .timer_sel = PWM_TIMER,
        .duty = 0,
        .hpoint = 0
    };
    ledc_channel_config(&ch_conf);

    // 20% duty for 8-bit resolution = 0.2 * 255 ≈ 51
    uint32_t duty = ( (1 << (PWM_RES - LEDC_TIMER_0 + 0)) - 1 ); // not used; compute manually below
    uint32_t duty_val = (uint32_t)( (float)255 * 0.20f ); // for 8-bit
    ledc_set_duty(PWM_SPEED_MODE, PWM_CHANNEL, duty_val);
    ledc_update_duty(PWM_SPEED_MODE, PWM_CHANNEL);

    ESP_LOGI("PWM", "GPIO %d set to %d Hz, duty %u (8-bit)", PWM_GPIO, PWM_FREQ, duty_val);

    while (true) {
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
