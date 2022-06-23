---
title: Filters in Minimal API applications
author: rick-anderson
description: Use filters in Minimal API applications
ms.author: riande
ms.date: 6/22/2022
monikerRange: '>= aspnetcore-7.0'
uid: fundamentals/minimal-apis/min-api-filters
---
# Filters in Minimal API apps

By [Rick Anderson](https://twitter.com/RickAndMSFT)

Minimal API filters allow developers to implement business logic that supports:

* Running code before the route handler.
* Inspecting and modifying parameters provided during a route handler invocation.
* Intercepting the response behavior of a route handler.

Filters can be helpful in the following scenarios:

* Validating the request parameters and body that are sent to an endpoint.
* Logging information about the request and response.
* Validating that a request is targeting a supported API version

Filters can be registered by providing a [Delegate](/dotnet/csharp/programming-guide/delegates/) that takes a [`routeHandlerInvocationContext`](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Http.Abstractions/src/RouteHandlerInvocationContext.cs) and returns  a [`RouteHandlerFilterDelegate`](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Http.Abstractions/src/RouteHandlerFilterDelegate.cs). The `RouteHandlerInvocationContext` provides access to the `HttpContext` associated with the request and a `Arguments` list indicating the arguments passed to the handler in the order in which they appear in the argument list of the handler.

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/Filters/Program.cs?name=snippet1)]

The preceding code:

* Calls the `AddFilter` extension method to add a filter to the `/colorSelector/{color}` endpoint.
* Returns the color specified except for the `Red`.
* Returns [Results.Problem](xref:Microsoft.AspNetCore.Http.Results.Problem%2A) when the `/colorSelector/Red` endpoint is requested.
* Uses `next` as the `RouteHandlerFilterDelegate`.

The filter is run before the endpoint handler. When multiple `AddFilter` invocations are made on a handler, the filters are executed in order of First In, Last Out (FILO) order, so the first filter registered run lasts.

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/Filters/Program.cs?name=snippet_abc)]

In the preceding code, the filters and handlers are executed in the following order:

* `CrouteFilter` -> `BrouteFilter` -> `ArouteFilter` -> route handler
* The `next` for `FilterC` is `FilterB` and so on.

 Consider a filter that validated endpoints which consume a `Todo` object in the body:

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/todo/Program.cs?name=snippet_filter1)]

In the preceding code:

* The `RouteHandlerInvocationContext` object provides access to the `MethodInfo` associated with the endpoint's handler and the `EndpointMetadata` that has been applied on the endpoint.
* The filter is registered using a `delegate` that takes a `RouteHandlerInvocationContext` and returns a `RouteHandlerFilterDelegate`. This factory pattern is useful to register a filter that depends on the signature of the target route handler.

In addition to being passed as delegates, filters can be registered by implementing the `IRouteHandlerFilter` interface. The follow code shows the preceding filter encapsulated in a class which implements `IRouteHandlerFilter`:

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/todo/RouteFilters/ToDoIsValidFilter.cs?name=snippet)]

Filters that implement the `IRouteHandlerFilter` interface can resolve dependencies from [Dependency Injection (DI)]](xref:fundamentals/dependency-injection), as shown in the previous code. Although filters can resolve dependencies from DI, filters themselves can ***not*** be resolved from DI.

The `ToDoIsValidFilter` is applied to the following endpoints:

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/todo/Program.cs?name=snippet_2flt)]

The following filter validates the `ToDo` object and modifies the `Name` field:

[!code-csharp[](~/fundamentals/minimal-apis/min-api-filters/7samples/todo/RouteFilters/ToDoIsValidFilter.cs?name=snippet2&highlight=7)]

## Additional Resources

* [View or download sample code](https://github.com/aspnet/Docs/tree/main/aspnetcore/fundamentals/minimal-apis/min-api-filters/7samples) ([how to download](xref:index#how-to-download-a-sample))
* <xref:tutorials/min-web-api>