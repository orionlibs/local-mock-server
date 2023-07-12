# Orion Local Mock Server
Java library facilitating mocking server responses that can be used by testing frameworks.

Please check the wiki

https://github.com/orionlibs/local-mock-server/wiki

You can import the library by using:  
```xml
<dependency>
    <groupId>io.github.orionlibs</groupId>
    <artifactId>local-mock-server</artifactId>
    <version>1.0.0</version>
</dependency>
```

To use this mock server, in your JUnit tests, for example, you can create a mock API class that could represent a real Spring MVC controller, for instance like:
```java
package io.github.orionlibs.local_mock_server.utils;

import java.io.IOException;
import okhttp3.Headers;
import okhttp3.HttpUrl;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

/**
 * It represents an API endpoint. It could be a Spring MVC Controller class that has controller methods.
 * This class is used by the test class of this library as a mock API endpoint
 */
public class MockAPI
{
    public static String testEndpoint(HttpUrl apiUrl) throws IOException
    {
        return testEndpoint(apiUrl, null);
    }


    public static String testEndpoint(HttpUrl apiUrl, Headers headers) throws IOException
    {
        OkHttpClient client = new OkHttpClient();
        Request.Builder requestBuilder = new Request.Builder();
        requestBuilder.url(apiUrl);
        if(headers != null)
        {
            requestBuilder.headers(headers);
        }
        Request request = requestBuilder.build();
        try(Response response = client.newCall(request).execute())
        {
            return response.body().string();
        }
    }
}
```
and then you can create a JUnit 5 test like:
```java
private LocalMockServer server;


@AfterEach
public void teardown() throws ServerShutdownException
{
    server.shutdown();
}


@Test
void test_LocalMockServer() throws Exception
{
    String url1ToCall = "http://127.0.0.1";
    int port1 = 80;
    String responseBody = "{\"status\" : \"OK\"}";
    server = new LocalMockServer(url1ToCall, port1, responseBody);
    assertEquals(responseBody, MockAPI.testEndpoint(server.getBaseURL()));
    server.shutdown();
    String url2ToCall = "http://127.0.0.1:8081";
    int port2 = 8081;
    server = new LocalMockServer(url2ToCall, port2, responseBody);
    assertEquals(responseBody, MockAPI.testEndpoint(server.getBaseURL()));
    assertEquals("/", server.getURI());
}
```