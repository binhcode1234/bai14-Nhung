# BÀI 1 : FreeRTOS điều khiển LED dùng Event Groups – STM32F103
📌 Giới thiệu

1,FreeRTOS + EventGroup để điều khiển 3 task nháy LED thông qua 1 nút nhấn.
Mỗi lần nhấn nút sẽ chuyển chế độ hoạt động của các task:
Mode	Task hoạt động	Mô tả
0	Không task nào	Tắt hết LED
1	Task1	Nháy LED PA1 nhanh
2	Task2	Nháy LED PA2 vừa
3	Task3	Nháy LED PA3 chậm
4	Task1 + Task2 + Task3	Cả 3 LED cùng nháy
Toàn bộ task đồng bộ với nhau bằng EventGroup của FreeRTOS.
Cấu hình phần cứng
MCU: STM32F103C8T6 (Blue Pill)
LED output:
PA1 → LED 1
PA2 → LED 2
PA3 → LED 3
Nút nhấn: PA0 (Input Pull-Up)
Ý tưởng hoạt động:
Có 1 task chính Task_Control đọc nút bấm, chuyển mode = 0 → 4.
Dựa vào mode, Task_Control sẽ Set/Clear bit trong EventGroup.
Các task LED chỉ chạy khi bit của chúng được Set.
Khi bit bị Clear, task vẫn tồn tại nhưng bị block trong xEventGroupWaitBits().
#BÀI 2 
