```
@Bean
    public CommandLineRunner initData(ApiDetailsRepository apiDetailsRepository) {
        return args -> {
            ApiDetails apiDetails1 = new ApiDetails();
            apiDetails1.setApplication("App1");
            apiDetails1.setEndpoint("http://localhost:8080/api/endpoint1");
            apiDetails1.setMethod("GET");
            Map<String, String> headers1 = new HashMap<>();
            headers1.put("Header1", "Value1");
            apiDetails1.setHeaders(headers1);
            Map<String, String> params1 = new HashMap<>();
            params1.put("Param1", "Value1");
            apiDetails1.setParams(params1);
            apiDetails1.setBodyType("json");
            apiDetails1.setRequestBody("{\"key\": \"value\"}");
            apiDetailsRepository.save(apiDetails1);

            ApiDetails apiDetails2 = new ApiDetails();
            apiDetails2.setApplication("App1");
            apiDetails2.setEndpoint("/api/endpoint2");
            apiDetails2.setMethod("POST");
            Map<String, String> headers2 = new HashMap<>();
            headers2.put("Header2", "Value2");
            apiDetails2.setHeaders(headers2);
            Map<String, String> params2 = new HashMap<>();
            params2.put("Param2", "Value2");
            apiDetails2.setParams(params2);
            apiDetails2.setBodyType("form-data");
            apiDetails2.setRequestBody("key1=value1&key2=value2");
            apiDetailsRepository.save(apiDetails2);

            ApiDetails apiDetails3 = new ApiDetails();
            apiDetails3.setApplication("App2");
            apiDetails3.setEndpoint("/api/endpoint3");
            apiDetails3.setMethod("PUT");
            Map<String, String> headers3 = new HashMap<>();
            headers3.put("Header3", "Value3");
            apiDetails3.setHeaders(headers3);
            Map<String, String> params3 = new HashMap<>();
            params3.put("Param3", "Value3");
            apiDetails3.setParams(params3);
            apiDetails3.setBodyType("json");
            apiDetails3.setRequestBody("{\"key\": \"value\"}");
            apiDetailsRepository.save(apiDetails3);
        };
    }
```
