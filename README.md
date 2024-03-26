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
    <div class="card mb-3">
        <div class="card-header">
            <h5 class="mb-0">Request</h5>
        </div>
        <div class="card-body">
            <div class="form-group">
                <label for="httpMethod">HTTP Method:</label>
                <select id="httpMethod" class="form-control">
                    <option value="GET">GET</option>
                    <option value="POST">POST</option>
                    <option value="PUT">PUT</option>
                    <option value="DELETE">DELETE</option>
                </select>
            </div>
            <div class="form-group">
                <label for="url">URL:</label>
                <input type="text" id="url" class="form-control" placeholder="Enter URL">
            </div>
        </div>
    </div>
    <div class="card mb-3">
        <div class="card-header">
            <h5 class="mb-0">Headers</h5>
        </div>
        <div class="card-body">
            <div id="headerList"></div>
            <div class="form-group">
                <input type="text" id="headerKey" class="form-control" placeholder="Key">
            </div>
            <div class="form-group">
                <input type="text" id="headerValue" class="form-control" placeholder="Value">
            </div>
            <button id="addHeaderBtn" class="btn btn-primary">Add Header</button>
        </div>
    </div>
    <div class="card mb-3">
        <div class="card-header">
            <h5 class="mb-0">Parameters</h5>
        </div>
        <div class="card-body">
            <div id="paramList"></div>
            <div class="form-group">
                <input type="text" id="paramKey" class="form-control" placeholder="Key">
            </div>
            <div class="form-group">
                <input type="text" id="paramValue" class="form-control" placeholder="Value">
            </div>
            <button id="addParamBtn" class="btn btn-primary">Add Parameter</button>
        </div>
    </div>
    <div class="card mb-3">
        <div class="card-header">
            <h5 class="mb-0">Authorization</h5>
        </div>
        <div class="card-body">
            <div class="form-group">
                <input type="text" id="authToken" class="form-control" placeholder="Bearer Token">
            </div>
        </div>
    </div>
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
    <button id="sendBtn" class="btn btn-primary">Send</button>
    <div class="card mt-3">
        <div class="card-header">
            <h5 class="mb-0">Response</h5>
        </div>
        <div class="card-body">
            <pre id="responseBody"></pre>
        </div>
    </div>
</div>
<script th:inline="javascript">
    /*<![CDATA[*/
    var applications = /*[[${applications}]]*/ [];
    /*]]>*/
</script>
<script th:src="@{/js/app.js}"></script>
</body>
</html>
```
