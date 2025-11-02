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

