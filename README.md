```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>API Testing Tool</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
</head>
<body>
    <div class="container mt-4">
        <h1 class="mb-4">API Testing Tool</h1>
        <div class="row mb-3">
            <div class="col-md-6">
                <select id="applicationDropdown" class="form-control">
                    <option value="">Select Application</option>
                </select>
            </div>
            <div class="col-md-6">
                <select id="endpointDropdown" class="form-control">
                    <option value="">Select Endpoint</option>
                </select>
            </div>
        </div>
        <!-- Rest of the form HTML -->
        <div class="card mb-3">
            <div class="card-header">
                <h5 class="mb-0">Request Body</h5>
            </div>
            <div class="card-body">
                <div class="form-group">
                    <select id="bodyType" class="form-control">
                        <option value="json">JSON</option>
                        <option value="form-data">Form Data</option>
                    </select>
                </div>
                <div class="form-group">
                    <textarea id="requestBody" rows="5" class="form-control"></textarea>
                </div>
            </div>
        </div>
        <button id="sendBtn" class="btn btn-success btn-lg btn-block mb-4">Send</button>
        <div class="card">
            <div class="card-header">
                <h5 class="mb-0">Response</h5>
            </div>
            <div class="card-body">
                <pre id="response" class="pre-scrollable"></pre>
            </div>
        </div>
    </div>

    <script th:inline="javascript">
        // jQuery code will be added here
    </script>
</body>
</html>
```
```
$(document).ready(function() {
    var headers = {};
    var params = {};

    // Load applications on page load
    $.get('/api/applications', function(applications) {
        var applicationDropdown = $('#applicationDropdown');
        applications.forEach(function(application) {
            applicationDropdown.append('<option value="' + application + '">' + application + '</option>');
        });
    });

    $('#applicationDropdown').change(function() {
        var application = $(this).val();
        if (application) {
            // Load endpoints based on selected application
            $.get('/api/endpoints', { application: application, enabled: true }, function(endpoints) {
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
            // Load API details based on selected application and endpoint
            $.get('/api/details', { application: application, endpoint: endpoint, enabled: true }, function(apiDetails) {
                $('#httpMethod').val(apiDetails.method);
                $('#url').val(apiDetails.url);

                // Pre-fill headers
                headers = {};
                $('#headerList').empty();
                apiDetails.headers.forEach(function(header) {
                    addHeader(header.key, header.value);
                });

                // Pre-fill params
                params = {};
                $('#paramList').empty();
                apiDetails.params.forEach(function(param) {
                    addParam(param.key, param.value);
                });

                $('#authToken').val(apiDetails.authToken);
                $('#bodyType').val(apiDetails.bodyType);
                $('#requestBody').val(apiDetails.requestBody);
            });
        } else {
            clearForm();
        }
    });

    $('#sendBtn').click(function() {
        var application = $('#applicationDropdown').val();
        var endpoint = $('#endpointDropdown').val();

        var requestOptions = {
            method: $('#httpMethod').val(),
            url: $('#url').val(),
            headers: headers,
            params: params,
            authToken: $('#authToken').val(),
            bodyType: $('#bodyType').val(),
            requestBody: $('#requestBody').val()
        };

        // Send API request
        $.post('/api/execute',
            {
                application: application,
                endpoint: endpoint,
                requestOptions: JSON.stringify(requestOptions)
            },
            function(response) {
                $('#response').text(JSON.stringify(response, null, 2));
            })
            .fail(function(error) {
                $('#response').text('Error: ' + error.responseJSON.message);
            });
    });

    // Helper functions

    function addHeader(key, value) {
        headers[key] = value;
        $('#headerList').append(
            '<li class="list-group-item">' +
                key + ': ' + value +
                '<button class="btn btn-danger btn-sm float-right removeHeader" data-key="' + key + '">Remove</button>' +
            '</li>'
        );
    }

    function addParam(key, value) {
        params[key] = value;
        $('#paramList').append(
            '<li class="list-group-item">' +
                key + ': ' + value +
                '<button class="btn btn-danger btn-sm float-right removeParam" data-key="' + key + '">Remove</button>' +
            '</li>'
        );
    }

    $(document).on('click', '.removeHeader', function() {
        var key = $(this).data('key');
        delete headers[key];
        $(this).parent().remove();
    });

    $(document).on('click', '.removeParam', function() {
        var key = $(this).data('key');
        delete params[key];
        $(this).parent().remove();
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
        $('#response').text('');
    }
});
```

```
@RestController
@RequestMapping("/api")
public class ApiController {

    private final ApiDetailsRepository apiDetailsRepository;
    private final WebClient webClient;

    public ApiController(ApiDetailsRepository apiDetailsRepository, WebClient webClient) {
        this.apiDetailsRepository = apiDetailsRepository;
        this.webClient = webClient;
    }

    @GetMapping("/applications")
    public List<String> getApplications() {
        return apiDetailsRepository.findDistinctApplications();
    }

    @GetMapping("/endpoints")
    public List<String> getEndpoints(@RequestParam String application, @RequestParam boolean enabled) {
        return apiDetailsRepository.findByApplicationAndEnabledTrue(application)
            .stream()
            .map(ApiDetails::getEndpoint)
            .collect(Collectors.toList());
    }

    @GetMapping("/details")
    public ApiDetails getApiDetails(@RequestParam String application, @RequestParam String endpoint, @RequestParam boolean enabled) {
        return apiDetailsRepository.findByApplicationAndEndpointAndEnabledTrue(application, endpoint);
    }

    @PostMapping("/execute")
    public ResponseEntity<?> executeRequest(@RequestBody ExecuteRequestDto executeRequestDto) {
        RequestOptions requestOptions = new ObjectMapper().convertValue(executeRequestDto.getRequestOptions(), RequestOptions.class);

        return webClient.method(HttpMethod.valueOf(requestOptions.getMethod()))
            .uri(requestOptions.getUrl())
            .headers(headers -> requestOptions.getHeaders().forEach(headers::add))
            .params(params -> requestOptions.getParams().forEach(params::add))
            .bodyValue(requestOptions.getRequestBody())
            .retrieve()
            .toEntity(String.class)
            .block();
    }

    @Data
    private static class ExecuteRequestDto {
        private String application;
        private String endpoint;
        private String requestOptions;
    }

    @Data
    private static class RequestOptions {
        private String method;
        private String url;
        private Map<String, String> headers;
        private Map<String, String> params;
        private String authToken;
        private String bodyType;
        private String requestBody;
    }
}
```
