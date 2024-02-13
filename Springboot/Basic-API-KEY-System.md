# 🗝️Basic API Key System
---
## ⚠️ Requirements
- Springboot project
- One or more working endpoints
- Dependencies: 
  - spring-boot-starter-web
---

### 1. API Key Config
First of all we need to create a file named **ApiKeyConfig.java** where we are going to have the basic configuration. 

In this file we will define the methods to load & check if the API Key is valid. This is a basic example:

```
// imports


@Configuration
public class ApiKeyConfig {

    private final JdbcTemplate jdbcTemplate;
    private final List<String> apiKeys;
    @Autowired
    public ApiKeyConfig(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        this.apiKeys = loadApiKeys();
    }

    private List<String> loadApiKeys(){
        List<String> results = new ArrayList<>();
        String sql = "SELECT id, value FROM api_keys";
        jdbcTemplate.query(sql, rs -> {
            String apiKey = rs.getString("value");
            results.add(apiKey);
        });
        return results;
    }

    public boolean isValidApiKey(String apiKey) {
        return apiKeys.contains(apiKey);
    }

}
```
In this case we will use a basic database connection where we stored the valid keys.

### 2. Interceptor
Once we have the basic API Key configuration created we need to have to intercept all the requests and use the configuration methos to do the needed validations.

**ApiKeyInterceptor.java**

```
// imports

@Component
public class ApiKeyInterceptor implements HandlerInterceptor {

    private final ApiKeyConfig apiKeyConfig;

    @Autowired
    public ApiKeyInterceptor(ApiKeyConfig apiKeyConfig) {
        this.apiKeyConfig = apiKeyConfig;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String apiKey = request.getHeader("X-API-Key");

        if (apiKey == null || !apiKeyConfig.isValidApiKey(apiKey)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        // For example, you can compare requiredPermissions with the requested endpoint permissions

        return true;
    }
}
```

### 3. WebMvcConfig
The last point is to add the interceptor to the needed api endpoints:

```
// imports

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Autowired
    private ApiKeyInterceptor apiKeyInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(apiKeyInterceptor)
                .addPathPatterns("/api/**");
    }
}
```

### 4. Result
Now to check if the API Key System is working you just have to add the **"X-API-Key" Header** on the Request **with a valid key** on its value and see if its wokring.

If you do a request without using the key the response will be Unauthorized.

