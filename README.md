# BÀI 1 : FreeRTOS điều khiển LED dùng Event Groups – STM32F103
# FreeRTOS EventGroup LED Control – STM32F103

## 📌 Giới thiệu
Dự án sử dụng FreeRTOS và EventGroup để điều khiển 3 LED thông qua 1 nút nhấn.  
Mỗi lần nhấn nút, chế độ sẽ thay đổi và các task LED sẽ nháy theo tần số khác nhau.

### Bảng chế độ hoạt động
| Mode | Task chạy | LED | Tần số |
|------|-----------|-----|--------|
| 0 | Không task nào | Tắt hết | — |
| 1 | Task1 | PA1 | Nhanh |
| 2 | Task2 | PA2 | Vừa |
| 3 | Task3 | PA3 | Chậm |
| 4 | Task1 + Task2 + Task3 | PA1 + PA2 + PA3 | Cùng nháy |

---

## ⚙️ Phần cứng
- STM32F103C8T6
- PA1 → LED1  
- PA2 → LED2  
- PA3 → LED3  
- PA0 → Nút nhấn (kéo lên nội – Input Pull-Up)

Kết nối:
```
PA0 ---- Nút nhấn ---- GND
PA1 ---- LED1 + R
PA2 ---- LED2 + R
PA3 ---- LED3 + R
```

---

## 🧠 Nguyên lý hoạt động
- Task_Control đọc nút nhấn và thay đổi biến `mode`.
- Dựa vào mode, Task_Control **Set/Clear bit** trong EventGroup.
- Các task LED sẽ chờ bit tương ứng:
  - BIT_TASK1 → Task1  
  - BIT_TASK2 → Task2  
  - BIT_TASK3 → Task3  
- Khi bit được set → Task LED chạy  
- Khi bit clear → Task LED bị block và dừng nháy

---
# BÀI 2 
DELAY
# STM32F103 – Blink LED Alternating (GPIO + Delay)

## 📌 Giới thiệu
Dự án thực hiện việc nháy **2 LED luân phiên** (LED1 sáng – LED2 tắt, sau đó đảo lại) sử dụng GPIO của STM32F103 và hàm Delay tự tạo bằng vòng lặp `NOP`.

---

## ⚙️ Phần cứng sử dụng
- **MCU**: STM32F103C8T6 (Blue Pill)  
- **LED Output**:
  - PB0 → LED1  
  - PB1 → LED2  

Kết nối LED:
```
PB0 ---- LED1 ---- R ---- GND
PB1 ---- LED2 ---- R ---- GND
```

---

## 🧠 Nguyên lý hoạt động
- Cấu hình PB0 và PB1 làm **Output Push-Pull**.
- Trong vòng lặp:
  - Bật PB0 – Tắt PB1 → Delay 500ms  
  - Tắt PB0 – Bật PB1 → Delay 500ms  
- Dùng vòng lặp `__NOP()` để tạo Delay đơn giản (không phụ thuộc SysTick).

---🧩 Mã nguồn (main.c)

## 🚀 Cách chạy
1. Tạo project STM32F10x Standard
1. Tạo project STM32F10x Standard Peripheral Library.  
2. Thêm file `main.c` vào project.  
3. Build bằng Keil / STM32CubeIDE / PlatformIO.  
4. Nạp chương trình qua ST-LINK.  
5. Quan sát 2 LED nháy luân phiên mỗi 500ms.
---
## ✅ Ghi chú
- Delay bằng vòng lặp không chính xác tuyệt đối nhưng đủ cho bài học cơ bản.  
- Khi làm dự án thực tế nên dùng SysTick hoặc Timer để tạo delay chuẩn.
---
SLEEP 
# STM32F103 – Blink LED bằng SysTick + Low-power WFI

## 📌 Giới thiệu
Dự án thực hiện việc nháy **2 LED luân phiên** sử dụng **SysTick timer** để tạo delay chính xác 1ms và lệnh **WFI** để đưa MCU vào chế độ tiết kiệm năng lượng trong khi chờ ngắt.

Đây là ví dụ chuẩn để học cách:
- Tạo delay bằng SysTick (không dùng vòng lặp bận)
- Lập trình ISR SysTick_Handler
- Sử dụng WFI để giảm tiêu thụ điện năng

---

## ⚙️ Phần cứng sử dụng
- **MCU**: STM32F103C8T6  
- **LED Output**:
  - PB0 → LED1  
  - PB1 → LED2  

Sơ đồ kết nối LED:
```
PB0 ---- LED1 ---- R ---- GND
PB1 ---- LED2 ---- R ---- GND
```

---

## 🧠 Nguyên lý hoạt động
- **SysTick** được cấu hình tạo ngắt mỗi **1ms**.  
- Mỗi lần SysTick ngắt → biến `tick` tăng lên.  
- Trong vòng lặp chính:
  - MCU vào chế độ tiết kiệm năng lượng với `__WFI()`  
  - Cứ mỗi **500ms**, chương trình đổi trạng thái 2 LED luân phiên.  

Ưu điểm:
✅ Delay chính xác hơn so với delay bằng vòng lặp  
✅ MCU ngủ khi không cần hoạt động → tiết kiệm điện  
✅ Tự học cách xử lý ngắt cơ bản

---

## 🧩 Mã nguồn (main.c)
```c
#include "stm32f10x.h"

volatile uint32_t tick = 0;  

void GPIO_Config(void);
void SysTick_Handler(void);

void SysTick_Handler(void)
{
    tick++; 
}

void GPIO_Config(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);

    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_2MHz;

    GPIO_Init(GPIOB, &GPIO_InitStructure);
    GPIO_ResetBits(GPIOB, GPIO_Pin_0 | GPIO_Pin_1);
}

int main(void)
{
    SystemInit();      
    GPIO_Config();     

    SysTick_Config(SystemCoreClock / 1000); // Ngắt mỗi 1ms

    uint32_t last = 0;
    uint8_t ledState = 0;  

    while (1)
    {
        __WFI();  // Chờ ngắt - tiết kiệm năng lượng

        if (tick - last >= 500)
        {
            last = tick;

            if (ledState == 0)
            {
                GPIO_SetBits(GPIOB, GPIO_Pin_0);
                GPIO_ResetBits(GPIOB, GPIO_Pin_1);
                ledState = 1;
            }
            else
            {
                GPIO_ResetBits(GPIOB, GPIO_Pin_0);
                GPIO_SetBits(GPIOB, GPIO_Pin_1);
                ledState = 0;
            }
        }
    }
}
```

---

## 🚀 Cách chạy
1. Tạo project STM32F1 (Standard Peripheral Library).  
2. Thay thế file `main.c` bằng mã trên.  
3. Build bằng Keil / STM32CubeIDE / PlatformIO.  
4. Nạp chương trình bằng ST-LINK.  
5. Quan sát 2 LED luân phiên sáng mỗi 500ms.

---

## ✅ Ghi chú
- `SysTick_Config(SystemCoreClock / 1000)` thiết lập ngắt 1ms.  
- `__WFI()` chỉ hoạt động khi global interrupt đang bật (`CPSIE i`).  
- Đây là cách chuẩn để tạo scheduler đơn giản không dùng RTOS.

---

## 📄 License
MIT License


