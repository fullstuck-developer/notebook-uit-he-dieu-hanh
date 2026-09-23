# HỆ ĐIỀU HÀNH
## CHƯƠNG 8: BỘ NHỚ ẢO
Trình bày các khái niệm cơ bản về bộ nhớ ảo, kỹ thuật cài đặt bộ nhớ ảo, một số vấn đề trong bộ nhớ ảo như cấp phát khung trang và tình trạng trì trệ.

### Các nội dung đã học
- Chương 1: Tổng quan về hệ điều hành
- Chương 2: Cấu trúc hệ điều hành
- Chương 3: Quản lý tiến trình
- Chương 4: Định thời CPU
- Chương 5: Đồng bộ hoá tiến trình
- Chương 6: Tắc nghẽn
- Chương 7: Quản lý bộ nhớ
- **Chương 8: Bộ nhớ ảo**
- Chương 9: Hệ điều hành Linux và Hệ điều hành Windows

---

## NỘI DUNG
1. Hiểu được các khái niệm tổng quan về bộ nhớ ảo
2. Hiểu và vận dụng kỹ thuật cài đặt bộ nhớ ảo demand paging
3. Hiểu được một số vấn đề trong bộ nhớ ảo: cấp phát frames và thrashing

---

## 1. Tổng quan về bộ nhớ ảo

### Nhắc lại về dynamic loading
- **Cơ chế**: chỉ khi nào cần được gọi đến thì một thủ tục mới được nạp vào bộ nhớ chính ⇒ tăng độ hiệu dụng của bộ nhớ bởi vì các thủ tục không được gọi đến sẽ không chiếm chỗ trong bộ nhớ.
- Rất hiệu quả trong trường hợp tồn tại khối lượng lớn mã chương trình có tần suất sử dụng thấp, không được sử dụng thường xuyên (ví dụ các thủ tục xử lý lỗi).
- **Hỗ trợ từ hệ điều hành**:
  - Thông thường, user chịu trách nhiệm thiết kế và hiện thực các chương trình có dynamic loading.
  - Hệ điều hành chủ yếu cung cấp một số thủ tục thư viện hỗ trợ, tạo điều kiện dễ dàng hơn cho lập trình viên.

### 8.1 Tổng quan về bộ nhớ ảo
- **Nhận xét**: không phải tất cả các phần của một tiến trình cần thiết phải được nạp vào bộ nhớ chính tại cùng một thời điểm.
- **Ví dụ**:
  - Đoạn mã điều khiển các lỗi hiếm khi xảy ra.
  - Các arrays, list, tables được cấp phát bộ nhớ (cấp phát tĩnh) nhiều hơn yêu cầu thực sự.
  - Một số tính năng ít khi được dùng của một chương trình.
  - Cả chương trình thì cũng có đoạn code chưa cần dùng.
- **Bộ nhớ ảo (virtual memory)**: Bộ nhớ ảo là một kỹ thuật cho phép xử lý một tiến trình không được nạp toàn bộ vào bộ nhớ vật lý.

**Ưu điểm của bộ nhớ ảo:**
- Số lượng tiến trình trong bộ nhớ nhiều hơn.
- Một tiến trình có thể thực thi ngay cả khi kích thước của nó lớn hơn bộ nhớ thực.
- Giảm nhẹ công việc của lập trình viên.
- Không gian tráo đổi giữa bộ nhớ chính và bộ nhớ phụ (swap space).

**Ví dụ:**
- swap partition trong Linux
- file `pagefile.sys` trong Windows

---

## 2. Cài đặt bộ nhớ ảo

### 8.2.1 Cài đặt bộ nhớ ảo
- Có hai kỹ thuật:
  - Phân trang theo yêu cầu (Demand Paging)
  - Phân đoạn theo yêu cầu (Demand Segmentation)
- Phần cứng memory management phải hỗ trợ paging và/hoặc segmentation.
- OS phải quản lý sự di chuyển của trang/đoạn giữa bộ nhớ chính và bộ nhớ thứ cấp.
- Trong chương này:
  - Chỉ quan tâm đến paging.
  - Phần cứng hỗ trợ hiện thực bộ nhớ ảo.
  - Chỉ tập trung vào các giải thuật của hệ điều hành.

### 8.2.2 Phân trang theo yêu cầu (Demand Paging)
- **Demand paging**: các trang của tiến trình chỉ được nạp vào bộ nhớ chính khi được yêu cầu.
- Khi có một tham chiếu đến một trang mà không có trong bộ nhớ chính (valid bit) thì phần cứng sẽ gây ra một ngắt (gọi là page-fault trap) kích khởi page-fault service routine (PFSR) của hệ điều hành.
- **PFSR (Page-Fault Service Routine)**:
  - Bước 1: Chuyển tiến trình về trạng thái blocked.
  - Bước 2: Phát ra một yêu cầu đọc đĩa để nạp trang được tham chiếu vào một frame trống; trong khi đợi I/O, một tiến trình khác được cấp CPU để thực thi.
  - Bước 3: Sau khi I/O hoàn tất, đĩa gây ra một ngắt đến hệ điều hành; PFSR cập nhật page table và chuyển tiến trình về trạng thái ready.

**Khi cần thay thế trang (Bước 2 của PFSR bổ sung):**
- Giả sử phải thay trang vì không tìm được frame trống, PFSR được bổ sung như sau:
  - Xác định vị trí trên đĩa của trang đang cần.
  - Tìm một frame trống:
    - Nếu có frame trống thì dùng nó.
    - Nếu không có frame trống thì dùng một giải thuật thay trang để chọn một trang hy sinh (victim page).
    - Ghi victim page lên đĩa; cập nhật page table và frame table tương ứng.
  - Đọc trang đang cần vào frame trống (đã có được từ bước 2); cập nhật page table và frame table tương ứng.

### 8.2.3 Thay thế trang nhớ
**Các vấn đề chủ yếu:**
- **Frame-allocation algorithm** (Thuật toán cấp phát khung trang):
  - Cấp phát cho tiến trình bao nhiêu frame của bộ nhớ thực?
- **Page-replacement algorithm** (Thuật toán thay thế trang):
  - Chọn frame của tiến trình sẽ được thay thế trang nhớ.
  - Mục tiêu: số lượng page-fault nhỏ nhất.
  - Được đánh giá bằng cách thực thi giải thuật đối với một chuỗi tham chiếu bộ nhớ (memory reference string) và xác định số lần xảy ra page fault.

**Ví dụ về chuỗi tham chiếu bộ nhớ (trang nhớ):**
Thứ tự tham chiếu các địa chỉ nhớ, với page size = 100:
0098, 0432, 0201, 0612, 0302, 0103, 0104, 0101, 0611, 0102, 0103, 0104, 0101, 0610, 0102, 0103, 0104, 0101, 0609, 0102, 0105
Các trang nhớ sau được tham chiếu lần lượt = chuỗi tham chiếu bộ nhớ:
`0, 4, 2, 6, 3, 1, 1, 1, 6, 1, 1, 1, 1, 6, 1, 1, 1, 1, 6, 1, 1`

---

## 3. Các giải thuật thay trang

Các dữ liệu cần biết ban đầu: Số khung trang, Tình trạng ban đầu, Chuỗi tham chiếu.

### 8.3.1 Các giải thuật thay trang
- Giải thuật thay trang FIFO (First-In-First-Out)
- Giải thuật thay trang OPT (Optimal)
- Giải thuật thay trang LRU (Least Recently Used)

### 8.3.2 Giải thuật thay trang FIFO
Giải thuật thay trang FIFO thay thế trang nhớ có thời gian được nạp vào bộ nhớ sớm nhất trong các trang nhớ.
- **Ví dụ**: Xét một tiến trình có 8 trang, 3 khung trang (ban đầu trống), chuỗi tham chiếu: `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1`
  - Tổng số lỗi trang: 15.

### 8.3.3 Nghịch lý Belady (Belady's Anomaly)
Bất thường (anomaly) Belady: số page fault tăng mặc dù tiến trình đã được cấp nhiều frame hơn.
- Ví dụ với chuỗi `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`:
  - Sử dụng 3 khung trang: 9 lỗi trang
  - Sử dụng 4 khung trang: 10 lỗi trang

### 8.3.4 Giải thuật thay trang OPT
Giải thuật thay trang OPT thay thế trang nhớ sẽ được tham chiếu trễ nhất trong tương lai ⇒ cần phải biết trước các trang sẽ được tham chiếu trong tương lai.
- **Ví dụ**: Với chuỗi tham chiếu như phần FIFO và 3 khung trang.
  - Tổng số lỗi trang: 9.

### 8.3.5 Giải thuật thay trang LRU
- Mỗi trang được ghi nhận (trong bảng phân trang) thời điểm được tham chiếu ⇒ trang LRU là trang nhớ có thời điểm tham chiếu nhỏ nhất (OS tốn chi phí tìm kiếm trang nhớ LRU này mỗi khi có page fault).
- Do vậy, LRU cần sự hỗ trợ của phần cứng và chi phí cho việc tìm kiếm. Ít CPU cung cấp đủ sự hỗ trợ phần cứng cho giải thuật LRU.
- **Ví dụ**: Với chuỗi tham chiếu như phần FIFO và 3 khung trang.
  - Tổng số lỗi trang: 12.

---

## 4. Vấn đề cấp phát Frames

### 8.4.1 Số lượng frame cấp cho tiến trình
- OS phải quyết định cấp cho mỗi tiến trình bao nhiêu frame.
  - Cấp ít frame ⇒ nhiều page fault.
  - Cấp nhiều frame ⇒ giảm mức độ multiprogramming.
- **Chiến lược cấp phát tĩnh (fixed-allocation)**:
  - Số frame cấp cho mỗi tiến trình không đổi, được xác định vào thời điểm loading và có thể tùy thuộc vào từng ứng dụng (kích thước của nó,…).
- **Chiến lược cấp phát động (variable-allocation)**:
  - Số frame cấp cho mỗi tiến trình có thể thay đổi trong khi nó chạy:
    - Nếu tỷ lệ page-fault cao ⇒ cấp thêm frame.
    - Nếu tỷ lệ page-fault thấp ⇒ giảm bớt frame.
  - Hệ điều hành phải mất chi phí để ước định các tiến trình.

### 8.4.2 Chiến lược cấp phát tĩnh
- **Cấp phát bằng nhau**:
  - Ví dụ, có 100 frame và 5 tiến trình → mỗi tiến trình được 20 frame.
- **Cấp phát theo tỉ lệ**: dựa vào kích thước tiến trình
  - $s_i = \text{size of process } p_i$
  - $S = \sum s_i$
  - $m = \text{total number of frames}$
  - $a_i = \text{allocation for } p_i = \frac{s_i}{S} \times m$
  - Ví dụ: $m = 64, s_1 = 10, s_2 = 127 \Rightarrow a_1 = \frac{10}{137} \times 64 \approx 5, a_2 = \frac{127}{137} \times 64 \approx 59$
- **Cấp phát theo độ ưu tiên**.

---

## 5. Vấn đề Thrashing

### 8.5.1 Trì trệ trên toàn bộ hệ thống
- Nếu một tiến trình không có đủ số frame cần thiết thì tỉ số page faults/sec rất cao.
- **Thrashing**: hiện tượng các trang nhớ của một tiến trình bị hoán chuyển vào/ra liên tục.
- Biểu đồ CPU utilization so với degree of multiprogramming: CPU utilization sẽ tăng đến một mức độ đa chương định, sau đó giảm đột ngột (do thrashing).

### 8.5.2 Mô hình cục bộ
- Để hạn chế thrashing, hệ điều hành phải cung cấp cho tiến trình càng “đủ” frame càng tốt. Bao nhiêu frame thì đủ cho một tiến trình thực thi hiệu quả?
- **Nguyên lý locality (locality principle)**:
  - Locality là tập các trang được tham chiếu gần nhau.
  - Một tiến trình gồm nhiều locality, và trong quá trình thực thi, tiến trình sẽ chuyển từ locality này sang locality khác.
- Vì sao hiện tượng thrashing xuất hiện?
  - Khi $\sum \text{size of locality} > \text{memory size}$

### 8.5.3 Giải pháp tập làm việc (Working-Set Model)
- Được thiết kế dựa trên nguyên lý locality.
- Xác định xem tiến trình thực sự sử dụng bao nhiêu frame.
- **Định nghĩa**:
  - $WS(t)$: các tham chiếu trang nhớ của tiến trình gần đây nhất cần được quan sát.
  - $\Delta$: khoảng thời gian tham chiếu.
- **Định nghĩa chi tiết**: Working set của tiến trình $P_i$, ký hiệu $WS_i$, là tập gồm $\Delta$ các trang được sử dụng gần đây nhất.
- **Nhận xét về $\Delta$**:
  - $\Delta$ quá nhỏ ⇒ không đủ bao phủ toàn bộ locality.
  - $\Delta$ quá lớn ⇒ bao phủ nhiều locality khác nhau.
  - $\Delta = \infty$ ⇒ bao gồm tất cả các trang được sử dụng.
  - Dùng working set của một tiến trình để xấp xỉ locality của nó.
- **Kích thước working set ($WSS_i$)**: $WSS_i$ = số lượng các trang trong $WS_i$.
- **Giải pháp working set**:
  - Đặt $D = \sum WSS_i$ = tổng các working-set size của mọi tiến trình trong hệ thống.
  - Nếu $D > m$ (số frame của hệ thống) ⇒ sẽ xảy ra thrashing.
  - Khi khởi tạo một tiến trình: cung cấp cho quá trình số lượng frame thỏa mãn working-set size của nó.
  - Nếu $D > m$ ⇒ tạm dừng một trong các tiến trình (các trang của tiến trình đó được chuyển ra đĩa cứng và các frame của nó được thu hồi).
- WS loại trừ được tình trạng trì trệ mà vẫn đảm bảo mức độ đa chương.
- Đọc thêm: Hệ thống tập tin, Hệ thống nhập xuất, Hệ thống phân tán.

---

## Tóm tắt lại nội dung buổi học
- Tổng quan về bộ nhớ ảo
- Cài đặt bộ nhớ ảo: Demand Paging
- Các giải thuật thay trang (Page Replacement Algorithms)
- Vấn đề cấp phát Frames
- Vấn đề Thrashing

## Bài tập
Xét chuỗi truy xuất bộ nhớ sau:
`1, 2, 3, 4, 2, 1, 5, 6, 2, 1, 2, 3, 7, 6, 3, 2, 1`

Có bao nhiêu lỗi trang xảy ra khi sử dụng các thuật toán thay thế sau đây, giả sử hệ thống có 4 khung trang.
a. LRU
b. FIFO
c. Chiến lược tối ưu (OPT)

