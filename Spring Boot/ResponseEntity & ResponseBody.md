
# 목차

1. [소개](#%EC%86%8C%EA%B0%9C)
2. [HTTP 개요](#http-%EA%B0%9C%EC%9A%94)
3. [HTTP 요청과 응답 구조](#http-%EC%9A%94%EC%B2%AD%EA%B3%BC-%EC%9D%91%EB%8B%B5-%EA%B5%AC%EC%A1%B0)
4. [HTTP Body의 개념](#http-body%EC%9D%98-%EA%B0%9C%EB%85%90)
    - [요청 본문(Request Body)](#%EC%9A%94%EC%B2%AD-%EB%B3%B8%EB%AC%B8request-body)
    - [응답 본문(Response Body)](#%EC%9D%91%EB%8B%B5-%EB%B3%B8%EB%AC%B8response-body)
5. [ResponseEntity와 ResponseBody 맛보기](#responseentity%EC%99%80-responsebody-%EB%A7%9B%EB%B3%B4%EA%B8%B0)
    - [ResponseEntity](#responseentity)
    - [ResponseBody](#responsebody)
6. [ResponseEntity와 ResponseBody 자세히 알아보기](#responseentity%EC%99%80-responsebody-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
    - [ResponseEntity 사용 예](#responseentity-%EC%82%AC%EC%9A%A9-%EC%98%88)
    - [@ResponseBody 사용 예](#responsebody-%EC%82%AC%EC%9A%A9-%EC%98%88)
    - [비교](#%EB%B9%84%EA%B5%90)
7. [결론](#%EA%B2%B0%EB%A1%A0)

# 소개

이 글은 HTTP(하이퍼텍스트 전송 프로토콜, HyperText Transfer Protocol)에 대한 개요와 HTTP 요청/응답 구조에 대해 설명합니다. 또한 Spring Framework를 사용하여 RESTful API를 구현할 때 유용한 `ResponseEntity`와 `ResponseBody`의 차이점과 사용 예에 대해 자세히 알아봅니다. 이 정보를 통해 보다 유연하고 효율적인 API 설계 방법을 이해할 수 있습니다.

# HTTP 개요

HTTP(하이퍼텍스트 전송 프로토콜, HyperText Transfer Protocol)는 웹에서 클라이언트와 서버 간에 데이터를 주고받기 위한 프로토콜입니다. HTTP는 요청/응답(request/response) 모델을 기반으로 하며, 클라이언트(일반적으로 웹 브라우저)는 서버에 요청을 보내고 서버는 요청에 대한 응답을 반환합니다.

HTTP 요청과 응답은 다음과 같은 구조로 이루어집니다:

1. **시작 라인(Start Line)**
    - 요청: 메소드(GET, POST, PUT, DELETE 등)와 URL, HTTP 버전이 포함됩니다.
    - 응답: HTTP 버전과 상태 코드(200, 404, 500 등), 상태 메시지가 포함됩니다.
2. **헤더(Headers)**
    - 요청과 응답 모두에서 추가 정보를 제공하는 키-값 쌍으로 이루어져 있습니다.
    - 예: `Content-Type`, `Authorization`, `Content-Length` 등
3. **본문(Body)**
    - 요청과 응답의 실제 데이터를 포함합니다.
    - GET 요청의 경우 본문이 비어 있을 수 있으며, POST, PUT 요청에서는 데이터를 전송하는 데 사용됩니다.
    - 응답 본문에는 HTML, JSON, XML 등 다양한 형식의 데이터가 포함될 수 있습니다.

#### HTTP Body의 개념

HTTP Body는 요청 또는 응답의 실제 데이터를 포함하는 부분입니다. 본문은 선택 사항이며, GET 요청에서는 일반적으로 사용되지 않습니다. POST, PUT, DELETE 요청에서는 데이터를 전송하기 위해 본문이 사용됩니다.

- **요청 본문(Request Body)**: 클라이언트가 서버로 데이터를 전송할 때 사용됩니다. 예를 들어, 사용자가 폼을 제출할 때 입력한 데이터를 서버로 보내는 경우가 있습니다.
```http
POST /api/v1/member HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 75

{
  "username": "john_doe",
  "email": "john@example.com"
}
```

- **응답 본문(Response Body)**: 서버가 클라이언트에게 데이터를 반환할 때 사용됩니다. 예를 들어, 서버가 요청된 리소스의 데이터를 JSON 형식으로 응답할 수 있습니다.
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123

{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com"
}
```

### ResponseEntity와 ResponseBody 맛보기

#### ResponseEntity

`ResponseEntity`는 Spring Framework에서 HTTP 응답을 구성하는 데 사용되는 클래스입니다. 이를 통해 상태 코드, 헤더, 본문을 명시적으로 설정할 수 있습니다.

```java
@PostMapping
public ResponseEntity<String> createMember(
        @RequestBody MemberCreateDto memberCreateDto
) {
    URI location = URI.create(memberService.createMember(memberCreateDto));
    return ResponseEntity.created(location).build();
}
```

위 코드에서 `ResponseEntity.created(location).build()`는 `201 Created` 상태 코드를 설정하고, `Location` 헤더에 새로 생성된 리소스의 URI를 설정합니다.

#### ResponseBody

```java
`@ResponseBody`는 메소드의 반환 값을 HTTP 응답 본문으로 변환하는 역할을 합니다. `@RestController`가 클래스에 적용되어 있는 경우, 모든 메소드에 자동으로 `@ResponseBody`가 적용됩니다.
```java
@GetMapping("/{memberId}")
public MemberFindDto findMemberById(
        @PathVariable Long memberId
) {
    return memberService.findMemberById(memberId);
}
```

위 코드에서 `findMemberById` 메소드는 `MemberFindDto` 객체를 반환하며, 이 객체는 HTTP 응답 본문으로 직렬화됩니다. 기본적으로 `200 OK` 상태 코드가 사용됩니다.

## ResponseEntity와 ResponseBody 자세히 알아보기

Spring Framework를 사용하여 RESTful API를 구현할 때, 우리는 종종 클라이언트에게 데이터를 반환하기 위해 `ResponseEntity`와 `ResponseBody`를 사용합니다. 이 두 클래스는 응답 본문을 구성하는 데 중요한 역할을 하지만, 사용 목적과 기능에서 차이가 있습니다. 이를 이해하기 위해 아래 예제 코드를 통해 각각의 역할을 살펴보겠습니다.

## ResponseEntity 사용 예

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/member")
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public ResponseEntity createMember(
            @RequestBody MemberCreateDto memberCreateDto
    ) {
        return ResponseEntity.created(URI.create(memberService.createMember(memberCreateDto))).build();
    }

    @GetMapping("/{memberId}")
    public ResponseEntity<MemberFindDto> findMemberById(
            @PathVariable Long memberId
    ) {
        return ResponseEntity.ok(memberService.findMemberById(memberId));
    }

    @DeleteMapping("/{memberId}")
    public ResponseEntity deleteMemberById(
            @PathVariable Long memberId
    ) {
        memberService.deleteMemberById(memberId);
        return ResponseEntity.noContent().build();
    }
}
```

`ResponseEntity`는 HTTP 응답 전체를 구성할 수 있는 클래스입니다. HTTP 상태 코드, 응답 본문, 헤더 등을 직접 설정할 수 있습니다. 위 예제에서 `createMember`, `findMemberById`, `deleteMemberById` 메소드들은 모두 `ResponseEntity`를 사용하여 응답을 반환하고 있습니다.

- **상태 코드 설정**: `ResponseEntity`는 HTTP 상태 코드를 명시적으로 설정할 수 있습니다. 예를 들어, `createMember` 메소드는 `201 Created` 상태 코드를 설정하고, `deleteMemberById` 메소드는 `204 No Content` 상태 코드를 설정합니다.
- **헤더 설정**: 응답 헤더를 설정할 수 있습니다. `createMember` 메소드는 `Location` 헤더에 새로 생성된 리소스의 URI를 설정합니다.
- **유연성**: `ResponseEntity`를 사용하면 다양한 응답을 유연하게 설정할 수 있습니다. HTTP 상태 코드와 헤더를 상황에 맞게 조절할 수 있습니다.

## @ResponseBody 사용 예
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/member")
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public String createMember(
            @RequestBody MemberCreateDto memberCreateDto
    ) {
        return memberService.createMember(memberCreateDto);
    }

    @GetMapping("/{memberId}")
    public MemberFindDto findMemberById(
            @PathVariable Long memberId
    ) {
        return memberService.findMemberById(memberId);
    }

    @DeleteMapping("/{memberId}")
    public void deleteMemberById(
            @PathVariable Long memberId
    ) {
        memberService.deleteMemberById(memberId);
    }
}

```

`ResponseBody`는 메소드의 반환 값을 HTTP 응답 본문으로 직렬화하는 데 사용됩니다. `ResponseEntity`와 달리 HTTP 상태 코드나 헤더를 직접 설정할 수 없습니다. 대신 메소드의 반환 값 자체가 응답 본문이 됩니다. `ResponseBody`는 일반적으로 단순한 응답을 반환할 때 사용됩니다.

- **간편성**: `ResponseBody`는 간단한 응답을 반환할 때 유용합니다. 메소드의 반환 값이 곧 응답 본문이 되므로, 추가 설정이 필요하지 않습니다.
- **직렬화**: 메소드의 반환 값이 JSON이나 XML 등으로 자동 직렬화됩니다.

#### 비교

|특징|ResponseEntity|ResponseBody|
|---|---|---|
|HTTP 상태 코드 설정|가능|불가능|
|응답 헤더 설정|가능|불가능|
|응답 본문 설정|가능|가능|
|사용 용도|복잡한 응답 구성 (상태 코드, 헤더 포함)|단순한 응답 본문 반환|

#### 결론

`ResponseEntity`는 HTTP 응답을 세밀하게 제어하고자 할 때 사용하며, `ResponseBody`는 간단한 데이터를 반환할 때 사용합니다. 복잡한 응답을 요구하는 상황에서는 `ResponseEntity`를 사용하여 상태 코드와 헤더를 설정하는 것이 좋으며, 단순한 데이터 응답의 경우 `ResponseBody`를 사용하는 것이 더 간편할 수 있습니다. 두 클래스를 적절히 활용하여 보다 유연하고 효율적인 RESTful API를 설계할 수 있습니다.
