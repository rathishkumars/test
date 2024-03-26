```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;
import java.util.Map;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ApiDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String application;
    private String endpoint;
    private String method;

    @ElementCollection
    private Map<String, String> headers;

    @ElementCollection
    private Map<String, String> params;

    private String bodyType;
    private String requestBody;

    // Getters and setters
}
```

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Embeddable;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Embeddable
public class Header {
    private String key;
    private String value;
}
```

```
@Data
@AllArgsConstructor
@NoArgsConstructor
@Embeddable
public class Param {
    private String key;
    private String value;
}
```
