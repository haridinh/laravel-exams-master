# Ứng dụng Trắc nghiệm - Laravel Project

## Thông tin sinh viên
- **Họ tên:** Đinh Bá Hải  
- **Mã sinh viên:** 22010126  
- **Tên dự án:** Ứng dụng Trắc nghiệm Online

---

## Mô tả dự án

Ứng dụng xây dựng bằng **Laravel** với mục tiêu tạo nền tảng thi trắc nghiệm trực tuyến, bao gồm:
- Quản lý bài thi, câu hỏi, đáp án.
- Tài khoản quản trị và thí sinh.
- Chức năng làm bài, tự động chấm điểm, xuất kết quả.

---

## 1. Sơ đồ cấu trúc hệ thống (Class Diagram)

![Class Diagram](https://i.imgur.com/RMcwXfm.png)

---

## 2. Thuật toán & Quy trình hoạt động

### 🛠️ Quản trị viên (Admin)
- **Quản lý bài thi:** Tạo, sửa, xóa, hiển thị danh sách bài thi.
- **Quản lý câu hỏi:** Gắn câu hỏi và đáp án vào từng bài thi.
- **Quản lý thí sinh:** Tạo/sửa/xóa danh sách người dùng dạng thí sinh.
- **Thống kê:** Xem kết quả, lịch sử làm bài, to-do list,...
  
![Admin Activity](https://i.imgur.com/fE0mokk.png)

---

### Thí sinh (User/Student)
- **Làm bài thi:**  
  - Đăng nhập hệ thống.  
  - Chọn bài thi có hiệu lực.  
  - Trả lời từng câu hỏi (trắc nghiệm 1 lựa chọn).  
  - Gửi kết quả.  
- **Chấm điểm:**  
  - Hệ thống tự động tính điểm, đánh giá đậu/rớt.  
  - Hiển thị kết quả.
  - Có thể **xuất ra file CSV** để lưu trữ.

![Student Activity](https://i.imgur.com/qgLGUNH.png)

---

### Thuật toán xử lý chính

#### A. **Chấm điểm bài thi**
1. Khi thí sinh nộp bài:
   - Lấy danh sách các câu trả lời đã chọn.
   - Kiểm tra đúng/sai, cộng điểm tương ứng theo `degree` mỗi câu.
   - Lưu kết quả từng câu hỏi vào bảng `results`.
   - Tính tổng điểm, xác định đậu/rớt (>= 50% điểm tối đa).
   - Ghi tổng điểm và trạng thái vào hồ sơ thí sinh.

#### B. **CRUD chuẩn Laravel**
- Áp dụng mô hình CRUD (Create, Read, Update, Delete) cho:
  - Bài thi (Exam)
  - Câu hỏi (Question)
  - Đáp án (Answer)
  - Thí sinh (Student)

---

## 3. Giao diện minh họa
![](https://i.imgur.com/VrsZrIU.png) 
![](https://i.imgur.com/8HNMeml.png) 
![](https://i.imgur.com/wLZTpyv.png) 
![](https://i.imgur.com/HVkgHPu.png) 
![](https://i.imgur.com/wPKCnQ3.png)
![](https://i.imgur.com/NyYwghK.png) 
![](https://i.imgur.com/VDLDOnW.png) 
![](https://i.imgur.com/nR47VUG.png) 
![](https://i.imgur.com/yMf1POg.png) 
![](https://i.imgur.com/JRTkYIk.png) 

---

## 4. Code xử lý logic chính

**File:** `app/Http/Controllers/User/HomeController.php`

```php
public function quizResult(Request $request) {
    if ($request->ajax()) {
        $collect = [];
        $dataIn = $request->all();
        $exam = Exam::find($request->examId);
        unset($dataIn['_token'], $dataIn['examId']);
        foreach($dataIn as $item) {
            $answer = Answer::find($item);
            $questionDegree = $answer->question->degree;
            $answerDegree = $answer->status ? $questionDegree : 0;
            $collect[] = $answerDegree;

            $result = new Result();
            $result->studentId = Auth::id();
            $result->examId = $exam->id;
            $result->questionId = $answer->question->id;
            $result->answerId = $answer->id;
            $result->degree = $answerDegree;
            $result->save();
        }

        $this->saveStudentDegree($collect, $exam);

        $result = $this->generateResultSnippet($exam, $dataIn, $collect);
        return response()->json([
            'result' => json_encode($result, JSON_INVALID_UTF8_IGNORE)
        ], Response::HTTP_OK);
    }
}
```
- Giải thích:
    - Nhận dữ liệu làm bài từ giao diện AJAX.
    - Kiểm tra đáp án đúng/sai, tính điểm cho từng câu.
    - Lưu từng kết quả vào bảng results.
    - Tính tổng điểm => xác định đậu/rớt.
    - Trả kết quả về frontend để hiển thị

---

## 5. Kết nối & Triển khai
- Không sử dụng cloud db.
- Sử dụng các bảo mật & quy ước.

---

## 6. Link repo:
https://github.com/haridinh/laravel-exams-master.git
