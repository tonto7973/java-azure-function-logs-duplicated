Sample project to demonstrate [duplicate logs](https://github.com/Azure/azure-functions-java-worker/issues/385) in Java Azure Functions.

- Plan: Consumption
- OS: Windows
- Runtime version: ~3
- Stack: Java
- Java Version: 8
- Platform: 32bit
- [Distributed Tracing](https://github.com/Azure/azure-functions-java-worker/wiki/Distributed-Tracing-for-Java-Azure-Functions): enabled
- Function Region: East US
- App Insights Region: East US

Sample code:
```java
public class HttpTriggerFunction {

    @FunctionName("HttpTrigger-Java")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS) HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {

        context.getLogger().log(Level.INFO, "I: This is an info");
        context.getLogger().log(Level.WARNING, "W: This is a warning {0}", "arg1");
        context.getLogger().log(Level.SEVERE, "S: This is an error {0}", 123);

        return request.createResponseBuilder(HttpStatus.OK).body("Done. Check the logs.").build();
    }
}
```

Settings:

```json
{
    "WEBSITE_RUN_FROM_PACKAGE": "1",
    "APPLICATIONINSIGHTS_CONNECTION_STRING": "InstrumentationKey=c12f9edb...",
    "APPINSIGHTS_INSTRUMENTATIONKEY": "c12f9edb...",
    "FUNCTIONS_EXTENSION_VERSION": "~3",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=09cbe5135baf4858bee9...",
    "WEBSITE_CONTENTSHARE": "azure-function-examples-200910102416",
    "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "DefaultEndpointsProtocol=https;AccountName=09cbe5135baf4858bee9...",
    // distributed tracing enabled
    "XDT_MicrosoftApplicationInsights_Java": "1",    
    "ApplicationInsightsAgent_EXTENSION_VERSION": "~2"
  }
```

Duplicate logs:
```csv
timestamp,message,severityLevel,itemType
"2020-09-10T09:47:56.6601788Z","Executing 'Functions.HttpTrigger-Java' (Reason='This function was programmatically called via the host APIs.', Id=0da012f4-c042-4aad-b67f-e8d85987d1c6)",1,trace
"2020-09-10T09:47:58.394Z","W: This is a warning arg1",2,trace
"2020-09-10T09:47:58.414Z","S: This is an error 123",3,trace
"2020-09-10T09:47:58.5958137Z","I: This is an info",1,trace
"2020-09-10T09:47:58.5962861Z","W: This is a warning {0}",2,trace
"2020-09-10T09:47:58.7652682Z","S: This is an error {0}",3,trace
"2020-09-10T09:47:58.765617Z","Function ""HttpTrigger-Java"" (Id: 0da012f4-c042-4aad-b67f-e8d85987d1c6) invoked by Java Worker",1,trace
"2020-09-10T09:47:58.9299472Z","Executed 'Functions.HttpTrigger-Java' (Succeeded, Id=0da012f4-c042-4aad-b67f-e8d85987d1c6, Duration=2329ms)",1,trace
```