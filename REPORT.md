# Báo cáo lab Nhóm 09

**Họ tên:** [Họ và tên của bạn] · **MSSV:** [Mã số sinh viên của bạn] · **JDK đã dùng** (kết quả `java -version`): `java version "17.0.19" 2026-04-21 LTS`

## Bài 1: Giá trị biên

| Ngày trễ | Hạng | Số tiền bạn điền | Cách tính |
|---:|---|---:|---|
| 24 | NORMAL | 48000 | 24 ngày × 2.000đ/ngày = 48.000đ (dưới mức trần 50.000đ) |
| 25 | NORMAL | 50000 | 25 ngày × 2.000đ/ngày = 50.000đ (vừa đúng chạm mức trần 50.000đ) |
| 26 | NORMAL | 50000 | 26 ngày × 2.000đ/ngày = 52.000đ, áp mức trần tối đa 50.000đ |
| 2 | VIP | 0 | Thuộc thời gian ân hạn 2 ngày đầu (0đ) |
| 3 | VIP | 1000 | Hết 2 ngày ân hạn, tính phí từ ngày thứ 3: (3 - 2) × 1.000đ/ngày = 1.000đ |
| 51 | VIP | 49000 | (51 - 2) × 1.000đ/ngày = 49.000đ (dưới mức trần 50.000đ) |
| 52 | VIP | 50000 | (52 - 2) × 1.000đ/ngày = 50.000đ (chạm mức trần 50.000đ) |
| 53 | VIP | 50000 | (53 - 2) × 1.000đ/ngày = 51.000đ, áp mức trần tối đa 50.000đ |

**Vì sao chọn các ngày này mà không chọn ngày 10?**
- Các ngày trên được chọn theo kỹ thuật **Phân tích giá trị biên (Boundary Value Analysis - BVA)** kết hợp với phân vùng tương đương. Các lỗi lập trình (đặc biệt là lỗi lệch 1 đơn vị - off-by-one errors như `>`, `>=`, `<`, `<=`) luôn có xác suất xảy ra cao nhất tại các điểm chuyển giao ranh giới logic.
- Cụ thể:
  + Với hạng **NORMAL**: Ngưỡng chạm trần 50.000đ đạt được tại mốc 25 ngày ($25 \times 2.000 = 50.000$). Bộ ba $\{24, 25, 26\}$ kiểm tra chính xác 3 trạng thái biên: ngay trước trần (48.000đ), đúng điểm chạm trần (50.000đ) và vượt trần (50.000đ) để đảm bảo hàm `Math.min(amount, 50_000L)` hoạt động chính xác.
  + Với hạng **VIP**:
    * Biên ân hạn: Miễn phí 2 ngày đầu và bắt đầu tính phí từ ngày thứ 3. Cặp $\{2, 3\}$ kiểm tra biên của điều kiện `daysLate <= graceDays`.
    * Biên áp trần 50.000đ: Đạt mức 50.000đ tại ngày thứ 52 ($50 \text{ ngày tính phí} \times 1.000 = 50.000$). Bộ ba $\{51, 52, 53\}$ kiểm tra chuyển tiếp từ dưới trần (49.000đ), chạm trần (50.000đ) và vượt trần (50.000đ).
  + Ngược lại, ngày 10 là giá trị nằm sâu bên trong vùng tương đương (chưa chạm biên ân hạn của VIP và chưa chạm mức trần 50.000đ của cả hai hạng), do đó không có khả năng phát hiện lỗi biên.

## Bài 2a: Coverage

| | Branch coverage của `LoanService` |
|---|---|
| Trước khi viết test | 0.66 (66.67% - 4/6 nhánh) |
| Sau khi viết 2 test | 1.00 (100% - 6/6 nhánh) |

**Test `daDatHanMuc` của bạn dùng `countActiveByMemberId` trả về giá trị nào? Vì sao chọn giá trị đó?**
- Test `daDatHanMuc` chọn `countActiveByMemberId` trả về giá trị **`3L`**.
- **Vì sao chọn giá trị đó:**
  + Đặc tả yêu cầu: *"Mỗi thành viên được mượn tối đa 3 cuốn: đang mượn 3 cuốn thì không được mượn thêm (`LIMIT_REACHED`)"*.
  + Theo phân tích giá trị biên (BVA):
    * Đang mượn 2 cuốn: còn hạn mức, được phép mượn tiếp (đã kiểm tra ở test `muonThanhCongKhiDangMuonHaiCuon`).
    * Đang mượn 3 cuốn: đã đạt tối đa hạn mức, lần mượn này phải bị từ chối với ngoại lệ `LIMIT_REACHED`.
  + Vì vậy, `3L` là giá trị biên nhỏ nhất và chuẩn xác nhất theo đặc tả để kích hoạt ngoại lệ. Nếu chọn giá trị lớn hơn (ví dụ 4L) thì test vẫn đạt coverage nhưng sẽ bỏ sót việc phát hiện lỗi khi người dùng đang mượn đúng 3 cuốn.

## Bài 2b: Mutation testing

**Lần chạy PIT đầu tiên:** có bao nhiêu mutant sống sót (SURVIVED)? Nằm ở dòng nào, là loại mutant gì?
- Có **1 mutant sống sót** (SURVIVED) trên tổng số 9 mutant (tỷ lệ diệt 8/9, đạt 89%).
- Vị trí: Dòng **28** trong file `LoanService.java` (`if (loans.countActiveByMemberId(memberId) > 3)`).
- Loại mutant: **ConditionalsBoundaryMutator** (`changed conditional boundary → SURVIVED`). PIT đã sửa đổi toán tử điều kiện từ `>` thành `>=`.

**Lỗi cài sẵn nằm ở đâu? Bạn phát hiện ra nó bằng cách nào?**
- Vị trí lỗi: Dòng **28** của `src/main/java/vn/edu/seminar/loan/LoanService.java`:
  `if (loans.countActiveByMemberId(memberId) > 3)`
- Cách phát hiện:
  1. Khi chạy PIT lần đầu với test `daDatHanMuc` ban đầu (dùng giá trị 4L), mutant thay đổi toán tử biên (`>` thành `>=`) sống sót vì cả `4 > 3` và `4 >= 3` đều trả về `true`.
  2. Báo cáo PIT chỉ ra dòng 28 có mutant sống sót. Đối chiếu với đặc tả nghiệp vụ: *"đang mượn 3 cuốn thì không được mượn thêm"*. Điều này khẳng định khi số sách đang mượn là 3 thì đã phải ném ngoại lệ `LIMIT_REACHED`.
  3. Khi sửa mock trong test về đúng giá trị biên `3L`, test lập tức **bị FAILED (đỏ)** trên code gốc, vì với `3 > 3` trả về `false`, hệ thống bỏ qua kiểm tra và cho mượn cuốn thứ 4. Điều này chứng minh lỗi cài sẵn nằm ở toán tử so sánh `> 3` trong code.

**Bạn đã sửa test và code như thế nào?**
- **Trong test (`LoanServiceTest.java`):** Đặt mock `countActiveByMemberId` trả về giá trị biên `3L`:
  ```java
  when(loans.countActiveByMemberId(1)).thenReturn(3L);
  ```
- **Trong code (`LoanService.java`):** Sửa điều kiện so sánh tại dòng 28 từ `> 3` thành `>= 3`:
  ```java
  if (loans.countActiveByMemberId(memberId) >= 3) {
      throw new LoanException(LIMIT_REACHED);
  }
  ```

**Kết quả PIT sau khi sửa** (Killed bao nhiêu / tổng bao nhiêu):
- **Killed 9 / 9 (100%)**
- Chi tiết:
  + `ConditionalsBoundaryMutator`: 2/2 killed (100%)
  + `NegateConditionalsMutator`: 3/3 killed (100%)
  + `NullReturnValsMutator`: 3/3 killed (100%)
  + `VoidMethodCallMutator`: 1/1 killed (100%)
  + Line coverage của mutated classes: 17/17 (100%)

## Câu hỏi tổng kết
**Coverage 100% có nghĩa là code không còn lỗi không? Hãy giải thích dựa trên những gì bạn thấy trong lab.**
- **Không. Code coverage 100% không có nghĩa là code đã sạch lỗi.**
- Giải thích từ thực tế bài lab:
  1. **Coverage chỉ ghi nhận code đã được thực thi (execution), không đánh giá tính đúng đắn của logic:** Ở bài 2a, khi ta viết test với số sách là `4L`, JaCoCo đã ghi nhận **100% branch coverage** và **100% line coverage** (build hoàn toàn xanh). Dù vậy, code vẫn chứa lỗi nghiệp vụ nghiêm trọng (`> 3` thay vì `>= 3`), khiến người đang mượn 3 cuốn vẫn mượn tiếp được sách thứ 4.
  2. **Coverage không kiểm tra được các giá trị biên:** Test chỉ cần kích hoạt nhánh rẽ (True/False) bằng một giá trị tùy ý là đã tính đủ 100% coverage, nhưng hoàn toàn có thể bỏ lọt các lỗi biên (off-by-one errors) do chọn sai test case.
  3. **Ý nghĩa của Mutation Testing:** Mutation testing kiểm tra chất lượng thực sự của test suite bằng cách cố tình tạo lỗi giả lập (mutants). Nhờ PIT chỉ ra mutant sống sót ở giá trị biên, ta mới phát hiện test chưa đủ chặt chẽ và bộc lộ được lỗi tiềm ẩn trong source code.
