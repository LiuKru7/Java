# ❗ Global Exception Handling with @ControllerAdvice

This guide explains how to centralize and handle exceptions across your Spring Boot application using `@ControllerAdvice`.

---

## 💡 What is @ControllerAdvice?

`@ControllerAdvice` is a Spring annotation that allows you to write global code for handling exceptions thrown by controller methods. It helps reduce boilerplate and ensures consistent error responses across your API.

---

## ⚠️ Global Exception Handler Example

Handles custom and validation exceptions globally.

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    // Handle custom exception for tenant/property not found
    @ExceptionHandler
    public ResponseEntity<?> tenantOrPropertyNotFoundExceptionHandler(TenantOrPropertyNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                ex.getMessage(),
                new Timestamp(System.currentTimeMillis())
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }

    // Handle validation errors from @Valid annotated requests
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        String errorMessage = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.joining("; "));

        ErrorResponse errorResponse = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                errorMessage,
                new Timestamp(System.currentTimeMillis())
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
}
```

---

## 📦 Error Response Model

A simple POJO class used for returning structured error information.

```java
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private Timestamp timestamp;
}
```

---

## ✅ Benefits of Using @ControllerAdvice

- 📌 Centralized error handling logic
- 🔄 Reusable across multiple controllers
- 💬 Returns informative error messages
- 📐 Clean and consistent API design

---

## 🔚 Example Output

```json
{
  "status": 400,
  "message": "username: must not be blank; password: must be at least 8 characters",
  "timestamp": "2024-04-14T14:05:23.123+00:00"
}
```