# DATABASE SCHEMA - KIẾN TRÚC HỆ THỐNG LUYỆN THI TIẾNG ANH

## 1. User

Lưu trữ thông tin người dùng trong hệ thống.

**Đối tượng chính**: User

**ID Range**: 0 - 999,999

```json
{
  "user_id": 123455,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "secret": "hashed_password_or_secret_key",
  "is_admin": false
}
```

### Cấu trúc chi tiết:
- **user_id**: Định danh duy nhất cho user (range: 0 - 999,999)
- **username**: Tên đăng nhập của user
- **email**: Địa chỉ email của user
- **secret**: Mật khẩu đã hash hoặc secret key để xác thực
- **is_admin**: true/false - user có phải admin hay không


## 2. Admin Question Bank

Lưu trữ các câu hỏi do admin nhập từ file PDF/Image sau khi được extract.

**Đối tượng chính**: Question

**ID Range**: 0 - 999,999,999

Hỗ trợ 2 loại câu hỏi:
1. **Single Choice (type: 0)**: Câu trắc nghiệm đơn lẻ
2. **Group Choice (type: 1)**: Nhóm câu hỏi liên tiếp dùng chung 1 context

```json
{
  "id": 123456788,
  "type": 0,
  "content": "What is the correct form of the verb?",
  "options": [
    "I go to school",
    "I goes to school",
    "I going to school",
    "I am go to school"
  ],
  "correct_answer": 0
}
```

```json
{
  "id": 234567889,
  "type": 1,
  "context": "Read the following passage:\n\nJohn has been working as an engineer for 10 years. He is very passionate about his work and always tries to improve his skills. Last month, he received a promotion.",
  "subquestions": [
    {
      "content": "How long has John been working as an engineer?",
      "options": [
        "10 years",
        "5 years",
        "15 years",
        "20 years"
      ],
      "correct_answer": 0
    },
    {
      "content": "What did John receive last month?",
      "options": [
        "A promotion",
        "A new job",
        "A salary increase",
        "A new office"
      ],
      "correct_answer": 0
    }
  ]
}
```

### Cấu trúc chi tiết:

#### Single Choice (type: 0):
- **id**: Định danh duy nhất cho câu hỏi (range: 0 - 999,999,999, ví dụ: 123456788)
- **type**: 0 (Single Choice)
- **content**: Nội dung câu hỏi
- **options**: Mảng 4 lựa chọn
- **correct_answer**: Index của đáp án đúng (0-3)

#### Group Choice (type: 1):
- **id**: Định danh duy nhất cho nhóm câu hỏi (range: 0 - 999,999,999, ví dụ: 234567889)
- **type**: 1 (Group Choice)
- **context**: Nội dung chung dùng cho tất cả các câu trong nhóm (đoạn văn, bảng, hình ảnh...)
- **subquestions**: Mảng các câu hỏi con trong nhóm
  - **content**: Nội dung của từng câu hỏi
  - **options**: Mảng 4 lựa chọn
  - **correct_answer**: Index của đáp án đúng (0-3)


## 3. User Exam

Lưu trữ đề thi của từng user - gồm 40 câu hỏi được generate từ Admin Bank hoặc tạo mới bằng AI.

**Đối tượng chính**: Exam

**ID Range**: 
- exam_id: 0 - 999,999
- user_id: 0 - 999,999
- question_id (generated): 1,000,000,000 - 1,999,999,999

```json
{
  "exam_id": 456788,
  "user_id": 123455,
  "questions": [
    {
      "id": 1123456788,
      "type": 0,
      "content": "What is the correct form of the verb?",
      "options": [
        "I go to school",
        "I goes to school",
        "I going to school",
        "I am go to school"
      ],
      "correct_answer": 0
    },
    {
      "id": 1234567889,
      "type": 1,
      "context": "Read the following passage:\n\nJohn has been working as an engineer for 10 years. He is very passionate about his work and always tries to improve his skills. Last month, he received a promotion.",
      "subquestions": [
        {
          "content": "How long has John been working as an engineer?",
          "options": [
            "10 years",
            "5 years",
            "15 years",
            "20 years"
          ],
          "correct_answer": 0
        },
        {
          "content": "What did John receive last month?",
          "options": [
            "A promotion",
            "A new job",
            "A salary increase",
            "A new office"
          ],
          "correct_answer": 0
        }
      ]
    },
    {
      "id": 1345678899,
      "admin_question_id": 567890122
    }
  ]
}
```

### Cấu trúc chi tiết:
- **exam_id**: Định danh đề thi (range: 0 - 999,999)
- **user_id**: Định danh user (range: 0 - 999,999)
- **questions**: Mảng 40 câu hỏi
  - **Câu hỏi được generate (type 0 & 1)**: Lưu toàn bộ dữ liệu giống Admin Question Bank, id range 1,000,000,000 - 1,999,999,999 (ví dụ: 1123456788, 1234567889)
  - **Câu hỏi tham chiếu từ Admin Question Bank**: Chỉ lưu `id` + `admin_question_id` (ví dụ: id 1345678899, admin_question_id 567890122)


## 4. User Attempt

Lưu trữ kết quả làm bài của user cho mỗi đề thi.

**Đối tượng chính**: Exam + Timestamp

**ID Range**:
- exam_id: 0 - 999,999
- user_id: 0 - 999,999

```json
{
  "exam_id": 456788,
  "user_id": 123455,
  "timestamp": "2024-11-11T10:30:00Z",
  "correct_count": 35,
  "questions": [
    {
      "id": 1123456788,
      "user_answer": 1,
      "is_correct": false,
      "feedback": "Bạn chọn sai. Đáp án đúng là 'I go to school' vì..."
    },
    {
      "id": 1234567889,
      "user_answer": 2,
      "is_correct": false,
      "feedback": "Bạn chọn sai. John đã làm kỹ sư được 10 năm rồi, không phải 15 năm."
    },
    {
      "id": 1234567890,
      "user_answer": 1,
      "is_correct": true,
      "feedback": "Đúng rồi! Câu 4 của đoạn văn nói 'he received a promotion'."
    }
  ]
}
```

### Cấu trúc chi tiết:
- **exam_id**: Định danh đề thi (range: 0 - 999,999)
- **user_id**: Định danh user (range: 0 - 999,999)
- **timestamp**: Thời gian user làm bài (ISO 8601 format)
- **correct_count**: Số câu trả lời đúng (0-40)
- **questions**: Mảng kết quả cho từng câu hỏi
  - **id**: Tham chiếu đến câu hỏi trong User Exam (int, ví dụ: 1123456788)
  - **user_answer**: Index của câu trả lời user chọn (0-3)
  - **is_correct**: true/false - câu trả lời có đúng không
  - **feedback**: Giải thích từ LLM về câu trả lời, ngắn gọn trong 1 câu.


## 5. Chat History

Lưu trữ lịch sử cuộc trò chuyện của user về các câu hỏi.

**ID Range**:
- user_id: 0 - 999,999
- exam_id: 0 - 999,999

```json
{
  "user_id": 123455,
  "question_id": 1123456788,
  "exam_id": 456788,
  "timestamp": "2024-11-11T10:35:00Z",
  "messages": [
    {
      "index": 0,
      "sender": 0,
      "content": "Tại sao không phải 'I goes to school'?"
    },
    {
      "index": 1,
      "sender": 1,
      "content": "Vì 'I' là ngôi thứ nhất số ít, nên động từ phải ở dạng không thêm 's'. Chỉ 'he', 'she', 'it' mới thêm 's' ở present simple."
    }
  ]
}
```

### Cấu trúc chi tiết:
- **user_id**: Định danh user (range: 0 - 999,999)
- **question_id**: Tham chiếu đến câu hỏi (từ User Exam)
- **exam_id**: Tham chiếu đến đề thi (range: 0 - 999,999, optional)
- **timestamp**: Thời gian bắt đầu cuộc trò chuyện (ISO 8601 format)
- **messages**: Mảng tin nhắn
  - **index**: Thứ tự tin nhắn (bắt đầu từ 0)
  - **sender**: 0 (user) hoặc 1 (assistant)
  - **content**: Nội dung tin nhắn
