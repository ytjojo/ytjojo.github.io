---
title: Gson、Jackson与retrofit的结合，用泛型优雅处理服务器返回数据
date: 2018-03-30 15:37:20
categories: git
tags: [gson,json,jackson,retrofit,Converter]
---

### 开源库
https://github.com/ytjojo/androidPractice/tree/master/retrofitex
这是用与商业项目的Retrofit扩展库，最大亮点是支持自定义注解
支持带进度的上传下载，多线程断点下载，支持cookie管理，支持token自动刷新（刷新条件可以根据http statucode 默认401 我们公司是403），token失效http status code 为 403 retrofit是不支持自动刷新token或者cookie的。支持错误重试
支持post请求缓存，可以设置过期时间，支持无网络返回缓存。
对https做了封装
对https 在4.4及一下手机做了兼容。
4.4手机下载或者上传图片的时候会出现如下错误
>javax.net.ssl.SSLHandshakeException: javax.net.ssl.SSLProtocolException: SSL handshake aborted: 
支持在单元测试请求接口，这样验证新接口不用跑在手机上，当然用Postman更好


### 前言
服务器返回的json格式大多数是固定的，我们公司返回结果中，最外层字段是固定的，code body msg，其中code是int值，msg是返回的消息
body则根据接口不同而不同，可以是基本数据类型，也可以是JsonObject 或者JsonArray。举个例子

```json
{
    "code": 200,
    "body": 100000
}

{
    "code": 200,
    "body": "success"
}
{
    "code": 200,
    "body": [{"name":"zhangsan","age":20},{"name":"lisi","age":21}]
}

{
    "code": 200,
    "body": {
        "orderNum":1001,
        "date":"2014-12-10 22:33"
    }
}
```
200表示接口正常返回（注意这个200是服务返回消息体的json中字段，不是http status code）。
### 用泛型统一返回对象


```java

public class ResponseWraper<T> implements Serializable {
    public T body;
    public int code;
    public String msg;
}



```
`


### 实现需求

1 可以只专注Body值，body不等于200走失败回调，error有失败的code和msg；
2 支持整个返回结果，
```java
//支持任何java类型
Observable<AnyJavaEntry> getResponse()

Observable<Integer> getResponse()
Observable<String> getResponse()
Observable<Boolean> getResponse()
```

Jackson 方式


```java

Observable<JsonNode> getArticles()
```
Gson方式
```java
@Get("url")
Observable<JsonElement> getArticles()
```

3支持结果有额外字段

如果code body 同级有额外字段，只需要继承这个泛型类ResponseWraper
```java

public class ArticleResponse extends ResponseWraper<ArrayList<Article>>  {
 	public ArrayList<String> urls;
}
//接口可以这样写，
@Get("url")
Observable<ArticleResponse> getArticles()

```
4 支持服务器不规范的返回结果
如整个相应体为空,我们退出登录接口返回相应体为空，http status code 200 ，但是相应体不包含任何字符串，也没有当初约定的code body msg这些json字段。当然也是支持走成功回调的只需要把返回值写成Observable<Object>
```java
Observable<Object> logout()

```
还有个接口返回的json只有200，{"code":200},这个接口客户端是不需要结果的，只需要知道成功失败，后台的大佬直接给个这种json，我其实希望body有个boolean的但是后台不肯改，好吧那我这边就支持这种方式吧。方法也很简单
```java
Observable<Object> closeChat()

```



### 完整代码如下

### Jackson


```java
final class JacksonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
    private final ObjectMapper mapper;
    private final Type type;
    public static Void DEFAULT_VOID_INSTIANCE;

    enum Irrelevant { INSTANCE; }

    JacksonResponseBodyConverter(Type type, ObjectMapper mapper) {
        this.type = type;
        this.mapper = mapper;
    }

    @Override
    public T convert(ResponseBody responseBody) throws IOException {

        BufferedSource bufferedSource = Okio.buffer(responseBody.source());
        String value = bufferedSource.readUtf8();
        bufferedSource.close();
        if (TextUtils.isEmpty(value)) {
            if (type == Object.class) {
                return (T) Irrelevant.INSTANCE;
            }
            throw new APIException(0, "response is empty");
        }
        JsonNode responseNode = mapper.readTree(value);
        JsonNode codeJsonNode = responseNode.get("code");
        JsonNode msgNode = responseNode.get("msg");
        String msg = msgNode == null ? null : msgNode.asText();
        int code = codeJsonNode != null ? codeJsonNode.asInt() : 0;
        if (codeJsonNode != null) {
            if (code != 200) {
				//这一步很关键
				//如果接口返回code不是200，就会走onError，
                throw new APIException(codeJsonNode.asInt(), msg, value);
            }
        } else {
            return parseNoCodeAndNoBodyType(responseNode, value);
        }
        JavaType javaType = mapper.constructType(type);
        if (!javaType.isTypeOrSubTypeOf(ResponseWraper.class)) {
            if (type == String.class) {
                JsonNode bodyNode = responseNode.get("body");
                if (bodyNode == null) {
                    return (T) "";
                }
                if (bodyNode.isTextual()) {
                    return (T) bodyNode.asText();
                }
                return (T) mapper.writeValueAsString(bodyNode);
            }
            if (type == Void.class) {
                assertVoidInstance();
                return (T) DEFAULT_VOID_INSTIANCE;
            }
            if (type == JsonNode.class) {
                return (T) responseNode;
            }
            if (type == Object.class) {
                return (T) Irrelevant.INSTANCE;
            }
            JavaType responseType = mapper.getTypeFactory().constructParametrizedType(
                    ResponseWraper.class, ResponseWraper.class, javaType);
            ResponseWraper responseWrapper = mapper.readValue(value, responseType);
            return (T) responseWrapper.body;
        }
        return mapper.readValue(value, javaType);
    }

    private void assertVoidInstance() {
        if (DEFAULT_VOID_INSTIANCE == null) {
            Constructor<?>[] cons = Void.class.getDeclaredConstructors();
            cons[0].setAccessible(true);
            try {
                DEFAULT_VOID_INSTIANCE = (Void) cons[0].newInstance(new Object[]{});
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
    }

    private T parseNoCodeAndNoBodyType(JsonNode root, String json) throws IOException {
        if (type == Void.class) {
            assertVoidInstance();
            return (T) DEFAULT_VOID_INSTIANCE;
        }
        if (type == JsonNode.class) {
            return (T) root;
        }
        if (type == Object.class) {
            return (T) Irrelevant.INSTANCE;
        }
        return mapper.readValue(json, mapper.constructType(type));
    }
}

```
Gson
```java


final class GsonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
	public static Void DEFAULT_VOID_INSTIANCE;
	private final Gson mGson;
	private final Type type;
	private TypeAdapter<T> adapter;
	enum Irrelevant { INSTANCE; }
	GsonResponseBodyConverter(Gson gson, Type type) {
		this.mGson = gson;
		this.type = type;
	}

	@Override public T convert(ResponseBody responseBody) throws IOException {

		BufferedSource bufferedSource = Okio.buffer(responseBody.source());
		String value = bufferedSource.readUtf8();
		bufferedSource.close();
		if (TextUtils.isEmpty(value)) {
			if(type == Object.class){
				return (T) Irrelevant.INSTANCE;
			}
               //这一步很关键
				//如果接口返回code不是200，就会走onError，
			throw new APIException(-3, "response is null");
		}
		int code = Integer.MAX_VALUE;
		JsonElement root = null;
		JsonObject response;
		try{
			root = mGson.fromJson(value, JsonElement.class);
			if(root.isJsonObject()){
				response = root.getAsJsonObject();
				JsonElement codeElement  = response.get("code");
				if(codeElement != null){
					code = codeElement.getAsInt();
				}else {
					JsonElement bodyJson = response.get("body");
					if(bodyJson == null){
						return parse(root,value);
					}
				}
			}else {
				return parse(root,value);
			}

		}catch (Exception e){
			throw new JsonException("json解析失败",e);
		}

		if (code != ServerResponse.RESULT_OK) {
			JsonElement msgJE = response.get("msg");
			String msg = msgJE == null ? null : msgJE.getAsString();
			throw new APIException(code, msg, value);
		}
		try{
			if (type instanceof Class) {
				if(type == EntireStringResult.class){
					return (T) new EntireStringResult(value);
				}
				if (type == String.class) {
					JsonElement bodyJson = response.get("body");
					if(bodyJson ==null){
						return null;
					}
					if(bodyJson.isJsonPrimitive()){
						return (T) bodyJson.getAsString();
					}
					return (T)(mGson.toJson(bodyJson));
				}
				if (type == Object.class) {
					return (T) Irrelevant.INSTANCE;
				}
				if (type == Void.class) {
					assertVoidInstance();
					return (T) DEFAULT_VOID_INSTIANCE;
				}
				if (type == JsonElement.class) {
					//如果返回结果是JSONObject则无需经过Gson
					return (T)(root);
				}
				if(!(ServerResponse.class.isAssignableFrom((Class<?>) type))){
					Type wrapperType = $Gson$Types.newParameterizedTypeWithOwner(null,ServerResponse.class,type);
					ServerResponse<?> wrapper = mGson.fromJson(value, wrapperType);
					return (T) wrapper.body;
				}
			}
			return mGson.fromJson(value, type);
		}catch (Exception e){
			throw new JsonException("json解析失败",e);
		}

	}
	private void assertVoidInstance(){
		if(DEFAULT_VOID_INSTIANCE ==null){
			Constructor<?>[] cons = Void.class.getDeclaredConstructors();
			cons[0].setAccessible(true);
			try {
				DEFAULT_VOID_INSTIANCE = (Void) cons[0].newInstance(new Object[]{});
			} catch (InstantiationException e) {
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			} catch (InvocationTargetException e) {
				e.printStackTrace();
			}
		}
	}
	private T parse(JsonElement root,String json){
		if(type == EntireStringResult.class){
			return (T) new EntireStringResult(json);
		}
		if (type == Object.class) {
			return (T) Irrelevant.INSTANCE;
		}
		if (type == Void.class) {
			assertVoidInstance();
			return (T) DEFAULT_VOID_INSTIANCE;
		}
		if (type == JsonElement.class) {
			//如果返回结果是JSONObject则无需经过Gson
			return (T)(root);
		}
		return mGson.fromJson(json, type);
	}

```


