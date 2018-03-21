---
title: 关于Gson和Jackson的花式使用都在这里了，一网打尽Json转换
date: 2018-03-19 20:11:45
categories: json
tags: [json,Gson,Jackson,json转换]
---


### 简单配置
Gson
```java
        GsonBuilder builder = new GsonBuilder();
        builder.enableComplexMapKeySerialization()
                .serializeNulls();
        builder.setDateFormat("yyyy-MM-dd HH:mm:ss");
        Gson gson = builder.create();
```

Jackson
```java
		ObjectMapper objectMapper = new ObjectMapper();
        SimpleDateFormat fmt = new SimpleDateFormat(
                "yyyy-MM-dd HH:mm:ss");
        objectMapper.setDateFormat(fmt);
        objectMapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);//字段和值加引号
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true);//允许转义字符
        objectMapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);//允许单引号
        objectMapper.configure(JsonParser.Feature. ALLOW_BACKSLASH_ESCAPING_ANY_CHARACTER, true);//允许反斜杠
        objectMapper.configure(JsonParser.Feature. ALLOW_NUMERIC_LEADING_ZEROS, true);//允许数字00005 赋值为5；
        objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY, true);
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES,true);
        objectMapper.configure(SerializationFeature.WRITE_EMPTY_JSON_ARRAYS, true);
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        //该特性可以允许JSON空字符串转换为POJO对象为null,否则抛异常。空字符串不是标准的pojo对应的json
        objectMapper.configure(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT,true);
```
### 字段值为null系列化输出为null
Gson
```java
     GsonBuilder builder = new GsonBuilder();
     builder.serializeNulls();
	 Gson gson = builder.create();
```
Jackson
```java
 ObjectMapper objectMapper = new ObjectMapper();
   objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);
```
### 设置默认日期格式
Gson
```java

  builder.setDateFormat("yyyy-MM-dd HH:mm:ss");
```

Jackson
```java
    SimpleDateFormat fmt = new SimpleDateFormat(
                "yyyy-MM-dd HH:mm:ss");
        objectMapper.setDateFormat(fmt);
```

### 类级别上处理部分字段序列化和反序列化忽略
两个框架都支持用transient修饰的字段忽略。

Gson处理方式，使用@Expose注解

```java

public class User { 
    @Expose 
	public String firstName; 
    @Expose(serialize = false) 
	public String lastName;
    @Expose (serialize = false, deserialize = false) 
    public emailAddress; 
    public String password;
    public transient boolean isDeveloper
 }

```
Jackson 用@JsonIgnore用与字段 get Set方法， @JsonIgnoreProperties用于类
json 、msg、status都会被忽略
```java
   @JsonIgnoreProperties({"msg","status"})
    public static class Body {
        @JsonIgnore
        private String json;
  		public String msg;
        public int status;
        public Body(){}
        @JsonIgnoreProperties
        public long meetingNumber;
        public String password;
    }

```

### 修改系列化字段名称
服务器给json字段有可能跟Entity类字段不一致，如何修改呢；

Gson，可以为同一个字段映射两个json字段，如下文中name字段，json中如果只有fullName,name值为fullname字段的值，只有username则name值为username字段值，两者都有的话是对应alternate配置的字段值

```java
 public class SomeClassWithFields {
   @SerializedName("name")
   public final String someField;  
   @SerializedName(value = "fullName", alternate = "username")    
   private String name; 
  
}
```
```java
public clsss RequestMsg{
private String osType;
@JsonProperty("os_type")
public String getOs_Type(){
return this.osType;
}
@JsonProperty(value="osType")
public void setOsType(String osType){
this.osType= osType;
}

}
```

### map的序列化对比
key简单对象序列化
Gson

```java
  HashMap<String,String> stringmap =new HashMap<>();
        stringmap.put("2","sss");
        stringmap.put("511","sss");
        json = mapper.writeValueAsString(stringmap);
        System.out.println(json);
        GsonBuilder builder = new GsonBuilder();
        builder.serializeNulls();
//        builder.enableComplexMapKeySerialization();
        Gson gson =  builder.create();
         json =  gson.toJson(stringmap);
        System.out.println(json);

```
输出结果
>{"2":"sss","511":"sss"}  
>{"2":"sss","511":"sss"}

Key为复杂对象的系列化



```java
    public static class Point {
        public int x;
        public int y;
        public Point(){}
        public Point(int x,int y){
            this.x = x;
            this.y = y;
        }
    }

	public void test(){
		String json = null;
	  final  ObjectMapper mapper =new ObjectMapper();
	   HashMap<Point,String> map = new HashMap<>();
        map.put(new Point(3,5),"sdw");
        map.put(new Point(6,2),"fasow");

        json = mapper.writeValueAsString(map);
        System.out.println(json);
        GsonBuilder builder = new GsonBuilder();
        builder.serializeNulls();
        Gson gson =  builder.create();
         json =  gson.toJson(map);
        System.out.println(json);
	}
```
输出结果为
>{"com.ytjojo.practice.Jackson$Point@f6c48ac":"sdw","com.ytjojo.practice.Jackson$Point@13deb50e":"fasow"}
>{"com.ytjojo.practice.Jackson$Point@f6c48ac":"sdw","com.ytjojo.practice.Jackson$Point@13deb50e":"fasow"}


可以发现直接调用的key.toString()方法，Point并没有序列化为json，最外层为{,如果GsonBuilde加上enableComplexMapKeySerialization()结果就不一样了，Gson序列化结果

>[[{"x":3,"y":5},"sdw"],[{"x":6,"y":2},"fasow"]]

可以发现整个map就是jsonArray，key和value组成的JsonArray是Array的子项目。
很遗憾，目前为止未发现Jackson原生支持这种序列化方式，如何让Jackson支持这种呢？看代码

```java

  final  ObjectMapper mapper =new ObjectMapper();
        SimpleModule simpleModule =  new SimpleModule();
        simpleModule.addSerializer(Map.class, new JsonSerializer<Map>() {
            @Override
            public void serialize(Map value, JsonGenerator gen, SerializerProvider serializers) throws IOException, JsonProcessingException {
                if(value != null){
                    gen.writeStartArray();
                    for(Object item: value.entrySet()){
                        Map.Entry entry = (Map.Entry) item;
                        gen.writeStartArray();
                        gen.writeObject(entry.getKey());
                        gen.writeObject(entry.getValue());
                        gen.writeEndArray();
                    }
                    gen.writeEndArray();
                }
            }
        });
        mapper.registerModule(simpleModule);
		
```
最后输出结果和Gson一样了
### map反序列化
#### 简单key反序列化
Gson和jackson都是统一的，不用做比较

#### 复杂key反序列化比较

Gson

```java
 String json = "[[{\"x\":10,\"y\":10},\"Ten\"],[{\"x\":20,\"y\":20},\"Twenty\"]]";
        GsonBuilder builder = new GsonBuilder();
        builder.serializeNulls();

        Gson gson =  builder.create();
      HashMap<Point,String> map = null;
         map =  gson.fromJson(json,new TypeToken<HashMap<Point,String>>(){}.getType());

        System.out.println(map.entrySet().iterator().next().getKey().x);

```
此时Gson能正常解析；
看看Jackson
```java
  String json = "[[{\"x\":10,\"y\":10},\"Ten\"],[{\"x\":20,\"y\":20},\"Twenty\"]]";
  final  ObjectMapper mapper =new ObjectMapper();
  HashMap<Point,String> map = null;
  map = mapper.readValue(json, new TypeReference<HashMap<Point,String>>() {});
        System.out.println(map.keySet().size()+"" +map);

```
发现报错了
>com.fasterxml.jackson.databind.JsonMappingException: Can not find a (Map) Key deserializer for type [simple type, class com.ytjojo.practice.Jackson$Point]
>at [Source: [[{"x":10,"y":10},"Ten"],[{"x":20,"y":20},"Twenty"]]; line: 1, column: 1]
>at com.fasterxml.jackson.databind.JsonMappingException.from(JsonMappingException.java:244)
>at com.fasterxml.jackson.databind.deser.DeserializerCache._handleUnknownKeyDeserializer(DeserializerCache.java:587)
>at com.fasterxml.jackson.databind.deser.DeserializerCache.findKeyDeserializer(DeserializerCache.java:168)
>at com.fasterxml.jackson.databind.DeserializationContext.findKeyDeserializer(DeserializationContext.java:500)
>at com.fasterxml.jackson.databind.deser.std.MapDeserializer.createContextual(MapDeserializer.java:231)
>at com.fasterxml.jackson.databind.DeserializationContext.handleSecondaryContextualization(DeserializationContext.java:685)
>at com.fasterxml.jackson.databind.DeserializationContext.findRootValueDeserializer(DeserializationContext.java:482)
>at com.fasterxml.jackson.databind.ObjectMapper._findRootDeserializer(ObjectMapper.java:3889)
>at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:3784)
>at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:2798)

Jackson默认是不支持解析key复杂对象的。
用ObjectMapper注册SimpleModule，SimpleModule加上自定义HashMap的JsonDeserializer


```java
String json = "[[{\"x\":10,\"y\":10},\"Ten\"],[{\"x\":20,\"y\":20},\"Twenty\"]]";
final  ObjectMapper mapper =new ObjectMapper();
SimpleModule simpleModule =  new SimpleModule();
//注册一定是HashMap.class不能是Map.class不然走不到自定义的JsonDeserializer里面去
simpleModule.addDeserializer(HashMap.class,new Key());
mapper.registerModule(simpleModule);
 HashMap<Point,String> map = null;
 map = mapper.readValue(json, new TypeReference<HashMap<Point,String>>() {});
System.out.println(map.keySet().size()+"" +map);

```
最关键的就是Key这个类了继承自JsonDeserializer，源码在此：

```java
public class Key extends JsonDeserializer<HashMap<Object, Object>>
        implements ContextualDeserializer {
    JavaType mJavaType;
    JavaType mKeyJavaType;//key对应的JavaType
    JavaType mValueJavaType;//Value对应的JavaType
    JsonDeserializer<Object> _valueDeserializer;
    JsonDeserializer<Object> _keyDeserializer;
    @Override
    public HashMap<Object, Object> deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
        HashMap hashMap = new HashMap();
        if (p.getCurrentToken() == JsonToken.START_ARRAY) {
            JsonToken next = p.nextToken();
            while (p.nextToken() != JsonToken.END_ARRAY)
                if (next != null && next == JsonToken.START_ARRAY) {
                    p.nextToken();
					//反序列化key
                    Object key = _keyDeserializer.deserialize(p, ctxt);
                    p.nextToken();
	              //反序列化Value
                    Object value = _valueDeserializer.deserialize(p, ctxt);
                    hashMap.put(key, value);
                    p.nextToken();
                }
        }
        return hashMap;
    }
    @Override
    public JsonDeserializer<?> createContextual(DeserializationContext ctxt, BeanProperty property) throws JsonMappingException {
        if (property != null) {
            //如果当前Hashmap是类的属性property不为null，如果根就是hashmap，就会为null；
            mJavaType = property.getType();  // -> beanProperty is null when the StringConvertible type is a root value
        } else {
            mJavaType = ctxt.getContextualType();
        }
        mKeyJavaType = mJavaType.getKeyType();
        mValueJavaType = mJavaType.getContentType();
        _keyDeserializer = ctxt.findContextualValueDeserializer(mKeyJavaType, property);
        _valueDeserializer = ctxt.findContextualValueDeserializer(mValueJavaType, property);
        return this;
    }

}
```
总体流程就是获得Key和value的javaType，然后找到对应的序列化类，根据JsonParser的token值在适当的位置进行反序列话，然后加如HashMap

### 服务器返回json字段是转义后的，带有反斜杠
比如下面这种json
>{"code":200,"body": "{\"meetingNumber\":\"2\",\"password\":\"sdwsdwsdw\"}"}
>这种json ，body字段被转义了，带有反斜杠\，估计后台的同学是直接用String类型赋值给body的，然后过json转换就会有转义字符。
>body对应的实体类
```java
    public static class Response {
        public int code;
        public Body body;

    }
    public static class Body {
		public Body(){}
        public long meetingNumber;
        public String password;
    }

```
不管用gson还是Jackson解析都是失败的。
gson报的错为
```java
com.google.gson.JsonSyntaxException: java.lang.IllegalStateException: Expected BEGIN_OBJECT but was STRING at line 1 column 22 path $.body

at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:224)
at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$1.read(ReflectiveTypeAdapterFactory.java:129)
at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:220)
```
Jackson报的错误为
```java
com.fasterxml.jackson.databind.JsonMappingException: Can not instantiate value of type [simple type, class com.ytjojo.practice.JsonTest$Body] from String value ('{"meetingNumber":"2","password":"sdwsdwsdw"}'); no single-String constructor/factory method
 at [Source: {"code":200,"body": "{\"meetingNumber\":\"2\",\"password\":\"sdwsdwsdw\"}"}; line: 1, column: 21] (through reference chain: com.ytjojo.practice.Response["body"])

	at com.fasterxml.jackson.databind.JsonMappingException.from(JsonMappingException.java:216)
	at com.fasterxml.jackson.databind.DeserializationContext.mappingException(DeserializationContext.java:894)
	at com.fasterxml.jackson.databind.deser.ValueInstantiator._createFromStringFallbacks(ValueInstantiator.java:316)
	at com.fasterxml.jackson.databind.deser.std.StdValueInstantiator.createFromString(StdValueInstantiator.java:288)
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.deserializeFromString(BeanDeserializerBase.java:1200)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer._deserializeOther(BeanDeserializer.java:144)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:135)
	at com.fasterxml.jackson.databind.deser.SettableBeanProperty.deserialize(SettableBeanProperty.java:490)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:101)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:260)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:125)
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:3788)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:2779)

```
怎么解决这个问题呢，显看Jackson的处理方式

#### Jackson有两种处理方式

方法1 用在字段上注解@JsonDeserialize

```java
    public static class Response {
        public int code;
        @JsonDeserialize(using = StringToObjectDeserializer.class)
        public Body body;

    }
    public static class Body {
		public Body(){}
        public long meetingNumber;
        public String password;
    }
```
StringToObjectDeserializer的源码
```java
  private static class StringToObjectDeserializer extends JsonDeserializer<Object> implements ContextualDeserializer {
        private JavaType valueType;

        @Override
        public JsonDeserializer<?> createContextual(DeserializationContext ctxt, BeanProperty property) throws JsonMappingException {
//            JavaType wrapperType = property.getType();
//            JavaType valueType = wrapperType.containedType(0);
			//
            if (property != null)
                valueType = property.getType();  // -> beanProperty is null when the StringConvertible type is a root value

            else {
                valueType = ctxt.getContextualType();
            }
            return this;
        }

        @Override
        public Object deserialize(JsonParser parser, DeserializationContext ctxt) throws IOException {
            String json =parser.getValueAsString();
			//ObjectMapper一般需要配置成单例，这里直接新建，不做配置了
            return new ObjectMapper().readValue(json,valueType);
        }

        @Override
        public Object getNullValue() {
			//可以直接返回null也可生成一个空对象
            Class<?> clazz=valueType.getRawClass();
            try {
                return clazz.newInstance();
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
            return super.getNullValue();
        }
    }
```
方法2用于全局配置，一劳永逸
```java
   SimpleModule simleModule = new SimpleModule().setDeserializerModifier(new StringToObjectDeserializerModifier());
   ObjectMapper objectMapper = new ObjectMapper().registerModule(simleModule);
	//配置 和解析代码不贴了。



```
StringToObjectDeserializerModifier源码很简单

```java

public static class StringToObjectDeserializerModifier extends BeanDeserializerModifier {
        public JsonDeserializer<?> modifyDeserializer(DeserializationConfig config,
                                                      BeanDescription beanDesc, JsonDeserializer<?> deserializer) {
            if (deserializer.getClass() == BeanDeserializer.class) {
                JavaType valueType = beanDesc.getType();

                return new StringToBeanDeserializer((BeanDeserializerBase) deserializer);
            }
            return deserializer;
        }


}

  public static class StringToBeanDeserializer extends BeanDeserializer {

        /**
         * Copy-constructor that can be used by sub-classes to allow
         * copy-on-write style copying of settings of an existing instance.
         */
        public StringToBeanDeserializer(BeanDeserializerBase src) {
            super(src, true);
        }

      
        @Override
        public Object deserializeFromString(JsonParser p, DeserializationContext ctxt) throws IOException {
            // First things first: id Object Id is used, most likely that's it
            if (_objectIdReader != null) {
                return deserializeFromObjectId(p, ctxt);
            }

        /* Bit complicated if we have delegating creator; may need to use it,
         * or might not...
         */
            if (_delegateDeserializer != null) {
                if (!_valueInstantiator.canCreateFromString()) {
                    Object bean = _valueInstantiator.createUsingDelegate(ctxt, _delegateDeserializer.deserialize(p, ctxt));
                    if (_injectables != null) {
                        injectValues(ctxt, bean);
                    }
                    return bean;
                }
            }
            String json = p.getValueAsString();
            JavaType javatype = getValueType();
            return new ObjectMapper().readValue(json, javatype);
        }

    }
```
#### Gson中的处理

gson中用到大杀器 @JsonAdapter。@JsonAdapter作用于字段可以自定义序列化方式，也可以自定反序列化。
其中class是继承com.google.gson.JsonDeserializer自定义反序列化;或者继承自JsonSerializer自定义系列化
本例的源码如下


```java
 public static class Response {
        public int code;
         @JsonAdapter(GsonStringToObjectDeserializer.class)
        public Body body;

    }
public static class Body {
		public Body(){}
        public long meetingNumber;
        public String password;
}


```
GsonStringToObjectDeserializer的源码如下
```java



public class GsonStringToObjectDeserializer implements JsonDeserializer<Object> {
    @Override
    public Object deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        String jsonValue = json.getAsString();
       return  new Gson().fromJson(jsonValue,typeOfT);
    }
}
```
这样我们的Jackson 和Gson都支持带有转义字段json解析成对应java实体对象了。

### 打造万能的Date解析方案
服务器返回的日期格式有时候不太规范，目前服务器返回的有一下四种


```java
    public final static String yyyyMMdd = "yyyy-MM-dd";
    public final static String YYYMMDDHHMM = "yyyy-MM-dd HH:mm";
    public final static String YYMMDDHHMMSS = "yyyy-MM-dd HH:mm:ss";
    public final static String ENGLISH_MMMddYYYY= "MMM dd, yyyy";//Apr 25, 2016
```
Jackson中这种差异处理方式是Date字段上加上@JsonFormat注解

```java
   @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm"
	private Date startTime;
	 @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss"
	private Date endTime;

```
Gson处理方式还是用大杀器@JsonAdapter
```java
	public static class Bean{
	 @JsonAdapter(GsonDateDeserializer.class)
	private Date startTime;
	 @JsonAdapter(GsonDateDeserializer.class)
	private Date endTime;
	}


	public class GsonDateDeserializer implements JsonDeserializer<Date> {
    @Override
    public Date deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        return DateTimeHelper.getDateFromString(json.getAsString());
    }
}
```
但是上面Jackson和gson各自处理方式还是不太方便，类的字段加上注解才管用，如过全局生效，只需要配置Gson 或者ObjectMapper就完美了
现在开撸，先上Gson代码
配置

```java

    GsonBuilder builder = new GsonBuilder();
        builder.enableComplexMapKeySerialization()
                .serializeNulls();
        builder.registerTypeAdapterFactory(DateTypeAdapter.FACTORY);
        Gson gson =builder.create();

```
DateTypeAdapter源码
```java
public final class DateTypeAdapter extends TypeAdapter<Date> {
  public static final TypeAdapterFactory FACTORY = new TypeAdapterFactory() {
    @SuppressWarnings("unchecked") // we use a runtime check to make sure the 'T's equal
    @Override public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> typeToken) {
      return typeToken.getRawType() == Date.class ? (TypeAdapter<T>) new DateTypeAdapter() : null;
    }
  };

  private final DateFormat enUsFormat
      = DateFormat.getDateTimeInstance(DateFormat.DEFAULT, DateFormat.DEFAULT, Locale.US);
  private final DateFormat localFormat
      = DateFormat.getDateTimeInstance(DateFormat.DEFAULT, DateFormat.DEFAULT);

  private final static SimpleDateFormat FORMAT_YYYMMDDHHMM =new SimpleDateFormat("yyyy-MM-dd HH:mm");
  private final static SimpleDateFormat FORMAT_GMT =new SimpleDateFormat("EEE MMM ddHH:mm:ss 'GMT' yyyy",Locale.US);
  private final static SimpleDateFormat FORMAT_UTC =new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
  @SuppressWarnings("unchecked")
  @Override public Date read(JsonReader in) throws IOException {
    if (in.peek() == JsonToken.NULL) {
      in.nextNull();
      return null;
    }
    return deserializeToDate(in.nextString());
  }

  private synchronized Date deserializeToDate(String json) {
    try {
      return localFormat.parse(json);
    } catch (ParseException ignored) {
    }
    if(json.length() ==16){
      try {
        return FORMAT_YYYMMDDHHMM.parse(json);
      } catch (ParseException e) {
        throw new JsonSyntaxException(json, e);
      }
    }
    if(json.contains("GMT")){
      try {
        return FORMAT_GMT.parse(json);
      } catch (ParseException e) {

      }
      try{
        Date date =new Date();
        date.setTime(GMTDateUtils.parseDate(json,0,json.length()));
        return date;
      }catch (IllegalArgumentException e){

      }

    }
    try {
      return FORMAT_UTC.parse(json);
    } catch (ParseException e) {

    }
    try {
      return enUsFormat.parse(json);
    } catch (ParseException ignored) {
    }
    ParseException parseException;
    try {
        return ISO8601Utils.parse(json, new ParsePosition(0));
    } catch (ParseException e) {
      parseException = e;
    }
    Date result= UTCDateUtils.parseDate(json);
    if(result == null){
      throw new JsonSyntaxException(json, parseException);
    }
    return result;
  }

  @Override public synchronized void write(JsonWriter out, Date value) throws IOException {
    if (value == null) {
      out.nullValue();
      return;
    }
    String dateFormatAsString = FORMAT_YYYMMDDHHMM.format(value);
    out.value(dateFormatAsString);
  }
}
```
Gson 处理方式就是自定义Date 的TypeAdapter，然后在GsonBuilder中注册该TypeAdapter.
其中utc格式处理和Gmt解析源码我就补贴了

Jackson全局配置源码如下

```java
  ObjectMapper objectMapper = new ObjectMapper();
	//忽略其他各种配置
  SuperDateDeserializer.regist(objectMapper);
```

```java

/**
 * SuperDateDeserializer
 */
public class SuperDateDeserializer extends DateDeserializers.DateDeserializer {

    public static void regist(ObjectMapper mapper) {
        mapper.registerModule(new SimpleModule().addDeserializer(Date.class, new SuperDateDeserializer()));
    }

    public SuperDateDeserializer() {
        super();
    }

    public SuperDateDeserializer(SuperDateDeserializer base, DateFormat df, String formatString) {
        super(base, df, formatString);
    }

    @Override
    protected SuperDateDeserializer withDateFormat(DateFormat df, String formatString) {
        return new SuperDateDeserializer(this, df, formatString);
    }

    @Override
    protected Date _parseDate(JsonParser p, DeserializationContext ctxt) throws IOException {

        if (_customFormat != null) {
            JsonToken t = p.getCurrentToken();
            if (t == JsonToken.VALUE_STRING) {
                String str = p.getText().trim();
                if (str.length() == 0) {
                    return (Date) getEmptyValue(ctxt);
                }
                SimpleDateFormat simpleDateFormat = (SimpleDateFormat) _customFormat;
                String patten = simpleDateFormat.toPattern();
                if (patten.length() != str.length()) {
                    return getDateFromString(str);
                }
                synchronized (_customFormat) {
                    try {
                        return _customFormat.parse(str);
                    } catch (ParseException e) {
                        throw new IllegalArgumentException("Failed to parse Date value '" + str
                                + "' (format: \"" + _formatString + "\"): " + e.getMessage());
                    }
                }
            }
            // Issue#381
            if (t == JsonToken.START_ARRAY && ctxt.isEnabled(DeserializationFeature.UNWRAP_SINGLE_VALUE_ARRAYS)) {
                p.nextToken();
                final Date parsed = _parseDate(p, ctxt);
                t = p.nextToken();
                if (t != JsonToken.END_ARRAY) {
                    throw ctxt.wrongTokenException(p, JsonToken.END_ARRAY,
                            "Attempted to unwrap single value array for single 'java.util.Date' "
                                    + "value but there was more than a single value in the array");
                }
                return parsed;
            }
        }
        JsonToken t = p.getCurrentToken();
        if (t == JsonToken.VALUE_STRING) {
            String str = p.getText().trim();
            return getDateFromString(str);
        }
        return super._parseDate(p, ctxt);
    }

    /**
     * 返回Date类型
     * @param dateStr 支持三种格式的输入字符串
     * @return Date值
     */
    public static Date getDateFromString(String dateStr) {
        if (TextUtils.isEmpty(dateStr)) {
            return null;
        }
        final String trimStr = dateStr.trim();
        SimpleDateFormat df = null;
        switch (trimStr.length()) {
            case 19:
                df = new SimpleDateFormat(YYMMDDHHMMSS);
                try {
                    return df.parse(trimStr);

                } catch (ParseException e) {
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;

            case 16:
                df = new SimpleDateFormat(YYYMMDDHHMM);
                try {
                    return df.parse(trimStr);

                } catch (ParseException e) {
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;
            case 10:
                df = new SimpleDateFormat(YYYYMMDD);
                try {
                    return df.parse(trimStr);

                } catch (ParseException e) {
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;
            case 12:
                df = new SimpleDateFormat(ENGLISH_MMMDDYYYY, Locale.ENGLISH);
                try {
                    return df.parse(trimStr);

                } catch (ParseException e) {
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;
            default:
			//如果想添加Uct和gmt格式处理就在这里添加
                return null;
        }
    }

    public static final String YYYYMMDD = "yyyy-MM-dd";
    public static final String YYYMMDDHHMM = "yyyy-MM-dd HH:mm";
    public static final String YYMMDDHHMMSS = "yyyy-MM-dd HH:mm:ss";
    public static final String ENGLISH_MMMDDYYYY = "MMM dd, yyyy";//Apr 25, 2016
}
```

Jackson处理方式还是ObjectMapper.registerModule,注册SimpleModule，在SimplModule中添加Date类型DateDeserializer
这样再也不用担心服务器返回各种奇怪日期格式造成解析失败了
### 手动调用Api生成json
Gson中
```java

        Gson gson = new Gson();
        StringWriter stringWriter = new StringWriter();
        JsonWriter jsonWriter =  gson.newJsonWriter(stringWriter);
        jsonWriter.beginObject();
        jsonWriter.name("name")
                .value("张三")
                .name("emails")
                .beginArray()
                .value("ytjojo@163.com")
                .value("ytjojo@qq.com")
                .endArray()
                .name("age")
                .value(20)
                .endObject();
        jsonWriter.flush();
        jsonWriter.close();
        String json = stringWriter.toString();
        System.out.println(json);
```
>输出结果{"name":"张三","emails":["ytjojo@163.com","ytjojo@qq.com"],"age":20}

Jackson生成json方法如下
```java
StringWriter stringWriter = new StringWriter();
        JsonGenerator generator = new ObjectMapper().getFactory().createGenerator(stringWriter);
        generator.writeStartObject();
        generator.writeStringField("name", "张三");
        generator.writeNumberField("age", 20);
        generator.writeFieldName("emails");
        generator.writeStartArray();

        generator.writeString("ytjojo@163.com");
        generator.writeString("ytjojo@qq.com");

        generator.writeEndArray();
        generator.writeFieldName("bean");
        generator.writeEndObject();
        generator.flush();
        generator.close();

        String json = stringWriter.toString();
        System.out.println(json);
```
>输出结果{"name":"张三","age":20,"emails":["ytjojo@163.com","ytjojo@qq.com"]}
>generator中还有个writeObject(Object pojo)方法，直接将pojo类写入json中，Gson貌似没这个功能

### 手动解析Json，Gson中为JsonElement，Jackson为JsonNode
Gson 
```java
       Gson gson = new Gson();
        String json = "{\"name\":\"张三\",\"age\":20,\"emails\":[\"ytjojo@163.com\",\"ytjojo@qq.com\"]}";
        JsonElement rootEl = gson.fromJson(json, JsonElement.class);
        if (rootEl.isJsonObject()) {
            JsonObject jsonObject = rootEl.getAsJsonObject();
            JsonElement nameEl = jsonObject.get("name");
            if (nameEl.isJsonPrimitive()) {
                String name = nameEl.getAsString();
                System.out.println(name);
            }
             JsonElement emailsEl = jsonObject.get("emails");
            if(emailsEl.isJsonArray()){
                JsonArray jsonArray = emailsEl.getAsJsonArray();
                for(int i= 0 ;i< jsonArray.size() ;i++){
                    String email =jsonArray.get(i).getAsString();
                    System.out.println(email);
                }
            }
            JsonElement ageEl = jsonObject.get("age");
            if(ageEl.isJsonPrimitive()){
                JsonPrimitive agePmt= ageEl.getAsJsonPrimitive();
                if(agePmt.isNumber()){
                    int age = agePmt.getAsInt();
                    System.out.println(age+"");
                }
            }

        }
```
>张三
>ytjojo@163.com
>ytjojo@qq.com
>20
>Jackson手动解析JsonNode
```java
  String json = "{\"name\":\"张三\",\"age\":20,\"emails\":[\"ytjojo@163.com\",\"ytjojo@qq.com\"]}";

        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode node = objectMapper.readTree(json);
        if(node.isObject()){
            JsonNode nameNode = node.path("name");//path和get方法功能类似，具体可以看文档
            if(nameNode.isTextual()){
                String name = nameNode.asText();
                System.out.println(name);
            }
            JsonNode ageNode = node.path("age");
            if(ageNode.isNumber()){
                int age = ageNode.asInt();
                System.out.println(age+"");
            }
            JsonNode emailsNode = node.path("emails");
            if(emailsNode.isArray()){
                for (int i = 0; i < emailsNode.size(); i++) {
                   JsonNode emailNode =  emailsNode.get(i);
                    if(emailNode.isTextual()){
                        String email = emailNode.asText();
                        System.out.println(email);
                    }
                }
            }
        }
```
### 构建泛型类得Type JacksonJavaType，
如何反序列化泛型类如ArrayList就需要生成Type或者JavaType，传给Gson和ObjectMapper进行反序列化。Gson和Jackson都有自己的Api来进程处理，代码入下：
Gson

```java
       Type type = new TypeToken<ArrayList<String>>(){}.getType();
 Primitives.isWrapperType()
        //等同于以下下代码
      type = $Gson$Types.newParameterizedTypeWithOwner(null,ArrayList.class,String.class);
		还可以判断是不是基本数据类型和包装类
        boolean isWrapperType =   Primitives.isWrapperType(Integer.class);//true
        boolean isPrimitive =   Primitives.isPrimitive(String.class);//false
        boolean isintPrimitive =   Primitives.isPrimitive(int.class);//true
        boolean isIntegerPrimitive =   Primitives.isPrimitive(Integer.class);//false

	//判断是不是泛型类行
      if(type instanceof ParameterizedType){
            System.out.println("ParameterizedType");//Arraylist属于泛型类
       }

```
Jackson生成泛型type
```java
Type  type = new TypeReference<ArrayList<String>>(){}.getType();
 JavaType javaType  = new ObjectMapper().getTypeFactory().constructParametricType(ArrayList.class,String.class);
 boolean isTypeOrSubTypeOf = javaType.isTypeOrSubTypeOf(List.class);//是不是List子类，true

```
有了Type和JavaType我们就可以解析出具体实体类。
### 完结
关于Gson 和Jackson的使用目前就介绍到这里，如果在项目新get的使用技巧会继续补充。Thanks to read！