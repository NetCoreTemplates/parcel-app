# parcel-webapp

.NET Core 2.1 Templates WebApp with Parcel + TypeScript

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/csharp-templates/parcel-webapp.png)](http://parcel-webapp.web-templates.io/)

> Browse [source code](https://github.com/NetCoreTemplates/parcel-webapp), view live demo [parcel-webapp.web-templates.io](http://parcel-webapp.web-templates.io) and install with [dotnet-new](http://docs.servicestack.net/dotnet-new):

    $ npm install -g @servicestack/cli

    $ dotnet-new parcel-webapp ProjectName

## About

This project combines the simplicity of developing modern JavaScript Apps with the [parcel template](https://github.com/NetCoreTemplates/parcel) with the simplicity of developing .NET Core Apps with [ServiceStack Templates Web Apps](http://templates.servicestack.net/docs/web-apps) to provide a unified solution for creating rich Apps in a live rapid development workflow without compilation allowing the creation of 
[pure Cloud Apps](http://templates.servicestack.net/docs/web-apps#pure-cloud-apps) and [simplified deployments](http://templates.servicestack.net/docs/deploying-web-apps).

## Layout

This template is maintained in the following directory structure:

 - `/app` - Your Web App's published source code and any plugins
 - `/client` - The Parcel managed Client App where client source code is maintained
 - `/server` - Extend your Web App with an optional `server.dll` plugin containing additional Server functionality
 - `/web` - The Web App binaries

Most development will happen within `/client` which is automatically published to `/app` using partial

### client

The difference with [templates-webapp](https://github.com/NetCoreTemplates/templates-webapp) is that the client source code is maintained in the `/client` folder and uses [Parcel JS](https://parceljs.org) to automatically bundle and publish your App to `/app/wwwroot` which is updated with live changes during development.

The client folder also contains the npm [package.json](https://github.com/NetCoreTemplates/parcel-webapp/blob/master/client/package.json) which contains all npm scripts required during development.

If this is the first time using Parcel, you will need to install its global CLI:

    $ npm install -g parcel-bundler

Then you can run a watched parcel build of your Web App with:

    $ npm run dev

Parcel is a zero configuration bundler which inspects your `.html` files to automatically transpile and bundle all your **.js** and **.css** assets and other web resources like TypeScript **.ts** source files into a static `.html` website synced at `/app/wwwroot`.

Then to start the ServiceStack Server to host your Web App run:

    $ npm run server

Which will host your App at `http://localhost:5000` which in `debug` mode will enable [hot reloading](http://templates.servicestack.net/docs/hot-reloading) 
which will automatically reload your web page as it detects any file changes made by parcel.

### server

To enable even richer functionality, this Web Apps template is also pre-configured with a custom Server project where you can extend your Web App with [Plugins](http://templates.servicestack.net/docs/web-apps#plugins) where all `Plugins`, `Services`, `Filters`, etc are automatically wired and made available to your Web App. 

This template includes a simple [ServerPlugin.cs](https://github.com/NetCoreTemplates/parcel-webapp/blob/master/server/ServerPlugin.cs) which contains an Empty `ServerPlugin` and `Hello` Service:

```csharp
public class ServerPlugin : IPlugin
{
    public void Register(IAppHost appHost)
    {
    }
}

[Route("/hello")]
[Route("/hello/{Name}")]
public class Hello : IReturn<HelloResponse>
{
    public string Name { get; set; }
}

public class HelloResponse
{
    public string Result { get; set; }
}

public class MyServices : Service
{
    public object Any(Hello request)
    {
        return new HelloResponse { Result = $"Hi {request.Name} from server.dll" };
    }
}
```

If you make any changes to `ServerPlugin.cs` you can run:

    $ npm run build-server

To build the project and copy the resulting `server.dll` to `/app/plugins/serer.dll` which will require restarting the server to load the latest plugin:

    $ npm run server

Which will automatically load any `Plugins`, `Services`, `Filters`, etc and make them available to your Web App. 

One benefit of creating your APIs with C# ServiceStack Services instead of [API Pages](http://templates.servicestack.net/docs/api-pages) is that you can generate TypeScript DTOs with:

    $ npm run dtos

Which saves generate DTOs for all your ServiceStack Services in [dtos.ts](https://github.com/NetCoreTemplates/parcel-webapp/blob/master/client/dtos.ts) which can then be accessed in your TypeScript source code.

If preferred you can instead develop Server APIs with API Pages, an example is included in [/client/api/hello.html](https://github.com/NetCoreTemplates/parcel-webapp/blob/master/client/api/hello.html)

```html
{{ { result: `Hi ${name} from /api/hello` } | return }}
```

Which as it uses the same data structure as the `Hello` Service above, can be called with ServiceStack's `JsonServiceClient` and generated TypeScript DTOs.

The [/client/index.ts](https://github.com/NetCoreTemplates/parcel-webapp/blob/master/client/index.ts) shows an example of this where initially the App calls  the C# `Hello` ServiceStack Service:

```ts
import { client } from "./shared";
import { Hello, HelloResponse } from "./dtos";

const result = document.querySelector("#result")!;

document.querySelector("#Name")!.addEventListener("input", async e => {
  const value = (e.target as HTMLInputElement).value;
  if (value != "") {
    const request = new Hello();
    request.name = value;
    const response = await client.get(request);
    // const response = await client.get<HelloResponse>("/api/hello", request); // call /api/hello instead
    result.innerHTML = response.result;
  } else {
    result.innerHTML = "";
  }
});
```

But while your App is running you can instead toggle the uncommented the line and hit `Ctrl+S` to save `index.ts` which Parcel will automatically transpile and publish to `/app/wwwroot` that will be detected by Hot Reload to automatically reload the page with the latest changes. Now typing in the text field will display the response from calling the `/api/hello.html` API instead.

### Deployments

During development Parcel maintains a debug and source-code friendly version of your App. Before deploying you can build an optimized production version of your App with:

    $ npm run build

Which will bundle and minify all `.css`, `.js` and `.html` assets and publish to `/app/wwwroot`.

Then to deploy Web Apps you just need to copy the `/app` and `/web` folders to any server with .NET Core 2.1 runtime installed. 
The [Deploying Web Apps](http://templates.servicestack.net/docs/deploying-web-apps) docs 