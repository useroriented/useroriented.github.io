# 一款更好的http模块

最近写angular2有个需求需要做http拦截器，发觉ng2的http模块拓展起来实在太麻烦。

1、首先需要constructor:

```js
@Injectable()
export class InterceptedHttp extends Http {
    constructor(backend: ConnectionBackend, defaultOptions: RequestOptions) {
        super(backend, defaultOptions);
    }
```

ng2的官网有这么一段话，我还是很赞同的:
> This is a best practice. Components are easier to test and debug when their constructors are simple, and all real work (especially calling a remote server) is handled in a separate method.

学过一阵子fp之后，喜欢用module pattern来代替oop的class pattern。然后class无法避免(eg.react需要class)。那么在使用class的情况下我不喜欢用constructor。constructor里面修改class成员会让class不够直观。所以除非必须，我是不会使用constructor的。

那么上面这个InterceptedHttp的实现方式就已经有点不爽了。

2、奇怪的注入方式:

```js
    providers: [
        {
            provide: Http,
            useFactory: httpFactory,
            deps: [XHRBackend, RequestOptions]
        }
    ]
```

我更期待类似service的注入方式:

```js
   providers: [HttpService]
```

受之前写elixir使用的http库[httpoison](https://github.com/edgurgel/httpoison)的启发。决定重新写一个http模块。下面是代码:

```js
import { Observable } from "rxjs"

export class Http {
	private defaultHeaders = {}

	/** 请求发出之前 */
	protected requestWillSend(url: string, body?: any, headers?: object) { }

	/** 请求发出之后 */
	protected requestDidSend(response: string | object) { }

	/** 请求之前会调用，这个函数的返回值是你的url */
	protected processUrl(url: string) {
		return url
	}

	/** 请求之前会调用，这个函数的返回值是你的headers */
	protected processRequestHeaders(headers = this.defaultHeaders) {
		return headers
	}

	protected processRequestBody(body: any) {
		return body
	}

	protected processResponseBody(body: object) {
		return body
	}

	protected get(url: string, headers?: object) {
		const processedUrl = this.processUrl(url)
		const processedHeaders = this.processRequestHeaders(headers)
		this.requestWillSend(processedUrl, undefined, processedHeaders)
		return Observable.ajax
			.get(processedUrl, processedHeaders)
			.map(res => res.responseType === "text" ? res.responseText : this.processResponseBody(res.response))
			.do(this.requestDidSend)
	}

	protected post(url: string, body?: any, headers?: object) {
		const processedUrl = this.processUrl(url)
		const processedHeaders = this.processRequestHeaders(headers)
		const processedBody = this.processRequestBody(body)
		this.requestWillSend(processedUrl, processedBody, processedHeaders)
		return Observable.ajax
			.post(processedUrl, processedBody, processedHeaders)
			.map(res => res.responseType === "text" ? res.responseText : this.processResponseBody(res.response))
			.do(this.requestDidSend)
	}

	protected put(url: string, body?: any, headers?: object) {
		const processedUrl = this.processUrl(url)
		const processedHeaders = this.processRequestHeaders(headers)
		const processedBody = this.processRequestBody(body)
		this.requestWillSend(processedUrl, processedBody, processedHeaders)
		return Observable.ajax
			.put(processedUrl, processedBody, processedHeaders)
			.map(res => res.responseType === "text" ? res.responseText : this.processResponseBody(res.response))
			.do(this.requestDidSend)
	}

	protected delete(url: string, headers?: object) {
		const processedUrl = this.processUrl(url)
		const processedHeaders = this.processRequestHeaders(headers)
		this.requestWillSend(processedUrl, processedHeaders)
		return Observable.ajax
			.delete(processedUrl, processedHeaders)
			.map(res => res.responseType === "text" ? res.responseText : this.processResponseBody(res.response))
			.do(this.requestDidSend)
	}
}

```

和rxjs如出一辙，都是返回的Observable。但是这个实现提供了声明周期`requestWillSend`和`requestDidSend`。实用的加工函数。而且Observable.ajax是会自动关闭你的subscription的。(事后了解到@angular/http也会自动关闭)。

有了这些函数就可以写出功能更丰富的service了，比如下面的`BaseService`:

```js
export default class BaseService extends Http {

	protected endpoint = environment.ual + environment.server

	public post(url: string, body?: any, headers?: object) {
		return super.post(url, body, { "Content-Type": "application/json", ...headers })
	}

	public put(url: string, body?: any, headers?: object) {
		return super.put(url, body, { "Content-Type": "application/json", ...headers })
	}

	protected processUrl(url: string) {
		return this.endpoint + url
	}

	/** TODO: 在这里处理给header加上权限 */
	// protected processRequestHeaders(headers: object) {
	// 	const authCookie = Cookie.get("credential")
	// 	return { ...headers, Authorization: "Bearer " + authCookie }
	// }

}
```

BaseService重写了部分方法，实现了额外的需求。利用`processUrl`给url加上了端点，避免了总是需要拼接路由的问题。利用`processRequestHeaders`可以给每个请求加上认证信息。等等。

最后只需要继承`BaseService`就能实现我们的service了:

```js
class UserService extends BaseService {
	public query() {
		return this.get("/user")
	}
}
```

至此我们做到了全程不使用`constructor`，简单的注入方式。