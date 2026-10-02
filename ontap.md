# TÀI LIỆU ÔN TẬP TOÀN DIỆN MÔN HỆ ĐIỀU HÀNH
> **Học phần:** Hệ điều hành (Operating Systems)  
> **Trọng tâm ôn thi:** Chương 5 (Đồng bộ tiến trình), Chương 7 (Quản lý bộ nhớ), Chương 8 (Bộ nhớ ảo).  
> **Tiêu chí:** Đầy đủ ý, súc tích, nhấn mạnh từ khóa cốt lõi (**in đậm**) và có hướng dẫn giải bài tập chi tiết.

---

## MỤC LỤC
1. [PHẦN 1: CHƯƠNG 5 - ĐỒNG BỘ TIẾN TRÌNH (PROCESS SYNCHRONIZATION)](#phần-1-chương-5---đồng-bộ-tiến-trình)
   - [1.1. Race Condition & Vùng tranh chấp (Critical Section)](#11-race-condition--vùng-tranh-chấp-critical-section)
   - [1.2. Ba yêu cầu bắt buộc cho lời giải CS](#12-ba-yêu-cầu-bắt-buộc-cho-lời-giải-cs)
   - [1.3. Các giải pháp phần mềm & Giải thuật Peterson](#13-các-giải-pháp-phần-mềm--giải-thuật-peterson)
   - [1.4. Các giải pháp hỗ trợ từ phần cứng](#14-các-giải-pháp-hỗ-trợ-từ-phần-cứng)
   - [1.5. Khóa Mutex (Mutex Locks)](#15-khóa-mutex-mutex-locks)
   - [1.6. Semaphore (Đèn hiệu)](#16-semaphore-đèn-hiệu)
   - [1.7. Monitor & Condition Variables](#17-monitor--condition-variables)
   - [1.8. Vấn đề Liveness (Deadlock, Starvation, Priority Inversion)](#18-vấn-đề-liveness)
   - [1.9. Ba bài toán đồng bộ kinh điển (Bounded-Buffer, Readers-Writers, Dining-Philosophers)](#19-ba-bài-toán-đồng-bộ-kinh-điển)
2. [PHẦN 2: CHƯƠNG 7 - QUẢN LÝ BỘ NHỚ (MEMORY MANAGEMENT)](#phần-2-chương-7---quản-lý-bộ-nhớ)
   - [2.1. Khái niệm cơ sở & Các kiểu địa chỉ](#21-khái-niệm-cơ-sở--các-kiểu-địa-chỉ)
   - [2.2. Chuyển đổi địa chỉ (Binding) & Nạp/Liên kết động](#22-chuyển-đổi-địa-chỉ-binding--nạpliên-kết-động)
   - [2.3. Các mô hình phân vùng bộ nhớ & Phân mảnh](#23-các-mô-hình-phân-vùng-bộ-nhớ--phân-mảnh)
   - [2.4. Cơ chế Phân trang (Paging) & Bảng trang](#24-cơ-chế-phân-trang-paging--bảng-trang)
   - [2.5. TLB & Thời gian truy xuất hiệu dụng (EAT)](#25-tlb--thời-gian-truy-xuất-hiệu-dụng-eat)
   - [2.6. Bảo vệ bộ nhớ & Cơ chế Hoán vị (Swapping)](#26-bảo-vệ-bộ-nhớ--cơ-chế-hoán-vị-swapping)
3. [PHẦN 3: CHƯƠNG 8 - BỘ NHỚ ẢO (VIRTUAL MEMORY)](#phần-3-chương-8---bộ-nhớ-ảo)
   - [3.1. Tổng quan Bộ nhớ ảo & Demand Paging](#31-tổng-quan-bộ-nhớ-ảo--demand-paging)
   - [3.2. Quy trình xử lý lỗi trang (Page-Fault Service Routine)](#32-quy-trình-xử-lý-lỗi-trang-pfsr)
   - [3.3. Các thuật toán thay thế trang (FIFO, OPT, LRU) & Nghịch lý Belady](#33-các-thuật-toán-thay-thế-trang)
   - [3.4. Cấp phát khung trang (Frame Allocation)](#34-cấp-phát-khung-trang-frame-allocation)
   - [3.5. Hiện tượng Trì trệ (Thrashing) & Mô hình Working-Set](#35-hiện-tượng-trì-trệ-thrashing--mô-hình-working-set)
4. [PHẦN 4: CÁC DẠNG BÀI TẬP TÍNH TOÁN ĐI THI (HƯỚNG DẪN CHI TIẾT)](#phần-4-các-dạng-bài-tập-tính-toán-đi-thi)
   - [Dạng 1: Cấp phát bộ nhớ động (First-fit, Best-fit, Worst-fit, Next-fit)](#dạng-1-cấp-phát-bộ-nhớ-động)
   - [Dạng 2: Phân trang & Tính toán số bit địa chỉ](#dạng-2-phân-trang--tính-toán-số-bit-địa-chỉ)
   - [Dạng 3: Tính thời gian truy xuất hiệu dụng (EAT với TLB)](#dạng-3-tính-thời-gian-truy-xuất-hiệu-dụng-eat)
   - [Dạng 4: Mô phỏng thuật toán thay trang (FIFO, OPT, LRU)](#dạng-4-mô-phỏng-thuật-toán-thay-trang)
   - [Dạng 5: Cấp phát khung trang theo tỉ lệ](#dạng-5-cấp-phát-khung-trang-theo-tỉ-lệ)
5. [PHẦN 5: BẢNG SO SÁNH TỔNG HỢP & PHẢN XẠ NHANH ĐI THI](#phần-5-bảng-so-sánh-tổng-hợp--phản-xạ-nhanh)

---

# PHẦN 1: CHƯƠNG 5 - ĐỒNG BỘ TIẾN TRÌNH

## 1.1. Race Condition & Vùng tranh chấp (Critical Section)
- **Race Condition (Hiện tượng tranh chấp dữ liệu):**
  - Là hiện tượng xảy ra khi **nhiều tiến trình cùng truy cập và thao tác đồng thời trên dữ liệu chia sẻ**.
  - **Kết quả cuối cùng phụ thuộc vào thứ tự thực thi** (đan xen lệnh) của các tiến trình.
  - Hậu quả: Dữ liệu bị **sai lệch, không nhất quán (inconsistency)**.
  - *Ví dụ 1 (Producer - Consumer):* Lệnh `count++` và `count--` ở mức máy gồm 3 thao tác (`Load`, `Inc/Dec`, `Store`). Khi bị ngắt xen kẽ giữa chừng do hết quantum time, giá trị `count` bị sai.
  - *Ví dụ 2 (Cấp phát PID):* Hai tiến trình cùng gọi `fork()` đồng thời, cùng đọc biến `next_available_pid` $\rightarrow$ bị cấp trùng một PID.
- **Critical Section (CS - Vùng tranh chấp):**
  - Là **đoạn mã lệnh** trong chương trình mà ở đó tiến trình **truy cập và thay đổi dữ liệu chia sẻ** (biến, bảng, tệp tin).
  - Cấu trúc chuẩn của một tiến trình khi xử lý CS:
    ```c
    while (true) {
        entry section      // Xin phép vào CS
            critical section   // Thao tác trên dữ liệu chia sẻ
        exit section       // Báo hiệu đã rời CS, nhường cho tiến trình khác
            remainder section  // Phần mã xử lý độc lập còn lại
    }
    ```

---

## 1.2. Ba yêu cầu bắt buộc cho lời giải CS
Mọi giải pháp cho bài toán vùng tranh chấp **bắt buộc phải thỏa mãn đồng thời 3 điều kiện**:
1. **Loại trừ tương hỗ (Mutual Exclusion):** 
   - Tại một thời điểm, **chỉ có duy nhất một tiến trình** được thực thi trong vùng tranh chấp của nó.
2. **Tiến triển (Progress):** 
   - Nếu không có tiến trình nào trong CS và có các tiến trình muốn vào CS, thì **chỉ những tiến trình không nằm trong remainder section** mới được tham gia quyết định tiến trình nào được vào tiếp theo.
   - Một tiến trình tạm dừng bên ngoài CS **không được ngăn cản** các tiến trình khác vào CS. (Ngăn chặn tình trạng **Deadlock**).
3. **Chờ đợi có giới hạn (Bounded Waiting):** 
   - Phải tồn tại một **giới hạn về số lần** các tiến trình khác được phép vào CS sau khi một tiến trình đã đưa ra yêu cầu vào CS và trước khi yêu cầu đó được chấp nhận.
   - Đảm bảo **không có tiến trình nào bị bỏ đói vô hạn (Starvation)**.

---

## 1.3. Các giải pháp phần mềm & Giải thuật Peterson

### a. Vô hiệu hóa ngắt (Disable Interrupt)
- Cơ chế: `Entry section` tắt ngắt; `Exit section` bật ngắt trở lại.
- **Hạn chế:**
  - Nguy hiểm nếu CS chạy vô tận hoặc chạy quá lâu (hệ thống bị treo, mất đáp ứng).
  - **Không hoạt động trên hệ thống đa xử lý (Multiprocessor)** vì việc tắt ngắt trên một CPU không ngăn được CPU khác truy cập bộ nhớ chung.

### b. Giải pháp phần mềm 1 (Dùng biến `turn`)
- Ý tưởng: Dùng biến chung `int turn` (khởi tạo bằng `0` hoặc `1`). Tiến trình $P_i$ chờ khi `turn == j`, sau CS gán `turn = j`.
- **Đánh giá:**
  - Thỏa **Mutual Exclusion**.
  - **Vi phạm Progress & Bounded Waiting:** Do luân phiên gượng ép (*strict alternation*). Nếu $P_j$ dừng ở remainder section hoặc không có nhu cầu vào CS, $P_i$ muốn vào lại sẽ bị chặn vĩnh viễn vì `turn` vẫn đang bằng $j$.

### c. Giải pháp phần mềm 2 (Dùng mảng `flag[]`)
- Ý tưởng: Dùng mảng `boolean flag[2]`. Tiến trình $P_i$ giơ cờ `flag[i] = true`, sau đó kiểm tra `while(flag[j]);`. Rời CS thì hạ cờ `flag[i] = false`.
- **Đánh giá:**
  - Thỏa **Mutual Exclusion**.
  - **Vi phạm Progress (Nguy cơ Deadlock):** Nếu cả 2 tiến trình đồng thời đặt cờ lên `true` cùng lúc trước vòng lặp `while`, cả hai sẽ kẹt mãi mãi trong `while(flag[j])`.

### d. Giải thuật Peterson (Hoàn chỉnh cho 2 tiến trình)
- Sử dụng kết hợp cả 2 biến: `int turn;` và `boolean flag[2];`.
- **Mã nguồn chuẩn:**
  ```c
  // Mã của tiến trình P_i (tiến trình đối thủ là P_j)
  while (true) {
      flag[i] = true;                 // 1. P_i sẵn sàng vào CS
      turn = j;                       // 2. Nhường lượt ưu tiên cho P_j
      while (flag[j] && turn == j);   // 3. Busy wait nếu P_j cũng sẵn sàng VÀ đang là lượt P_j
      
      /* CRITICAL SECTION */
      
      flag[i] = false;                // 4. Rời CS, hạ cờ
      
      /* REMAINDER SECTION */
  }
  ```
- **Chứng minh:** Thỏa mãn cả 3 điều kiện:
  - *Mutual Exclusion:* Để cả 2 cùng vào CS thì `flag[0] == flag[1] == true`, nhưng biến `turn` chỉ có thể nhận 1 giá trị (0 hoặc 1) tại 1 thời điểm $\rightarrow$ mâu thuẫn.
  - *Progress:* Tiến trình nào nhường lượt sau sẽ phải chờ tiến trình kia vào trước. Tiến trình ở remainder section có `flag = false` nên không cản trở tiến trình khác.
  - *Bounded Waiting:* Mỗi tiến trình chỉ phải chờ tối đa 1 lượt của tiến trình kia.
- **Hạn chế trên kiến trúc hiện đại:**
  - CPU và trình biên dịch hiện đại có cơ chế **tối ưu sắp xếp lại lệnh (Instruction Reordering)** đối với các thao tác độc lập (ví dụ đảo thứ tự gán `flag[i]` và `turn`).
  - Hậu quả: Giải thuật Peterson có thể bị sai trên hệ thống đa lõi hiện đại.
  - Khắc phục: Phải sử dụng rào cản bộ nhớ (**Memory Barrier**).

---

## 1.4. Các giải pháp hỗ trợ từ phần cứng
1. **Memory Barrier (Lớp chắn bộ nhớ):**
   - Chỉ thị phần cứng bắt buộc mọi thao tác đọc/ghi bộ nhớ trước nó phải được hoàn tất và lan truyền đến tất cả CPU trước khi các lệnh sau nó được thực hiện.
   - Khắc phục lỗi Instruction Reordering trên kiến trúc nhớ sắp xếp yếu (*weakly ordered*).
2. **Lệnh đơn nguyên `TestAndSet`:**
   ```c
   boolean test_and_set(boolean *target) {
       boolean rv = *target;
       *target = true;
       return rv;
   }
   // Áp dụng bảo vệ CS:
   while (test_and_set(&lock)); // Busy wait khi lock đang true
   /* CS */
   lock = false;
   ```
3. **Lệnh đơn nguyên `CompareAndSwap` (CAS):**
   ```c
   int compare_and_swap(int *value, int expected, int new_value) {
       int temp = *value;
       if (*value == expected)
           *value = new_value;
       return temp;
   }
   // Áp dụng: while (compare_and_swap(&lock, 0, 1) != 0); /* CS */ lock = 0;
   ```
4. **Biến đơn nguyên (Atomic Variables):**
   - Hỗ trợ các kiểu dữ liệu nguyên thủy (như `atomic_int`) thực hiện các thao tác tăng, giảm, cập nhật tự động mà không bị ngắt quãng.

---

## 1.5. Khóa Mutex (Mutex Locks)
- Là công cụ đồng bộ phần mềm đơn giản nhất để đảm bảo **Mutual Exclusion**.
- Có 1 biến cờ trạng thái boolean `available`.
- Hai thao tác nguyên tử:
  - `acquire()`: Yêu cầu khóa (nếu khóa bận thì chờ/ngủ, nếu rỗi thì chiếm khóa).
  - `release()`: Giải phóng khóa.

### So sánh 2 cơ chế Mutex:
| Tiêu chí | Spinlock (Busy Waiting) | Mutex Sleep - Wakeup (Non-busy waiting) |
| :--- | :--- | :--- |
| **Cơ chế** | Vòng lặp `while(!available);` liên tục kiểm tra. | Nếu bận, gọi `block()` đưa tiến trình vào hàng đợi **ngủ**; khi mở khóa gọi `wakeup()` đánh thức. |
| **Tài nguyên CPU** | **Lãng phí chu kỳ CPU** do liên tục lặp kiểm tra. | **Tiết kiệm CPU**, nhường CPU cho tiến trình khác. |
| **Chi phí ngữ cảnh** | Không tốn chi phí chuyển ngữ cảnh (context switch). | Tốn chi phí chuyển ngữ cảnh (lưu/phục hồi PCB). |
| **Khi nào nên dùng?** | Khi **thời gian thực thi trong CS rất ngắn** (thường dùng trong kernel, hệ thống đa xử lý). | Khi **thời gian trong CS dài** hoặc hệ thống đơn CPU. |

---

## 1.6. Semaphore (Đèn hiệu)
- **Bản chất:** Semaphore $S$ là một **biến số nguyên** chỉ được truy cập qua 2 thao tác nguyên tử: `wait()` (còn gọi là $P$) và `signal()` (còn gọi là $V$).

### a. Phân loại Semaphore:
1. **Binary Semaphore:** Giá trị chỉ có thể là **0** hoặc **1** (hoạt động tương tự như Khóa Mutex).
2. **Counting Semaphore:** Giá trị là số nguyên **không giới hạn**, dùng để **quản lý số lượng tài nguyên hữu hạn**.

### b. Định nghĩa thao tác (Cài đặt không Busy Waiting):
```c
typedef struct {
    int value;
    struct process *list; // Hàng đợi các tiến trình đang chờ
} semaphore;

void wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        // Thêm tiến trình này vào S->list
        block(); // Đưa tiến trình vào trạng thái ngủ
    }
}

void signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        // Lấy một tiến trình P ra khỏi S->list
        wakeup(P); // Đưa P vào Ready Queue
    }
}
```
> **Ý NGHĨA QUAN TRỌNG CỦA `S->value` (CÂU HỎI THI CỰC KỲ PHỔ BIẾN):**
> - Khi **`S->value >= 0`**: Biểu thị **số lượng tài nguyên khả dụng** (số tiến trình có thể gọi `wait()` mà không bị chặn).
> - Khi **`S->value < 0`**: Độ lớn tuyệt đối **$|S\text{->value}|$ chính là số lượng tiến trình đang bị nghẽn (blocked)** và đang chờ trong hàng đợi `S->list`.

### c. Ba mô hình ứng dụng cốt lõi của Semaphore:
1. **Bảo vệ Vùng tranh chấp (Mutual Exclusion):** Khởi tạo `sem = 1`. Trước CS gọi `wait(sem)`, sau CS gọi `signal(sem)`.
2. **Quy định Thứ tự thực thi (Execution Ordering):** Để lệnh $S_1$ của $P_1$ luôn chạy trước $S_2$ của $P_2$:
   - Khởi tạo `synch = 0`.
   - $P_1$: Thực thi $S_1 \rightarrow$ `signal(synch);`
   - $P_2$: `wait(synch);` $\rightarrow$ Thực thi $S_2$.
3. **Quản lý giới hạn tài nguyên:** Khởi tạo $S = N$ (với $N$ là số lượng tài nguyên).

---

## 1.7. Monitor & Condition Variables
- **Monitor:**
  - Là **kiểu dữ liệu trừu tượng (ADT)** cấp cao đóng gói dữ liệu dùng chung và các hàm thao tác.
  - **Tự động đảm bảo tính loại trừ tương hỗ (Mutual Exclusion)**: Tại một thời điểm, chỉ có duy nhất một tiến trình được thực thi bên trong monitor.
- **Biến điều kiện (Condition Variable):**
  - Cung cấp cơ chế cho phép tiến trình tạm dừng và chờ đợi điều kiện cụ thể bên trong monitor.
  - Khai báo: `condition x, y;`
  - Thao tác:
    - `x.wait()`: Tiến trình gọi lệnh bị **tạm dừng (block)** và đưa vào hàng đợi `condition queue x`. Khóa monitor được nhường cho tiến trình khác.
    - `x.signal()`: Đánh thức **đúng một tiến trình** đang bị block trên hàng đợi `x`. Nếu không có tiến trình nào đang chờ thì lệnh `signal()` **vô tác dụng** (khác với semaphore sẽ tăng biến đếm).

---

## 1.8. Vấn đề Liveness
- **Liveness:** Thuộc tính đảm bảo rằng các tiến trình phải có sự tiến triển thực sự (không bị tắc nghẽn vĩnh viễn).
- **Deadlock (Bế tắc):** Hai hoặc nhiều tiến trình chờ đợi vô hạn một sự kiện mà sự kiện đó chỉ có thể được tạo ra bởi một trong các tiến trình đang chờ đó.
  - *Ví dụ:* $P_0$ giữ $S$ chờ $Q$; $P_1$ giữ $Q$ chờ $S$.
- **Starvation (Đói tài nguyên):** Một tiến trình bị kẹt vô hạn trong hàng đợi, không bao giờ được cấp tài nguyên.
- **Priority Inversion (Nghịch đảo độ ưu tiên):**
  - Xảy ra khi tiến trình ưu tiên cao ($H$) bị chặn bởi tiến trình ưu tiên thấp ($L$) vì $L$ đang giữ tài nguyên/khóa mà $H$ cần, trong khi tiến trình ưu tiên trung bình ($M$) lại chiếm CPU của $L$.
  - **Giải pháp:** Sử dụng giao thức thừa hưởng độ ưu tiên (**Priority Inheritance Protocol**): tạm thời nâng độ ưu tiên của tiến trình $L$ lên bằng $H$ cho đến khi $L$ giải phóng tài nguyên.

---

## 1.9. Ba bài toán đồng bộ kinh điển

### 1. Bounded-Buffer (Producer - Consumer)
- **Tài nguyên & Semaphore:**
  - `mutex = 1`: Đảm bảo loại trừ tương hỗ khi thêm/xóa phần tử trong buffer.
  - `empty = n`: Đếm số vị trí trống còn lại (khởi tạo bằng kích thước đệm $n$).
  - `full = 0`: Đếm số vị trí đã có dữ liệu (khởi tạo bằng 0).
- **Mã nguồn:**
  ```c
  // PRODUCER
  while (true) {
      /* tạo sản phẩm next_produced */
      wait(empty);        // Chờ có chỗ trống
      wait(mutex);        // Khóa buffer
      // Thêm sản phẩm vào buffer
      signal(mutex);      // Mở khóa buffer
      signal(full);       // Báo hiệu có thêm 1 sản phẩm
  }

  // CONSUMER
  while (true) {
      wait(full);         // Chờ có sản phẩm
      wait(mutex);        // Khóa buffer
      // Lấy sản phẩm khỏi buffer
      signal(mutex);      // Mở khóa buffer
      signal(empty);      // Báo hiệu có thêm 1 chỗ trống
      /* tiêu thụ sản phẩm */
  }
  ```
  *(Lưu ý thi: Thứ tự `wait(empty)` rồi mới `wait(mutex)` là bắt buộc. Nếu đảo ngược sẽ dẫn đến **Deadlock** khi bộ đệm đầy).*

### 2. Readers - Writers (Biến thể 1 - Ưu tiên Readers)
- **Quy tắc:** Cho phép nhiều Readers đọc cùng lúc; Writer độc quyền (chỉ 1 Writer và không có Reader nào).
- **Dữ liệu chia sẻ:**
  - `semaphore rw_mutex = 1`: Khóa tài nguyên dùng chung giữa Reader và Writer.
  - `semaphore mutex = 1`: Bảo vệ biến đếm `read_count`.
  - `int read_count = 0`: Đếm số Reader đang đọc.
- **Mã nguồn:**
  ```c
  // WRITER
  wait(rw_mutex);
  /* ghi/cập nhật dữ liệu */
  signal(rw_mutex);

  // READER
  wait(mutex);
  read_count++;
  if (read_count == 1)      // Reader đầu tiên đến -> khóa Writer
      wait(rw_mutex);
  signal(mutex);

  /* đọc dữ liệu */

  wait(mutex);
  read_count--;
  if (read_count == 0)      // Reader cuối cùng rời đi -> mở khóa cho Writer
      signal(rw_mutex);
  signal(mutex);
  ```
  *(Hạn chế: Ưu tiên Reader có thể khiến **Writer bị đói tài nguyên - Starvation** nếu liên tục có Reader mới đến).*

### 3. Dining-Philosophers (Triết gia ăn tối)
- **Phát biểu:** 5 triết gia, 5 chiếc đũa (`semaphore chopstick[5] = {1, 1, 1, 1, 1}`). Triết gia $i$ cần cả 2 đũa: đũa trái $i$ và đũa phải $(i+1)\%5$.
- **Nguy cơ Deadlock:** Nếu cả 5 triết gia đồng thời cầm đũa bên trái cùng lúc, tất cả sẽ chờ đũa bên phải mãi mãi.
- **Các giải pháp ngăn chặn Deadlock:**
  1. Chỉ cho phép tối đa 4 triết gia cùng ngồi vào bàn.
  2. Chỉ cho phép nhặt đũa khi cả 2 chiếc đều rảnh (thực hiện kiểm tra trong Critical Section).
  3. **Giải pháp bất đối xứng:** Triết gia vị trí lẻ nhặt đũa trái trước, đũa phải sau; triết gia vị trí chẵn nhặt đũa phải trước, đũa trái sau.

---

# PHẦN 2: CHƯƠNG 7 - QUẢN LÝ BỘ NHỚ

## 2.1. Khái niệm cơ sở & Các kiểu địa chỉ
- **Mục tiêu quản lý bộ nhớ:** Tối ưu hóa việc phân phối bộ nhớ, tăng mức độ đa chương (**Degree of Multiprogramming**), bảo vệ và chia sẻ bộ nhớ giữa các tiến trình.
- **Phân loại địa chỉ:**
  - **Địa chỉ vật lý (Physical Address):** Địa chỉ thực tế trên thanh RAM vật lý (từ $0$ đến $Max$).
  - **Địa chỉ luận lý / ảo (Logical / Virtual Address):** Địa chỉ do CPU phát ra khi thực thi chương trình.
  - **Địa chỉ tương đối (Relocatable / Relative):** Biểu diễn độ lệch tương đối so với một mốc (ví dụ địa chỉ nền của module).
  - **Địa chỉ tuyệt đối (Absolute):** Địa chỉ cố định tương ứng trực tiếp với địa chỉ vật lý.
- **Linker & Loader:**
  - **Linker:** Gom các object module và thư viện lại thành một file thực thi duy nhất (**load module**).
  - **Loader:** Nạp load module từ đĩa vào bộ nhớ chính để thực thi.

---

## 2.2. Chuyển đổi địa chỉ (Binding) & Nạp/Liên kết động

### a. Ba thời điểm kết gán địa chỉ (Address Binding):
1. **Thời điểm biên dịch (Compile Time):** 
   - Sử dụng khi biết trước vị trí nạp trong RAM. Tạo mã có **địa chỉ tuyệt đối**.
   - Nhược điểm: Phải biên dịch lại toàn bộ nếu địa chỉ nạp thay đổi (ví dụ: file `.COM` MS-DOS).
2. **Thời điểm nạp (Load Time):** 
   - Compiler tạo địa chỉ tương đối; khi nạp, Loader sẽ cộng địa chỉ nền để ra địa chỉ vật lý (**tái định vị tĩnh**).
   - Nhược điểm: Nếu tiến trình bị di chuyển vùng nhớ thì phải reload lại.
3. **Thời điểm thực thi (Execution Time / Run Time):** 
   - Quá trình ánh xạ bị trì hoãn cho đến khi lệnh thực sự chạy.
   - **Cần sự hỗ trợ của phần cứng MMU** (Memory Management Unit) với các thanh ghi Base/Limit hoặc Paging/Segmentation.
   - Ưu điểm: Cho phép tiến trình di chuyển linh hoạt trong RAM khi đang chạy (áp dụng trong hầu hết HĐH hiện đại).

### b. Dynamic Loading & Dynamic Linking:
- **Dynamic Loading (Nạp động):** 
  - Một hàm/thủ tục **chỉ được nạp vào RAM khi nó thực sự được gọi đến**.
  - Tối ưu cho các đoạn mã lớn nhưng hiếm khi dùng (ví dụ: hàm xử lý lỗi hệ thống).
- **Dynamic Linking (Liên kết động):** 
  - Việc liên kết thư viện ngoài được hoãn lại đến lúc thực thi.
  - Sử dụng một đoạn mã sơ khai (**stub**) để tham chiếu đến thư viện (`.DLL` trên Windows, `.so` trên Linux).
  - **Ưu điểm vượt trội:** Tiết kiệm RAM và đĩa cứng nhờ cơ chế **chia sẻ mã (Code Sharing)** giữa nhiều tiến trình.

---

## 2.3. Các mô hình phân vùng bộ nhớ & Phân mảnh

### a. Phân mảnh bộ nhớ (Fragmentation):
- **Phân mảnh nội (Internal Fragmentation):** 
  - Vùng nhớ được cấp phát **lớn hơn vùng nhớ tiến trình yêu cầu**. Phần dư thừa nằm bên trong vùng cấp phát nhưng không dùng đến và tiến trình khác không thể xài.
  - *Thường gặp:* Phân chia cố định (Fixed Partitioning), Phân trang (Paging - ở khung trang cuối).
- **Phân mảnh ngoại (External Fragmentation):** 
  - Tổng dung lượng bộ nhớ trống còn lại đủ để thỏa mãn yêu cầu, nhưng các vùng trống **bị phân tán rời rạc, không liên tục**, nên không cấp phát được.
  - *Thường gặp:* Phân chia động (Dynamic Partitioning), Phân đoạn (Segmentation).
  - *Giải pháp:* **Kết khối (Compaction)** - gom tất cả vùng nhớ trống về một phía (chỉ thực hiện được khi binding ở execution time).

### b. Bốn chiến lược cấp phát phân vùng động (Placement Strategies):
1. **First-fit (Phù hợp đầu tiên):** Cấp phát khối trống đầu tiên đủ lớn (tính từ đầu bộ nhớ). **Nhanh nhất**.
2. **Best-fit (Phù hợp nhất):** Cấp phát khối trống nhỏ nhất trong số các khối đủ lớn. **Tạo ra các mảnh vụn nhỏ nhất**.
3. **Worst-fit (Kém phù hợp nhất):** Cấp phát khối trống lớn nhất. **Để lại phần dư lớn nhất** (dễ tận dụng lại).
4. **Next-fit (Phù hợp kế tiếp):** Giống First-fit nhưng bắt đầu tìm từ vị trí vừa cấp phát trước đó.

---

## 2.4. Cơ chế Phân trang (Paging) & Bảng trang
- **Nguyên lý:**
  - Bộ nhớ vật lý được chia thành các khối kích thước cố định bằng nhau gọi là **Khung trang (Frames)**.
  - Bộ nhớ luận lý được chia thành các khối có kích thước bằng khung trang gọi là **Trang (Pages)**.
  - Kích thước trang luôn là lũy thừa của 2 ($2^n$, thường từ 4KB đến vài MB).
  - Cho phép cấp phát **không liên tục**. **Loại bỏ hoàn toàn phân mảnh ngoại**, chỉ còn phân mảnh nội ở trang cuối cùng.
- **Chuyển đổi địa chỉ trong Paging:**
  - Địa chỉ ảo gồm 2 phần: **Page number ($p$)** và **Offset ($d$)**.
  - Nếu không gian địa chỉ ảo là $2^m$ và kích thước trang là $2^n$:
    - Số bit dành cho **Offset ($d$) = $n$ bits**.
    - Số bit dành cho **Page number ($p$) = $m - n$ bits**.
    - Số mục trong bảng trang = $2^{m-n}$.
  - Cơ chế ánh xạ: CPU đưa $p$ vào Bảng trang (**Page Table**) tra ra frame number $f$. Địa chỉ vật lý tương ứng là ghép $(f, d)$.
- **Cài đặt Bảng trang:**
  - Bảng trang được lưu trong RAM.
  - **PTBR (Page-Table Base Register):** Thanh ghi trỏ vào địa chỉ bắt đầu của bảng trang.
  - **PTLR (Page-Table Length Register):** Thanh ghi lưu độ dài bảng trang (dùng để kiểm tra hợp lệ).
  - *Nhược điểm:* Mỗi thao tác truy xuất dữ liệu/lệnh tốn **2 lần truy xuất bộ nhớ** (Lần 1 tra bảng trang, Lần 2 đọc dữ liệu).

---

## 2.5. TLB & Thời gian truy xuất hiệu dụng (EAT)
- **TLB (Translation Lookaside Buffer):**
  - Là bộ nhớ đệm phần cứng liên kết nhanh (**Associative Cache**) chuyên dùng để lưu các mục bảng trang được truy cập gần đây.
- **Effective Access Time (EAT - Thời gian truy xuất hiệu dụng):**
  - Gọi $\alpha$ là tỉ lệ tìm thấy trong TLB (**Hit ratio**).
  - Gọi $\epsilon$ là thời gian tra cứu TLB (**Lookup time**).
  - Gọi $x$ là chu kỳ truy xuất bộ nhớ chính (**Memory access time**).
  - Nếu TLB Hit: Thời gian truy xuất $= \epsilon + x$.
  - Nếu TLB Miss: Thời gian truy xuất $= \epsilon + x + x = \epsilon + 2x$.
  - **CÔNG THỨC EAT:**
    $$\mathbf{EAT = (\epsilon + x)\alpha + (\epsilon + 2x)(1 - \alpha) = (2 - \alpha)x + \epsilon}$$
    *(Nếu đề bài giả định thời gian tra TLB $\epsilon \approx 0$, công thức trở thành: $\mathbf{EAT = (2 - \alpha)x}$).*

---

## 2.6. Bảo vệ bộ nhớ & Cơ chế Hoán vị (Swapping)
- **Bảo vệ trong Paging:**
  - **Protection bits:** Quy định quyền (Read-only, Read-Write, Execute).
  - **Valid/Invalid bit:**
    - `valid`: Trang thuộc không gian địa chỉ hợp lệ của tiến trình và đang nằm trong RAM.
    - `invalid`: Trang không hợp lệ (không thuộc tiến trình) hoặc chưa được nạp vào RAM.
- **Hoán vị (Swapping):**
  - Tạm thời đưa một tiến trình ra khỏi RAM và lưu vào bộ nhớ phụ (**Swap space / Backing store**), sau đó nạp lại khi có điều kiện.
  - Chính sách: **Round-robin** (hết quantum thì swap out), **Roll out/Roll in** (theo độ ưu tiên: tiến trình thấp bị đẩy ra nhường chỗ cho tiến trình ưu tiên cao).

---

# PHẦN 3: CHƯƠNG 8 - BỘ NHỚ ẢO

## 3.1. Tổng quan Bộ nhớ ảo & Demand Paging
- **Khái niệm Bộ nhớ ảo (Virtual Memory):**
  - Kỹ thuật cho phép thực thi một tiến trình mà **không cần nạp toàn bộ tiến trình vào bộ nhớ vật lý**.
  - Cho phép không gian địa chỉ luận lý **lớn hơn rất nhiều** dung lượng RAM vật lý.
  - Tách rời hoàn toàn góc nhìn bộ nhớ của lập trình viên khỏi cấu trúc vật lý.
- **Ưu điểm:**
  - Tăng đáng kể mức độ đa chương (**Degree of Multiprogramming**).
  - Tiết kiệm I/O nạp chương trình (chỉ nạp phần cần dùng).
  - Dễ dàng chia sẻ bộ nhớ giữa các tiến trình qua cơ chế Copy-on-Write.
- **Demand Paging (Phân trang theo yêu cầu):**
  - Cơ chế: **Chỉ nạp một trang vào bộ nhớ chính khi trang đó được CPU tham chiếu đến**.
  - Dùng cờ `valid/invalid bit` trong bảng phân trang để nhận biết trang đã có trong RAM hay chưa.

---

## 3.2. Quy trình xử lý lỗi trang (PFSR)
- **Page Fault (Lỗi trang):** Xảy ra khi CPU truy xuất một địa chỉ thuộc về một trang có cờ là `invalid` (chưa có trong RAM).
- **Quy trình 6 bước của Page-Fault Service Routine (PFSR):**
  1. Phần cứng MMU phát hiện trang `invalid` $\rightarrow$ Phát sinh ngắt ngoại lệ (**Page-fault trap**) chuyển quyền điều khiển cho OS.
  2. OS lưu trạng thái của tiến trình, đưa tiến trình vào trạng thái **Blocked (Chờ I/O)**.
  3. OS kiểm tra tính hợp lệ của địa chỉ tham chiếu:
     - Nếu địa chỉ bất hợp pháp $\rightarrow$ Báo lỗi Segmentation fault, hủy tiến trình.
     - Nếu hợp pháp $\rightarrow$ Tiến hành nạp trang từ đĩa (backing store).
  4. OS tìm một **khung trang (frame) trống**:
     - Nếu có frame trống: Sử dụng frame đó.
     - Nếu không có: Dùng **thuật toán thay trang** để chọn một **trang nạn nhân (victim page)**. Nếu trang nạn nhân bị sửa đổi (dirty bit = 1), phải ghi nó về đĩa trước khi ghi đè.
  5. Đọc nội dung trang cần thiết từ đĩa vào khung trang đã chọn.
  6. Sau khi I/O hoàn tất: Cập nhật lại Bảng trang (gán frame number, sửa cờ thành `valid`) và đưa tiến trình về trạng thái **Ready** để tiếp tục thực thi lại chỉ thị gây lỗi trang.

---

## 3.3. Các thuật toán thay thế trang

### 1. Thuật toán FIFO (First-In-First-Out)
- **Nguyên lý:** Thay thế trang được nạp vào RAM sớm nhất.
- **Ưu điểm:** Đơn giản, dễ cài đặt bằng hàng đợi queue.
- **Nghịch lý Belady (Belady's Anomaly):** 
  - Là hiện tượng **tỷ lệ lỗi trang tăng lên khi số lượng khung trang được cấp tăng lên**.
  - *Chuỗi kinh điển chứng minh Belady:* `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`. Với 3 frames có 9 lỗi trang, nhưng với 4 frames lại tăng lên 10 lỗi trang!

### 2. Thuật toán OPT (Optimal - Tối ưu)
- **Nguyên lý:** Thay thế trang **sẽ lâu được sử dụng nhất trong tương lai**.
- **Đặc điểm:**
  - Cho số lượng lỗi trang **thấp nhất tuyệt đối**.
  - **Không bao giờ bị nghịch lý Belady**.
  - **Không thể hiện thực trong thực tế** vì HĐH không thể biết trước chuỗi truy xuất tương lai.
  - Mục đích: Dùng làm **chuẩn đối sánh (benchmark)** để đánh giá các thuật toán khác.

### 3. Thuật toán LRU (Least Recently Used)
- **Nguyên lý:** Thay thế trang **đã lâu nhất chưa được truy cập trong quá khứ** (xấp xỉ thuật toán OPT dựa trên nguyên lý cục bộ).
- **Đặc điểm:**
  - Hiệu quả cao, số lỗi trang xấp xỉ OPT.
  - **Không bị nghịch lý Belady** (thuộc nhóm giải thuật stack).
  - Chi phí cài đặt cao: Cần hỗ trợ phần cứng (bộ đếm Counter hoặc ngăn xếp Stack) để cập nhật mốc thời gian mỗi lần truy cập.

---

## 3.4. Cấp phát khung trang (Frame Allocation)
- **Cấp phát tĩnh (Fixed Allocation):**
  - **Cấp phát đều (Equal Allocation):** Chia đều số frame cho các tiến trình: $m / n$.
  - **Cấp phát theo tỉ lệ (Proportional Allocation):** Chia dựa theo kích thước tiến trình $s_i$:
    $$a_i = \frac{s_i}{\sum s_i} \times m$$
    *(với $m$ là tổng số frames khả dụng, $s_i$ là kích thước tiến trình $P_i$)*.
  - **Cấp phát theo độ ưu tiên:** Ưu tiên cấp nhiều frame hơn cho tiến trình có mức ưu tiên cao.
- **Cấp phát động (Variable Allocation):** Điều chỉnh số frame trong quá trình chạy dựa trên tần suất lỗi trang (**Page-fault frequency**).

---

## 3.5. Hiện tượng Trì trệ (Thrashing) & Mô hình Working-Set

### a. Thrashing (Hiện tượng trì trệ / đảo trang liên tục):
- **Định nghĩa:** Là hiện tượng một tiến trình **dành nhiều thời gian cho việc hoán chuyển trang (swap in/out) hơn là thực thi lệnh**.
- **Hậu quả:** Tỉ lệ lỗi trang tăng vọt, **hiệu suất sử dụng CPU (CPU Utilization) sụt giảm nghiêm trọng**. HĐH thấy CPU rảnh tưởng thiếu tiến trình nên lại nạp thêm tiến trình mới, khiến hệ thống tê liệt hoàn toàn.
- **Nguyên nhân cốt lõi:** Một tiến trình không được cấp đủ số frame tối thiểu để chứa toàn bộ **tập cục bộ (locality)** của nó:
  $$\sum \text{Kích thước Locality} > \text{Dung lượng RAM}$$

### b. Mô hình Working-Set (Working-Set Model):
- Dựa trên **nguyên lý cục bộ (Locality Principle)**.
- Định nghĩa tham số cửa sổ thời gian trượt $\Delta$ (**Working-set window**).
- Tập $WS_i(t)$ là tập hợp các trang được tham chiếu trong $\Delta$ lần truy xuất gần nhất của tiến trình $P_i$.
- Kích thước working set: $WSS_i = |WS_i|$.
- Gọi $D = \sum WSS_i$ là tổng nhu cầu khung trang của toàn bộ hệ thống:
  - Nếu **$D \le m$**: Đủ bộ nhớ, hệ thống vận hành ổn định.
  - Nếu **$D > m$**: Thiếu bộ nhớ, **nguy cơ xảy ra Thrashing**.
  - **Giải pháp của HĐH:** Tạm dừng (**suspend**) một số tiến trình, swap toàn bộ trang của tiến trình đó ra đĩa để nhường frame cho các tiến trình còn lại đạt đủ $WSS$.

---

# PHẦN 4: CÁC DẠNG BÀI TẬP TÍNH TOÁN ĐI THI

## Dạng 1: Cấp phát bộ nhớ động
**Đề bài mẫu:** Bộ nhớ có các phân vùng trống theo thứ tự: `600K, 500K, 200K, 300K`. Cần cấp phát cho các tiến trình theo thứ tự: `P1 (212K), P2 (417K), P3 (112K), P4 (426K)`. Xác định kết quả cấp phát theo First-fit, Best-fit, Worst-fit, Next-fit.

### Hướng dẫn giải chi tiết:
1. **First-fit (Khối đầu tiên đủ lớn):**
   - $P_1 (212K) \rightarrow$ Phân vùng 600K (còn dư 388K).
   - $P_2 (417K) \rightarrow$ Phân vùng 500K (còn dư 83K).
   - $P_3 (112K) \rightarrow$ Phân vùng 600K đang dư 388K (còn dư 276K).
   - $P_4 (426K) \rightarrow$ Không có phân vùng nào đủ lớn (kể cả 200K, 300K, hay các phần dư 276K, 83K) $\rightarrow$ **P4 phải chờ**.
2. **Best-fit (Khối nhỏ nhất đủ lớn):**
   - $P_1 (212K) \rightarrow$ Chọn phân vùng 300K (dư 88K).
   - $P_2 (417K) \rightarrow$ Chọn phân vùng 500K (dư 83K).
   - $P_3 (112K) \rightarrow$ Chọn phân vùng 200K (dư 88K).
   - $P_4 (426K) \rightarrow$ Chọn phân vùng 600K (dư 174K).
   - $\rightarrow$ **Tất cả các tiến trình đều được cấp phát thành công!**
3. **Worst-fit (Khối lớn nhất):**
   - $P_1 (212K) \rightarrow$ Chọn 600K (dư 388K).
   - $P_2 (417K) \rightarrow$ Chọn 500K (dư 83K).
   - $P_3 (112K) \rightarrow$ Các khối hiện có: 388K, 83K, 200K, 300K $\rightarrow$ Chọn lớn nhất là 388K (dư 276K).
   - $P_4 (426K) \rightarrow$ Các khối còn lại: 276K, 83K, 200K, 300K $\rightarrow$ Không khối nào $\ge 426K \rightarrow$ **P4 phải chờ**.
4. **Next-fit (Tiếp tục từ vị trí trước):**
   - $P_1 (212K) \rightarrow$ Khối 600K (dư 388K). Con trỏ ở khối 600K.
   - $P_2 (417K) \rightarrow$ Xét tiếp khối 500K $\rightarrow$ Cấp vào 500K (dư 83K). Con trỏ ở 500K.
   - $P_3 (112K) \rightarrow$ Xét tiếp khối 200K $\rightarrow$ Cấp vào 200K (dư 88K). Con trỏ ở 200K.
   - $P_4 (426K) \rightarrow$ Xét tiếp khối 300K (không đủ), quay vòng lại 600K (còn dư 388K - không đủ), 500K (83K - không đủ) $\rightarrow$ **P4 phải chờ**.

> **Kết luận:** Trong trường hợp này, **Best-fit là thuật toán hiệu quả nhất** vì cấp phát được cho cả 4 tiến trình.

---

## Dạng 2: Phân trang & Tính toán số bit địa chỉ

### Bài tập 2.1:
**Đề bài:** Xét không gian địa chỉ có 12 trang, mỗi trang kích thước 2KB, ánh xạ vào bộ nhớ vật lý có 32 khung trang.
a. Địa chỉ logic gồm bao nhiêu bit?  
b. Địa chỉ physical gồm bao nhiêu bit?

**Lời giải:**
- Kích thước trang = $2\text{KB} = 2 \times 2^{10} = 2^{11}\text{ bytes} \Rightarrow$ Số bit offset **$d = 11\text{ bits}$**.
- **Câu a:** 
  - Không gian địa chỉ có 12 trang. Để định danh được 12 trang ($0 \dots 11$), cần: $\lceil \log_2(12) \rceil = 4\text{ bits}$ cho số hiệu trang $p$.
  - $\Rightarrow$ Địa chỉ logic gồm: $p + d = 4 + 11 =$ **15 bits** (Tổng kích thước không gian địa chỉ là $12 \times 2\text{KB} = 24\text{KB}$).
- **Câu b:**
  - Bộ nhớ vật lý có 32 khung trang $= 2^5$ khung trang $\Rightarrow$ Cần **$5\text{ bits}$** cho số hiệu khung trang $f$.
  - $\Rightarrow$ Địa chỉ vật lý gồm: $f + d = 5 + 11 =$ **16 bits** (Tổng bộ nhớ vật lý là $32 \times 2\text{KB} = 64\text{KB} = 2^{16}\text{ bytes}$).

### Bài tập 2.2:
**Đề bài:** Máy tính 32-bit địa chỉ ảo, bảng trang 2 cấp: 9 bit cấp 1, 11 bit cấp 2, phần còn lại cho offset. Xác định:
a. Kích thước của 1 trang nhớ?  
b. Không gian địa chỉ ảo có bao nhiêu trang?

**Lời giải:**
- Tổng số bit địa chỉ ảo = 32 bits.
- Số bit offset $d = 32 - (9 + 11) = 32 - 20 = 12\text{ bits}$.
- **Câu a:** Kích thước trang $= 2^{12}\text{ bytes} = 4096\text{ bytes} =$ **4 KB**.
- **Câu b:** Số trang trong không gian địa chỉ ảo $= 2^{32 - 12} = 2^{20} =$ **1,048,576 trang (1M trang)**.

---

## Dạng 3: Tính thời gian truy xuất hiệu dụng (EAT)
**Đề bài:** Hệ thống phân trang có thời gian truy xuất RAM bình thường $x = 200\text{ ns}$.
a. Nếu bảng trang lưu trong RAM và không có TLB, thời gian truy xuất là bao nhiêu?  
b. Nếu có TLB với hit-ratio $\alpha = 75\%$, thời gian tìm trong TLB xem như $\epsilon = 0\text{ ns}$, tính EAT?  
c. Nếu thời gian tìm trong TLB là $\epsilon = 20\text{ ns}$ và hit-ratio $\alpha = 90\%$, tính EAT?

**Lời giải:**
- **Câu a:** Khi không có TLB, mỗi truy xuất cần 2 lần vào RAM (1 lần đọc bảng trang + 1 lần đọc dữ liệu):
  $$\text{Thời gian} = 2 \times x = 2 \times 200\text{ ns} = \mathbf{400\text{ ns}}$$
- **Câu b:** Áp dụng công thức với $\epsilon = 0$:
  $$\text{EAT} = (\alpha) \times x + (1 - \alpha) \times 2x = (0.75 \times 200) + (0.25 \times 400) = 150 + 100 = \mathbf{250\text{ ns}}$$
- **Câu c:** Áp dụng công thức tổng quát với $\epsilon = 20\text{ ns}$, $\alpha = 0.9$:
  $$\text{EAT} = (\epsilon + x)\alpha + (\epsilon + 2x)(1 - \alpha) = (20 + 200) \times 0.9 + (20 + 400) \times 0.1 = (220 \times 0.9) + (420 \times 0.1) = 198 + 42 = \mathbf{240\text{ ns}}$$

---

## Dạng 4: Mô phỏng thuật toán thay trang (FIFO, OPT, LRU)
**Đề bài:** Xét chuỗi tham chiếu bộ nhớ gồm 17 phần tử sau:  
`1, 2, 3, 4, 2, 1, 5, 6, 2, 1, 2, 3, 7, 6, 3, 2, 1`  
Giả sử hệ thống có **4 khung trang (ban đầu trống)**. Đếm số lỗi trang (Page Fault - PF) đối với:
a. FIFO  
b. OPT  
c. LRU

### Lời giải chi tiết:

#### a. Thuật toán FIFO:
Thay trang nạp vào sớm nhất (hàng đợi FIFO).
| Tham chiếu | 1 | 2 | 3 | 4 | 2 | 1 | 5 | 6 | 2 | 1 | 2 | 3 | 7 | 6 | 3 | 2 | 1 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Khung 1** | 1 | 1 | 1 | 1 | 1 | 1 | **5** | 5 | 5 | 5 | 5 | **3** | 3 | 3 | 3 | 3 | **1** |
| **Khung 2** | | 2 | 2 | 2 | 2 | 2 | 2 | **6** | 6 | 6 | 6 | 6 | **7** | 7 | 7 | 7 | 7 |
| **Khung 3** | | | 3 | 3 | 3 | 3 | 3 | 3 | **2** | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| **Khung 4** | | | | 4 | 4 | 4 | 4 | 4 | 4 | **1** | 1 | 1 | 1 | **6** | 6 | 6 | 6 |
| **Lỗi trang?**| **x**| **x**| **x**| **x**| | | **x**| **x**| **x**| **x**| | **x**| **x**| **x**| | | **x**|
- **Tổng số lỗi trang (FIFO): 11 lỗi trang.**

#### b. Thuật toán OPT:
Thay trang sẽ được tham chiếu trễ nhất trong tương lai.
| Tham chiếu | 1 | 2 | 3 | 4 | 2 | 1 | 5 | 6 | 2 | 1 | 2 | 3 | 7 | 6 | 3 | 2 | 1 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Khung 1** | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | **7** | 7 | 7 | 7 | **1** |
| **Khung 2** | | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| **Khung 3** | | | 3 | 3 | 3 | 3 | **5** | **6** | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 |
| **Khung 4** | | | | 4 | 4 | 4 | 4 | 4 | 4 | 4 | 4 | **3** | 3 | 3 | 3 | 3 | 3 |
| **Lỗi trang?**| **x**| **x**| **x**| **x**| | | **x**| **x**| | | | **x**| **x**| | | | **x**|
*(Tại bước 5: bộ đệm {1,2,3,4}, trang 5 vào thay thế 3 vì 3 mãi sau mới dùng. Tại bước 6: trang 6 vào thay thế 5).*  
- **Tổng số lỗi trang (OPT): 9 lỗi trang.**

#### c. Thuật toán LRU:
Thay trang có thời điểm sử dụng gần đây nhất là xa nhất (lâu nhất chưa xài).
| Tham chiếu | 1 | 2 | 3 | 4 | 2 | 1 | 5 | 6 | 2 | 1 | 2 | 3 | 7 | 6 | 3 | 2 | 1 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Khung 1** | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | **7** | 7 | 7 | 7 | **1** |
| **Khung 2** | | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| **Khung 3** | | | 3 | 3 | 3 | 3 | **5** | 5 | 5 | **5**$\rightarrow$**x** | | **3** | 3 | 3 | 3 | 3 | 3 |
| **Khung 4** | | | | 4 | 4 | 4 | 4 | **6** | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 |
| **Lỗi trang?**| **x**| **x**| **x**| **x**| | | **x**| **x**| | | | **x**| **x**| | | | **x**|
*(Tại bước 5: bộ nhớ có {1,2,3,4}, trang 3 lâu nhất chưa dùng $\rightarrow$ thay bằng 5. Tại bước 6: có {1,2,5,4}, trang 4 lâu nhất $\rightarrow$ thay bằng 6. Đến khi gặp 3: thay thế trang 5).*  
- **Tổng số lỗi trang (LRU): 10 lỗi trang.**

---

## Dạng 5: Cấp phát khung trang theo tỉ lệ
**Đề bài:** Hệ thống có tổng cộng $m = 64$ khung trang khả dụng. Có 2 tiến trình:
- $P_1$ có kích thước $s_1 = 10$ trang.
- $P_2$ có kích thước $s_2 = 127$ trang.
Tính số khung trang cấp phát cho mỗi tiến trình theo thuật toán Proportional Allocation.

**Lời giải:**
- Tổng kích thước các tiến trình: $S = s_1 + s_2 = 10 + 127 = 137$ trang.
- Số frame cấp cho $P_1$:
  $$a_1 = \frac{s_1}{S} \times m = \frac{10}{137} \times 64 \approx 4.67 \Rightarrow \mathbf{5\text{ frames}}$$
- Số frame cấp cho $P_2$:
  $$a_2 = \frac{s_2}{S} \times m = \frac{127}{137} \times 64 \approx 59.33 \Rightarrow \mathbf{59\text{ frames}}$$

---

# PHẦN 5: BẢNG SO SÁNH TỔNG HỢP & PHẢN XẠ NHANH

### 1. Bảng so sánh các kỹ thuật và khái niệm cốt lõi
| Cặp so sánh | Tiêu chí phân biệt mấu chốt |
| :--- | :--- |
| **Phân mảnh nội vs. Phân mảnh ngoại** | - **Nội:** Vùng nhớ dư thừa **nằm bên trong** khối đã cấp phát (Paging, Fixed Partitioning).<br>- **Ngoại:** Vùng nhớ trống **nằm rải rác bên ngoài**, tổng dung lượng đủ nhưng không liên tục (Dynamic Partitioning, Segmentation). |
| **Dynamic Loading vs. Dynamic Linking** | - **Loading:** Tự nạp hàm vào RAM khi được gọi (tiết kiệm bộ nhớ cho code ít dùng).<br>- **Linking:** Dùng Stub liên kết thư viện chia sẻ lúc chạy, **nhiều process dùng chung một bản copy trên RAM**. |
| **Binary Semaphore vs. Mutex Lock** | - Mutex có khái niệm **Ownership** (tiến trình nào lock thì chính nó phải unlock).<br>- Semaphore không có ownership (tiến trình này gọi `wait()`, tiến trình khác có thể gọi `signal()`). |
| **FIFO vs. OPT vs. LRU** | - **FIFO:** Dễ cài đặt nhất, hiệu suất trung bình, **bị Nghịch lý Belady**.<br>- **OPT:** Hiệu suất cao nhất lý thuyết, không bị Belady, **không hiện thực được**.<br>- **LRU:** Hiệu suất tiệm cận OPT, không bị Belady, **cần phần cứng hỗ trợ**. |
| **Paging vs. Swapping** | - **Paging:** Di chuyển từng **trang nhớ đơn lẻ** giữa RAM và đĩa.<br>- **Swapping:** Di chuyển **toàn bộ tiến trình** ra/vào bộ nhớ phụ. |

### 2. Bộ câu hỏi phản xạ nhanh lý thuyết đi thi
1. **Câu hỏi:** Tại sao tắt ngắt (disable interrupt) không giải quyết được bài toán vùng tranh chấp trên hệ thống đa CPU?
   - **Trả lời:** Vì lệnh tắt ngắt chỉ có hiệu lực trên CPU đang thực thi lệnh đó. Các CPU khác vẫn tiếp tục chạy và truy cập vào bộ nhớ chung bình thường.
2. **Câu hỏi:** Giải thuật Peterson giải quyết được cho bao nhiêu tiến trình?
   - **Trả lời:** Được thiết kế chuẩn cho **2 tiến trình**.
3. **Câu hỏi:** Giá trị của biến Semaphore $S$ nói lên điều gì khi $S = -3$?
   - **Trả lời:** Có **3 tiến trình đang bị block** và nằm chờ trong hàng đợi của Semaphore đó.
4. **Câu hỏi:** Nghịch lý Belady là gì? Xảy ra ở giải thuật thay trang nào?
   - **Trả lời:** Là hiện tượng số lỗi trang tăng lên khi số khung trang cấp phát tăng lên. Xảy ra điển hình ở thuật toán **FIFO**.
5. **Câu hỏi:** Tại sao thuật toán OPT không thể cài đặt thực tế trong HĐH?
   - **Trả lời:** Vì HĐH không thể biết trước được chuỗi tham chiếu trang trong tương lai của tiến trình.
6. **Câu hỏi:** Thrashing xảy ra khi nào và cách HĐH khắc phục bằng mô hình Working-Set?
   - **Trả lời:** Xảy ra khi tổng kích thước locality vượt quá dung lượng RAM thực ($D > m$). HĐH khắc phục bằng cách tạm dừng (suspend) một số tiến trình để thu hồi frame, đảm bảo các tiến trình còn lại có đủ số frame bằng kích thước working set của chúng.
