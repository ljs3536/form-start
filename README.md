# /24-09-30

## 프로젝트 생성

# /24-10-02

## 타임리프 스프링 통합

타임리프는 스프링 없이도 동작하지만, 스프링과 통합을 위한 다양한 기능을 편리하게 제공한다.
그리고 이런 부분은 스프링으로 백엔드를 개발하는 개발자 입장에서 타임리프를 선택하는 하나의 이유가 된다.

### 스프링 통합으로 추가되는 기능들
- 스프링의 SpringEL 문법 통합
- '${@myBean.doSomething()}'처럼 스프링 빈 호출 지원
- 편리한 폼 관리를 위한 추가 속성
  - th:object (기능 강화, 폼 커멘드 객체 선택)
  - th:field, th:errors, th:errorclass
- 폼 컴포넌트 기능
  - checkbox, radio button, List등을 편리하게 사용할 수 있는 기능 지원
- 스프링의 메시지, 국제화 기능의 편리한 통합
- 스프링의 검증, 오류 처리 통합
- 스프링의 변환 서비스 통합

# /24-10-03

## 입력 폼 처리

# /24-10-04

## 요구사항 추가
타임리프를 사용해서 폼에서 체크박스, 라디오 버튼, 셀렉트 박스를 편리하게 사용하는 방법을 학습해보자.

- 판매여부
  - 판매 오픈 여부
  - 체크 박스로 선택할 수 있다.
- 등록 지역
  - 서울, 부산, 제주
  - 체크 박스로 다중 선택할 수 있다.
- 상품 종류
  - 도서, 식품, 기타
  - 라디오 버튼으로 하나만 선택할 수 있다.
- 배송 방식
  - 빠른 배송
  - 일반 배송
  - 느린 배송
  - 셀렉트 박스로 하나만 선택할 수 있다.



### 정리
th:object, th:field 덕분에 폼을 개발 할 때 약간의 편리함을 얻었다.
이것의 진짜 위력은 뒤에 설명할 검증(Validation)에서 나타난다.

# /24-10-05

## 체크박스 - 단일1

### 주의! 체크박스를 선택하지 않을 떄
HTML에서 체크 박스를 선택하지 않고 폼을 전송하면 open이라는 필드 자체가 서버로 전송되지 않는다.

HTML 체크박스는 선택이 안되면 클라이언트에서 서버로 값 자체를 보내지 않는다. 
수정의 경우에는 상황에 따라서 이 방식이 문제가 될 수 있다.
사용자가 의도적으로 체크되어 있던 값을 체크를 해제해도 저장 시 아무 값도 넘어가지 않기 때문에,
서버 구현에 따라서 값이 오지 않은 것으로 판단해서 값을 변경하지 않을 수도 있다.

이런 문제를 해결하기 위해서 스프링MVC는 약간의 트릭을 사용하는데, 
히든 필드를 하나 만들어서, _open처럼 기존 체크 박스 이름 앞에 언더스코어를 붙여서 전송하면 체크를 해제했다고 인식할 수 있다.
히든 필드는 항상 전송된다. 따라서 체크를 해제한 경우 여기에서 open은 전송되지 않고,
open은 전송되지 않고, _open만 전송되는데, 이 경우 스프링 MVC는 체크를 해제했다고 판단한다.

### 체크해제를 인식하기 위한 히든 필드
"<input type="hidden" name="_open" value="on"/>"

### 체크박스 체크
open=on&_open=on
체크박스를 체크하면 스프링MVC가 open에 값이 있는것을 확인하고 사용한다. 이떄 _open은 무시한다.

### 체크박스 미체크
_open=on
체크박스를 체크하지 않으면 스프링MVC가 _open만 있는 것을 확인하고, open의 값이 체크되지 않았다고 인식한다.
이 경우 서버에서 Boolean 타입을 찍어보면 결과가 null이 아니라 false인 것을 확인할 수 있다.

# /24-10-06

## 체크박스 - 단일2

### 타임리프
개발할 때 마다 이렇게 히든 필드를 추가하는 것은 상당히 번거롭다. 타임리프가 제공하는 폼 기능을 사용하면 이런 부분을 자동으로 처리할 수 있다.

### 타임리프의 체크확인
체크박스에서 판매 여부를 선택해서 저장하면, 조회시에 checked 속성이 추가된 것을 확인할 수 있다.
이런 부분을 개발자가 직접 처리하려면 상당히 번거롭다.
타임리프의 th:field를 같이 사용하면, 값이 true인 경우 체크를 자동으로 처리해준다.

# /24-10-07

## 체크 박스 - 멀티

### @ModelAttribute의 특별한 사용법
등록 폼, 상세화면, 수정 폼에서 모두 서울, 부산, 제주라는 체크 박스를 반복해서 보여주어야 한다.
이렇게 하려면 각각의 컨트롤러에서 model.addAttribute(...)을 사용해서 체크 박스를 구성하는 데이터를 반복해서 넣어주어야 한다.
@ModelAttribute는 이렇게 컨트롤러에 있는 별도의 메서드에 적용할 수 있다.
이렇게하면 해당 컨트롤러를 요청할 때 regions에서 반환환 값이 자동으로 모델(model)에 담기게 된다.
물론 이렇게 사용하지 않고, 각각의 컨트롤러 메서드에서 모델에 직접 데이터를 담아서 처리해도 된다.

### th:for="${#ids.prev('open')}"
멀티 체크박스는 같은 이름의 여러 체크박스를 만들 수 있다. 그런데 문제는 이렇게 반복해서 HTML태그를 생성할 때, 
생성된 HTML 태그 속성에서 name은 같아도 되지만, id는 모두 달라야한다.
따라서 타임리프는 체크박스를 each 루프 안에서 반복해서 만들 때 임의로 1,2,3 숫자를 뒤에 붙여준다.

# /24-10-08

## 라디오 버튼

### 타임리프에서 ENUM 직접 사용하기

#### 타임리프에서 ENUM 직접 접근
<div th:each="type:${T(hello.itemservice.domain.item.ItemType).values()}">
스프링EL 문법으로 ENUM을 직접 사용할 수 있다.
ENUM에 values()를 호출하면 해당 ENUM의 모든 정보가 배열로 반환된다.
그런데 이렇게 사용하면 ENUM패키지의 위치가 변경되거나 할때 자바 컴파일러가 타임리프까지 컴파일 오류를 잡을 수 없으므로 추천하지는 않는다.

# /24-10-10

## 셀렉트 박스 

