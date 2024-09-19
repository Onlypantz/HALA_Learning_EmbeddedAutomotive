
# Lesson 4: PROTOCOL

## 1. SPI

### *1.1 Khái niệm*

Machine technique: Có dây, nối tiếp.

Số dây: 4 dây

+ MOSI - Master out, Slave in.
+ MISO - Master in, Slave out.
+ SCLK - Serial clock
+ CS/SS - Chip Select

![Spi_diagram.png]

Direction: Full duplex, Half duplex.

SPI bus: 1 Master, N Slaves.

Data check: None.

Speed: tối đa 60Mps ở khoảng cách ngắn, thường dùng để giao tiếp giữa các chip trên cùng board.

### *1.2. Cách lấy dữ liệu*

Dựa vào setting của CPOL và CPHA, SPI có 4 modes:

+ Mode 1 (CPOL=0, CPHA=0): ở idle thì clock kéo xuống low, lấy data ở sườn lên (rising edge).
+ Mode 2 (CPOL=0, CPHA=1): ở idle thì clock kéo xuống low, lấy data ở sườn xuống (failing edge).
+ Mode 3 (CPOL=1, CPHA=0): ở idle thì clock kéo lên high, lấy data ở sườn xuống (failing edge).
+ Mode 4 (CPOL=1, CPHA=0): ở idle thì clock kéo lên high, lấy data ở sườn lên (rising edge).

### *1.3. SPI frame*

SPI frame có độ dài 8bits data frame.

### *1.4. Các bước tạo message trên line*

Để truyền/nhận sử dụng SPI protocol thì:

1. Cấu hình clock, chọn chế độ hoạt động SPI bằng CPOL và CPHA.
2. Kéo chân CS của SPI Slave tương ứng về LOW.
3. Master tạo xung clock trên SCLK.
4. Các thanh ghi trên Master và Slave cập nhật giá trị, dịch bit.
5. Lặp lại quá trình từ 2. cho đến khi truyền được 8 bits.

## 2. I2C

### *2.1 Khái niệm*

Machine technique: Có dây, nối tiếp.

Số dây: 2 dây

+ SCL - Serial clock line.
+ SDA - Serial Data line.

![I2C_diagram.png]

Direction: Half duplex.

I2C bus: N Masters, M Slaves.

Data check: ACK. 

Speed: 

+ Standard mode: 100 kbps.
+ Fast mode: 400 kbps.
+ Fast mode plus: 1 Mbps.
+ High-speed mode: 3.4 Mbps.
+ Ultra fast mode: 5Mbps.

### *2.2. Cách lấy dữ liệu*

I2C có 2 chế độ làm việc chính:

+ Master mode.
+ Slave mode.

Data được lấy khi SCL ở HIGH. Data chỉ có thể được thay đổi khi SCL ở LOW.

### *2.3. I2C frame*

Các I2C device giao tiếp với nhau thông qua các Messages. Mỗi Message gồm các frame sau:

![I2C_Message.png]

+ Start bit (1 bit): tạo ra bằng cách kéo SDA xuống LOW, sau đó kéo SCL xuống LOW.
+ Address bits (7 hoặc 10 bits): được cấu hình trước ở thanh ghi. Master truyền Address bits đến tất cả các Slaves. Các Slaves nhận và phản hồi master bằng ACK/NACK. Các Slaves so sánh Address bits mà Master truyền với Address bản thân, nếu giống thì bắt đầu thực hiện request từ Master (R/W bit).
+ Data bits (8 bits): Master truyền data cho Slave nếu Write mode (R/W = 1), Master nhận data từ Slave nếu Read mode (R/W = 0).
+ Stop bit (1 bit): tạo ra bằng cách kéo SDA lên HIGH trong khi SCL đang HIGH.

### *2.4. Các bước tạo message trên line*

Để truyền/nhận sử dụng I2C protocol thì:

1. Master gửi tín hiệu bắt đầu bằng cách kéo SDA xuống LOW trong khi SCL đang ở HIGH. Sau đó kéo SCL về LOW.
2. Master tạo xung clock trên SCL.
3. Master gửi Address kèm 1 bit R/W cho tất cả Slaves. Slaves nhận kiểm tra Address frame và so sánh với Address bản thân. 

+ Nếu Address đủ và giống vs Slave Address thì phản hồi bằng cách truyền ACK/NACK bit = LOW cho Master.
+ Nếu Address không đủ hoặc khác Slave Address thì Slave rời khỏi line SDA (SDA ở clock thứ 9 vẫn bằng HIGH).

4. Dựa vào R/W bit (Read: R/W = 0, Write: R/W = 1):

+ R/W = 0 (Master request đọc giá trị từ Slave): Sau khi Slave kéo SDA về LOW như bước 3, Slave truyền Data frame cho Master. Master nhận và truyền lại ACK, quá tình lặp lại cho đến khi hết data cần truyền.
+ R/W = 1 (Master request ghi giá trị vào Slave): Sau khi Slave kéo SDA về LOW như bước 3, Master truyền Data frame cho Slave. Slave nhận và truyền lại ACK, quá tình lặp lại cho đến khi hết data cần truyền.

5. Khi hết data cần truyền, Master tạo ra Stop bit bằng cách kéo SDA lên HIGH trong khi SCL ở mức HIGH.

## 3. UART

### *3.1 Khái niệm*

Machine technique: Có dây, nối tiếp.

Số dây: 3 dây

+ Tx - Transmit.
+ Rx - Receive.
+ GND - Ground.

![UART_diagram.png]

Direction: Full duplex, Half duplex, Simplex.

UART bus: Peer to Peer (không chia master-slave).

Data check: Parity.

Speed: 5 Mbps.

### *3.2. Cách lấy dữ liệu*

Dựa vào Baundrate được cấu hình, ta có thể lấy được Data trong UART frame.

### *3.3. UART frame*

UART devices giao tiếp với nhau theo chuẩn các UART frame. Mỗi frame bao gồm:

![UART_frame.png]

+ Ở idle, UART bus được kéo lên HIGH.
+ Start bit (1 bit): tạo ra bằng cách kéo line xuống LOW.
+ Data bits (5 đến 9 bits): data cần truyền trong 1 frame.
+ Parity bit (1 bits): Kiểm tra tính toàn vẹn của frame bằng tính chẵn lẻ. Nếu số lượng bit 1 là lẻ thì Parity bit là 1, ngược lại Parity bit là 0.
+ Stop bit (1,1.5,2 bits): số lượng bit được cấu hình trước, tạo ra bằng cách kéo line lên HIGH.

### *3.4. Các bước tạo message trên line*

Để truyền/nhận sử dụng UART protocol thì:

1. Cấu hình clock, cấu hình baundrate, cấu hình sử dụng Parity bit.
2. UART bus được kéo xuống LOW bởi UART device để tạo Start bit.
3. Transmiting UART truyền data frame (5 đến 9 bits) kèm bit Parity (nếu cấu hình sử dụng) đến Receiving UART.
4. UART bus được kéo lên HIGH bởi UART device để tạo Stop bit.   
