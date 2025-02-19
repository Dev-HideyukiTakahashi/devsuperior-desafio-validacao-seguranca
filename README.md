## 🔨 **Referência para Spring Validation**
---

### ⚙️ ConstraintValidator Customizado Exemplo

* Criando uma Annotation
* Exemplo com UserInsertValidator.Class

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

@Constraint(validatedBy = UserInsertValidator.class) 
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface UserInsertValid {
  String message() default "Validation error";
  Class<?>[] groups() default {};
  Class<? extends Payload>[] payload() default {};
}
```
* Criando o Validator
* Exemplo com UserInsertDTO.Class

```java
import java.util.ArrayList;
import java.util.List;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.devsuperior.dscatalog.dto.UserInsertDTO;
import com.devsuperior.dscatalog.entities.User;
import com.devsuperior.dscatalog.repositories.UserRepository;
import com.devsuperior.dscatalog.resources.exceptions.FieldMessage;

public class UserInsertValidator implements ConstraintValidator<UserInsertValid, UserInsertDTO> {
  @Override
  public void initialize(UserInsertValid ann) {
  }

  @Override
  public boolean isValid(UserInsertDTO dto, ConstraintValidatorContext context) {
    List<FieldMessage> list = new ArrayList<>();

    // Coloque aqui seus testes de validação, acrescentando objetos FieldMessage à lista
    // Como exemplo, se email já existe
    User user = userRepository.findByEmail(dto.getEmail());
    if (user != null) {
      list.add(new FieldMessage("email", "Email já existe"));
    }

    for (FieldMessage e : list) {
      context.disableDefaultConstraintViolation();
      context.buildConstraintViolationWithTemplate(e.getMessage())
             .addPropertyNode(e.getFieldName())
             .addConstraintViolation();
    }

    return list.isEmpty();
  }
}
```

---

### 🛠️ **Tratando erros de Validation**
  * Criando uma exception personalizada para response

#### Custom Error

```java
public class CustomError {

  private Instant timestamp;
  private Integer status;
  private String error;
  private String path;

  constructors, getters ...
}
```

#### Field Message

```java
public class FieldMessage {

  private String fieldName;
  private String message;

  constructors, getters ...
}
```

#### Validation Error

```java
public class ValidationError extends CustomError {

  private List<FieldMessage> errors = new ArrayList<>();

  public ValidationError(Instant timestamp, Integer status, String error, String path) {
    super(timestamp, status, error, path);
  }

  public List<FieldMessage> getErrors() {
    return errors;
  }

  public void addError(String fieldName, String message) {
    errors.removeIf(x -> x.getFieldName().equals(fieldName));
    errors.add(new FieldMessage(fieldName, message));
  }

}
```

#### Handler
```java
@ControllerAdvice
public class ControllerExceptionHandler {

  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<CustomError> methodArgumentNotValid(MethodArgumentNotValidException e,
      HttpServletRequest request) {
    Instant timestamp = Instant.now();
    Integer status = HttpStatus.UNPROCESSABLE_ENTITY.value();
    String err = "Dados inválidos!";
    String path = request.getRequestURI();
    ValidationError error = new ValidationError(timestamp, status, err, path);
    e.getFieldErrors().forEach(x -> error.addError(x.getField(), x.getDefaultMessage()));

    return ResponseEntity.status(status).body(error);
  }

```
