---
layout: post
title:  "gRPC testing with .NET Core Test Server"
date:   2020-08-20
image: /images/posts/computer-error.jpg
tags: [.NET Core, gRPC, Testing, Bard.gRPC]
---

Recently I have been doing some work with gRPC in .NET core. I'm a big fan of functional testing against my APIs so I tried to spin up my gRPC API and test it with .NET Core's TestServer when I got this exception.

```
Grpc.Core.RpcException: Status(StatusCode="Internal", Detail="Bad gRPC response. Response protocol downgraded to HTTP/1.1.")
```

After a bit of digging around the Internet I discovered this fix that the problem was due to HTTP Request message version not matching the HTTP Response message version.

<!--more-->


This could be fixed by creating a custom [DelegatingHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.delegatinghandler?view=netcore-3.1) that modified the response version issued by the Test Server.

```c#
    internal class GrpcMessageHandler : DelegatingHandler
    {
        public GrpcMessageHandler(HttpMessageHandler innerHandler) : base(innerHandler)
        {
        }

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            var response = await base.SendAsync(request, cancellationToken);
            
            response.Version = request.Version;

            return response;
        }
    }
```

However as a convenience I have bundled up an extension method in my open source library [Bard.gRPC](https://docs.bard.net.nz/bard/grpc/grpc) which means you can enrich your http client with the added behaviour.

``` c#
 var hostBuilder = new HostBuilder()
                .ConfigureWebHost(builder =>
                    builder
                        .UseStartup<Startup>()
                        .UseTestServer()
                        .UseEnvironment("development"));

            _host = hostBuilder.Start();
            
            var testClient = _host
                .GetTestClient()
                .ForGrpc();
```

I'll blog some more in more detail about my new API testing framework [Bard](https://docs.bard.net.nz/bard/grpc/grpc) in the coming weeks.



