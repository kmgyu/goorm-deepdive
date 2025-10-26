# IntelliJ IDEA Live Templates 정리

라이브 템플릿은 자주 사용하는 코드 조각을 짧은 **단축어(abbreviation)** 로 빠르게 입력할 수 있게 도와주는 기능입니다. `psvm`을 입력하면 `public static void main(String[] args)` 로 자동 확장되는 것처럼, 반복적인 코드를 빠르게 작성할 수 있습니다.

---

## 1. 라이브 템플릿의 유형

IntelliJ IDEA에서는 다음과 같은 네 가지 종류의 라이브 템플릿을 제공합니다.

### 1.1 단순 템플릿 (Simple Templates)

- 고정된 텍스트만 포함된 가장 기본적인 형태입니다.
    
- 입력한 축약어가 고정된 코드로 그대로 확장됩니다.
    

|축약어|확장 코드|
|---|---|
|`psfs`|`public static final String`|
|`psvm`, `main`|`public static void main(String[] args) { }`|
|`sout`|`System.out.println();`|

---

### 1.2 매개변수화된 템플릿 (Parameterized Templates)

- 템플릿 내부에 **변수(variable)** 를 포함하여 사용자 입력을 받거나 자동 계산된 값을 넣을 수 있습니다.
    
- 변수는 `$VARIABLE_NAME$` 형식으로 감쌉니다.
    

|축약어|확장 코드 예시|
|---|---|
|`for`|`for (int i = 0; i < ; i++) { }`|
|`ifn`|`if (var == null) { }`|

---

### 1.3 감싸기 템플릿 (Surround Templates)

- 선택한 코드 블록을 특정 코드 구조로 감쌉니다.
    
- 예: `T` → `<tag></tag>` 형태로 감싸기
    
- 사용법: 코드 선택 후 `Ctrl + Alt + J` → 템플릿 선택
    

---

### 1.4 후위 코드 완성 (Postfix Code Completion)

- 템플릿과 비슷하지만 **표현식 뒤에 단축어를 붙여서 즉시 변환**합니다.
    
- 예:  
    `someCondition.if` →  
    `if (someCondition) { }`
    

---

## 2. 라이브 템플릿 설정 방법

라이브 템플릿을 추가/수정하려면 다음 경로로 이동하세요:

- `Settings(환경설정)` → `Editor` → `Live Templates`  
    또는 단축키 `Ctrl + Alt + S`
    

### 설정 절차

1. 좌측에서 언어 그룹 선택 (예: Java)
    
2. 우측 상단의 `+` 버튼 클릭 → `Live Template` 선택
    
3. **Abbreviation**: 단축어 입력 (예: `test`)
    
4. **Description**: 설명 입력 (예: `JUnit5 test method`)
    
5. **Template Text**: 템플릿 본문 작성
    
    - 변수는 `$변수명$` 형식
        
    - 커서가 이동할 마지막 위치는 `$END$`
        

예시:

java

복사편집

`@DisplayName("$DISPLAY_NAME$") @Test void $METHOD_NAME$() {     // given     $END$     // when      // then }`

6. **Define** 버튼 클릭 → 사용할 언어 환경 선택 (예: Java)
    

---

## 3. import 자동 처리 팁

### 3.1 Shorten FQ Names 옵션

- 라이브 템플릿에서 클래스/어노테이션을 **풀 패키지 경로(Fully Qualified Name)** 로 작성한 경우, 자동으로 import 문을 추가해줍니다.
    
- 예:
    
    java
    
    복사편집
    
    `@org.junit.jupiter.api.Test`
    
    → `Shorten FQ names` 옵션 활성 시:
    
    java
    
    복사편집
    
    `import org.junit.jupiter.api.Test; @Test`
    

> 설정 위치: 템플릿 작성 화면 하단의 `Shorten FQ names` 체크

### 3.2 Use static import if possible 옵션

- static 메서드나 상수를 사용할 때 `Assertions.assertEquals()` 대신 `assertEquals()` 등으로 자동 import하도록 설정할 수 있습니다.
    

---

## 4. 기타 설정 정보

- 템플릿은 **언어별 그룹**으로 구분됩니다. 템플릿을 다른 그룹으로 옮기려면 우클릭 → `Move`.
    
- **동일한 단축어라도 그룹에 따라 서로 다른 코드로 확장**될 수 있습니다.
    
- **수정된 기본 템플릿은 파란 글씨**로 표시되어 쉽게 구분됩니다.
    
- **언어 방언(Dialect)** 이 설정된 경우, 해당 방언에만 적용됩니다.
    

---

## 5. 활용 예시

Live Template을 작성하고 사용하는 모습:

![Live Template 예시](https://hudi.blog/static/gwt-6d79fe4746dbe1090bfdc6649c857d2e.gif)