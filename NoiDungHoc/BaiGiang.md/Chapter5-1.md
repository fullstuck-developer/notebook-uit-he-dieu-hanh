# HỆ ĐIỀU HÀNH
## CHƯƠNG 5: ĐỒNG BỘ TIẾN TRÌNH (PHẦN 1)

Trong chương này, các vấn đề về đồng bộ tiến trình sẽ được thảo luận và làm rõ bao gồm: vì sao cần phải đồng bộ, các tiêu chuẩn về lời giải cho bài toán đồng bộ và các kỹ thuật đồng bộ.

---

## MỤC TIÊU
1. Trình bày được khái niệm race condition và mô tả được vấn đề vùng tranh chấp
2. Mô tả được các yêu cầu dành cho lời giải của bài toán vùng tranh chấp
3. Liệt kê được các giải pháp đồng bộ dựa trên ngắt (giải pháp phần mềm) và vấn đề của chúng
4. Trình bày được giải pháp đồng bộ dựa trên phần cứng bao gồm `test_and_set`, `compare_and_swap`, và biến đơn nguyên

---

## NỘI DUNG
1. Race condition
2. Vấn đề vùng tranh chấp
3. Lời giải cho vấn đề vùng tranh chấp
4. Các giải pháp dựa trên ngắt (giải pháp phần mềm)
5. Giải pháp phần cứng

---

## 01. RACE CONDITION

### 5.1.1 Bài toán Producer vs. Consumer
Bài toán Producer vs. Consumer mô tả về 02 tiến trình bao gồm: “Sản xuất” và “Bán hàng”. Nếu gọi biến `count` mô tả số lượng hàng hóa, thì tiến trình “Sản xuất” sẽ làm tăng giá trị của `count`; ngược lại, tiến trình “Bán hàng” sẽ làm giảm giá trị này. Khi ”Sản xuất” và “Bán hàng” diễn ra đồng thời, biến `count` sẽ chịu tác động của việc tăng và giảm cùng lúc. Khi đó, liệu rằng giá trị của `count` có còn đúng với logic?

- Gồm 02 tiến trình diễn ra đồng thời với nhau:
  - **Producer**: liên tục tạo ra hàng hóa $\rightarrow$ tăng biến `count`
  - **Consumer**: liên tục bán hàng $\rightarrow$ giảm biến `count`

- Thông thường các tiến trình đều sẽ được đặt trong vòng `while(1)` để thực thi liên tục.
- Khi các tiến trình thực thi đồng thời, các dữ kiện sau sẽ **KHÔNG** thể xác định được:
  - Tiến trình nào thực thi trước?
  - Tiến trình nào thực thi lâu hơn (do giải thuật định thời CPU)?
  - Tiến trình sẽ hết quantum time khi nào?

**Mã nguồn minh họa:**
*Producer*
```c
item nextProduce;
while(1){
    while(count == BUFFER_SIZE); 
    /*khong lam gi*/
    buffer[in] = nextProduce;
    count++;
    in = (in+1)%BUFFER_SIZE;
}
```

*Consumer*
```c
item nextConsumer;
while(1){
    while(count == 0);
    /*khong lam gi*/
    nextConsumer = buffer[out];
    count--;
    out = (out+1)%BUFFER_SIZE; 
}
```
*(Các tiến trình chia sẻ một bounded buffer và một biến count đếm số phần tử trong buffer)*

**Phân tích lệnh count++ và count--:**
`count++`
```text
Load:  reg1 = count
Inc:   reg1 = reg1 + 1
Store: count = reg1
```

`count--`
```text
Load:  reg2 = count
Dec:   reg2 = reg2 - 1
Store: count = reg2
```
*\*reg1, reg2 là các thanh ghi*

Giả sử **count = 5**, hãy cho biết giá trị của count với 02 trường hợp đan xen thực thi dưới đây?

*Quantum = 3 cycles (Kết quả: Giá trị không chính xác)*
```text
T1 | Producer: reg1 = count      (reg1 = 5)
T2 | Producer: reg1 = reg1 + 1   (reg1 = 6)
T3 | Producer: count = reg1      (count = 6)
T4 | Consumer: reg2 = count      (reg2 = 6)
T5 | Consumer: reg2 = reg2 – 1   (reg2 = 5)
T6 | Consumer: count = reg2      (count = 5)
```

*Quantum = 2 cycles (Kết quả: Giá trị không chính xác)*
```text
T1 | Producer: reg1 = count      (reg1 = 5)
T2 | Producer: reg1 = reg1 + 1   (reg1 = 6)
T3 | Consumer: reg2 = count      (reg2 = 5)
T4 | Consumer: reg2 = reg2 – 1   (reg2 = 4)
T5 | Producer: count = reg1      (count = 6)
T6 | Consumer: count = reg2      (count = 4)
```
$\rightarrow$ Quá trình thực thi của 2 lệnh `count++` và `count--` bị đan xen vào nhau gây sai lệch dữ liệu.

---

### 5.1.2 Bài toán Cấp phát PID
Khi một tiến trình P gọi hàm `fork()`, một tiến trình con sẽ được tạo ra, hệ điều hành sẽ cấp cho tiến trình con một số định danh gọi là PID. Như vậy nếu có 2 tiến trình P0 và P1 cùng gọi hàm `fork()` đồng thời với nhau thì chuyện gì sẽ xảy ra?

- 02 tiến trình P0 và P1 đang tạo tiến trình con bằng cách gọi hàm `fork()`.
- Biến `next_available_pid()` được kernel sử dụng để tạo ra PID cho tiến trình mới.
- Tiến trình con của P0 và P1 đồng thời yêu cầu PID và nhận được kết quả như nhau.
- Cần có cơ chế để ngăn P0 và P1 truy cập biến `next_available_pid` cùng lúc, để tránh tình trạng một PID được cấp cho 2 tiến trình.

---

### 5.1.3. Race condition
**Race condition** là hiện tượng xảy ra khi các tiến trình cùng truy cập đồng thời vào dữ liệu được chia sẻ. Kết quả cuối cùng sẽ phụ thuộc vào thứ tự thực thi của các tiến trình đang chạy đồng thời với nhau.

- Trong bài toán Producer vs. Consumer dữ liệu được chia sẻ là biến `count` bị tác động đồng thời bởi cả 02 tiến trình Producer và Consumer. Trong bài toán cấp phát PID, dữ liệu được chia sẻ là biến `next_available_pid` bị tranh giành bởi tiến trình thực thi đồng thời là P0 và P1.
- Race condition có thể dẫn đến việc dữ liệu bị sai và không nhất quán (inconsistency).
- Để dữ liệu chia sẻ được nhất quán, cần bảo đảm sao cho tại mỗi thời điểm chỉ có một tiến trình được thao tác lên dữ liệu chia sẻ. Do đó, cần có cơ chế đồng bộ hoạt động của các tiến trình này.

---

## 02. VẤN ĐỀ VÙNG TRANH CHẤP
Vùng tranh chấp (hay còn gọi là critical section) là vùng code mà ở đó các tiến trình thực hiện tác động lên dữ liệu được chia sẻ.

### 5.2 Vấn đề vùng tranh chấp
- Xem xét một hệ thống có n tiến trình {$P_0, P_1, . . . , P_{n-1}$}
- Mỗi tiến trình có một **vùng tranh chấp** là một **đoạn code**:
  - Thực hiện việc thay đổi giá trị của dữ liệu được chia sẻ (có thể là các biến, bảng dữ liệu, file,...)
  - Khi một tiến trình đang thực hiện vùng tranh chấp của mình thì các tiến trình khác **KHÔNG** được thực hiện vùng tranh chấp của chúng.
- Vấn đề vùng tranh chấp chính là thiết kế cách thức xử lý các vấn đề trên.

Mỗi tiến trình phải yêu cầu để được phép tiến vào vùng tranh chấp của mình thông qua `entry section`, sau đó thực thi vùng tranh chấp – `critical section` - rồi tiến đến `exit section`, và sau cùng là thực thi `remainder section`.

```c
while(1){
    entry section
        critical section
    exit section
        remainder section
}
```

---

## 03. LỜI GIẢI CHO BÀI TOÁN VÙNG TRANH CHẤP

### 5.3.1. Yêu cầu dành cho lời giải
Vấn đề vùng tranh chấp là một vấn đề phức tạp, do đó, ta cần có những yêu cầu cụ thể để đảm bảo rằng lời giải cho bài toán này có thể đáp ứng được các tiêu chuẩn như các tiến trình đều phải được thực thi, không bị xảy ra hiện tượng đói, dữ liệu không bị thiếu nhất quán hay không để xảy ra tình trạng deadlock.

Lời giải cho bài toán vùng tranh chấp phải đảm bảo 03 yêu cầu sau:
1. **Mutual exclusion (loại trừ tương hỗ)**: Khi một tiến trình P đang thực thi trong vùng tranh chấp (CS) của nó thì không có tiến trình Q nào khác đang thực thi trong CS của Q.
2. **Progress (tiến triển)**: Một tiến trình tạm dừng bên ngoài vùng tranh chấp không được ngăn cản các tiến trình khác vào vùng tranh chấp. (Hiện tượng tiến trình P chờ điều kiện từ tiến trình Q khi tiến trình Q cũng đang chờ điều kiện từ tiến trình P để được vào vùng tranh chấp được gọi là **deadlock**).
3. **Bounded waiting (chờ đợi giới hạn)**: Mỗi tiến trình chỉ phải chờ để được vào vùng tranh chấp trong một khoảng thời gian có hạn định nào đó. Không xảy ra tình trạng đói tài nguyên (starvation).

---

### 5.3.2. Phân loại giải pháp
Có nhiều hướng tiếp cận cho lời giải của bài toán vùng tranh chấp. Tùy thuộc vào tiêu chí mà chúng ta có thể phân loại thành các giải pháp phần mềm/phần cứng, các giải pháp đòi hỏi/không đòi hỏi sự hỗ trợ của hệ điều hành, các giải pháp yêu cầu sự chờ đợi của tiến trình,...

**Phân loại theo sự hỗ trợ của phần cứng**
- **Giải pháp phần mềm/Giải pháp dựa trên ngắt**:
  - Không cần sự hỗ trợ từ phần cứng, có thể được thực hiện thông qua các kỹ thuật lập trình.
  - Ví dụ: giải thuật Peterson, giải thuật Bakery, giải thuật Dekker.
- **Giải pháp dựa trên phần cứng**:
  - Cần sự hỗ trợ của một vài phần cứng đặc biệt, ví dụ cung cấp cơ chế đơn nguyên cho một vài chỉ thị/lệnh nhất định.
  - Ví dụ: Test & Set, Compare & Swap.

**Phân loại theo sự hỗ trợ của hệ điều hành**
- **Busy waiting**:
  - Không cần sự hỗ trợ của hệ điều hành.
  - Sử dụng kỹ thuật lập trình để tiến trình/tiểu trình phải chờ đợi (trong khi liên tục kiểm tra điều kiện) để được vào vùng tranh chấp.
- **Sleep & Wake up**:
  - Cần hệ điều hành cung cấp cơ chế (thông qua system call) để:
    - *Tạm dừng (block) tiến trình*: đưa tiến trình gọi lệnh này vào trạng thái ngủ (sleep) nếu không được vào vùng tranh chấp.
    - *Đánh thức tiến trình*: khi một tiến trình ra khỏi vùng tranh chấp, tiến trình này có thể “đánh thức” (wake up) một tiến trình khác đang ngủ để tiến trình đó vào vùng tranh chấp.

---

## 04. CÁC GIẢI PHÁP DỰA TRÊN NGẮT (GIẢI PHÁP PHẦN MỀM)
Trong phần này, chúng ta sẽ nghiên cứu cách cài đặt các giải pháp phần mềm – thực hiện theo cơ chế ngắt - để thực hiện giải quyết bài toán vùng tranh chấp. Các giải pháp này cần phải thỏa mãn 03 yêu cầu dành cho lời giải bài toán vùng tranh chấp.

### 5.4. Các giải pháp dựa trên ngắt
- `Entry section`: vô hiệu hóa ngắt
- `Exit section`: kích hoạt ngắt
- Liệu rằng giải pháp này có thể giải quyết được bài toán?
  - Chuyện gì sẽ xảy ra nếu vùng tranh chấp là đoạn code chạy trong vòng 1 giờ?
  - Liệu có tiến trình nào bị đói không?
  - Nếu có 2 CPUs cùng chạy thì sao?

---

### 5.4.1. Giải pháp phần mềm 1
Ý tưởng của giải pháp này sử dụng một biến `turn` để kiểm tra xem tiến trình tới lượt thực hiện của tiến trình nào với sự hỗ trợ của 2 thao tác đơn nguyên là `load` và `store`.

- Giải pháp dành cho 2 tiến trình.
- Giả sử 2 lệnh hợp ngữ `load` và `store` là 2 thao tác **đơn nguyên** (không thể bị cắt ngang).
- 2 tiến trình cùng chia sẻ một biến `turn` (`int turn;`)
- Biến `turn` có tác dụng chỉ ra tiến trình nào tới lượt để vào vùng tranh chấp.
- Giá trị của `turn` sẽ được khởi tạo là `i`.

**Mã nguồn tiến trình $P_i$**:
```c
while (true)
{ 
    while (turn == j);   /* entry section, vô hiệu hóa ngắt */
                         /* Nếu kiểm tra thấy đang lượt của P_j, thì P_i chờ và không làm gì cả */
                         /* turn != j -> turn = i: tới lượt của P_i nên thoát khỏi vòng lặp while và tiến vào vùng tranh chấp */
    
    /* critical section */
    
    turn = j;            /* exit section, kích hoạt ngắt */
                         /* Sau khi P_i thực thi vùng tranh chấp xong thì trả lượt lại về cho P_j */
    
    /* remainder section */ 
}
```

- **Mutual exclusion được đảm bảo**:
  - $P_i$ chỉ được phép vào vùng tranh chấp khi: `turn = i`
  - và `turn` không thể vừa bằng `i`, vừa bằng `j` được.
- **Kiểm tra Progress $\rightarrow$ Không đảm bảo**: 
  - Do $P_0$ có thể phải chờ $P_1$ trả `turn = 0`, mà $P_1$ lại tốn thời gian chạy Remainder Section, không vào CS nhưng cản $P_0$ vào CS do `turn` vẫn đang bằng 1.
- **Kiểm tra Bounded Waiting $\rightarrow$ Không đảm bảo**:
  - $P_0$ không biết phải chờ bao lâu để được vào CS do $P_1$ tốn nhiều thời gian chạy Remainder Section.

---

### 5.4.2. Giải pháp phần mềm 2
Ý tưởng của giải pháp này sử dụng một mảng `flag[]` để kiểm tra xem tiến trình tới lượt thực hiện của tiến trình nào với sự hỗ trợ của 2 thao tác đơn nguyên là `load` và `store`.

- Giải pháp dành cho 2 tiến trình.
- Giả sử 2 lệnh hợp ngữ `load` và `store` là 2 thao tác **đơn nguyên** (không thể bị cắt ngang).
- 2 tiến trình cùng chia sẻ một biến `flag` (`boolean flag[2];`)
- Mảng `flag[]` được dùng để xác định liệu tiến trình đã sẵn sàng để vào vùng tranh chấp chưa.
  - `flag[i] = true;` cho biết là $P_i$ đã sẵn sàng để vào vùng tranh chấp.
- Giá trị của `flag[i]` sẽ được khởi tạo là `false`.

**Mã nguồn tiến trình $P_i$**:
```c
while (true)
{ 
    flag[i] = true;      /* P_i sẵn sàng để vào vùng tranh chấp */
    while (flag[j]);     /* Nếu kiểm tra thấy đang lượt của P_j, thì P_i chờ và không làm gì cả */
    
    /* critical section */
    
    flag[i] = false;     /* Sau khi P_i thực thi vùng tranh chấp xong thì trả lượt lại về cho P_j */
    
    /* remainder section */ 
}
```
- Mutual exclusion, Progress và Bounded waiting có được đảm bảo? (Chưa giải quyết triệt để).

---

### 5.4.3. Giải pháp Peterson
Giải pháp phần mềm 1 và 2 đã đưa ra ý tưởng về cách đảm bảo mutual exclusion tuy vẫn chưa thực hiện tốt việc đảm bảo progress và bounded waiting. Khắc phục các nhược điểm của các giải pháp trên, Peterson đã đề xuất một giải pháp đảm bảo được cả 03 yêu cầu về lời giải của bài toán vùng tranh chấp.

- Giải pháp dành cho 2 tiến trình.
- Giả sử 2 lệnh hợp ngữ `load` và `store` là 2 thao tác **đơn nguyên** (không thể bị cắt ngang).
- 2 tiến trình cùng chia sẻ hai biến:
  - `int turn;`
  - `boolean flag[2];`
- Biến `turn` có tác dụng chỉ ra tiến trình nào tới lượt để vào vùng tranh chấp.
- Mảng `flag[]` được dùng để xác định liệu tiến trình đã sẵn sàng để vào vùng tranh chấp chưa. (`flag[i] = true;` cho biết là $P_i$ đã sẵn sàng).

**Mã nguồn tiến trình $P_i$**:
```c
while (true){ 
    flag[i] = true;       /* P_i sẵn sàng để vào vùng tranh chấp */
    turn = j;             /* Nhường lượt cho P_j */
    while (flag[j] && turn == j); /* Nếu như P_j sẵn sàng và đang lượt của P_j thì P_i chờ */
    
    /* critical section */
    
    flag[i] = false;      /* P_i bỏ trạng thái sẵn sàng vào vùng tranh chấp */
    
    /* remainder section */
}
```

- **Mutual exclusion được đảm bảo**:
  - $P_i$ chỉ được phép vào vùng tranh chấp khi: hoặc `flag[j] = false` hoặc `turn = i`
  - và `turn` không thể vừa bằng `i`, vừa bằng `j` được.
- **Kiểm tra Progress $\rightarrow$ Đảm bảo**:
  - $P_1$ không được vào CS khi `flag[0] == true` và `turn == 0`
  - $P_0$ không thực hiện CS: Entry section (`turn = 1`), Exit section (`flag[0] = false`)
  - Nếu $P_0$ không vào CS, $P_0$ KHÔNG ngăn cản $P_1$ vào CS và ngược lại.
- **Kiểm tra Bounded Waiting $\rightarrow$ Đảm bảo**:
  - $P_1$ phải chờ tối đa là 1 lần $P_0$ vào vùng tranh chấp.
  - CS thường rất nhỏ nên thời gian chờ đợi sẽ rất ngắn.

---

### 5.4.4. Giải pháp Peterson và kiến trúc hiện đại
Mặc dù giải pháp Peterson đã được chứng minh là hiệu quả ở trên, tuy nhiên trên các kiến trúc hiện đại thì việc này chưa được đảm bảo. Trên các hệ thống hiện đại, để cải thiện hiệu suất của hệ thống thì bộ vi xử lý và/hoặc trình biên dịch có thể sắp xếp lại các thao tác độc lập với nhau. 

- Để cải thiện hiệu suất, vi xử lý và/hoặc trình biên dịch sẽ sắp xếp lại các thao tác mà độc lập với nhau.
- Việc hiểu vì sao giải pháp Peterson không hoạt động trên kiến trúc hiện đại sẽ giúp hiểu rõ hơn về race condition.
- Với các tiến trình đơn tiểu trình thì việc thực hiện các lệnh sẽ không có gì thay đổi.
- Với các tiến trình đa tiểu trình, việc sắp xếp lại các thao tác có thể dẫn đến kết quả không nhất quán hoặc không dự đoán được.

**Ví dụ về kiến trúc hiện đại:**
- Có 2 tiểu trình cùng chia sẻ dữ liệu: `boolean flag = false; int x = 0;`
- **Thread1** thực hiện:
```c
while (!flag);
print x;
```
- **Thread2** thực hiện:
```c
x = 100;
flag = true;
```
- Kết quả kỳ vọng được in ra là: `100`

- Tuy nhiên, bởi vì biến `flag` và biến `x` là độc lập với nhau nên các thao tác của Thread2 có thể bị **sắp xếp lại** thứ tự thực hiện thành:
```c
flag = true;
x = 100;
```
- Trong trường hợp này, kết quả có thể được in ra là: `0`

**Xét lại giải pháp Peterson trên kiến trúc hiện đại:**
- Việc gán `flag[]` và `turn` có thể bị **sắp xếp lại** thứ tự thực thi.
- Dẫn đến $P_0$ và $P_1$ có thể cùng vào CS.
- Để đảm bảo giải pháp Peterson hoạt động chính xác trên kiến trúc máy tính hiện đại, ta phải sử dụng **Memory Barrier**.

---

## 05. CÁC HỖ TRỢ TỪ PHẦN CỨNG

### 5.5.1. Memory Barrier
Một Memory Barrier (lớp bảo vệ bộ nhớ) là một lệnh bắt buộc bất kỳ thay đổi nào trên bộ nhớ phải được lan truyền đến tất cả các bộ xử lý.

**Memory model (mô hình bộ nhớ)**
- **Memory model** trong hệ điều hành là mô hình hoạt động của bộ nhớ trong hệ thống, bao gồm cách thức quản lý và truy xuất đến các vùng nhớ được cấp phát cho các tiến trình và luồng trong hệ thống. Memory model định nghĩa các quy tắc và ràng buộc cho việc sử dụng bộ nhớ, đảm bảo tính đúng đắn, an toàn và hiệu quả của các hoạt động trên bộ nhớ.
- Hai mô hình bộ nhớ phổ biến bao gồm:
  - **Mô hình bộ nhớ được sắp xếp mạnh**: các thay đổi bộ nhớ trên một bộ xử lý sẽ được các bộ xử lý khác biết ngay lập tức.
  - **Mô hình bộ nhớ được sắp xếp yếu**: các thay đổi bộ nhớ trên một bộ xử lý CÓ THỂ sẽ KHÔNG được các bộ xử lý khác biết ngay lập tức.

**Hoạt động của Memory Barrier**
- **Memory barrier** là một chỉ thị (instruction) mà bắt buộc mọi thay đổi trong bộ nhớ phải được truyền tải (hiển thị) đến tất cả bộ xử lý khác.
- Khi một chỉ thị memory barrier được thực hiện, hệ thống sẽ đảm bảo là tất cả thao tác `load` (nạp dữ liệu) và `store` (ghi dữ liệu) đều đã được hoàn thành trước khi các thao tác `load` và `store` sau đó được thực hiện.
- Do đó, kể cả khi các lệnh bị sắp xếp lại, memory barrier bảo đảm rằng các thao tác ghi dữ liệu đều đã được hoàn thành trong bộ nhớ và được truyền tải đến các bộ xử lý khác trước khi các thao tác nạp dữ liệu hoặc ghi dữ liệu được thực thi trong tương lai.

**Ví dụ về Memory Barrier**
Xét lại ví dụ trước đó, chúng ta có thể dùng memory barrier để Thread1 chắc chắn in ra 100.

*Thread1:*
```c
while (!flag)
    memory_barrier();
print x;
```
*Thread2:*
```c
x = 100;
memory_barrier();
flag = true; 
```
- Với Thread1, ta dùng `memory_barrier()` để đảm bảo rằng giá trị của `flag` được đọc trước khi đọc giá trị của `x`.
- Với Thread 2, ta dùng `memory_barrier()` để đảm bảo thao tác gán `x = 100` diễn ra trước khi gán `flag = true`.

### 5.5.2. Lệnh phần cứng: test_and_set
### 5.5.3. Lệnh phần cứng: compare_and_swap
### 5.5.4. Biến đơn nguyên
*(Sinh viên tự nghiên cứu các mục trên và trình bày tại lớp).*

---

## THẢO LUẬN & TÓM TẮT
**Tóm tắt lại nội dung buổi học:**
- Race condition
- Vấn đề vùng tranh chấp
- Lời giải cho vấn đề vùng tranh chấp
- Các giải pháp dựa trên ngắt (giải pháp phần mềm)
- Giải pháp phần cứng

