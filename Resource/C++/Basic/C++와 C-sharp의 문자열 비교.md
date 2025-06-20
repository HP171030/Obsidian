
## 1. C++ `std::string` vs C# `string` vs C# `StringBuilder`

* **C++ `std::string`:**
    * **가변(Mutable)** 문자열 클래스
    * 내용을 직접 변경할 수 있으며, 메모리 관리를 자동으로 처리 (힙에 동적 할당).
    * C#의 `StringBuilder`와 유사

* **C# `string`:**
    * **불변(Immutable)** 문자열 객체
    * 한 번 생성되면 내용을 변경할 수 없음. 수정 시 매번 새로운 객체를 생성
    * C++의 `const char*` (문자열 리터럴)의 '불변성' 개념과 유사하지만, `string`은 클래스이고 `const char*`는 원시 포인터라는 차이가 있음

* **C# `StringBuilder`:**
    * C#에서 가변 문자열을 효율적으로 다루기 위한 클래스
    * `string`의 불변성으로 인한 성능 문제를 해결하기 위해 도입

## 2. C++에서 C# `string`과 가장 유사한 타입

* **`std::string_view` (C++17 이후):**
    * 문자열 데이터를 **소유하지 않는(non-owning) 읽기 전용 뷰**
    * 문자열 데이터의 주소와 문자열 길이를 가지고 있는 클래스라 보면 됨
    * `substr()`, `find()`, `length()` 등 `std::string`과 유사한 멤버 함수를 제공
    * C# `string`처럼 읽기 전용으로 사용되며, 값 복사 없이 전달되어 효율적

* **`const std::string&`:**
    * `std::string` 객체에 대한 상수 참조
    * 함수 내에서 문자열 내용을 변경할 수 없도록 강제
    * C# `string`처럼 불필요한 복사 없이 문자열을 전달할 때 유용함

* **문자열 리터럴 (`"..."`) 또는 `const char*`:**
    * C++에서 가장 강력하게 불변성이 보장되는 문자열

## 3. 요약

| 특징             | C++ `std::string`                                  | C# `string`                                     | C# `StringBuilder`                                 |
| :--------------- | :------------------------------------------------- | :---------------------------------------------- | :------------------------------------------------- |
| 가변성           | 가변(Mutable)                                     | 불변(Immutable)                                 | 가변(Mutable)                                     |
| 메모리 관리      | 힙 메모리 동적 할당 및 자동 관리                       | .NET 런타임의 관리되는 힙                         | 힙 메모리 동적 할당 및 관리 (C++ `std::string`과 유사) |
| 주 용도           | 가변 문자열 처리                                   | 불변 문자열 처리, 문자열 리터럴                  | 가변 문자열 처리                                   |
| C# 타입과 유사성 | C# `StringBuilder`와 가장 유사 (가변성, 효율적인 수정) | C++에서는 직접 대응되는 타입이 없음 (불변성만 보면 `const char*`와 유사) | C++ `std::string`과 유사                                |

- strlen(fieldName) : fieldName 문자열의 길이를 가져오는 함수
- cin.getLine(fieldName,size) : 콘솔에 입력한 문자열을 fieldName에 size 만큼 가져옴(공백 포함) => cin.get(fieldName,size) 같음