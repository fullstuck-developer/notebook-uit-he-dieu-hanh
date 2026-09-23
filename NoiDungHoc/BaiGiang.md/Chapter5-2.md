# HỆ ĐIỀU HÀNH
## CHƯƠNG 5: ĐỒNG BỘ TIẾN TRÌNH (PHẦN 2)

*Bên cạnh việc sử dụng các ngắt hoặc cần hỗ trợ từ phần cứng, hệ điều hành cũng cung cấp các cơ chế giúp thực thi việc đồng bộ các tiến trình/tiểu trình. Phần 2 của chương đồng bộ tiến trình sẽ giới thiệu các kỹ thuật sử dụng semaphore, mutex, monitor vốn là các kỹ thuật rất phổ biến trong đồng bộ tiến trình.*

### MỤC TIÊU
1. Diễn tả được cơ chế hoạt động của mutex lock, semaphore và monitor trong việc giải quyết bài toán vùng tranh chấp.
2. Phân tích chương trình ứng dụng mutex lock và semaphore.
3. Viết được chương trình sử dụng mutex lock và semaphore để thực hiện đồng bộ thứ tự hoạt động của tiến trình/tiểu trình.
4. Trình bày được vấn đề Liveness trong hoạt động đồng bộ tiến trình.

### NỘI DUNG
6. Mutex locks
7. Semaphore
8. Monitor
9. Liveness

---

### Review: GIẢI QUYẾT BÀI TOÁN VÙNG TRANH CHẤP
* Các giải pháp dựa trên ngắt **gây lãng phí** tài nguyên CPU khi liên tục kiểm tra điều kiện chờ đợi tiến vào CS (busy waiting).
* Các giải pháp hỗ trợ từ phần cứng thì **khá phức tạp** và **không thể truy cập** được bởi lập trình viên của các chương trình ứng dụng.

=> Các nhà thiết kế hệ điều hành xây dựng các **công cụ phần mềm cấp cao** để giải quyết vấn đề vùng tranh chấp:
* Mutex locks
* Semaphore
* Monitor

---

## 6. MUTEX LOCKS

### 5.6.1. Định nghĩa mutex locks
Mutex (viết tắt của **Mut**ual **ex**clusion) locks hay còn gọi là “khóa mutex” là kỹ thuật giúp đảm bảo yêu cầu loại trừ tương hỗ khi các tiến trình thực thi đồng thời với nhau. Mutex locks hoạt động như một ổ khóa, khi tiến trình tiến vào vùng tranh chấp thì cần phải yêu cầu khóa ổ khóa lại, tương tự, sau khi ra khỏi vùng tranh chấp thì tiến trình cần yêu cầu mở khóa mutex để tiến trình khác có thể tiến vào.

**Mô hình hoạt động:**
```c
while (true) {
    acquire lock
        critical section
    release lock
        remainder section
}
```

```c
acquire() {
    while (!available);
        /* busy wait */
    available = false;
} 

release() {
    available = true;
} 
```
Thao tác gọi `acquire()` hoặc `release()` phải được thực hiện **đơn nguyên** -> Có thể được hiện thực thông qua lệnh phần cứng đơn nguyên như `compare_and_swap`.

**Yêu cầu busy waiting -> Lãng phí CPU**
Giải pháp này còn được gọi là **spinlock**.

### 5.6.2. Mutex locks không busy waiting
Để tránh busy waiting trong việc sử dụng khóa mutex, hệ điều hành cung cấp cơ chế cho phép khóa tiến trình – hay chủ động đưa tiến trình vào trạng thái ngủ, song song đó là cơ chế đánh thức tiến trình để đưa tiến trình vào trạng thái hoạt động trở lại. 

* Để tránh busy waiting trong mutex locks, ta tạm thời đặt tiến trình vào trạng thái ngủ khi khóa bị khóa, và sau đó đánh thức tiến trình dậy khi khóa được mở.
* Hệ điều hành cần cung cấp 2 thao tác:
    * `block`: tạm dừng và đặt tiến trình gọi thao tác này vào trong hàng đợi – **trạng thái ngủ**.
    * `wakeup`: xóa một tiến trình ra khỏi hàng đợi và đặt lại vào trong hàng đợi sẵn sàng – **đánh thức**.

**So sánh 2 phương pháp:**

*Busy waiting:*
```c
acquire() {
    while (!available);
        /* busy wait */
    available = false;
} 

release() {
    available = true;
} 
```

*Không busy waiting:*
```c
acquire() {
    if (!available)
        block();
    available = false;
} 

release() {
    available = true;
    wakeup(Q);
} 
```

**Tiến trình P muốn vào CS:**
* Nếu khóa mutex **đang mở**: tiến trình khóa lại và tiến vào CS.
* Nếu khóa mutex **đang khóa**: tiến trình P bị block và vào trạng thái ngủ.

**Tiến trình P sau khi hoàn thành CS:**
* Mở khóa mutex.
* Đánh thức tiến trình Q (nếu có) đang ngủ trong hàng chờ.

### 5.6.3. Cách sử dụng mutex locks
Lưu ý:
* Mutex lock thường sẽ được khai báo toàn cục và được khởi tạo trong hàm `main`.
* Cần phải xác định đúng vùng tranh chấp trước khi thực hiện các thao tác trên khóa mutex (`acquire` và `release`).

Quy trình:
1. Khai báo và khởi tạo mutex.
2. Tiến trình/Tiểu trình yêu cầu khóa Mutex – thông qua `acquire()` – để thực hiện CS.
3. Tại 1 thời điểm chỉ có 1 tiến trình/tiểu trình thành công khóa mutex và thực hiện CS.
4. Tiến trình/Tiểu trình mở khóa Mutex – thông qua `release()` – sau khi rời CS.
5. Hủy khóa mutex sau khi không sử dụng nữa.

---

## 7. SEMAPHORE

### 5.7.1. Định nghĩa semaphore
Bên cạnh mutex locks, semaphore cũng là một trong những công cụ đồng bộ phổ biến được nhiều hệ điều hành cung cấp. Với semaphore, lập trình viên có thể ứng dụng trong nhiều trường hợp khác nhau bao gồm đồng bộ thứ tự thực thi của các tiến trình/tiểu trình, thứ tự thực thi của các thao tác nằm trên nhiều tiến trình/tiểu trình khác nhau, và đảm bảo loại trừ tương hỗ.

* Semaphore là công cụ đồng bộ cung cấp các cách sử dụng linh hoạt (hơn khóa Mutex) để các tiến trình có thể đồng bộ các hoạt động/hành vi của mình.
* Semaphore **S** về bản chất là một **biến số nguyên**.
* Chỉ có thể được truy cập thông qua 2 thao tác: `wait()` và `signal()` – hay còn được gọi là `P()` và `V()`.

| Định nghĩa thao tác wait() | Định nghĩa thao tác signal() |
| --- | --- |
| ```c wait(S) { while (S <= 0) ; // busy wait S--; } ``` | ```c signal(S) { S++; } ``` |
| * Nếu semaphore S **không dương** thì tiến trình/tiểu trình phải chờ. <br> * Khi tiến trình được thực hiện CS thì **trừ semaphore đi 1**. <br> * Được sử dụng khi muốn **sử dụng tài nguyên**. | * **Tăng** giá trị của semaphore lên 1. <br> * Được sử dụng khi **trả lại tài nguyên**. |

**Ví dụ trên nhà hàng:**
* Restaurant có 15 bàn trống.
* Khai báo semaphore `freeTable`
* Khởi tạo `freeTable = 15`
* Khách P1 đến: gọi `wait(freeTable);` `<dùng bữa>;` -> lúc này `freeTable = 14`.
* Khách P1 đi: gọi `signal(freeTable);` -> lúc này `freeTable = 15`.
* Có 10 khách đến thì: `freeTable = 5`.
* Có 15 khách đến thì: `freeTable = 0`.
* Khách P16 đến: gọi `wait(freeTable);` -> do `freeTable = 0` nên P16 bị chờ ở ngoài.
* Khách P15 đi: gọi `signal(freeTable);` -> lúc này `freeTable = 1`.
* Khách P16 được cấp tài nguyên và vào trong, `wait(freeTable);` -> lúc này `freeTable = 0` trở lại.

### 5.7.2. Phân loại semaphore
Semaphore được chia thành 2 loại gồm: counting semaphore và binary semaphore.

* **Counting semaphore:**
    * Giá trị là số nguyên **không giới hạn**
* **Binary semaphore:**
    * Giá trị là **0** hoặc **1**
    * Có tác dụng giống với **khóa mutex**

> *Lưu ý:* Có thể sử dụng counting semaphore như một binary semaphore.

### 5.7.3. Hiện thực semaphore
**Hiện thực semaphore với busy waiting:**
`int S; // semaphore là một số nguyên`
* Cần phải đảm bảo rằng không có 2 tiến trình nào cùng lúc thực hiện thao tác `wait()` và `signal()` của một semaphore.
* Việc hiện thực semaphore cũng là một bài toán vùng tranh chấp, hàm `wait()` và `signal()` cũng nằm trong vùng tranh chấp.
* Có thể thực hiện **busy waiting** ở trong vùng tranh chấp:
    * Đoạn code thực hiện ngắn.
    * Busy waiting sẽ ngắn nếu CS hiếm khi được thực thi.
* Tuy nhiên, một số chương trình có thể tốn nhiều thời gian trong CS -> đây không phải giải pháp tốt.

**Hiện thực semaphore không busy waiting:**
* Mỗi semaphore được gắn với một hàng đợi.
* Mỗi phần tử trong hàng chờ có 2 thành phần:
    * Giá trị số nguyên (giá trị của semaphore).
    * Con trỏ chỉ đến phần tử tiếp theo (danh sách liên kết đơn).
* Hệ điều hành cần cung cấp 2 thao tác:
    * `block`: tạm dừng và đặt tiến trình gọi thao tác này vào trong hàng đợi – **trạng thái ngủ**.
    * `wakeup`: xóa một tiến trình ra khỏi hàng đợi và đặt lại vào trong hàng đợi sẵn sàng – **đánh thức**.

```c
typedef struct { 
    int value; 
    struct process *list; 
} semaphore;
```

| Định nghĩa thao tác wait() | Định nghĩa thao tác signal() |
| --- | --- |
| ```c wait(semaphore *S) { S->value--; if (S->value < 0) { // add this process to S->list; block(); } } ``` | ```c signal(semaphore *S) { S->value++; if (S->value <= 0) { // remove a process P from S->list; wakeup(P); } } ``` |

**Ví dụ trên tiến trình:**
Khởi tạo semaphore `count = 1`
* Tiến trình A vào region: `count = 0`
* Tiến trình B vào region: `count = -1` -> B vào Queue.
* Tiến trình C vào region: `count = -2` -> C vào Queue. (Queue hiện có B, C).
* Tiến trình A rời region: `count = -1` -> Lấy B ra khỏi Queue và wakeup B. (Queue hiện còn C).
* Tiến trình B rời region: `count = 0` -> Lấy C ra khỏi Queue và wakeup C. (Queue hiện trống).
* Tiến trình C rời region: `count = 1`.

### 5.7.4. Ứng dụng semaphore

**1. Đảm bảo loại trừ tương hỗ**
* Semaphore hoạt động như một khóa mutex.
* Bao CS bằng thao tác `wait()` và `signal()`.
* Khởi tạo giá trị của semaphore là `1` -> Chỉ tiến trình nào gọi `wait()` trước thì mới được tiến vào CS.

```c
// Tiến trình P1
wait(sem1);
// CS
signal(sem1);

// Tiến trình P2
wait(sem1);
// CS
signal(sem1);
```

**2. Đảm bảo thứ tự thực thi**
Đồng bộ P1 và P2 sao cho S1 luôn luôn thực thi trước S2.
* Khởi tạo semaphore `synch = 0`.
* Phân tích thứ tự thực thi:
    * Nếu S1 thực thi trước: `signal(synch)` làm `synch` lên 1, sau đó P2 gọi `wait(synch)` không sao.
    * Nếu S2 thực thi trước: gọi `wait(synch)` nhưng `synch` đang = 0 nên sẽ bị block, chờ đến khi S1 thực thi.

```c
// Tiến trình P1
S1;
signal(synch);

// Tiến trình P2
wait(synch);
S2;
```

**3. Đảm bảo điều kiện**
Đồng bộ tiến trình `Produce` và `Consume` sao cho `sells <= products`.
* Bước 1: Dựa vào điều kiện, xác định tài nguyên.
    * *Số đơn vị* mà `sells` được tăng <=> Số hàng còn lại trong kho (`stock`).
* Bước 2: Xác định số lượng semaphore.
    * Quản lý 1 tài nguyên -> Cần **1 semaphore** `stock`.
* Bước 3: Đặt `wait()` và `signal()`.
* Bước 4: Dựa vào trạng thái của hệ thống, xác định giá trị của semaphore.
    * `products = 0`, `sells = 0` -> Khởi tạo `stock = 0`.

```c
semaphore stock = 0;

// Tiến trình Produce
products++;
signal(stock);

// Tiến trình Consume
wait(stock);
sells++;
```

### 5.7.5. Một số nhận xét về semaphore
**Xét semaphore S:**
* Khi `S->value >= 0`: số lần mà các tiến trình/tiểu trình có thể thực thi `wait(S)` mà không bị blocked là `S->value`.
* Khi `S->value < 0`: số tiến trình/tiểu trình đang đợi trên S là `|S->value|`.

**Atomic và mutual exclusion:**
* Không được xảy ra trường hợp 2 tiến trình cùng đang ở trong thân lệnh `wait(S)` và `signal(S)` (cùng thao tác trên 1 semaphore S) tại một thời điểm (ngay cả với hệ thống multiprocessor).
* -> *Đoạn mã định nghĩa các lệnh `wait(S)` và `signal(S)` cũng chính là vùng tranh chấp.*
* Vùng tranh chấp của các tác vụ `wait(S)` và `signal(S)` thông thường rất nhỏ: khoảng 10 lệnh.
* Giải pháp cho vùng tranh chấp `wait(S)` và `signal(S)`:
    * **Uniprocessor:** có thể dùng cơ chế cấm ngắt (disable interrupt). Nhưng phương pháp này không làm việc trên hệ thống multiprocessor.
    * **Multiprocessor:** có thể dùng các giải pháp software (như giải thuật Dekker, Peterson) hoặc giải pháp hardware (TestAndSet, Swap).
* Vì CS rất nhỏ nên chi phí cho busy waiting sẽ rất thấp.

### 5.7.6. Các vấn đề khi sử dụng semaphore
Việc sử dụng semaphore yêu cầu tính cẩn thận và chính xác rất cao. Thứ tự của các lệnh `wait()` và `signal()` hay giá trị khởi tạo của semaphore có thể ảnh hưởng đến tính đúng đắn hay hiệu suất của chương trình. 

**Deadlock:**
Khởi tạo semaphore `S = 1`, `Q = 1`

```c
// Tiến trình P1
wait(S);
wait(Q);
...
signal(S);
signal(Q);

// Tiến trình P2
wait(Q);
wait(S);
...
signal(Q);
signal(S);
```
Tồn tại khả năng xảy ra:
* P1 gọi `wait(S)` // S = 0
* P2 gọi `wait(Q)` // Q = 0
* P1 gọi `wait(Q)` // P1 bị blocked vì Q đang bằng 0
* P2 gọi `wait(S)` // P2 bị blocked vì S đang bằng 0

**-> DEADLOCK (hai bên chờ nhau)**

> *Lưu ý:* Cần phải lưu ý giá trị khởi tạo và thứ tự sắp xếp các thao tác khi sử dụng semaphore.

---

## 8. MONITOR

### 5.8.1. Định nghĩa monitor
Nhiều loại vấn đề có thể dễ dàng phát sinh khi xử lý các vấn đề liên quan đến vùng tranh chấp nếu các lập trình viên sử dụng semaphore và khóa mutex không đúng cách. Một giải pháp được đề xuất để giải quyết các lỗi trên đó là sử dụng các công cụ đồng bộ đơn giản như một cấu trúc ngôn ngữ bậc cao, và đó chính là **monitor**.

* Là một kiểu dữ liệu trừu tượng (abstract data type) đóng gói những thành phần sau:
    * **Các biến nội bộ:** được khai báo bên trong monitor và chỉ có thể được truy cập bởi các hàm nội bộ trong monitor.
    * **Các thủ tục:** Một tập các thao tác được định nghĩa bởi lập trình viên và được thực thi theo loại trừ tương hỗ, các thủ tục này cũng chỉ có thể truy cập các biến nội bộ được khai báo ở trên.
    * **Đoạn code khởi tạo**

**Mã giả của một monitor:**
```c
monitor monitor-name
{
    // shared variable declarations
    procedure P1 (…) { …. }
    procedure P2 (…) { …. }
    procedure Pn (…) {……}
    initialization code (…) { … }
}
```

**Hiện thực monitor với semaphore:**
* Variables:
    * `semaphore mutex`
    * `mutex = 1`
* Mỗi thủ tục P sẽ được thay thế bởi đoạn mã bên dưới:
    ```c
    wait(mutex);
    …
    body of P;
    … 
    signal(mutex);
    ```

**Đặc điểm của monitor:**
* Tiến trình “vào monitor” bằng cách gọi một trong các thủ tục được định nghĩa trong monitor.
* Chỉ có một tiến trình có thể vào monitor tại một thời điểm -> mutual exclusion được bảo đảm.
* Tuy nhiên, cấu trúc của monitor không thực sự mạnh mẽ cho các mô hình đồng bộ khác -> cần định nghĩa thêm cơ chế đồng bộ -> **cấu trúc condition**.

### 5.8.2. Condition variable
Condition variable (biến điều kiện) là các biến có kiểu dữ liệu là condition. Chỉ có 2 thao tác có thể được thực hiện trên các biến kiểu condition đó là `wait()` và `signal()`. Condition variable cho phép lập trình viên hiện thực các mô hình đồng bộ riêng biệt theo đúng nhu cầu của từng chương trình.

* Nhằm cho phép tiến trình đợi “trong monitor”, chỉ có thể được truy cập bên trong monitor.
* Khai báo: `condition x, y;`
* Chỉ có thể thao tác lên condition variable bằng 02 thao tác:
    * `x.wait()` – tiến trình thực thi thao tác này sẽ bị block trên condition variable `x` cho đến khi thao tác `x.signal()` được thực thi.
    * `x.signal()` – phục hồi quá trình thực thi của một tiến trình (nếu có) bị block trên condition variable `x`.
        * *Nếu có nhiều tiến trình bị block: chỉ một tiến trình được phục hồi.*
        * *Nếu không có tiến trình nào bị block: không có tác dụng.*

**Đặc điểm của condition variable:**
* Các tiến trình có thể đợi ở **entry queue** hoặc đợi ở các **condition queue** (`x, y,…`).
* Khi thực hiện lệnh `x.wait()`, tiến trình sẽ được chuyển vào **condition queue x**.
* Lệnh `x.signal()` chuyển một tiến trình từ condition queue x vào monitor.
* Khi đó, để bảo đảm mutual exclusion, tiến trình gọi `x.signal()` sẽ bị blocked và được đưa vào **urgent queue**.

**Sử dụng condition variable:**
Đồng bộ P1 và P2 sao cho hàm F1 luôn luôn thực thi phần S1 trước khi hàm F2 thực thi phần S2.
```c
monitor MONITOR_NAME {
    condition x; 
    // x = 0 (logic ngầm)
    boolean done = false; // logic khởi tạo

    procedure F1() {
        S1;
        done = true;
        x.signal();
    }

    procedure F2() {
        if (done == false) {
            x.wait();
        }
        S2;
    }
}
```

---

## 9. LIVENESS
Quá trình đồng bộ tiến trình có thể gây ra các lỗi nghiêm trọng trong việc làm tiến trình bị “kẹt” và không thể tiếp tục chạy. Liveness là thuật ngữ để chỉ một tập các đặc điểm mà hệ thống phải thỏa mãn để đảm bảo rằng các tiến trình thực sự đang chạy.

### Appendix A: Liveness
* Tiến trình có thể phải chờ vô thời hạn để cố gắng yêu cầu các công cụ đồng bộ như mutex hay semaphore -> vi phạm tiêu chí **progress** và **bounded waiting**.
* **Liveness** là thuật ngữ để chỉ một tập các đặc điểm mà hệ thống phải thỏa mãn để đảm bảo tiến trình thực sự chạy.
* Chờ đợi không giới hạn là một ví dụ tiêu biểu cho việc liveness thất bại (tiến trình không còn chạy).

**DEADLOCK:** 
* Là tình trạng **hai hay nhiều tiến trình** đang **chờ đợi không giới hạn** cho một sự kiện mà sự kiện này chỉ có thể được thực hiện bởi một trong các tiến trình đang chờ ở trên.

**Một số dạng khác của deadlock:**
* **Starvation – đói:**
    * Một tiến trình có thể không bao giờ được thoát ra khỏi hàng đợi của semaphore mà nó đang chờ.
* **Priority inversion – nghịch đảo ưu tiên:**
    * Vấn đề định thời khi tiến trình có độ ưu tiên thấp giữ khóa mà đang được cần bởi tiến trình có độ ưu tiên cao.
    * Có thể được giải quyết bằng priority inheritance protocol.

---

### TÓM TẮT LẠI NỘI DUNG BUỔI HỌC
* **Mutex locks**
* **Semaphore**
* **Monitor**
* **Liveness**

