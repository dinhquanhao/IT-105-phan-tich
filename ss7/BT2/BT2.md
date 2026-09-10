# BT2 – Khử quan hệ nhiều-nhiều bằng Association Class

## Class Diagram

```text
┌──────────────────┐
│     Student      │
├──────────────────┤
│ - studentId: int │
│ - name: String   │
└──────────────────┘
        │
        │ 1
        │
        │ 0..*
┌──────────────────────┐
│      Enrollment      │
├──────────────────────┤
│ - enrollDate: Date   │
└──────────────────────┘
        │
        │ 0..*
        │
        │ 1
┌──────────────────┐
│      Course      │
├──────────────────┤
│ - courseId: int  │
│ - name: String   │
└──────────────────┘
```

## Multiplicity

```text
Student 1 ───── 0..* Enrollment
Course  1 ───── 0..* Enrollment
```

## Giải thích

- Một Student có thể chưa đăng ký khóa học nào hoặc đăng ký nhiều khóa học: `0..*`.
- Mỗi Enrollment thuộc về đúng 1 Student: `1`.
- Một Course có thể chưa có học viên hoặc có nhiều học viên đăng ký: `0..*`.
- Mỗi Enrollment thuộc về đúng 1 Course: `1`.

## Kết luận

Quan hệ ban đầu:

```text
Student * ───── * Course
```

được khử thành:

```text
Student 1 ───── 0..* Enrollment 0..* ───── 1 Course
```

Lớp Enrollment cho phép lưu thông tin riêng của từng lần đăng ký, cụ thể là `enrollDate: Date`.
