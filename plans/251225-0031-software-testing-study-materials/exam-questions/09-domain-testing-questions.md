# Topic 9: Domain Testing & Data Generation - Câu Hỏi Vận Dụng
**CS423 Software Testing | Final Exam Preparation**

---

## Câu 1: Equivalence Class Partitioning Design (10 điểm)

### Tình huống:
Bạn đang test form đăng ký tài khoản ngân hàng online với các trường:
- **Số CMND/CCCD**: 9 hoặc 12 chữ số
- **Số điện thoại**: 10 chữ số, bắt đầu bằng 03/05/07/08/09
- **Số tiền gửi ban đầu**: Tối thiểu 50,000 VND, tối đa 500,000,000 VND
- **Ngày sinh**: Phải đủ 18 tuổi, không quá 70 tuổi

### Yêu cầu:
a) Xác định các Equivalence Classes cho mỗi trường (4 điểm)
b) Chọn representative value cho mỗi class (3 điểm)
c) Thiết kế minimum test cases để cover all classes (3 điểm)

### Đáp án mẫu:

**a) Equivalence Classes Identification (4 điểm):**

| Field | Class Type | Description | Example |
|-------|------------|-------------|---------|
| **CMND/CCCD** | | | |
| | Valid-9 | 9 digits exactly | 123456789 |
| | Valid-12 | 12 digits exactly | 123456789012 |
| | Invalid-Short | < 9 digits | 12345678 |
| | Invalid-Between | 10-11 digits | 1234567890 |
| | Invalid-Long | > 12 digits | 1234567890123 |
| | Invalid-Alpha | Contains letters | 12345678A |
| | Invalid-Special | Contains special chars | 123-456-789 |
| **Số điện thoại** | | | |
| | Valid-03 | Starts with 03 | 0312345678 |
| | Valid-05 | Starts with 05 | 0512345678 |
| | Valid-07 | Starts with 07 | 0712345678 |
| | Valid-08 | Starts with 08 | 0812345678 |
| | Valid-09 | Starts with 09 | 0912345678 |
| | Invalid-Prefix | Invalid prefix (01,02,04,06) | 0112345678 |
| | Invalid-Length | Not 10 digits | 091234567 |
| **Số tiền gửi** | | | |
| | Invalid-Low | < 50,000 | 49,999 |
| | Valid-Min | = 50,000 | 50,000 |
| | Valid-Normal | 50,000 - 500,000,000 | 1,000,000 |
| | Valid-Max | = 500,000,000 | 500,000,000 |
| | Invalid-High | > 500,000,000 | 500,000,001 |
| **Ngày sinh (tuổi)** | | | |
| | Invalid-Young | < 18 tuổi | 17 tuổi |
| | Valid-Min | = 18 tuổi | 18 tuổi |
| | Valid-Normal | 18-70 tuổi | 35 tuổi |
| | Valid-Max | = 70 tuổi | 70 tuổi |
| | Invalid-Old | > 70 tuổi | 71 tuổi |

**b) Representative Values (3 điểm):**

```python
test_data = {
    'cmnd_cccd': {
        'valid_9': '123456789',
        'valid_12': '123456789012',
        'invalid_short': '12345678',        # 8 digits
        'invalid_between': '1234567890',    # 10 digits
        'invalid_long': '1234567890123',    # 13 digits
        'invalid_alpha': '12345678A',
        'invalid_special': '123-456-789',
    },
    'phone': {
        'valid_03': '0312345678',
        'valid_05': '0567890123',
        'valid_07': '0789012345',
        'valid_08': '0823456789',
        'valid_09': '0934567890',
        'invalid_prefix_01': '0112345678',
        'invalid_prefix_02': '0212345678',
        'invalid_length_short': '091234567',  # 9 digits
        'invalid_length_long': '09123456789', # 11 digits
    },
    'deposit': {
        'invalid_low': 49_999,
        'valid_min': 50_000,
        'valid_normal': 10_000_000,
        'valid_max': 500_000_000,
        'invalid_high': 500_000_001,
    },
    'age': {
        # Assuming today is 2024-01-15
        'invalid_young': '2007-01-16',  # 16 tuổi 364 ngày
        'valid_min': '2006-01-15',       # Exactly 18
        'valid_normal': '1989-06-15',    # 35 tuổi
        'valid_max': '1954-01-15',       # Exactly 70
        'invalid_old': '1953-01-14',     # 71 tuổi
    }
}
```

**c) Minimum Test Cases (3 điểm):**

```python
# Single Valid Test (1 test covering all valid classes)
VALID_COMPLETE = {
    'cmnd': '123456789',      # Valid-9
    'phone': '0912345678',    # Valid-09
    'deposit': 1_000_000,     # Valid-Normal
    'dob': '1989-06-15',      # Valid-Normal
    'expected': 'SUCCESS'
}

# Invalid Test Cases (1 per invalid class - can't combine invalid classes)
INVALID_CASES = [
    # CMND/CCCD invalid classes
    {'cmnd': '12345678', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_CMND_SHORT'},
    {'cmnd': '1234567890', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_CMND_INVALID'},
    {'cmnd': '1234567890123', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_CMND_LONG'},
    {'cmnd': '12345678A', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_CMND_FORMAT'},

    # Phone invalid classes
    {'cmnd': '123456789', 'phone': '0112345678', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_PHONE_PREFIX'},
    {'cmnd': '123456789', 'phone': '091234567', 'deposit': 1_000_000, 'dob': '1989-06-15', 'expected': 'ERROR_PHONE_LENGTH'},

    # Deposit invalid classes
    {'cmnd': '123456789', 'phone': '0912345678', 'deposit': 49_999, 'dob': '1989-06-15', 'expected': 'ERROR_DEPOSIT_MIN'},
    {'cmnd': '123456789', 'phone': '0912345678', 'deposit': 500_000_001, 'dob': '1989-06-15', 'expected': 'ERROR_DEPOSIT_MAX'},

    # Age invalid classes
    {'cmnd': '123456789', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '2007-01-16', 'expected': 'ERROR_AGE_YOUNG'},
    {'cmnd': '123456789', 'phone': '0912345678', 'deposit': 1_000_000, 'dob': '1953-01-14', 'expected': 'ERROR_AGE_OLD'},
]

# Additional valid combinations for better coverage
VALID_COMBINATIONS = [
    {'cmnd': '123456789012', 'phone': '0312345678', 'deposit': 50_000, 'dob': '2006-01-15', 'expected': 'SUCCESS'},  # CCCD-12, min deposit, min age
    {'cmnd': '123456789', 'phone': '0567890123', 'deposit': 500_000_000, 'dob': '1954-01-15', 'expected': 'SUCCESS'},  # max deposit, max age
]

# Total: 1 valid + 10 invalid + 2 additional = 13 test cases
# This covers all 21 equivalence classes with minimum tests
```

---

## Câu 2: Boundary Value Analysis (10 điểm)

### Tình huống:
Bạn đang test chức năng tính phí vận chuyển:
- Khoảng cách: 0-2000 km
- Trọng lượng: 0.1-50 kg
- Công thức: `fee = distance * 500 + weight * 10000`
- Phí tối thiểu: 30,000 VND
- Phí tối đa: 2,000,000 VND

### Yêu cầu:
a) Xác định boundary values cho distance và weight (3 điểm)
b) Thiết kế BVA test cases (4 điểm)
c) Kết hợp ECP + BVA để tối ưu test cases (3 điểm)

### Đáp án mẫu:

**a) Boundary Values Identification (3 điểm):**

```
DISTANCE (0-2000 km):
├── -1      (invalid, below min)
├── 0       (boundary, at min - có thể invalid tùy business)
├── 0.1     (just above min)
├── 1000    (nominal)
├── 1999.9  (just below max)
├── 2000    (boundary, at max)
└── 2001    (invalid, above max)

WEIGHT (0.1-50 kg):
├── 0       (invalid, below min)
├── 0.09    (invalid, just below min)
├── 0.1     (boundary, at min)
├── 0.11    (just above min)
├── 25      (nominal)
├── 49.99   (just below max)
├── 50      (boundary, at max)
└── 50.01   (invalid, above max)

FEE BOUNDARIES:
├── Calculated < 30,000    → Return 30,000 (min fee)
├── 30,000 ≤ Calculated ≤ 2,000,000 → Return Calculated
└── Calculated > 2,000,000 → Return 2,000,000 (max fee)
```

**b) BVA Test Cases (4 điểm):**

```python
class TestShippingFeeBVA:
    """Boundary Value Analysis test cases"""

    # Distance boundary tests (weight = 25kg nominal)
    DISTANCE_BVA = [
        # (distance, weight, expected_result)
        (-1, 25, 'ERROR_DISTANCE'),        # Below min
        (0, 25, 'ERROR_DISTANCE'),          # At min (assuming 0 is invalid)
        (0.1, 25, 250_050),                 # Just above min: 0.1*500 + 25*10000
        (1, 25, 250_500),                   # Min+1
        (1000, 25, 750_000),                # Nominal
        (1999, 25, 1_249_500),              # Max-1
        (2000, 25, 1_250_000),              # At max
        (2001, 25, 'ERROR_DISTANCE'),       # Above max
    ]

    # Weight boundary tests (distance = 1000km nominal)
    WEIGHT_BVA = [
        (1000, 0, 'ERROR_WEIGHT'),          # Below min
        (1000, 0.09, 'ERROR_WEIGHT'),       # Just below min
        (1000, 0.1, 501_000),               # At min: 1000*500 + 0.1*10000
        (1000, 0.11, 501_100),              # Just above min
        (1000, 25, 750_000),                # Nominal
        (1000, 49.99, 999_900),             # Just below max
        (1000, 50, 1_000_000),              # At max
        (1000, 50.01, 'ERROR_WEIGHT'),      # Above max
    ]

    # Fee cap boundary tests
    FEE_CAP_BVA = [
        # Cases that hit minimum fee cap (30,000)
        (10, 0.1, 30_000),                  # Calculated: 5,000+1,000 = 6,000 → Cap to 30,000
        (50, 0.5, 30_000),                  # Calculated: 25,000+5,000 = 30,000 → Exactly min

        # Cases that hit maximum fee cap (2,000,000)
        (2000, 50, 2_000_000),              # Calculated: 1,000,000+500,000 = 1,500,000 → Under cap
        # Need extreme values to hit max cap:
        # (2000*500 + 50*10000 = 1,500,000) < 2,000,000
        # If distance could be higher or we adjust formula...
    ]

    @pytest.mark.parametrize("distance,weight,expected", DISTANCE_BVA)
    def test_distance_boundaries(self, distance, weight, expected):
        if isinstance(expected, str):
            with pytest.raises(ValueError, match=expected):
                calculate_shipping_fee(distance, weight)
        else:
            assert calculate_shipping_fee(distance, weight) == expected

    @pytest.mark.parametrize("distance,weight,expected", WEIGHT_BVA)
    def test_weight_boundaries(self, distance, weight, expected):
        if isinstance(expected, str):
            with pytest.raises(ValueError, match=expected):
                calculate_shipping_fee(distance, weight)
        else:
            assert calculate_shipping_fee(distance, weight) == expected
```

**c) Combined ECP + BVA Optimization (3 điểm):**

```python
# Strategy: Use BVA for boundaries, ECP for middle values
# This reduces redundant tests while maintaining coverage

OPTIMIZED_TEST_CASES = [
    # === VALID CASES ===

    # Boundaries (BVA)
    {'distance': 0.1, 'weight': 0.1, 'expected': 30_000, 'note': 'Min-Min → Cap to min fee'},
    {'distance': 0.1, 'weight': 50, 'expected': 500_050, 'note': 'Min-Max'},
    {'distance': 2000, 'weight': 0.1, 'expected': 1_001_000, 'note': 'Max-Min'},
    {'distance': 2000, 'weight': 50, 'expected': 1_500_000, 'note': 'Max-Max'},

    # Boundary-1 / Boundary+1 (BVA)
    {'distance': 0.11, 'weight': 0.11, 'expected': 30_000, 'note': 'Min+1, Min+1'},
    {'distance': 1999, 'weight': 49.99, 'expected': 1_499_400, 'note': 'Max-1, Max-1'},

    # Nominal/Representative (ECP)
    {'distance': 500, 'weight': 10, 'expected': 350_000, 'note': 'Normal case'},
    {'distance': 1000, 'weight': 25, 'expected': 750_000, 'note': 'Middle values'},

    # === INVALID CASES ===

    # Below minimum (BVA)
    {'distance': -1, 'weight': 25, 'expected': 'ERROR', 'note': 'Distance < 0'},
    {'distance': 0, 'weight': 25, 'expected': 'ERROR', 'note': 'Distance = 0'},
    {'distance': 1000, 'weight': 0, 'expected': 'ERROR', 'note': 'Weight = 0'},
    {'distance': 1000, 'weight': 0.09, 'expected': 'ERROR', 'note': 'Weight < min'},

    # Above maximum (BVA)
    {'distance': 2001, 'weight': 25, 'expected': 'ERROR', 'note': 'Distance > max'},
    {'distance': 1000, 'weight': 50.01, 'expected': 'ERROR', 'note': 'Weight > max'},

    # Combined invalid (ECP - only 1 needed)
    {'distance': -100, 'weight': 100, 'expected': 'ERROR', 'note': 'Both invalid'},
]

# Implementation
def test_optimized_cases():
    for tc in OPTIMIZED_TEST_CASES:
        if tc['expected'] == 'ERROR':
            with pytest.raises(ValueError):
                calculate_shipping_fee(tc['distance'], tc['weight'])
        else:
            result = calculate_shipping_fee(tc['distance'], tc['weight'])
            assert result == tc['expected'], \
                f"Failed: {tc['note']} - Expected {tc['expected']}, got {result}"

# Total: 15 test cases cover all boundaries and partitions
# Without optimization: 7 distance × 8 weight = 56 combinations
# Savings: 73% reduction
```

---

## Câu 3: Pairwise/Combinatorial Testing (10 điểm)

### Tình huống:
Bạn đang test chức năng booking khách sạn với các parameters:
- Room Type: Single, Double, Suite (3)
- Board: Room Only, Breakfast, Half Board, Full Board (4)
- Duration: 1-3 nights, 4-7 nights, 8+ nights (3)
- Payment: Credit Card, Debit Card, Bank Transfer, Cash (4)
- Guest Type: Adult Only, With Children, With Pets (3)

### Yêu cầu:
a) Tính số test cases exhaustive vs pairwise (2 điểm)
b) Thiết kế pairwise test set đảm bảo all pairs covered (5 điểm)
c) Thêm constraints và điều chỉnh test set (3 điểm)

### Đáp án mẫu:

**a) Exhaustive vs Pairwise Calculation (2 điểm):**

```
EXHAUSTIVE TESTING:
Room × Board × Duration × Payment × Guest
  3  ×   4   ×    3     ×    4    ×   3  = 432 test cases

PAIRWISE TESTING:
- Number of pairs: C(5,2) = 10 pairs
- Each pair needs coverage
- Theoretical minimum: max(3×4, 4×4, 3×3) = 16 test cases
- Practical: ~20-25 test cases (with PICT/ACTS)

REDUCTION: (432-25)/432 = 94% reduction
```

**b) Pairwise Test Set Design (5 điểm):**

```python
# PICT Input File: hotel_booking.txt
"""
Room: Single, Double, Suite
Board: RoomOnly, Breakfast, HalfBoard, FullBoard
Duration: Short, Medium, Long
Payment: CreditCard, DebitCard, BankTransfer, Cash
Guest: AdultOnly, WithChildren, WithPets
"""

# Generated Pairwise Test Cases (using PICT)
PAIRWISE_TESTS = [
    # TC  Room     Board       Duration  Payment       Guest
    (1,  'Single', 'RoomOnly',  'Short', 'CreditCard',  'AdultOnly'),
    (2,  'Single', 'Breakfast', 'Medium', 'DebitCard',   'WithChildren'),
    (3,  'Single', 'HalfBoard', 'Long',   'BankTransfer', 'WithPets'),
    (4,  'Single', 'FullBoard', 'Short',  'Cash',        'WithChildren'),
    (5,  'Double', 'RoomOnly',  'Medium', 'BankTransfer', 'WithChildren'),
    (6,  'Double', 'Breakfast', 'Long',   'Cash',        'AdultOnly'),
    (7,  'Double', 'HalfBoard', 'Short',  'DebitCard',   'AdultOnly'),
    (8,  'Double', 'FullBoard', 'Medium', 'CreditCard',  'WithPets'),
    (9,  'Suite',  'RoomOnly',  'Long',   'DebitCard',   'WithChildren'),
    (10, 'Suite',  'Breakfast', 'Short',  'BankTransfer', 'AdultOnly'),
    (11, 'Suite',  'HalfBoard', 'Medium', 'Cash',        'WithChildren'),
    (12, 'Suite',  'FullBoard', 'Long',   'CreditCard',  'AdultOnly'),
    (13, 'Single', 'RoomOnly',  'Long',   'CreditCard',  'WithChildren'),
    (14, 'Double', 'RoomOnly',  'Short',  'Cash',        'WithPets'),
    (15, 'Suite',  'RoomOnly',  'Medium', 'CreditCard',  'WithPets'),
    (16, 'Single', 'Breakfast', 'Short',  'CreditCard',  'WithPets'),
    (17, 'Double', 'Breakfast', 'Medium', 'BankTransfer', 'WithPets'),
    (18, 'Suite',  'HalfBoard', 'Short',  'CreditCard',  'WithChildren'),
    (19, 'Single', 'FullBoard', 'Medium', 'DebitCard',   'AdultOnly'),
    (20, 'Double', 'FullBoard', 'Short',  'BankTransfer', 'AdultOnly'),
]

# Verification: All pairs covered
def verify_pairwise_coverage(tests):
    """Verify all pairs appear at least once"""
    parameters = ['Room', 'Board', 'Duration', 'Payment', 'Guest']
    values = {
        'Room': ['Single', 'Double', 'Suite'],
        'Board': ['RoomOnly', 'Breakfast', 'HalfBoard', 'FullBoard'],
        'Duration': ['Short', 'Medium', 'Long'],
        'Payment': ['CreditCard', 'DebitCard', 'BankTransfer', 'Cash'],
        'Guest': ['AdultOnly', 'WithChildren', 'WithPets'],
    }

    # Check all pairs
    uncovered_pairs = []

    for i, param1 in enumerate(parameters):
        for j, param2 in enumerate(parameters):
            if i >= j:
                continue

            for val1 in values[param1]:
                for val2 in values[param2]:
                    # Check if this pair exists in any test
                    pair_found = False
                    for test in tests:
                        if test[i] == val1 and test[j] == val2:
                            pair_found = True
                            break

                    if not pair_found:
                        uncovered_pairs.append((param1, val1, param2, val2))

    if uncovered_pairs:
        print(f"Missing pairs: {uncovered_pairs}")
        return False
    return True

assert verify_pairwise_coverage(PAIRWISE_TESTS), "Not all pairs covered!"
```

**c) Adding Constraints (3 điểm):**

```python
# PICT Input with Constraints
"""
Room: Single, Double, Suite
Board: RoomOnly, Breakfast, HalfBoard, FullBoard
Duration: Short, Medium, Long
Payment: CreditCard, DebitCard, BankTransfer, Cash
Guest: AdultOnly, WithChildren, WithPets

# Constraints (Business Rules)
# Rule 1: Pets only allowed in Suite
IF [Guest] = "WithPets" THEN [Room] = "Suite";

# Rule 2: Children require at least Breakfast
IF [Guest] = "WithChildren" THEN [Board] IN {"Breakfast", "HalfBoard", "FullBoard"};

# Rule 3: Long stays (8+ nights) require non-cash payment
IF [Duration] = "Long" THEN [Payment] <> "Cash";

# Rule 4: Suite with FullBoard requires Credit Card
IF [Room] = "Suite" AND [Board] = "FullBoard" THEN [Payment] = "CreditCard";
"""

# Updated Test Cases after Constraints
CONSTRAINED_PAIRWISE = [
    # Valid combinations respecting constraints
    (1,  'Single', 'RoomOnly',   'Short',  'Cash',        'AdultOnly'),
    (2,  'Single', 'Breakfast',  'Medium', 'DebitCard',   'WithChildren'),  # Children → Breakfast+
    (3,  'Single', 'HalfBoard',  'Long',   'BankTransfer', 'AdultOnly'),     # Long → no Cash
    (4,  'Double', 'Breakfast',  'Short',  'CreditCard',  'WithChildren'),
    (5,  'Double', 'HalfBoard',  'Medium', 'Cash',        'AdultOnly'),
    (6,  'Double', 'FullBoard',  'Long',   'CreditCard',  'AdultOnly'),
    (7,  'Suite',  'RoomOnly',   'Short',  'DebitCard',   'WithPets'),       # Pets → Suite
    (8,  'Suite',  'Breakfast',  'Medium', 'BankTransfer', 'WithPets'),
    (9,  'Suite',  'HalfBoard',  'Long',   'DebitCard',   'WithPets'),
    (10, 'Suite',  'FullBoard',  'Short',  'CreditCard',  'AdultOnly'),      # Suite+FullBoard → CC
    (11, 'Suite',  'FullBoard',  'Medium', 'CreditCard',  'WithChildren'),   # Combined rules
    (12, 'Suite',  'FullBoard',  'Long',   'CreditCard',  'WithPets'),       # All constraints
    # Additional cases to maintain pair coverage
    (13, 'Single', 'FullBoard',  'Short',  'BankTransfer', 'AdultOnly'),
    (14, 'Double', 'RoomOnly',   'Medium', 'DebitCard',   'AdultOnly'),
    (15, 'Double', 'Breakfast',  'Long',   'BankTransfer', 'WithChildren'),
]

# Negative test cases (violating constraints)
CONSTRAINT_VIOLATION_TESTS = [
    # Pets in non-Suite room → Should fail
    {'Room': 'Single', 'Guest': 'WithPets', 'expected': 'ERROR_PETS_SUITE_ONLY'},
    {'Room': 'Double', 'Guest': 'WithPets', 'expected': 'ERROR_PETS_SUITE_ONLY'},

    # Children with RoomOnly → Should fail
    {'Board': 'RoomOnly', 'Guest': 'WithChildren', 'expected': 'ERROR_CHILDREN_NEED_MEALS'},

    # Long stay with Cash → Should fail
    {'Duration': 'Long', 'Payment': 'Cash', 'expected': 'ERROR_LONG_STAY_NO_CASH'},

    # Suite+FullBoard without CreditCard → Should fail
    {'Room': 'Suite', 'Board': 'FullBoard', 'Payment': 'DebitCard', 'expected': 'ERROR_PREMIUM_CC_ONLY'},
]
```

---

## Câu 4: Faker Data Generation (10 điểm)

### Tình huống:
Bạn cần generate test data cho hệ thống quản lý học sinh:
- 1000 học sinh với thông tin: họ tên, ngày sinh (6-18 tuổi), lớp (1-12), điểm TB (0-10)
- 50 giáo viên với: họ tên, email, môn dạy, số năm kinh nghiệm (1-30)
- 12 lớp học với: tên lớp, giáo viên chủ nhiệm, sĩ số

### Yêu cầu:
a) Viết Faker script để generate students data (3 điểm)
b) Viết Faker script để generate teachers với unique emails (3 điểm)
c) Viết script đảm bảo referential integrity giữa các bảng (4 điểm)

### Đáp án mẫu:

**a) Generate Students Data (3 điểm):**

```python
from faker import Faker
from datetime import date, timedelta
import random

fake = Faker('vi_VN')  # Vietnamese locale

def generate_students(count=1000):
    """Generate student data with Vietnamese names"""
    students = []

    for i in range(1, count + 1):
        # Age 6-18 years old
        min_age = 6
        max_age = 18
        today = date.today()

        # Calculate birth date range
        max_birth = today - timedelta(days=min_age * 365)
        min_birth = today - timedelta(days=max_age * 365)

        dob = fake.date_between(start_date=min_birth, end_date=max_birth)

        # Determine grade based on age (rough approximation)
        age = (today - dob).days // 365
        grade = min(12, max(1, age - 5))  # Grade = Age - 5, capped 1-12

        students.append({
            'id': i,
            'ho_ten': fake.name(),  # Vietnamese name
            'ngay_sinh': str(dob),
            'lop': grade,
            'diem_tb': round(random.uniform(0, 10), 2),
            'gioi_tinh': random.choice(['Nam', 'Nữ']),
            'dia_chi': fake.address(),
            'phu_huynh_phone': fake.phone_number(),
        })

    return students

# Generate with specific distribution
def generate_students_by_grade(total=1000):
    """Generate students with even distribution across grades"""
    students = []
    students_per_grade = total // 12

    for grade in range(1, 13):
        for _ in range(students_per_grade):
            # Calculate appropriate age for grade
            age = grade + 5
            dob = fake.date_of_birth(minimum_age=age, maximum_age=age + 1)

            students.append({
                'id': len(students) + 1,
                'ho_ten': fake.name(),
                'ngay_sinh': str(dob),
                'lop': grade,
                'diem_tb': generate_realistic_score(grade),
                'gioi_tinh': random.choice(['Nam', 'Nữ']),
            })

    return students

def generate_realistic_score(grade):
    """Generate score with realistic distribution (normal curve)"""
    # Mean = 6.5, std = 1.5, capped 0-10
    score = random.gauss(6.5, 1.5)
    return round(max(0, min(10, score)), 2)
```

**b) Generate Teachers với Unique Emails (3 điểm):**

```python
def generate_teachers(count=50):
    """Generate teachers with unique emails"""
    teachers = []
    used_emails = set()

    subjects = [
        'Toán', 'Văn', 'Anh', 'Lý', 'Hóa', 'Sinh',
        'Sử', 'Địa', 'GDCD', 'Tin học', 'Thể dục', 'Nhạc', 'Mỹ thuật'
    ]

    for i in range(1, count + 1):
        name = fake.name()

        # Generate unique email from name
        base_email = generate_email_from_name(name)
        email = base_email

        # Handle duplicates
        counter = 1
        while email in used_emails:
            email = f"{base_email.split('@')[0]}{counter}@school.edu.vn"
            counter += 1

        used_emails.add(email)

        teachers.append({
            'id': i,
            'ho_ten': name,
            'email': email,
            'mon_day': random.choice(subjects),
            'so_nam_kinh_nghiem': random.randint(1, 30),
            'phone': fake.phone_number(),
            'trinh_do': random.choice(['Cử nhân', 'Thạc sĩ', 'Tiến sĩ']),
        })

    return teachers

def generate_email_from_name(name):
    """Convert Vietnamese name to email format"""
    import unicodedata
    import re

    # Remove Vietnamese diacritics
    normalized = unicodedata.normalize('NFD', name.lower())
    ascii_name = normalized.encode('ascii', 'ignore').decode('ascii')

    # Split into parts
    parts = ascii_name.split()

    if len(parts) >= 2:
        # Format: firstname.lastname@domain
        email = f"{parts[-1]}.{parts[0]}@school.edu.vn"
    else:
        email = f"{parts[0]}@school.edu.vn"

    return email

# Alternative using Faker's unique feature
def generate_teachers_with_faker_unique(count=50):
    """Use Faker's built-in unique constraint"""
    teachers = []

    for i in range(1, count + 1):
        teachers.append({
            'id': i,
            'ho_ten': fake.name(),
            'email': fake.unique.email(),  # Guaranteed unique
            'mon_day': fake.random_element([
                'Toán', 'Văn', 'Anh', 'Lý', 'Hóa', 'Sinh'
            ]),
            'so_nam_kinh_nghiem': fake.random_int(min=1, max=30),
        })

    # Reset unique cache for next run
    fake.unique.clear()

    return teachers
```

**c) Referential Integrity Script (4 điểm):**

```python
class SchoolDataGenerator:
    """Generate school data with referential integrity"""

    def __init__(self, locale='vi_VN'):
        self.fake = Faker(locale)
        self.teachers = []
        self.classes = []
        self.students = []

    def generate_all(self, num_teachers=50, num_classes=12, num_students=1000):
        """Generate all data maintaining relationships"""
        # Step 1: Generate teachers first (no dependencies)
        self.teachers = self._generate_teachers(num_teachers)

        # Step 2: Generate classes (depends on teachers)
        self.classes = self._generate_classes(num_classes)

        # Step 3: Generate students (depends on classes)
        self.students = self._generate_students(num_students)

        return self._validate_integrity()

    def _generate_teachers(self, count):
        teachers = []
        used_emails = set()

        for i in range(1, count + 1):
            email = self.fake.unique.email()
            used_emails.add(email)

            teachers.append({
                'id': i,
                'ho_ten': self.fake.name(),
                'email': email,
                'mon_day': self.fake.random_element([
                    'Toán', 'Văn', 'Anh', 'Lý', 'Hóa', 'Sinh'
                ]),
                'kinh_nghiem': self.fake.random_int(1, 30),
            })

        return teachers

    def _generate_classes(self, count):
        classes = []

        # Get available teachers for homeroom
        available_teachers = [t['id'] for t in self.teachers]

        for i in range(1, count + 1):
            # Assign homeroom teacher (each teacher can only be homeroom for 1 class)
            if available_teachers:
                homeroom_teacher_id = random.choice(available_teachers)
                available_teachers.remove(homeroom_teacher_id)
            else:
                # If no more unique teachers, allow sharing
                homeroom_teacher_id = random.choice([t['id'] for t in self.teachers])

            classes.append({
                'id': i,
                'ten_lop': f"Lớp {i}",
                'khoi': (i - 1) // 4 + 10,  # Khối 10, 11, 12
                'gvcn_id': homeroom_teacher_id,  # FK to teachers
                'si_so': 0,  # Will be updated after students generated
            })

        return classes

    def _generate_students(self, count):
        students = []

        # Distribute students evenly across classes
        students_per_class = count // len(self.classes)

        for class_obj in self.classes:
            for _ in range(students_per_class):
                student_id = len(students) + 1
                khoi = class_obj['khoi']
                age = khoi + 3  # Khối 10 → 13 tuổi, etc.

                students.append({
                    'id': student_id,
                    'ho_ten': self.fake.name(),
                    'ngay_sinh': str(self.fake.date_of_birth(
                        minimum_age=age, maximum_age=age + 1
                    )),
                    'lop_id': class_obj['id'],  # FK to classes
                    'diem_tb': round(random.gauss(6.5, 1.5), 2),
                })

            # Update class size
            class_obj['si_so'] = students_per_class

        return students

    def _validate_integrity(self):
        """Validate referential integrity"""
        errors = []

        # Check all class.gvcn_id references valid teacher
        teacher_ids = {t['id'] for t in self.teachers}
        for cls in self.classes:
            if cls['gvcn_id'] not in teacher_ids:
                errors.append(f"Class {cls['id']} references invalid teacher {cls['gvcn_id']}")

        # Check all student.lop_id references valid class
        class_ids = {c['id'] for c in self.classes}
        for student in self.students:
            if student['lop_id'] not in class_ids:
                errors.append(f"Student {student['id']} references invalid class {student['lop_id']}")

        # Check class sizes match actual student count
        for cls in self.classes:
            actual_count = sum(1 for s in self.students if s['lop_id'] == cls['id'])
            if actual_count != cls['si_so']:
                errors.append(f"Class {cls['id']}: si_so={cls['si_so']} but has {actual_count} students")

        if errors:
            print("Integrity errors:")
            for e in errors:
                print(f"  - {e}")
            return False

        print("✓ All referential integrity constraints satisfied")
        return True

    def export_json(self, filename='school_data.json'):
        """Export all data to JSON"""
        import json
        data = {
            'teachers': self.teachers,
            'classes': self.classes,
            'students': self.students,
        }
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    def export_sql(self, filename='school_data.sql'):
        """Export as SQL INSERT statements"""
        with open(filename, 'w', encoding='utf-8') as f:
            # Teachers
            for t in self.teachers:
                f.write(f"""INSERT INTO teachers (id, ho_ten, email, mon_day, kinh_nghiem)
VALUES ({t['id']}, '{t['ho_ten']}', '{t['email']}', '{t['mon_day']}', {t['kinh_nghiem']});\n""")

            # Classes
            for c in self.classes:
                f.write(f"""INSERT INTO classes (id, ten_lop, khoi, gvcn_id, si_so)
VALUES ({c['id']}, '{c['ten_lop']}', {c['khoi']}, {c['gvcn_id']}, {c['si_so']});\n""")

            # Students
            for s in self.students:
                f.write(f"""INSERT INTO students (id, ho_ten, ngay_sinh, lop_id, diem_tb)
VALUES ({s['id']}, '{s['ho_ten']}', '{s['ngay_sinh']}', {s['lop_id']}, {s['diem_tb']});\n""")

# Usage
generator = SchoolDataGenerator()
generator.generate_all(num_teachers=50, num_classes=12, num_students=1000)
generator.export_json()
generator.export_sql()
```

---

## Câu 5: Decision Table Testing (10 điểm)

### Tình huống:
Bạn đang test chức năng tính phí đăng ký thẻ tín dụng:
- Loại thẻ: Standard, Gold, Platinum
- Thu nhập: < 10tr, 10-30tr, > 30tr
- Lịch sử tín dụng: Tốt, Trung bình, Xấu
- Kết quả: Phê duyệt với phí, Yêu cầu bổ sung hồ sơ, Từ chối

Business Rules:
1. Standard + Thu nhập >= 10tr + Lịch sử Tốt/TB → Phê duyệt (phí 500k)
2. Gold + Thu nhập >= 20tr + Lịch sử Tốt → Phê duyệt (phí 1tr)
3. Platinum + Thu nhập > 30tr + Lịch sử Tốt → Phê duyệt (phí 2tr)
4. Lịch sử Xấu → Từ chối
5. Các trường hợp khác → Yêu cầu bổ sung

### Yêu cầu:
a) Xây dựng Decision Table đầy đủ (4 điểm)
b) Giản lược Decision Table (3 điểm)
c) Thiết kế test cases từ Decision Table (3 điểm)

### Đáp án mẫu:

**a) Full Decision Table (4 điểm):**

```
CONDITIONS                          | RULES
                                    | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | ...
------------------------------------|----|----|----|----|----|----|----|----|----|-
Loại thẻ = Standard                 | Y  | Y  | Y  | Y  | Y  | Y  | N  | N  | N  |
Loại thẻ = Gold                     | N  | N  | N  | N  | N  | N  | Y  | Y  | Y  |
Loại thẻ = Platinum                 | N  | N  | N  | N  | N  | N  | N  | N  | N  |
Thu nhập < 10tr                     | Y  | Y  | Y  | N  | N  | N  | Y  | Y  | Y  |
Thu nhập 10-30tr                    | N  | N  | N  | Y  | Y  | Y  | N  | N  | N  |
Thu nhập > 30tr                     | N  | N  | N  | N  | N  | N  | N  | N  | N  |
Lịch sử = Tốt                       | Y  | N  | N  | Y  | N  | N  | Y  | N  | N  |
Lịch sử = Trung bình                | N  | Y  | N  | N  | Y  | N  | N  | Y  | N  |
Lịch sử = Xấu                       | N  | N  | Y  | N  | N  | Y  | N  | N  | Y  |
------------------------------------|----|----|----|----|----|----|----|----|----|-
ACTIONS                             |    |    |    |    |    |    |    |    |    |
Phê duyệt (phí 500k)                |    |    |    | X  | X  |    |    |    |    |
Phê duyệt (phí 1tr)                 |    |    |    |    |    |    |    |    |    |
Phê duyệt (phí 2tr)                 |    |    |    |    |    |    |    |    |    |
Yêu cầu bổ sung                     | X  | X  |    |    |    |    | X  | X  |    |
Từ chối                             |    |    | X  |    |    | X  |    |    | X  |
```

**Complete Table (27 rules = 3 × 3 × 3):**

| Rule | Card | Income | History | Action |
|------|------|--------|---------|--------|
| R1 | Standard | <10tr | Good | Request Docs |
| R2 | Standard | <10tr | Medium | Request Docs |
| R3 | Standard | <10tr | Bad | Reject |
| R4 | Standard | 10-30tr | Good | Approve 500k |
| R5 | Standard | 10-30tr | Medium | Approve 500k |
| R6 | Standard | 10-30tr | Bad | Reject |
| R7 | Standard | >30tr | Good | Approve 500k |
| R8 | Standard | >30tr | Medium | Approve 500k |
| R9 | Standard | >30tr | Bad | Reject |
| R10 | Gold | <10tr | Good | Request Docs |
| R11 | Gold | <10tr | Medium | Request Docs |
| R12 | Gold | <10tr | Bad | Reject |
| R13 | Gold | 10-30tr | Good | Request Docs |
| R14 | Gold | 10-30tr | Medium | Request Docs |
| R15 | Gold | 10-30tr | Bad | Reject |
| R16 | Gold | >30tr | Good | Approve 1tr |
| R17 | Gold | >30tr | Medium | Request Docs |
| R18 | Gold | >30tr | Bad | Reject |
| R19 | Platinum | <10tr | Good | Request Docs |
| R20 | Platinum | <10tr | Medium | Request Docs |
| R21 | Platinum | <10tr | Bad | Reject |
| R22 | Platinum | 10-30tr | Good | Request Docs |
| R23 | Platinum | 10-30tr | Medium | Request Docs |
| R24 | Platinum | 10-30tr | Bad | Reject |
| R25 | Platinum | >30tr | Good | Approve 2tr |
| R26 | Platinum | >30tr | Medium | Request Docs |
| R27 | Platinum | >30tr | Bad | Reject |

**b) Simplified Decision Table (3 điểm):**

```
# Observation 1: Bad history always → Reject (regardless of other conditions)
# Observation 2: Can collapse by card type and outcome

SIMPLIFIED TABLE:
=================

| # | Conditions | Action |
|---|------------|--------|
| 1 | History = Bad | Reject |
| 2 | Standard + Income >= 10tr + History ∈ {Good, Medium} | Approve 500k |
| 3 | Gold + Income > 30tr + History = Good | Approve 1tr |
| 4 | Platinum + Income > 30tr + History = Good | Approve 2tr |
| 5 | Otherwise (remaining valid combinations) | Request Docs |

CODE REPRESENTATION:
```python
def evaluate_credit_card_application(card_type, income, history):
    # Rule 1: Bad history → Reject (highest priority)
    if history == 'Bad':
        return ('Reject', 0)

    # Rule 2: Standard card approval
    if card_type == 'Standard' and income >= 10_000_000:
        return ('Approve', 500_000)

    # Rule 3: Gold card approval
    if card_type == 'Gold' and income > 30_000_000 and history == 'Good':
        return ('Approve', 1_000_000)

    # Rule 4: Platinum card approval
    if card_type == 'Platinum' and income > 30_000_000 and history == 'Good':
        return ('Approve', 2_000_000)

    # Rule 5: Default → Request additional documents
    return ('RequestDocs', 0)
```

**c) Test Cases from Decision Table (3 điểm):**

```python
# Each test case covers one rule/condition combination
TEST_CASES = [
    # ID, Card, Income, History, Expected Action, Expected Fee

    # REJECT cases (all bad history)
    ('TC01', 'Standard', 5_000_000, 'Bad', 'Reject', 0),
    ('TC02', 'Gold', 25_000_000, 'Bad', 'Reject', 0),
    ('TC03', 'Platinum', 50_000_000, 'Bad', 'Reject', 0),

    # APPROVE Standard (500k)
    ('TC04', 'Standard', 10_000_000, 'Good', 'Approve', 500_000),   # BVA: min income
    ('TC05', 'Standard', 15_000_000, 'Medium', 'Approve', 500_000), # ECP: medium history
    ('TC06', 'Standard', 50_000_000, 'Good', 'Approve', 500_000),   # High income

    # APPROVE Gold (1tr)
    ('TC07', 'Gold', 30_000_001, 'Good', 'Approve', 1_000_000),     # BVA: just above 30tr
    ('TC08', 'Gold', 50_000_000, 'Good', 'Approve', 1_000_000),     # ECP: high income

    # APPROVE Platinum (2tr)
    ('TC09', 'Platinum', 30_000_001, 'Good', 'Approve', 2_000_000), # BVA: just above 30tr
    ('TC10', 'Platinum', 100_000_000, 'Good', 'Approve', 2_000_000),# ECP: very high

    # REQUEST DOCS cases
    ('TC11', 'Standard', 5_000_000, 'Good', 'RequestDocs', 0),      # Low income
    ('TC12', 'Standard', 9_999_999, 'Good', 'RequestDocs', 0),      # BVA: just below 10tr
    ('TC13', 'Gold', 20_000_000, 'Good', 'RequestDocs', 0),         # Gold but income 10-30tr
    ('TC14', 'Gold', 30_000_000, 'Good', 'RequestDocs', 0),         # BVA: exactly 30tr (not >30tr)
    ('TC15', 'Gold', 50_000_000, 'Medium', 'RequestDocs', 0),       # Good income but medium history
    ('TC16', 'Platinum', 25_000_000, 'Good', 'RequestDocs', 0),     # Platinum but income < 30tr
    ('TC17', 'Platinum', 50_000_000, 'Medium', 'RequestDocs', 0),   # Good income but medium history
]

@pytest.mark.parametrize("tc_id,card,income,history,exp_action,exp_fee", TEST_CASES)
def test_credit_card_evaluation(tc_id, card, income, history, exp_action, exp_fee):
    action, fee = evaluate_credit_card_application(card, income, history)
    assert action == exp_action, f"{tc_id}: Expected {exp_action}, got {action}"
    assert fee == exp_fee, f"{tc_id}: Expected fee {exp_fee}, got {fee}"
```

---

## Câu 6: Mockaroo Data Generation (10 điểm)

### Tình huống:
Bạn cần generate 10,000 records test data cho hệ thống đặt hàng online sử dụng Mockaroo với các yêu cầu:
- Orders: order_id, customer_id, order_date, total_amount, status
- Status distribution: 60% completed, 25% processing, 10% pending, 5% cancelled
- Total amount: $10-$5000 with realistic distribution (more orders at lower amounts)

### Yêu cầu:
a) Thiết kế Mockaroo schema với formulas (4 điểm)
b) Giải thích cách tạo realistic distribution (3 điểm)
c) So sánh Mockaroo vs Faker cho use case này (3 điểm)

### Đáp án mẫu:

**a) Mockaroo Schema Design (4 điểm):**

```
MOCKAROO SCHEMA:
================

Field Name      | Type              | Formula/Options
----------------|-------------------|------------------
order_id        | Row Number        | -
customer_id     | Number            | Min: 1, Max: 5000
order_date      | Date              | Min: 01/01/2024, Max: 12/31/2024
total_amount    | Custom Formula    | See formula below
status          | Custom List       | Weighted distribution
shipping_addr   | Full Address      | -
payment_method  | Custom List       | Credit Card, PayPal, Bank Transfer

FORMULAS:
=========

# total_amount - Exponential distribution (more low values)
= round(10 + (random(0,1) ^ 0.3) * 4990, 2)

# Explanation:
# random(0,1)^0.3 creates right-skewed distribution
# More values near 0, fewer near 1
# Scale to $10-$5000 range

# status - Weighted random
= random_weighted("completed:60,processing:25,pending:10,cancelled:5")

# Alternative using if/else:
= if(random(0,100) <= 60, "completed",
    if(random(0,100) <= 85, "processing",
      if(random(0,100) <= 95, "pending", "cancelled")))
```

**Mockaroo API Usage:**

```python
import requests

MOCKAROO_API_KEY = "your_api_key"

schema = {
    "fields": [
        {"name": "order_id", "type": "Row Number"},
        {"name": "customer_id", "type": "Number", "min": 1, "max": 5000},
        {"name": "order_date", "type": "Date", "min": "01/01/2024", "max": "12/31/2024"},
        {
            "name": "total_amount",
            "type": "Formula",
            "formula": "round(10 + (random(0,1) ^ 0.3) * 4990, 2)"
        },
        {
            "name": "status",
            "type": "Custom List",
            "values": ["completed"] * 60 + ["processing"] * 25 +
                      ["pending"] * 10 + ["cancelled"] * 5
        },
        {"name": "shipping_addr", "type": "Full Address"},
        {
            "name": "payment_method",
            "type": "Custom List",
            "values": ["Credit Card", "PayPal", "Bank Transfer"]
        }
    ]
}

# Generate via API
response = requests.post(
    f"https://api.mockaroo.com/api/generate.json?key={MOCKAROO_API_KEY}&count=10000",
    json=schema
)

orders = response.json()
```

**b) Realistic Distribution Explanation (3 điểm):**

```python
# PROBLEM: Linear random gives uniform distribution
# uniform(10, 5000) → Equal probability for $50 and $4500
# Reality: Most orders are smaller amounts

# SOLUTION 1: Exponential distribution
import numpy as np

def generate_realistic_amounts(n=10000, min_val=10, max_val=5000):
    """
    Use exponential distribution for realistic e-commerce order amounts

    Most e-commerce sites see:
    - 50% of orders under $100
    - 30% of orders $100-$500
    - 15% of orders $500-$2000
    - 5% of orders > $2000
    """
    # Exponential: many low values, few high values
    # Lambda controls how quickly values decrease
    lambda_param = 0.003  # Tuned for desired distribution

    amounts = []
    for _ in range(n):
        # Generate exponential value
        raw = np.random.exponential(1/lambda_param)
        # Clip to range and add minimum
        amount = min(max_val, max(min_val, raw + min_val))
        amounts.append(round(amount, 2))

    return amounts

# SOLUTION 2: Power law (Pareto principle)
def generate_pareto_amounts(n=10000, min_val=10, max_val=5000):
    """
    Use power law: 80% of orders are 20% of total value
    """
    alpha = 1.5  # Shape parameter (lower = more skewed)

    amounts = []
    for _ in range(n):
        # Pareto distribution
        raw = min_val * (1 - np.random.random()) ** (-1/alpha)
        amount = min(max_val, raw)
        amounts.append(round(amount, 2))

    return amounts

# SOLUTION 3: Weighted buckets (most controllable)
def generate_bucketed_amounts(n=10000):
    """
    Explicitly control distribution by buckets
    """
    buckets = [
        (0.50, 10, 100),     # 50% of orders: $10-$100
        (0.30, 100, 500),    # 30% of orders: $100-$500
        (0.15, 500, 2000),   # 15% of orders: $500-$2000
        (0.05, 2000, 5000),  # 5% of orders: $2000-$5000
    ]

    amounts = []
    for _ in range(n):
        # Select bucket based on weights
        r = np.random.random()
        cumulative = 0
        for weight, low, high in buckets:
            cumulative += weight
            if r <= cumulative:
                amount = np.random.uniform(low, high)
                amounts.append(round(amount, 2))
                break

    return amounts

# Verify distribution
amounts = generate_bucketed_amounts(10000)
print(f"Under $100: {sum(1 for a in amounts if a < 100) / 100:.1f}%")
print(f"$100-$500: {sum(1 for a in amounts if 100 <= a < 500) / 100:.1f}%")
print(f"$500-$2000: {sum(1 for a in amounts if 500 <= a < 2000) / 100:.1f}%")
print(f"Over $2000: {sum(1 for a in amounts if a >= 2000) / 100:.1f}%")
```

**c) Mockaroo vs Faker Comparison (3 điểm):**

| Aspect | Mockaroo | Faker |
|--------|----------|-------|
| **Setup** | None (web-based) | pip install |
| **Learning Curve** | Low (UI-based) | Medium (code) |
| **Max Records Free** | 1,000 | Unlimited |
| **Paid Tier** | Up to 10M | N/A |
| **Distribution Control** | Formulas | Full code control |
| **Relationships** | Limited (same row) | Full (any logic) |
| **Reproducibility** | API with seed | seed parameter |
| **CI/CD Integration** | Via API | Native (code) |
| **Custom Types** | Formulas | Custom providers |
| **Performance** | Server-side | Local (faster) |

**Recommendation:**

```python
# USE MOCKAROO WHEN:
# - Quick prototyping (< 1,000 records)
# - Non-developers need to generate data
# - Standard data types (names, addresses, emails)
# - One-time data generation

# USE FAKER WHEN:
# - CI/CD pipeline integration
# - Complex relationships between entities
# - Custom data distributions
# - Large volumes (> 10,000 records)
# - Need reproducibility with seeds

# FOR THIS USE CASE (10,000 orders with distributions):
# → FAKER is better because:
#   1. 10,000 records exceeds Mockaroo free tier
#   2. Custom distribution logic easier in code
#   3. Can integrate with test framework
#   4. Reproducible with seed

# Faker implementation for this use case:
from faker import Faker
import random

fake = Faker()
Faker.seed(12345)  # Reproducibility

def generate_orders(n=10000):
    orders = []
    status_weights = ['completed'] * 60 + ['processing'] * 25 + \
                     ['pending'] * 10 + ['cancelled'] * 5

    for i in range(1, n + 1):
        orders.append({
            'order_id': i,
            'customer_id': random.randint(1, 5000),
            'order_date': str(fake.date_between(start_date='-1y', end_date='today')),
            'total_amount': generate_realistic_amount(),
            'status': random.choice(status_weights),
            'shipping_addr': fake.address(),
            'payment_method': random.choice(['Credit Card', 'PayPal', 'Bank Transfer']),
        })

    return orders

def generate_realistic_amount():
    """Bucketed distribution for realistic order amounts"""
    r = random.random()
    if r < 0.50:
        return round(random.uniform(10, 100), 2)
    elif r < 0.80:
        return round(random.uniform(100, 500), 2)
    elif r < 0.95:
        return round(random.uniform(500, 2000), 2)
    else:
        return round(random.uniform(2000, 5000), 2)
```

---

## Câu 7: State Transition Testing (10 điểm)

### Tình huống:
Bạn đang test hệ thống quản lý đơn hàng với các trạng thái:
- DRAFT → SUBMITTED → CONFIRMED → SHIPPED → DELIVERED
- Có thể CANCELLED từ: DRAFT, SUBMITTED, CONFIRMED
- Có thể RETURNED từ: DELIVERED (trong 7 ngày)

### Yêu cầu:
a) Vẽ State Transition Diagram (3 điểm)
b) Xây dựng State Transition Table (3 điểm)
c) Thiết kế test cases bao gồm valid và invalid transitions (4 điểm)

### Đáp án mẫu:

**a) State Transition Diagram (3 điểm):**

```
                        ┌─────────────────────────────────────────┐
                        │                                          │
                        ▼                                          │
    ┌──────────┐  submit   ┌───────────┐  confirm   ┌───────────┐ │
    │  DRAFT   │──────────▶│ SUBMITTED │───────────▶│ CONFIRMED │ │
    └──────────┘           └───────────┘            └───────────┘ │
         │                       │                        │       │
         │ cancel                │ cancel                 │ cancel│
         │                       │                        │       │
         ▼                       ▼                        ▼       │
    ┌────────────────────────────────────────────────────────┐   │
    │                      CANCELLED                          │   │
    └────────────────────────────────────────────────────────┘   │
                                                                  │
                        ┌───────────┐   ship    ┌───────────┐    │
                        │ CONFIRMED │──────────▶│  SHIPPED  │    │
                        └───────────┘           └───────────┘    │
                                                      │          │
                                                      │ deliver  │
                                                      ▼          │
                        ┌───────────┐  return   ┌───────────┐    │
                        │  RETURNED │◀──────────│ DELIVERED │────┘
                        └───────────┘   (≤7d)   └───────────┘

LEGEND:
═══════
→  Valid transition
States: DRAFT, SUBMITTED, CONFIRMED, SHIPPED, DELIVERED, CANCELLED, RETURNED
```

**b) State Transition Table (3 điểm):**

```
STATE TRANSITION TABLE:
=======================

Current State  | Event      | Next State  | Conditions         | Actions
---------------|------------|-------------|--------------------|-----------------
DRAFT          | submit     | SUBMITTED   | -                  | Validate order
DRAFT          | cancel     | CANCELLED   | -                  | Log cancellation
SUBMITTED      | confirm    | CONFIRMED   | Payment verified   | Reserve inventory
SUBMITTED      | cancel     | CANCELLED   | -                  | Refund if paid
CONFIRMED      | ship       | SHIPPED     | Inventory available| Generate tracking
CONFIRMED      | cancel     | CANCELLED   | -                  | Refund + release inv
SHIPPED        | deliver    | DELIVERED   | Received by customer| Update tracking
DELIVERED      | return     | RETURNED    | Within 7 days      | Create RMA
CANCELLED      | -          | -           | Terminal state     | -
RETURNED       | -          | -           | Terminal state     | -

INVALID TRANSITIONS (should raise error):
=========================================

Current State  | Event      | Expected Error
---------------|------------|-----------------------------
DRAFT          | confirm    | "Cannot confirm draft order"
DRAFT          | ship       | "Cannot ship draft order"
DRAFT          | deliver    | "Cannot deliver draft order"
DRAFT          | return     | "Cannot return draft order"
SUBMITTED      | ship       | "Order not confirmed yet"
SUBMITTED      | deliver    | "Order not shipped yet"
SUBMITTED      | return     | "Order not delivered yet"
CONFIRMED      | deliver    | "Order not shipped yet"
CONFIRMED      | return     | "Order not delivered yet"
SHIPPED        | confirm    | "Order already confirmed"
SHIPPED        | cancel     | "Cannot cancel shipped order"
SHIPPED        | return     | "Order not delivered yet"
DELIVERED      | cancel     | "Cannot cancel delivered order"
DELIVERED      | return     | "Return period expired" (> 7 days)
CANCELLED      | (any)      | "Order is cancelled"
RETURNED       | (any)      | "Order is returned"
```

**c) Test Cases (4 điểm):**

```python
import pytest
from datetime import datetime, timedelta

class TestOrderStateTransitions:
    """Test all valid and invalid state transitions"""

    # VALID TRANSITIONS (Happy Path)
    VALID_TRANSITIONS = [
        ('DRAFT', 'submit', 'SUBMITTED'),
        ('DRAFT', 'cancel', 'CANCELLED'),
        ('SUBMITTED', 'confirm', 'CONFIRMED'),
        ('SUBMITTED', 'cancel', 'CANCELLED'),
        ('CONFIRMED', 'ship', 'SHIPPED'),
        ('CONFIRMED', 'cancel', 'CANCELLED'),
        ('SHIPPED', 'deliver', 'DELIVERED'),
        ('DELIVERED', 'return', 'RETURNED'),  # Within 7 days
    ]

    @pytest.mark.parametrize("current,event,expected", VALID_TRANSITIONS)
    def test_valid_transition(self, current, event, expected):
        """Test all valid state transitions"""
        order = Order(state=current)

        if event == 'return':
            # Set delivered date within return period
            order.delivered_at = datetime.now() - timedelta(days=3)

        getattr(order, event)()  # Call method dynamically

        assert order.state == expected, \
            f"Expected {current} --{event}--> {expected}, got {order.state}"

    # INVALID TRANSITIONS
    INVALID_TRANSITIONS = [
        # Can't skip states
        ('DRAFT', 'confirm', 'Cannot confirm draft order'),
        ('DRAFT', 'ship', 'Cannot ship draft order'),
        ('DRAFT', 'deliver', 'Cannot deliver draft order'),
        ('SUBMITTED', 'ship', 'Order not confirmed'),
        ('SUBMITTED', 'deliver', 'Order not shipped'),
        ('CONFIRMED', 'deliver', 'Order not shipped'),

        # Can't go backward
        ('CONFIRMED', 'submit', 'Invalid transition'),
        ('SHIPPED', 'confirm', 'Order already confirmed'),

        # Can't cancel after shipped
        ('SHIPPED', 'cancel', 'Cannot cancel shipped order'),
        ('DELIVERED', 'cancel', 'Cannot cancel delivered order'),

        # Terminal states
        ('CANCELLED', 'submit', 'Order is cancelled'),
        ('CANCELLED', 'confirm', 'Order is cancelled'),
        ('RETURNED', 'submit', 'Order is returned'),
    ]

    @pytest.mark.parametrize("current,event,error_msg", INVALID_TRANSITIONS)
    def test_invalid_transition(self, current, event, error_msg):
        """Test all invalid state transitions raise appropriate errors"""
        order = Order(state=current)

        with pytest.raises(InvalidStateTransition) as exc_info:
            getattr(order, event)()

        assert error_msg.lower() in str(exc_info.value).lower()
        assert order.state == current, "State should remain unchanged"

    # CONDITIONAL TRANSITIONS
    def test_return_within_period(self):
        """Return allowed within 7 days of delivery"""
        order = Order(state='DELIVERED')
        order.delivered_at = datetime.now() - timedelta(days=5)

        order.return_order()

        assert order.state == 'RETURNED'

    def test_return_after_period_fails(self):
        """Return rejected after 7 days"""
        order = Order(state='DELIVERED')
        order.delivered_at = datetime.now() - timedelta(days=10)

        with pytest.raises(InvalidStateTransition) as exc_info:
            order.return_order()

        assert "period expired" in str(exc_info.value).lower()
        assert order.state == 'DELIVERED'

    def test_return_at_boundary(self):
        """BVA: Return at exactly 7 days"""
        order = Order(state='DELIVERED')
        order.delivered_at = datetime.now() - timedelta(days=7)

        order.return_order()  # Should succeed at boundary

        assert order.state == 'RETURNED'

    def test_return_just_after_boundary(self):
        """BVA: Return at 7 days + 1 second fails"""
        order = Order(state='DELIVERED')
        order.delivered_at = datetime.now() - timedelta(days=7, seconds=1)

        with pytest.raises(InvalidStateTransition):
            order.return_order()

    # FULL PATH TESTS
    def test_complete_happy_path(self):
        """Test complete order lifecycle"""
        order = Order()

        assert order.state == 'DRAFT'

        order.submit()
        assert order.state == 'SUBMITTED'

        order.confirm()
        assert order.state == 'CONFIRMED'

        order.ship()
        assert order.state == 'SHIPPED'

        order.deliver()
        assert order.state == 'DELIVERED'

    def test_cancel_from_each_allowed_state(self):
        """Test cancellation from all allowed states"""
        for start_state in ['DRAFT', 'SUBMITTED', 'CONFIRMED']:
            order = Order(state=start_state)
            order.cancel()
            assert order.state == 'CANCELLED'

    def test_state_history_tracking(self):
        """Verify state transitions are logged"""
        order = Order()

        order.submit()
        order.confirm()
        order.ship()

        expected_history = [
            ('DRAFT', 'SUBMITTED', 'submit'),
            ('SUBMITTED', 'CONFIRMED', 'confirm'),
            ('CONFIRMED', 'SHIPPED', 'ship'),
        ]

        assert len(order.state_history) == 3
        for i, (from_state, to_state, event) in enumerate(expected_history):
            assert order.state_history[i]['from'] == from_state
            assert order.state_history[i]['to'] == to_state
            assert order.state_history[i]['event'] == event
```

---

## Câu 8: Test Data Seeding Strategy (10 điểm)

### Tình huống:
Bạn cần setup test data cho integration tests của hệ thống e-commerce với yêu cầu:
- Mỗi test phải có data độc lập
- Data phải realistic
- Tests có thể chạy parallel
- Cleanup sau mỗi test

### Yêu cầu:
a) Thiết kế test data factory pattern (4 điểm)
b) Implement data isolation strategy (3 điểm)
c) Viết cleanup mechanism (3 điểm)

### Đáp án mẫu:

**a) Test Data Factory Pattern (4 điểm):**

```python
from faker import Faker
from dataclasses import dataclass, field
from typing import Optional, List
import uuid

fake = Faker()

@dataclass
class UserFactory:
    """Factory for creating test users"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    email: str = field(default_factory=lambda: fake.unique.email())
    name: str = field(default_factory=fake.name)
    password: str = "TestPassword123!"
    role: str = "customer"

    @classmethod
    def create(cls, **overrides):
        """Create user with optional overrides"""
        defaults = cls()
        for key, value in overrides.items():
            setattr(defaults, key, value)
        return defaults

    @classmethod
    def create_admin(cls, **overrides):
        """Convenience method for admin users"""
        return cls.create(role="admin", **overrides)

    def save(self, db):
        """Persist to database"""
        db.execute("""
            INSERT INTO users (id, email, name, password_hash, role)
            VALUES (?, ?, ?, ?, ?)
        """, (self.id, self.email, self.name, hash_password(self.password), self.role))
        return self


@dataclass
class ProductFactory:
    """Factory for creating test products"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    name: str = field(default_factory=fake.catch_phrase)
    price: float = field(default_factory=lambda: round(fake.pyfloat(min_value=10, max_value=1000), 2))
    stock: int = field(default_factory=lambda: fake.random_int(min=0, max=100))
    category_id: Optional[str] = None

    @classmethod
    def create(cls, **overrides):
        return cls(**{**cls().__dict__, **overrides})

    @classmethod
    def create_in_stock(cls, quantity=10, **overrides):
        """Create product with guaranteed stock"""
        return cls.create(stock=quantity, **overrides)

    @classmethod
    def create_out_of_stock(cls, **overrides):
        """Create product with zero stock"""
        return cls.create(stock=0, **overrides)

    def save(self, db):
        db.execute("""
            INSERT INTO products (id, name, price, stock, category_id)
            VALUES (?, ?, ?, ?, ?)
        """, (self.id, self.name, self.price, self.stock, self.category_id))
        return self


@dataclass
class OrderFactory:
    """Factory for creating test orders"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    user_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    status: str = "pending"
    total: float = 0.0
    items: List = field(default_factory=list)

    @classmethod
    def create(cls, **overrides):
        return cls(**{**cls().__dict__, **overrides})

    @classmethod
    def create_with_items(cls, user, products, quantities=None, **overrides):
        """Create order with items from products list"""
        if quantities is None:
            quantities = [1] * len(products)

        order = cls.create(user_id=user.id, **overrides)
        total = 0

        for product, qty in zip(products, quantities):
            order.items.append({
                'product_id': product.id,
                'quantity': qty,
                'price': product.price,
            })
            total += product.price * qty

        order.total = round(total, 2)
        return order

    def save(self, db):
        db.execute("""
            INSERT INTO orders (id, user_id, status, total)
            VALUES (?, ?, ?, ?)
        """, (self.id, self.user_id, self.status, self.total))

        for item in self.items:
            db.execute("""
                INSERT INTO order_items (order_id, product_id, quantity, price)
                VALUES (?, ?, ?, ?)
            """, (self.id, item['product_id'], item['quantity'], item['price']))

        return self


# Master Factory - combines all factories
class TestDataFactory:
    """Central factory for all test data"""

    def __init__(self, db):
        self.db = db
        self.created_records = []  # Track for cleanup

    def create_user(self, **kwargs):
        user = UserFactory.create(**kwargs)
        user.save(self.db)
        self.created_records.append(('users', user.id))
        return user

    def create_product(self, **kwargs):
        product = ProductFactory.create(**kwargs)
        product.save(self.db)
        self.created_records.append(('products', product.id))
        return product

    def create_order(self, user=None, products=None, **kwargs):
        if user is None:
            user = self.create_user()
        if products is None:
            products = [self.create_product()]

        order = OrderFactory.create_with_items(user, products, **kwargs)
        order.save(self.db)
        self.created_records.append(('orders', order.id))
        return order

    def cleanup(self):
        """Delete all created records in reverse order"""
        for table, record_id in reversed(self.created_records):
            self.db.execute(f"DELETE FROM {table} WHERE id = ?", (record_id,))
        self.created_records.clear()
```

**b) Data Isolation Strategy (3 điểm):**

```python
import pytest
import threading
from contextlib import contextmanager

class TestDatabase:
    """Thread-safe test database with isolation"""

    _local = threading.local()

    @classmethod
    def get_connection(cls):
        """Get thread-local database connection"""
        if not hasattr(cls._local, 'connection'):
            cls._local.connection = create_connection()
        return cls._local.connection

    @classmethod
    @contextmanager
    def transaction(cls):
        """Transaction-based isolation"""
        conn = cls.get_connection()
        try:
            conn.begin()
            yield conn
            conn.rollback()  # Always rollback for test isolation
        except:
            conn.rollback()
            raise


class IsolatedTestCase:
    """Base class for isolated tests"""

    @pytest.fixture(autouse=True)
    def setup_isolation(self, db):
        """Each test runs in its own transaction"""
        self.factory = TestDataFactory(db)

        # Start transaction
        db.begin()

        yield

        # Rollback (or cleanup + commit)
        db.rollback()

    # Alternative: Use unique prefixes for test data
    @pytest.fixture
    def unique_prefix(self):
        """Generate unique prefix for this test run"""
        import time
        return f"test_{int(time.time() * 1000)}_{threading.current_thread().name}_"


# Using unique identifiers for parallel test isolation
class ParallelSafeFactory:
    """Factory that ensures unique data across parallel tests"""

    def __init__(self, db, test_id=None):
        self.db = db
        self.test_id = test_id or str(uuid.uuid4())[:8]
        self.created_ids = []

    def create_user(self, **kwargs):
        # Ensure unique email even in parallel execution
        if 'email' not in kwargs:
            kwargs['email'] = f"{self.test_id}_{fake.unique.user_name()}@test.com"

        user = UserFactory.create(**kwargs)
        user.save(self.db)
        self.created_ids.append(('users', user.id))
        return user

    def create_product(self, **kwargs):
        # Prefix product name with test_id
        if 'name' not in kwargs:
            kwargs['name'] = f"[{self.test_id}] {fake.catch_phrase()}"

        product = ProductFactory.create(**kwargs)
        product.save(self.db)
        self.created_ids.append(('products', product.id))
        return product


# Pytest fixtures for isolation
@pytest.fixture
def isolated_db():
    """Provide isolated database for each test"""
    with TestDatabase.transaction() as db:
        yield db


@pytest.fixture
def factory(isolated_db):
    """Provide factory with automatic cleanup"""
    f = TestDataFactory(isolated_db)
    yield f
    f.cleanup()


# Example test using isolation
class TestOrderCreation:
    def test_create_order_success(self, factory):
        # Data created here is isolated to this test
        user = factory.create_user()
        product = factory.create_product(price=100, stock=10)

        order = factory.create_order(user=user, products=[product])

        assert order.total == 100
        assert order.status == 'pending'
        # After test, all data is rolled back/cleaned up
```

**c) Cleanup Mechanism (3 điểm):**

```python
class TestDataCleaner:
    """Comprehensive cleanup mechanism"""

    def __init__(self, db):
        self.db = db
        self.cleanup_registry = []

    def register(self, table, record_id, priority=0):
        """Register record for cleanup with priority (higher = cleanup first)"""
        self.cleanup_registry.append({
            'table': table,
            'id': record_id,
            'priority': priority
        })

    def cleanup_all(self):
        """Clean up all registered records respecting FK constraints"""
        # Sort by priority (higher first) to handle FK constraints
        sorted_records = sorted(
            self.cleanup_registry,
            key=lambda x: x['priority'],
            reverse=True
        )

        for record in sorted_records:
            try:
                self.db.execute(
                    f"DELETE FROM {record['table']} WHERE id = ?",
                    (record['id'],)
                )
            except Exception as e:
                print(f"Warning: Failed to delete {record['table']}.{record['id']}: {e}")

        self.cleanup_registry.clear()

    def cleanup_by_prefix(self, prefix):
        """Clean up all records with given prefix in any text field"""
        tables = ['users', 'products', 'orders', 'order_items']

        for table in tables:
            self.db.execute(f"""
                DELETE FROM {table}
                WHERE id LIKE ?
                OR name LIKE ?
                OR email LIKE ?
            """, (f"{prefix}%", f"%{prefix}%", f"%{prefix}%"))


# Pytest cleanup fixtures
@pytest.fixture(scope="function")
def clean_factory(db):
    """Factory with automatic cleanup after each test"""
    factory = TestDataFactory(db)

    yield factory

    # Cleanup in correct order (respecting FK constraints)
    cleanup_order = ['order_items', 'orders', 'products', 'users']

    for table, record_id in factory.created_records:
        # Will be cleaned in proper order
        pass

    # Actual cleanup
    for table in cleanup_order:
        ids = [r[1] for r in factory.created_records if r[0] == table]
        if ids:
            placeholders = ','.join(['?'] * len(ids))
            db.execute(f"DELETE FROM {table} WHERE id IN ({placeholders})", ids)


@pytest.fixture(scope="session", autouse=True)
def global_cleanup(db):
    """Clean up any orphaned test data at end of test session"""
    yield

    # Clean up any records with 'test_' prefix that might have been left
    db.execute("DELETE FROM order_items WHERE order_id LIKE 'test_%'")
    db.execute("DELETE FROM orders WHERE id LIKE 'test_%'")
    db.execute("DELETE FROM products WHERE name LIKE '[test_%'")
    db.execute("DELETE FROM users WHERE email LIKE 'test_%'")


# Context manager for automatic cleanup
@contextmanager
def test_data_scope(db):
    """Context manager ensuring cleanup on exit"""
    factory = TestDataFactory(db)

    try:
        yield factory
    finally:
        factory.cleanup()


# Usage example
def test_order_workflow():
    with test_data_scope(db) as factory:
        user = factory.create_user()
        product = factory.create_product(stock=10)
        order = factory.create_order(user, [product])

        # Test logic here
        assert order.total > 0

    # Data automatically cleaned up here
```

---

## Câu 9: Edge Case Identification (10 điểm)

### Tình huống:
Bạn đang test form nhập số điện thoại quốc tế với yêu cầu:
- Hỗ trợ format: +[country_code][number]
- Country code: 1-3 digits
- Number: 4-14 digits
- Có thể có spaces, dashes, parentheses

### Yêu cầu:
a) Liệt kê tất cả edge cases có thể (4 điểm)
b) Phân loại edge cases theo category (3 điểm)
c) Thiết kế test cases cho các edge cases quan trọng nhất (3 điểm)

### Đáp án mẫu:

**a) Edge Cases Identification (4 điểm):**

```
EDGE CASES BY COMPONENT:
========================

COUNTRY CODE (+1 to +999):
--------------------------
1. Minimum: +1 (1 digit)
2. Maximum: +999 (3 digits)
3. Leading zeros: +01, +001 (valid? depends on spec)
4. Just below min: +(empty) → Invalid
5. Just above max: +1000 (4 digits) → Invalid
6. Non-numeric: +1A, +ABC → Invalid
7. Negative: +-1 → Invalid
8. Decimal: +1.5 → Invalid
9. Missing plus: 1234567890 → Invalid/needs handling

PHONE NUMBER (4-14 digits):
---------------------------
10. Minimum: 4 digits (1234)
11. Maximum: 14 digits (12345678901234)
12. Below min: 3 digits (123) → Invalid
13. Above max: 15 digits → Invalid
14. At boundary-1: 3 digits, 5 digits
15. At boundary+1: 13 digits, 15 digits
16. All same digit: 1111111111
17. All zeros: 0000000000 (valid?)
18. Starts with zero: 0123456789

FORMATTING CHARACTERS:
----------------------
19. With spaces: +1 234 567 8901
20. With dashes: +1-234-567-8901
21. With parentheses: +1 (234) 567-8901
22. Mixed format: +1 (234) 567-8901
23. Leading/trailing spaces: " +1234567890 "
24. Multiple consecutive spaces: +1  234   5678
25. Only formatting chars: +1 --- () ---
26. Formatting within country code: +(1) 234567890

SPECIAL CHARACTERS:
-------------------
27. Unicode digits: +1 ٢٣٤٥٦٧٨٩٠ (Arabic numerals)
28. Full-width digits: +１２３４５６７８９０
29. Letters mixed in: +1-ABC-DEF-1234
30. Special symbols: +1.234.567.8901 (dots)
31. Emoji: +1📱2345678
32. SQL injection: +1'; DROP TABLE--
33. XSS: +1<script>alert(1)</script>
34. Null character: +1\x00234567890

EMPTY/NULL:
-----------
35. Empty string: ""
36. Null value: null
37. Whitespace only: "   "
38. Just plus sign: "+"
39. Just country code: "+1"
40. Just number: "1234567890"

LENGTH COMBINATIONS:
--------------------
41. Min country + min number: +1 1234 (total 5 digits)
42. Max country + max number: +999 12345678901234 (total 17 digits)
43. Min country + max number: +1 12345678901234
44. Max country + min number: +999 1234
```

**b) Edge Cases Categories (3 điểm):**

```python
EDGE_CASES = {
    'BOUNDARY_VALUES': {
        'description': 'Values at or near limits',
        'cases': [
            {'id': 'BV01', 'input': '+1 1234', 'desc': 'Min country + min number'},
            {'id': 'BV02', 'input': '+999 12345678901234', 'desc': 'Max country + max number'},
            {'id': 'BV03', 'input': '+1 123', 'desc': 'Number below min (3 digits)'},
            {'id': 'BV04', 'input': '+1 123456789012345', 'desc': 'Number above max (15 digits)'},
            {'id': 'BV05', 'input': '+1000 1234567890', 'desc': 'Country code above max'},
        ],
        'priority': 'HIGH',
    },

    'FORMAT_VARIATIONS': {
        'description': 'Different valid formatting styles',
        'cases': [
            {'id': 'FV01', 'input': '+1 234 567 8901', 'desc': 'Spaces'},
            {'id': 'FV02', 'input': '+1-234-567-8901', 'desc': 'Dashes'},
            {'id': 'FV03', 'input': '+1 (234) 567-8901', 'desc': 'Parentheses'},
            {'id': 'FV04', 'input': '+12345678901', 'desc': 'No formatting'},
            {'id': 'FV05', 'input': '  +1 234 567 8901  ', 'desc': 'Leading/trailing spaces'},
        ],
        'priority': 'HIGH',
    },

    'INVALID_CHARACTERS': {
        'description': 'Characters that should be rejected',
        'cases': [
            {'id': 'IC01', 'input': '+1 ABC DEF 1234', 'desc': 'Letters in number'},
            {'id': 'IC02', 'input': '+1.234.567.8901', 'desc': 'Dots as separators'},
            {'id': 'IC03', 'input': '+1#234#567#8901', 'desc': 'Hash symbols'},
            {'id': 'IC04', 'input': '+1*234*567*8901', 'desc': 'Asterisks'},
        ],
        'priority': 'MEDIUM',
    },

    'EMPTY_NULL': {
        'description': 'Empty or null inputs',
        'cases': [
            {'id': 'EN01', 'input': '', 'desc': 'Empty string'},
            {'id': 'EN02', 'input': None, 'desc': 'Null value'},
            {'id': 'EN03', 'input': '   ', 'desc': 'Whitespace only'},
            {'id': 'EN04', 'input': '+', 'desc': 'Plus only'},
            {'id': 'EN05', 'input': '+1', 'desc': 'Country code only'},
        ],
        'priority': 'HIGH',
    },

    'SECURITY': {
        'description': 'Security-related inputs',
        'cases': [
            {'id': 'SEC01', 'input': "+1'; DROP TABLE users;--", 'desc': 'SQL injection'},
            {'id': 'SEC02', 'input': '+1<script>alert(1)</script>', 'desc': 'XSS attempt'},
            {'id': 'SEC03', 'input': '+1\x00234567890', 'desc': 'Null byte injection'},
            {'id': 'SEC04', 'input': '+1' + '2' * 10000, 'desc': 'Buffer overflow attempt'},
        ],
        'priority': 'CRITICAL',
    },

    'INTERNATIONALIZATION': {
        'description': 'International number formats',
        'cases': [
            {'id': 'I18N01', 'input': '+84 912345678', 'desc': 'Vietnam'},
            {'id': 'I18N02', 'input': '+1 800 555 1234', 'desc': 'US toll-free'},
            {'id': 'I18N03', 'input': '+44 20 7946 0958', 'desc': 'UK London'},
            {'id': 'I18N04', 'input': '+81 3 1234 5678', 'desc': 'Japan Tokyo'},
            {'id': 'I18N05', 'input': '+86 10 1234 5678', 'desc': 'China Beijing'},
        ],
        'priority': 'HIGH',
    },

    'SPECIAL_NUMBERS': {
        'description': 'Special or unusual number patterns',
        'cases': [
            {'id': 'SP01', 'input': '+1 0000000000', 'desc': 'All zeros'},
            {'id': 'SP02', 'input': '+1 1111111111', 'desc': 'All same digit'},
            {'id': 'SP03', 'input': '+1 1234567890', 'desc': 'Sequential'},
            {'id': 'SP04', 'input': '+1 0123456789', 'desc': 'Starts with zero'},
        ],
        'priority': 'MEDIUM',
    },
}
```

**c) Test Cases for Critical Edge Cases (3 điểm):**

```python
import pytest

class TestPhoneNumberEdgeCases:
    """Test critical edge cases for phone number validation"""

    # BOUNDARY VALUE TESTS (CRITICAL)
    @pytest.mark.parametrize("phone,expected,description", [
        # Minimum valid
        ("+1 1234", True, "Min country (1) + min number (4)"),
        ("+12 1234", True, "Country 2 digits + min number"),
        ("+999 1234", True, "Max country (3) + min number"),

        # Maximum valid
        ("+1 12345678901234", True, "Min country + max number (14)"),
        ("+999 12345678901234", True, "Max country + max number"),

        # Just outside boundaries
        ("+1 123", False, "Number too short (3 digits)"),
        ("+1 123456789012345", False, "Number too long (15 digits)"),
        ("+1000 1234567890", False, "Country code too long (4 digits)"),
        ("+ 1234567890", False, "Empty country code"),
    ])
    def test_boundary_values(self, phone, expected, description):
        result = validate_phone(phone)
        assert result == expected, f"Failed: {description}"

    # FORMAT VARIATION TESTS (HIGH)
    @pytest.mark.parametrize("phone,expected", [
        ("+1 234 567 8901", True),       # Spaces
        ("+1-234-567-8901", True),       # Dashes
        ("+1 (234) 567-8901", True),     # Parentheses
        ("+12345678901", True),           # No formatting
        ("  +1 234 567 8901  ", True),   # Trimmed
        ("+1  234   567  8901", True),   # Multiple spaces
    ])
    def test_format_variations(self, phone, expected):
        assert validate_phone(phone) == expected

    # SECURITY TESTS (CRITICAL)
    @pytest.mark.parametrize("phone,description", [
        ("+1'; DROP TABLE users;--", "SQL injection"),
        ("+1<script>alert(1)</script>", "XSS"),
        ("+1\x00234567890", "Null byte"),
        ("+1" + "2" * 10000, "Overflow attempt"),
        ("+1\n\r234567890", "Newline injection"),
    ])
    def test_security_inputs(self, phone, description):
        # Should not raise exception, just return False
        try:
            result = validate_phone(phone)
            assert result == False, f"Security input should be rejected: {description}"
        except Exception as e:
            pytest.fail(f"Should not raise exception for {description}: {e}")

    # EMPTY/NULL TESTS (HIGH)
    @pytest.mark.parametrize("phone,expected", [
        ("", False),
        (None, False),
        ("   ", False),
        ("+", False),
        ("+1", False),          # No number part
        ("1234567890", False),  # No country code
    ])
    def test_empty_null_inputs(self, phone, expected):
        result = validate_phone(phone)
        assert result == expected

    # INTERNATIONAL NUMBERS (HIGH)
    @pytest.mark.parametrize("phone,country,expected", [
        ("+84 912345678", "Vietnam", True),
        ("+1 800 555 1234", "US Toll-free", True),
        ("+44 20 7946 0958", "UK", True),
        ("+81 3 1234 5678", "Japan", True),
        ("+86 10 1234 5678", "China", True),
        ("+91 98765 43210", "India", True),
        ("+49 30 1234567", "Germany", True),
    ])
    def test_international_formats(self, phone, country, expected):
        assert validate_phone(phone) == expected, f"Failed for {country}"

    # EDGE CASE COMBINATIONS
    def test_all_zeros_number(self):
        """All zeros should be valid syntactically"""
        assert validate_phone("+1 0000000000") == True

    def test_starts_with_zero(self):
        """Number starting with zero"""
        assert validate_phone("+1 0123456789") == True

    def test_unicode_digits_rejected(self):
        """Non-ASCII digits should be rejected"""
        assert validate_phone("+1 ٢٣٤٥٦٧٨٩٠١") == False  # Arabic

    def test_fullwidth_digits_rejected(self):
        """Full-width digits should be rejected"""
        assert validate_phone("+１ ２３４５６７８９０") == False  # Japanese
```

---

## Câu 10: Toolshop Test Case Design (10 điểm)

### Tình huống:
Bạn cần test chức năng User Registration của Toolshop application (tham khảo HW02):
- First Name: 2-50 characters, letters only
- Last Name: 2-50 characters, letters only
- Email: valid email format
- Password: 8-20 chars, 1 upper, 1 lower, 1 digit
- Date of Birth: must be 18+ years old

### Yêu cầu:
a) Áp dụng ECP để xác định equivalence classes (3 điểm)
b) Áp dụng BVA cho các boundary values (3 điểm)
c) Tạo minimum test set với 100% coverage (4 điểm)

### Đáp án mẫu:

**a) Equivalence Classes (3 điểm):**

```python
EQUIVALENCE_CLASSES = {
    'first_name': {
        'valid': [
            {'id': 'FN_V1', 'desc': '2-50 letters', 'example': 'John'},
            {'id': 'FN_V2', 'desc': 'With spaces allowed?', 'example': 'Mary Jane'},
            {'id': 'FN_V3', 'desc': 'Unicode letters', 'example': 'José'},
        ],
        'invalid': [
            {'id': 'FN_I1', 'desc': 'Too short (1 char)', 'example': 'J'},
            {'id': 'FN_I2', 'desc': 'Too long (51+ chars)', 'example': 'A' * 51},
            {'id': 'FN_I3', 'desc': 'Contains numbers', 'example': 'John123'},
            {'id': 'FN_I4', 'desc': 'Contains special chars', 'example': 'John@Doe'},
            {'id': 'FN_I5', 'desc': 'Empty', 'example': ''},
            {'id': 'FN_I6', 'desc': 'Only whitespace', 'example': '   '},
        ],
    },

    'last_name': {
        # Same as first_name
        'valid': [
            {'id': 'LN_V1', 'desc': '2-50 letters', 'example': 'Smith'},
        ],
        'invalid': [
            {'id': 'LN_I1', 'desc': 'Too short', 'example': 'S'},
            {'id': 'LN_I2', 'desc': 'Too long', 'example': 'B' * 51},
            {'id': 'LN_I3', 'desc': 'Contains numbers', 'example': 'Smith2'},
        ],
    },

    'email': {
        'valid': [
            {'id': 'EM_V1', 'desc': 'Standard format', 'example': 'user@domain.com'},
            {'id': 'EM_V2', 'desc': 'With subdomain', 'example': 'user@sub.domain.com'},
            {'id': 'EM_V3', 'desc': 'With plus tag', 'example': 'user+tag@domain.com'},
            {'id': 'EM_V4', 'desc': 'With dots', 'example': 'first.last@domain.com'},
        ],
        'invalid': [
            {'id': 'EM_I1', 'desc': 'No @ symbol', 'example': 'userdomain.com'},
            {'id': 'EM_I2', 'desc': 'No domain', 'example': 'user@'},
            {'id': 'EM_I3', 'desc': 'No TLD', 'example': 'user@domain'},
            {'id': 'EM_I4', 'desc': 'Double @', 'example': 'user@@domain.com'},
            {'id': 'EM_I5', 'desc': 'Spaces', 'example': 'user @domain.com'},
            {'id': 'EM_I6', 'desc': 'Empty', 'example': ''},
        ],
    },

    'password': {
        'valid': [
            {'id': 'PW_V1', 'desc': 'All requirements met', 'example': 'Password1'},
            {'id': 'PW_V2', 'desc': 'With special chars', 'example': 'Password1!'},
            {'id': 'PW_V3', 'desc': 'Exactly 8 chars', 'example': 'Passwo1d'},
            {'id': 'PW_V4', 'desc': 'Exactly 20 chars', 'example': 'Aa1' + 'x' * 17},
        ],
        'invalid': [
            {'id': 'PW_I1', 'desc': 'Too short (7)', 'example': 'Passw1d'},
            {'id': 'PW_I2', 'desc': 'Too long (21)', 'example': 'Aa1' + 'x' * 18},
            {'id': 'PW_I3', 'desc': 'No uppercase', 'example': 'password1'},
            {'id': 'PW_I4', 'desc': 'No lowercase', 'example': 'PASSWORD1'},
            {'id': 'PW_I5', 'desc': 'No digit', 'example': 'Passwordd'},
            {'id': 'PW_I6', 'desc': 'With spaces', 'example': 'Pass word1'},
        ],
    },

    'date_of_birth': {
        'valid': [
            {'id': 'DOB_V1', 'desc': 'Exactly 18', 'example': '18 years ago today'},
            {'id': 'DOB_V2', 'desc': 'Adult (30)', 'example': '30 years ago'},
            {'id': 'DOB_V3', 'desc': 'Senior (70)', 'example': '70 years ago'},
        ],
        'invalid': [
            {'id': 'DOB_I1', 'desc': 'Under 18', 'example': '17 years ago'},
            {'id': 'DOB_I2', 'desc': 'Future date', 'example': 'tomorrow'},
            {'id': 'DOB_I3', 'desc': 'Invalid format', 'example': 'not-a-date'},
        ],
    },
}
```

**b) Boundary Value Analysis (3 điểm):**

```python
from datetime import date, timedelta

# Calculate DOB for exact ages
today = date.today()

BOUNDARY_VALUES = {
    'first_name_length': {
        # [min-1, min, min+1, nominal, max-1, max, max+1]
        'invalid_below': 'J',                    # 1 char (min-1)
        'at_min': 'Jo',                          # 2 chars (min)
        'above_min': 'Joe',                      # 3 chars (min+1)
        'nominal': 'John',                       # 4 chars
        'below_max': 'A' * 49,                   # 49 chars (max-1)
        'at_max': 'A' * 50,                      # 50 chars (max)
        'invalid_above': 'A' * 51,               # 51 chars (max+1)
    },

    'password_length': {
        'invalid_below': 'Passw1d',              # 7 chars
        'at_min': 'Passwo1d',                    # 8 chars
        'above_min': 'Passwor1d',                # 9 chars
        'nominal': 'Password123',               # 11 chars
        'below_max': 'Aa1' + 'x' * 16,          # 19 chars
        'at_max': 'Aa1' + 'x' * 17,             # 20 chars
        'invalid_above': 'Aa1' + 'x' * 18,      # 21 chars
    },

    'date_of_birth_age': {
        # BVA for 18+ requirement
        'invalid_17': today - timedelta(days=17*365),         # 17 years old
        'invalid_17_364': today - timedelta(days=17*365 + 364), # 17y 364d
        'at_min_18': today - timedelta(days=18*365),          # Exactly 18
        'above_min': today - timedelta(days=18*365 + 1),      # 18y 1d
        'nominal': today - timedelta(days=30*365),            # 30 years
    },
}

# Generate actual test values
def generate_boundary_dob(years, days_offset=0):
    """Generate DOB for given age with optional day offset"""
    return today - timedelta(days=years * 365 + days_offset)

DOB_BOUNDARY_TESTS = [
    # Age 17 years 364 days (just under 18) → Invalid
    {'dob': generate_boundary_dob(17, 364), 'expected': False, 'desc': 'Just under 18'},

    # Age exactly 18 years → Valid
    {'dob': generate_boundary_dob(18, 0), 'expected': True, 'desc': 'Exactly 18'},

    # Age 18 years 1 day → Valid
    {'dob': generate_boundary_dob(18, 1), 'expected': True, 'desc': 'Just over 18'},
]
```

**c) Minimum Test Set with 100% Coverage (4 điểm):**

```python
import pytest
from datetime import date, timedelta

# Minimum test set using Single Fault Assumption
# (Only one invalid condition per test for invalid cases)

MINIMUM_TEST_SET = [
    # ============= VALID TESTS (1 test covers all valid classes) =============
    {
        'id': 'TC01',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'john.doe@test.com',
        'password': 'Password123',
        'dob': '1990-06-15',
        'expected': 'SUCCESS',
        'covers': ['All valid ECP classes'],
    },

    # Additional valid for boundary coverage
    {
        'id': 'TC02',
        'first_name': 'Jo',                      # BVA: min length (2)
        'last_name': 'Do',
        'email': 'jo@test.com',
        'password': 'Passwo1d',                  # BVA: min length (8)
        'dob': str(date.today() - timedelta(days=18*365)),  # BVA: exactly 18
        'expected': 'SUCCESS',
        'covers': ['BVA: min first_name', 'BVA: min password', 'BVA: min age'],
    },

    {
        'id': 'TC03',
        'first_name': 'A' * 50,                  # BVA: max length
        'last_name': 'B' * 50,
        'email': 'max@test.com',
        'password': 'Aa1' + 'x' * 17,           # BVA: max length (20)
        'dob': '1950-01-01',                     # Valid senior
        'expected': 'SUCCESS',
        'covers': ['BVA: max first_name', 'BVA: max password'],
    },

    # ============= INVALID TESTS (one per invalid class) =============

    # First Name invalid cases
    {
        'id': 'TC04',
        'first_name': 'J',                       # Too short
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Password1',
        'dob': '1990-01-01',
        'expected': 'ERROR_FIRST_NAME_SHORT',
        'covers': ['ECP: first_name too short'],
    },
    {
        'id': 'TC05',
        'first_name': 'A' * 51,                  # Too long
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Password1',
        'dob': '1990-01-01',
        'expected': 'ERROR_FIRST_NAME_LONG',
        'covers': ['ECP: first_name too long'],
    },
    {
        'id': 'TC06',
        'first_name': 'John123',                 # Contains numbers
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Password1',
        'dob': '1990-01-01',
        'expected': 'ERROR_FIRST_NAME_FORMAT',
        'covers': ['ECP: first_name with numbers'],
    },

    # Email invalid cases
    {
        'id': 'TC07',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'invalid-email',                # No @ symbol
        'password': 'Password1',
        'dob': '1990-01-01',
        'expected': 'ERROR_EMAIL_FORMAT',
        'covers': ['ECP: email no @'],
    },
    {
        'id': 'TC08',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': '',                             # Empty email
        'password': 'Password1',
        'dob': '1990-01-01',
        'expected': 'ERROR_EMAIL_REQUIRED',
        'covers': ['ECP: email empty'],
    },

    # Password invalid cases
    {
        'id': 'TC09',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Passw1d',                   # Too short (7)
        'dob': '1990-01-01',
        'expected': 'ERROR_PASSWORD_SHORT',
        'covers': ['BVA: password < 8'],
    },
    {
        'id': 'TC10',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'password1',                 # No uppercase
        'dob': '1990-01-01',
        'expected': 'ERROR_PASSWORD_UPPERCASE',
        'covers': ['ECP: no uppercase'],
    },
    {
        'id': 'TC11',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Passwordd',                 # No digit
        'dob': '1990-01-01',
        'expected': 'ERROR_PASSWORD_DIGIT',
        'covers': ['ECP: no digit'],
    },

    # DOB invalid cases
    {
        'id': 'TC12',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Password1',
        'dob': str(date.today() - timedelta(days=17*365)),  # Under 18
        'expected': 'ERROR_AGE_UNDERAGE',
        'covers': ['ECP: under 18'],
    },
    {
        'id': 'TC13',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'test@test.com',
        'password': 'Password1',
        'dob': str(date.today() + timedelta(days=1)),  # Future date
        'expected': 'ERROR_DOB_FUTURE',
        'covers': ['ECP: future DOB'],
    },
]

# Test implementation
@pytest.mark.parametrize("test_case", MINIMUM_TEST_SET)
def test_registration(test_case):
    result = register_user(
        first_name=test_case['first_name'],
        last_name=test_case['last_name'],
        email=test_case['email'],
        password=test_case['password'],
        dob=test_case['dob']
    )

    if test_case['expected'] == 'SUCCESS':
        assert result.success == True
    else:
        assert result.success == False
        assert test_case['expected'] in result.error_code

# Coverage Summary:
# - 3 valid tests covering all valid ECP + key BVA
# - 10 invalid tests covering each invalid ECP
# - Total: 13 tests for 100% ECP + BVA coverage

# Without optimization: Would need 6 × 6 × 6 × 6 × 3 = 3888 exhaustive tests
# Savings: 99.7% reduction
```

---

## Tổng kết

| Câu | Chủ đề | Điểm |
|-----|--------|------|
| 1 | Equivalence Class Partitioning Design | 10 |
| 2 | Boundary Value Analysis | 10 |
| 3 | Pairwise/Combinatorial Testing | 10 |
| 4 | Faker Data Generation | 10 |
| 5 | Decision Table Testing | 10 |
| 6 | Mockaroo Data Generation | 10 |
| 7 | State Transition Testing | 10 |
| 8 | Test Data Seeding Strategy | 10 |
| 9 | Edge Case Identification | 10 |
| 10 | Toolshop Test Case Design | 10 |
| **Tổng** | | **100** |

---

*Thời gian ôn tập đề xuất: 3-4 giờ*
*Thực hành: Viết test cases cho Toolshop registration với ECP + BVA, generate data với Faker*
