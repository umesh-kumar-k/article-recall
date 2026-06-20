---
aliases:
  - Seamless data fetching with httpResource
Source 1: https://blog.angular.dev/seamless-data-fetching-with-httpresource-71ba7c4169b9
---
# Seamless data fetching with httpResource

## HttpResource
* **Key Points:**
  - httpResource is built on top of the resource primitive and uses HttpClient as loader. It acts as a frontend for @angular/common/http. It makes HTTP requests through the Angular HTTP stack, including interceptors. As the underlying stack remains the same, testing will rely on the same tools.
  - By default, an httpResource will perform a GET request and return an unknown typed JSON response.
  - It is important to note that httpResource differs from the HttpClient as it initiates the request eagerly (unlike the HttpClient Observable-based requests which must be subscribed).
  - Like resource, it configures a reactive request. If any of the source signals in the request computation change, a new HTTP request will be made.
  - For more advanced requests, it is possible to define a request object similar to HttpClient's request.
  - While the resource pattern is meant only for retrieving asynchronous data, httpResource will allow any request method (like POST in the previous example). This still doesn't mean that you should be using httpResource to change data on the server. For instance, if you need to submit form data, use the HttpClient methods.
  - An httpResource will return and parse the response as JSON but it is possible to use it for other return types.
  - The API has multiple dedicated methods available for other response types:
    - httpResource.text(() => ({ … })); // returns a text in value()
    - httpResource.blob(() => ({ … })); // returns a Blob object in value()
    - httpResource.arrayBuffer(() => ({ … })); // returns an ArrayBuffer in value()
* **Technical Entities (Classes/Functions/APIs):** `httpResource`, `HttpClient`, `@angular/common/http`, `interceptors`, `GET`, `POST`, `text()`, `blob()`, `arrayBuffer()`
* **Code Snippet:**
```typescript
currentUserId = getCurrentUserId(); // returns a signal

user = httpResource(() => `/api/user/${currentUserId()}`); // A reactive function as argument
```

## Shape of an HttpResource
* **Key Points:**
  - An httpResource , similar to other `resource`, exposes several signals:
    - value() — which contains the result of the http request (when successful) and is programmatically overwritable
    - status() — with the status of the resource (idle, loading, error etc)
    - error() — with the request error / parsing error
    - isLoading() — which is true while the request is pending
  - It also includes dedicated signals for metadata about the response:
    - headers() — with the response's headers
    - statusCode() — with the response's status code
    - progress() — with the progress of the request (if required in the request object)
  - These new signals streamline the writing of requests by exposing this useful information without requiring a specific argument like for the HttpClient to request the HttpResponse.
* **Technical Entities (Classes/Functions/APIs):** `value()`, `status()`, `error()`, `isLoading()`, `headers()`, `statusCode()`, `progress()`

## Embracing the Ecosystem for Type Safety
* **Key Points:**
  - When performing http requests we often want to ensure that the data we receive conforms the shape that we expect. This is commonly known as schema validation.
  - In the JavaScript ecosystem we often reach out for battle-tested libraries like Zod or Valibot for schema validation. The httpResource offers direct integration for those libraries by using the parse parameter. The returned type of this parse function will provide the type to the resource itself, ensuring type safety alongside the schema validation.
  - The following example uses Zod to parse and validate the response from the StarWars API. The resource is then typed the same as the output type of the Zod's parsing.
* **Technical Entities (Classes/Functions/APIs):** `parse`, `Zod`, `Valibot`
* **Code Snippet:**
```typescript
export class AppComponent {
  id = signal(1);

  swPersonResource = httpResource(
    () => `https://swapi.dev/api/people/${this.id()}`,
    { parse: starWarsPersonSchema.parse }
  );
}

const starWarsPersonSchema = z.object({
  name: z.string(),
  height: z.number({ coerce: true }),
  edited: z.string().datetime(),
  films: z.array(z.string()),
});
```

## Experimental API
* **Key Points:**
  - The httpResource is available as a part of the Angular v19.2 release. This is an experimental API and is not ready for production because the shape of this API can still change before it is promoted to stable. With that in mind, we would love for you to try out the API and let us know what you think. You can learn more about this API, resource and more in the RFC on GitHub.
  - Thank you for being a part of the Angular community and we look forward to continuing the reactivity journey together. Happy coding!