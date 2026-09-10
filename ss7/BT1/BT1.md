# BT1 – Thiết kế đóng gói lớp Student

## Class Diagram

```text
┌─────────────────────────────────────┐
│               Student               │
├─────────────────────────────────────┤
│ - id: int                           │
│ - name: String                      │
│ - age: int                          │
│ - score: double                     │
├─────────────────────────────────────┤
│ + getId(): int                      │
│ + setId(id: int): void              │
│ + getName(): String                 │
│ + setName(name: String): void       │
│ + getAge(): int                     │
│ + setAge(age: int): void            │
│ + getScore(): double                │
│ + setScore(score: double): void     │
└─────────────────────────────────────┘
```

## Ý nghĩa Access Modifier

- `-` = private: áp dụng cho toàn bộ thuộc tính.
- `+` = public: áp dụng cho toàn bộ phương thức.
- `#` = protected: không sử dụng trong bài này.

## Pseudocode setScore()

```text
FUNCTION setScore(score):
    IF score < 0 OR score > 10 THEN
        REJECT the update
    ELSE
        this.score = score
    END IF
END FUNCTION
```

## Logic

Chỉ cho phép cập nhật điểm khi `0 <= score <= 10`.
Nếu điểm nhỏ hơn 0 hoặc lớn hơn 10 thì từ chối cập nhật.
