```
import com.example.api.model.ApiDetails;
import com.example.api.repository.ApiDetailsRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.util.UriBuilder;
import reactor.core.publisher.Mono;

import java.util.List;
import java.util.Map;

@Controller
public class ApiController {
    @Autowired
    private ApiDetailsRepository apiDetailsRepository;

    @GetMapping("/api/endpoint1")
    public ResponseEntity<String> endpoint1(
            @RequestHeader("Header1") String headerValue,
            @RequestParam(name = "Param1", required = false) String paramValue
    ) {
        // Process the request and generate the response
        String responseBody = "Response from /api/endpoint1 - Header1: " + headerValue;
        if (paramValue != null) {
            responseBody += ", Param1: " + paramValue;
        }
        return ResponseEntity.ok(responseBody);
    }

    @GetMapping("/")
    public String index(Model model) {
        List<String> applications = apiDetailsRepository.findAll().stream()
                .map(ApiDetails::getApplication)
                .distinct()
                .toList();
        model.addAttribute("applications", applications);
        return "index";
    }

    @GetMapping("/endpoints")
    @ResponseBody
    public List<String> getEndpoints(@RequestParam String application) {
        return apiDetailsRepository.findByApplication(application).stream()
                .map(ApiDetails::getEndpoint)
                .toList();
    }

    @GetMapping("/api-details")
    @ResponseBody
    public ApiDetails getApiDetails(@RequestParam String application, @RequestParam String endpoint) {
        return apiDetailsRepository.findByApplicationAndEndpoint(application, endpoint);
    }

    @PostMapping("/api/send-request")
    @ResponseBody
    public Mono<String> sendRequest(@RequestBody Map<String, Object> requestData) {
        String url = (String) requestData.get("url");
        String method = (String) requestData.get("method");
        Map<String, String> headers = (Map<String, String>) requestData.get("headers");
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        if (requestData.containsKey("params")) {
            Map<String, String> paramMap = (Map<String, String>) requestData.get("params");
            paramMap.forEach(params::add);
        }
        Object body = requestData.get("body");

        System.out.println("Request URL: " + url);
        System.out.println("Request Method: " + method);
        System.out.println("Request Headers: " + headers);
        System.out.println("Request Params: " + params);
        System.out.println("Request Body: " + body);

        WebClient.RequestBodySpec requestSpec = WebClient.create()
                .method(HttpMethod.valueOf(method))
                .uri(uriBuilder -> {
                    UriBuilder builder = uriBuilder.path(url);
                    params.forEach(builder::queryParam);
                    return builder.build();
                });

        if (headers != null) {
            requestSpec.headers(httpHeaders -> headers.forEach(httpHeaders::add));
        }

        if (body != null) {
            if (body instanceof String) {
                requestSpec.contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(body);
            } else if (body instanceof Map) {
                MultiValueMap<String, String> formData = new LinkedMultiValueMap<>();
                ((Map<String, String>) body).forEach(formData::add);
                requestSpec.contentType(MediaType.APPLICATION_FORM_URLENCODED)
                        .body(BodyInserters.fromFormData(formData));
            }
        }

        return requestSpec.retrieve()
                .bodyToMono(String.class)
                .doOnNext(response -> System.out.println("Response: " + response))
                .doOnError(error -> System.out.println("Error: " + error.getMessage()));
    }
}
```
