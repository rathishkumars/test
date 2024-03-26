```

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

```$(document).ready(function() {
    var headers = {};
    var params = {};

    // Populate applications dropdown
    var applicationDropdown = $('#applicationDropdown');
    applications.forEach(function(app) {
        applicationDropdown.append('<option value="' + app + '">' + app + '</option>');
    });

    $('#applicationDropdown').change(function() {
        var application = $(this).val();
        if (application) {
            $.get('/endpoints', { application: application }, function(endpoints) {
                var endpointDropdown = $('#endpointDropdown');
                endpointDropdown.empty();
                endpointDropdown.append('<option value="">Select Endpoint</option>');
                endpoints.forEach(function(endpoint) {
                    endpointDropdown.append('<option value="' + endpoint + '">' + endpoint + '</option>');
                });
            });
        } else {
            clearForm();
        }
    });

    $('#endpointDropdown').change(function() {
        var application = $('#applicationDropdown').val();
        var endpoint = $(this).val();
        if (application && endpoint) {
            $.get('/api-details', { application: application, endpoint: endpoint }, function(apiDetails) {
                populateForm(apiDetails);
            });
        } else {
            clearForm();
        }
    });

    $('#addHeaderBtn').click(function() {
        var key = $('#headerKey').val();
        var value = $('#headerValue').val();
        if (key && value) {
            addHeader(key, value);
            $('#headerKey').val('');
            $('#headerValue').val('');
        }
    });

    $('#addParamBtn').click(function() {
        var key = $('#paramKey').val();
        var value = $('#paramValue').val();
        if (key && value) {
            addParam(key, value);
            $('#paramKey').val('');
            $('#paramValue').val('');
        }
    });

    $(document).on('click', '.remove-header', function() {
        var key = $(this).data('key');
        removeHeader(key);
    });

    $(document).on('click', '.remove-param', function() {
        var key = $(this).data('key');
        removeParam(key);
    });

    $('#sendBtn').click(function() {
        var url = $('#url').val();
        var method = $('#httpMethod').val();
        var authToken = $('#authToken').val();
        var bodyType = $('#bodyType').val();
        var requestBody = $('#requestBody').val();

        if (authToken) {
            headers['Authorization'] = 'Bearer ' + authToken;
        }

        var requestData = {
            url: url,
            method: method,
            headers: headers
        };

        if (Object.keys(params).length > 0) {
            requestData.params = params;
        }

        if (bodyType === 'json') {
            requestData.body = requestBody;
        } else if (bodyType === 'form-data') {
            var formData = new FormData();
            var formDataPairs = requestBody.split('&');
            for (var i = 0; i < formDataPairs.length; i++) {
                var pair = formDataPairs[i].split('=');
                var key = decodeURIComponent(pair[0]);
                var value = decodeURIComponent(pair[1]);
                formData.append(key, value);
            }
            requestData.body = formData;
        }

        $.ajax({
            url: '/api/send-request',
            method: 'POST',
            contentType: 'application/json',
            data: JSON.stringify(requestData),
            success: function(response) {
                displayResponse(response);
            },
            error: function(xhr, status, error) {
                displayError(error);
            }
        });
    });
    function populateForm(apiDetails) {
        $('#httpMethod').val(apiDetails.method);
        $('#url').val(apiDetails.endpoint);
        headers = apiDetails.headers || {};
        params = apiDetails.params || {};
        $('#headerList').empty();
        Object.entries(headers).forEach(function([key, value]) {
            addHeader(key, value);
        });
        $('#paramList').empty();
        Object.entries(params).forEach(function([key, value]) {
            addParam(key, value);
        });
        $('#authToken').val(apiDetails.authToken || '');
        $('#bodyType').val(apiDetails.bodyType);
        $('#requestBody').val(apiDetails.requestBody);
    }

    function clearForm() {
        $('#httpMethod').val('GET');
        $('#url').val('');
        headers = {};
        params = {};
        $('#headerList').empty();
        $('#paramList').empty();
        $('#authToken').val('');
        $('#bodyType').val('json');
        $('#requestBody').val('');
        $('#responseBody').text('');
    }

    function addHeader(key, value) {
        $('#headerList').append(`
            <div class="header-item" data-key="${key}">
                <span class="header-key">${key}</span>: <span class="header-value">${value}</span>
                <button class="btn btn-danger btn-sm remove-header" data-key="${key}">Remove</button>
            </div>
        `);
        headers[key] = value;
    }

    function removeHeader(key) {
        delete headers[key];
        $(`#headerList .header-item[data-key="${key}"]`).remove();
    }

    function addParam(key, value) {
        $('#paramList').append(`
            <div class="param-item" data-key="${key}">
                <span class="param-key">${key}</span>: <span class="param-value">${value}</span>
                <button class="btn btn-danger btn-sm remove-param" data-key="${key}">Remove</button>
            </div>
        `);
        params[key] = value;
    }

    function removeParam(key) {
        delete params[key];
        $(`#paramList .param-item[data-key="${key}"]`).remove();
    }

    function displayResponse(response) {
        $('#responseBody').text(response);
    }

    function displayError(error) {
        $('#responseBody').text('Error: ' + error);
    }
});

```

```
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
