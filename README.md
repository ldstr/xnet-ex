# xNet-Ex
A work-in-progress "extended" version of the popular (unmaintained) xNet fork, Leaf.xNet.

# TODO:
- Fix nullable issues/fully migrate to .NET Core
- Migrate from/fix obsolete and deprecated system classes
- Add full async/await support
- Use JsonSerializer (System.Text.Json) for saving/loading cookies
- Add AOT support
- Translate code docs into English (en-US)
- Move the `HttpResponse` indexer to the `Headers` property and implement IEnumerable for it
- Implement new property `StoreResponseCookies` for `HttpRequest`: `HttpResponse` should have `Cookies` as `IReadOnlyKeyValueCollection<string,Cookie>` with indexer.
- Add cross-platform support

# Features
### HTTP Methods
- GET
- POST
- PATCH
- DELETE
- PUT
- OPTIONS

### Keep temporary headers (when redirected)
It's enabled by default. But you can disable this behavior:
```csharp
httpRequest.KeepTemporaryHeadersOnRedirect = false;
httpRequest.AddHeader(HttpHeader.Referer, "https://google.com");
httpRequest.Get("http://google.com").None();
// After redirection to www.google.com - request won't have Referer header because KeepTemporaryHeadersOnRedirect = false
```

### Middle response headers (when redirected)
```csharp
httpRequest.EnableMiddleHeaders = true;

// This requrest has a lot of redirects
var resp = httpRequest.Get("https://account.sonyentertainmentnetwork.com/");
var md = resp.MiddleHeaders;
```

### Cross Domain Cookies
Used native cookie storage from .NET with domain shared access support.  
Cookies enabled by default. If you wait to disable parsing it use:
```csharp
HttpRequest.UseCookies = false;
```
Cookies now escaping values. If you wait to disable it use:
```csharp
HttpRequest.Cookies.EscapeValuesOnReceive = false;

// UnescapeValuesOnSend by default = EscapeValuesOnReceive
// so set if to false isn't necessary
HttpRequest.Cookies.UnescapeValuesOnSend = false;
```

### Select SSL Protocols (downgrade when required)
```csharp
// By Default (SSL 2 & 3 not used)
httpRequest.SslProtocols = SslProtocols.Tls | SslProtocols.Tls12 | SslProtocols.Tls11;
```

### My HTTPS proxy returns bad response
Sometimes HTTPS proxy require relative address instead of absolute.
This behavior can be changed:
```csharp
http.Proxy.AbsoluteUriInStartingLine = false;
```

### Modern User-Agent Randomization
UserAgents were updated in January 2019.
```csharp
httpRequest.UserAgentRandomize();
// Call it again if you want change it again

// or set property
httpRequest.UserAgent = Http.RandomUserAgent();
```
When you need a specific browser just use the `Http` class same way:
- `ChromeUserAgent()`
- `FirefoxUserAgent()`
- `IEUserAgent()`
- `OperaUserAgent()`
- `OperaMiniUserAgent()`

## Cyrilic and Unicode Form parameters
```csharp
var urlParams = new RequestParams {
    { ["привет"] = "мир"  },
    { ["param2"] = "val2" }
}
// Or
// urlParams["привет"] = "мир";
// urlParams["param2"] = "val2";

string content = request.Post("https://google.com", urlParams).ToString();
```

## A lot of Substring functions
```csharp
string title = html.Substring("<title>", "</title>");

// substring or default
string titleWithDefault  = html.Substring("<title>", "</title>") ?? "Nothing";
string titleWithDefault2 = html.Substring("<title>", "</title>", fallback: "Nothing");

// substring or empty
string titleOrEmpty  = html.SubstringOrEmpty("<title>", "</title>");
string titleOrEmpty2 = html.Substring("<title>", "</title>") ?? ""; // "" or string.Empty
string titleOrEmpty3 = html.Substring("<title>", "</title>", fallback: string.Empty);

// substring or thrown exception when not found
// it will throw new SubstringException with left and right arguments in the message
string titleOrException  = html.SubstringEx("<title>", "</title>");
// when you need your own Exception
string titleOrException2 = html.Substring("<title>", "</title>")
    ?? throw MyCustomException();


```

# How to:
### Get started
Add in the beggining of file.
```csharp
using Leaf.xNet;
```
And use one of this code templates:

```csharp
using (var request = new HttpRequest()) {
    // Do something
}

// Or
HttpRequest request = null;
try {
    request = new HttpRequest();
    // Do something 
}
catch (HttpException ex) {
    // Http error handling
    // You can use ex.Status or ex.HttpStatusCode for more details.
}
catch (Exception ex) {
	// Unhandled exceptions
}
finally {
    // Cleanup in the end if initialized
    request?.Dispose();
}

```

### Send multipart requests with fields and files
Methods `AddField()` and `AddFile()` has been removed (unstable).
Use this code:
```csharp
var multipartContent = new MultipartContent()
{
    {new StringContent("Harry Potter"), "login"},
    {new StringContent("Crucio"), "password"},
    {new FileContent(@"C:\hp.rar"), "file1", "hp.rar"}
};

// When response isn't required
request.Post("https://google.com", multipartContent).None();

// Or
var resp = request.Post("https://google.com", multipartContent);
// And then read as string
string respStr = resp.ToString();
```

### Get page source (response body) and find a value between strings
```csharp
string html = request.Get("https://google.com").ToString();
string title = html.Substring("<title>", "</title>");
```

### Get response headers
```csharp
var httpResponse = httpRequest.Get("https://yoursever.com");
string responseHeader = httpResponse["X-User-Authentication-Token"];
```

### Download a file
```csharp
var resp = request.Get("http://google.com/file.zip");
resp.ToFile("C:\\myDownloadedFile.zip");
```

### Get Cookies
```csharp
string response = request.Get("https://twitter.com/login").ToString();
var cookies = request.Cookies.GetCookies("https://twitter.com");
foreach (Cookie cookie in cookies) {
    // concat your string or do what you want
    Console.WriteLine($"{cookie.Name}: {cookie.Value}");
}
```

### Proxy
Your proxy server:
```csharp
// Type: HTTP / HTTPS 
httpRequest.Proxy = HttpProxyClient.Parse("127.0.0.1:8080");
// Type: Socks4
httpRequest.Proxy = Socks4ProxyClient.Parse("127.0.0.1:9000");
// Type: Socks4a
httpRequest.Proxy = Socks4aProxyClient.Parse("127.0.0.1:9000");
// Type: Socks5
httpRequest.Proxy = Socks5ProxyClient.Parse("127.0.0.1:9000");

```

Debug proxy server (Charles / Fiddler):
```csharp
// HTTP / HTTPS (by default is HttpProxyClient at 127.0.0.1:8888)
httpRequest.Proxy = ProxyClient.DebugHttpProxy;

// Socks5 (by default is Socks5ProxyClient at 127.0.0.1:8889)
httpRequest.Proxy = ProxyClient.DebugSocksProxy;
```

### Add a Cookie to HttpRequest.Cookies storage
```csharp
request.Cookies.Set(string name, string value, string domain, string path = "/");

// or
var cookie = new Cookie(string name, string value, string domain, string path);
request.Cookies.Set(cookie);
```
