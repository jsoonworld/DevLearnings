# Exception 관리
1차 스프린트에서는 계층형으로 구현하였고 Exception 관련해서는 다음과 같이 분류를 하였다.

![](https://velog.velcdn.com/images/harperkwon/post/d09031a3-c42d-42fe-90e1-d9ab0352b5b9/image.png)

1차 스프린트를 하면서 Exception 관련해서 구현과 확장에 있어서 타협한 부분도 있고 고민을 충분하게 해보지 못한 것 같아서 이번에 다시 정리하면서 1차 스프린트에서 구현한 방식이 최선이었을까? 혹은 다음 스프린트로 넘어갈 때 팀의 상황을 고려한다면 어떻게 구현하는 것이 좋을까 고민해 보고자 한다.

## GlobalExceptionHandler
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ErrorResponse> handleCustomException(CustomException e) {
        return ResponseEntity
                .badRequest()
                .body(ErrorResponse.of(
                        e.getErrorMessage().getStatus(),
                        e.getErrorMessage().getMessage()
                ));
    }
}
```

## CustomException
```java
@Getter
public class CustomException extends RuntimeException {

    private ErrorMessage errorMessage;

    public CustomException(ErrorMessage errorMessage) {
        super(errorMessage.getMessage());
        this.errorMessage = errorMessage;
    }

}
```

## ErrorResponse
```java
@Builder
public record ErrorResponse(
        int status,
        String message
) {
    public static ErrorResponse of(int status, String message) {
        return ErrorResponse.builder()
                .status(status)
                .message(message)
                .build();
    }
}
```

## SuccessResponse
```java
@JsonPropertyOrder({"status", "message", "result"})
public record SuccessResponse<T>(
        int status,
        String message,
        @JsonInclude(JsonInclude.Include.NON_NULL)
        T result
) {
    public static <T> SuccessResponse of(SuccessMessage successMessage) {
        return new SuccessResponse(successMessage.getStatus(), successMessage.getMessage(), null);
    }

    public static <T> SuccessResponse of(SuccessMessage successMessage, T result) {
        return new SuccessResponse(successMessage.getStatus(), successMessage.getMessage(), result);
    }
}
```

## ErrorMessage
```java
@Getter
@AllArgsConstructor
public enum ErrorMessage {
    ;
    private final int status;
    private final String message;
}
```

## SuccessMessage
```java
@Getter
@AllArgsConstructor
public enum SuccessMessage {

    ;

    private final int status;
    private final String message;
}

```

## 고민
```java
@Builder
public record ErrorResponse(
        int status,
        String message
) {
    public static ErrorResponse of(int status, String message) {
        return ErrorResponse.builder()
                .status(status)
                .message(message)
                .build();
    }
}
```
다음 코드를 보면 status를 반환하고 있고 message에서는 다음과 같은 enum을 보낸다.
```java
INVALID_TOKEN(401, "유효하지 않은 토큰입니다"),
```

status가 과연 필요한 것일까 궁금하다. 메시지 내용에 따라 status가 정해지는 것이 아닌가? 또한 Controller에서 ResponseEntity를 사용하는데 중복으로 명시를 해줘야 하는 것일까?
```java
return ResponseEntity.ok
```

```java
import org.springframework.lang.Nullable;

public enum HttpStatus implements HttpStatusCode {
    CONTINUE(100, HttpStatus.Series.INFORMATIONAL, "Continue"),
    SWITCHING_PROTOCOLS(101, HttpStatus.Series.INFORMATIONAL, "Switching Protocols"),
    PROCESSING(102, HttpStatus.Series.INFORMATIONAL, "Processing"),
    EARLY_HINTS(103, HttpStatus.Series.INFORMATIONAL, "Early Hints"),
    /** @deprecated */
    @Deprecated(
        since = "6.0.5"
    )
    CHECKPOINT(103, HttpStatus.Series.INFORMATIONAL, "Checkpoint"),
    OK(200, HttpStatus.Series.SUCCESSFUL, "OK"),
    CREATED(201, HttpStatus.Series.SUCCESSFUL, "Created"),
    ACCEPTED(202, HttpStatus.Series.SUCCESSFUL, "Accepted"),
    NON_AUTHORITATIVE_INFORMATION(203, HttpStatus.Series.SUCCESSFUL, "Non-Authoritative Information"),
    NO_CONTENT(204, HttpStatus.Series.SUCCESSFUL, "No Content"),
    RESET_CONTENT(205, HttpStatus.Series.SUCCESSFUL, "Reset Content"),
    PARTIAL_CONTENT(206, HttpStatus.Series.SUCCESSFUL, "Partial Content"),
    MULTI_STATUS(207, HttpStatus.Series.SUCCESSFUL, "Multi-Status"),
    ALREADY_REPORTED(208, HttpStatus.Series.SUCCESSFUL, "Already Reported"),
    IM_USED(226, HttpStatus.Series.SUCCESSFUL, "IM Used"),
    MULTIPLE_CHOICES(300, HttpStatus.Series.REDIRECTION, "Multiple Choices"),
    MOVED_PERMANENTLY(301, HttpStatus.Series.REDIRECTION, "Moved Permanently"),
    FOUND(302, HttpStatus.Series.REDIRECTION, "Found"),
    /** @deprecated */
    @Deprecated
    MOVED_TEMPORARILY(302, HttpStatus.Series.REDIRECTION, "Moved Temporarily"),
    SEE_OTHER(303, HttpStatus.Series.REDIRECTION, "See Other"),
    NOT_MODIFIED(304, HttpStatus.Series.REDIRECTION, "Not Modified"),
    /** @deprecated */
    @Deprecated
    USE_PROXY(305, HttpStatus.Series.REDIRECTION, "Use Proxy"),
    TEMPORARY_REDIRECT(307, HttpStatus.Series.REDIRECTION, "Temporary Redirect"),
    PERMANENT_REDIRECT(308, HttpStatus.Series.REDIRECTION, "Permanent Redirect"),
    BAD_REQUEST(400, HttpStatus.Series.CLIENT_ERROR, "Bad Request"),
    UNAUTHORIZED(401, HttpStatus.Series.CLIENT_ERROR, "Unauthorized"),
    PAYMENT_REQUIRED(402, HttpStatus.Series.CLIENT_ERROR, "Payment Required"),
    FORBIDDEN(403, HttpStatus.Series.CLIENT_ERROR, "Forbidden"),
    NOT_FOUND(404, HttpStatus.Series.CLIENT_ERROR, "Not Found"),
```

이렇게 다 담을 수 없을 정도로 남겨져있고 활용할 수 있는데..라는 생각이 든다. 물론 Custom 하는 것에서 오는 장점도 있고 반복을 함으로써 강조의 효과도 있다고 생각한다.. 정답은 그 중간일까..?
다만, 이 코드를 읽을 서버 파트 동료, 나의 코드를 전달받을 수도 있는 사람에게 코드를 보고 명확하게 파악할 수 있도록 명시해 주는 것이 좋다는 생각이 다시 든다.

```java
@Getter
@AllArgsConstructor
public enum ErrorMessage {

    // 소셜 로그인
    INVALID_TOKEN(401, "유효하지 않은 토큰입니다"),
    INVALID_KEY(401, "유효하지 않은 키입니다"),
    FAILED_SOCIAL_LOGIN(404, "소셜 로그인애 실패하였습니다"),
    INVALID_USER(404, "유효하지 않은 유저입니다"),
    FAILED_TOKEN_REISSUE(404, "토큰 재발급에 실패하였습니다"),
    UNAUTHORIZED_JWT_EXCEPTION(401, "유효하지 않은 토큰입니다"),
```

에러 메시지를 표기할 때, 범주가 큰 순으로 정리를 했다. 예를 들어 
`FAILED_TOKEN_REISSUE`를 보면 `FAILED` > `TOKEN` > `REISSUE` 순으로 범주가 줄어드는 것을 볼 수 있을 것 같다. 이렇게 분류를 한 이유는 확장성을 고려해서 메시지를 나누고 관리할 때 범주가 큰 순부터 분류를 하면 용이할 것 같아서이다. 클라이언트(web, app) 사람들에게 어떤 표현, 순서가 직관적이고 편한지 물어봤을 때, 이 순서를 선호한다고 말씀도 해주셨다. 서버 입장에서도 이렇게 표기하고 관리하는 것이 용이할까!? 다시 한번 생각해 보고 싶다.

```java
@Builder
public record ErrorResponse(
        int status,
        String message
) {
    public static ErrorResponse of(int status, String message){
        return ErrorResponse.builder()
                .status(status)
                .message(message)
                .build();
    }

    public static ErrorResponse of(int status, String message, BindingResult bindingResult){
        return ErrorResponse.builder()
                .status(status)
                .message(message)
                .build();
    }

    /**
     * Q: record 에서는 getter 를 만들어 주는데 가독성을 위해서 남겨둘지 말지
     */
    @Getter
    public static class ValidationError {
        private final String field;
        private final String message;

        private ValidationError(FieldError fieldError){
            this.field = fieldError.getField();
            this.message = fieldError.getDefaultMessage();
        }

        public static List<ValidationError> of(final BindingResult bindingResult){
            return bindingResult.getFieldErrors().stream()
                    .map(ValidationError::new)
                    .toList();
        }
    }
}
```

Exception` 에서 고민되는 마지막 부분이다. `ValidationError` 첫 번째로 record type인데 Getter를 써줘야 할까? 가시성으로. 두 번째로 `ValidationError`의 필요성이다. Getter 사용에 관해서는 record를 이해하고 사용하는 입장에서는 필요가 없다는 생각이 든다. 두 번째 고민에 대해서는 아직 명확하게 필요성을 느끼고 있지 않아서 고민이 든다.




