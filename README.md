## 我最新的网络框架请移步[https://github.com/qibin0506/Glin](https://github.com/qibin0506/Glin)
# AndNet
AndNet是一个Android开中中二次封装的网络框架，可以任意轻松切换使用的底层网络请求框架，AndNet使用Parser-Callback模式，可以轻松实现从网络请求到数据解析的整个操作步骤。

AndNet的网络请求框架默认使用OkHttp，当然你完全可以轻松的实现自己的请求操作并且替换，而你的业务逻辑代码无需任何变动。

## 更新日志

### 0.1.2版本更新

1. 加入put、delete请求的支持
2. 可直接post、put一段json
3. cancel加入stack为空判断
 
***

### 0.1.1版本更新

1. 加入debug
2. 修复WeakReference带来的问题
3. 重用网络框架的cancel功能，加入tag标识

## 使用

### 1 初始化

``` java
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        OkHttpStack okHttpStack = new OkHttpStack();
        okHttpStack.debug(true);
        Net.init(okHttpStack);
       
    }
}
```

### 2 定义parser

``` java
public class CommParser<T> implements Net.Parser<T> {
	
	private String mKey;
	
	public CommParser(String key) {
		mKey = key;
	}

	@Override
	public Result<T> parse(String response) {
		Result<T> result = new Result<T>();
		try {
			JSONObject baseObject = JSON.parseObject(response);
			if(!baseObject.getBooleanValue("success")) {
				result.setMsg(baseObject.getString("message"));
			}else {
				Class<T> klass = Helper.generateType(getClass());
				if(klass == null) throw new Exception();
				
				T t = baseObject.getObject(mKey, klass);
				result.setStatus(Result.SUCCESS);
				result.setResult(t);
				return result;
			}
		} catch (Exception e) {
			e.printStackTrace();
			result.setMsg(Net.ERR_PARSE_MSG);
		}
		
		result.setStatus(Result.ERROR);
 		return result;
	}
}

```

### 3 get请求

``` java
Net.get("http://192.168.3.116/?name=loader&age=18&city=jinan",
                new CommParser<User>("user") {}, new Net.Callback<User>() {
            @Override
            public void callback(Result<User> result) {
                if(result.getStatus() == Result.SUCCESS) {
                    User user = result.getResult();

                    mTextView.setText(user.getName());
                    mTextView.append("\n" + user.getAge());
                    mTextView.append("\n" + user.getCity());
                }else {
                    mTextView.setText(result.getMsg());
                }
            }
        }, getClass().getName());
```

### 4 post请求
``` java
        User user = new User();
        user.setName("qibin");
        user.setCity("shandong");
        user.setAge(18);

        Net.post("http://192.168.3.116/", user, new CommParser<User>("user") {
                }, new Net.Callback<User>() {
                    @Override
                    public void callback(Result<User> result) {
                        if(result.getStatus() == Result.SUCCESS) {
                            mTextView.setText(result.getResult().toString());
                        }else {
                            mTextView.setText(result.getMsg());
                        }
                    }
                }, getClass().getName());
```

### 5 文件上传

``` java

        RequestParams params = new RequestParams("name", "qibin");
        params.add("file", new File(Environment.getExternalStorageDirectory()
                + "/dl.jar"));
        Net.post("http://192.168.3.116/upload.php", params, new Net.NoParser(),
                new Net.Callback<String>() {
            @Override
            public void callback(Result<String> result) {
                mTextView.setText(result.getResult() + "");
            }
        }, getClass().getName());
```

### cancel

``` java

class MyActivity extend Activity {
    
    @Override
    public void onDestroy() {
        Net.cancel(getClass().getName());
    }
}

```

### 6 定制HttpStack
``` java
public class VolleyStack<T> extends AbsHttpStack<T> {

    private Application mContext;

    public VolleyStack(Application context) {
        mContext = context;
    }

    /**
     * get请求
     *
     * @param url      网址
     * @param parser   解析器
     * @param callback 回调
     */
    @Override
    public void get(String url, Net.Parser<T> parser,
                    Net.Callback<T> callback, final Object tag) {
        invoke(Request.Method.GET, url, null, parser, callback, tag);
    }

    /**
     * post请求
     *
     * @param url      访问的url
     * @param params   post参数
     * @param parser   解析器
     * @param callback 回调
     */
    @Override
    public void post(String url, RequestParams params,
                     Net.Parser<T> parser,
                     Net.Callback<T> callback,
                     final Object tag) {
        invoke(Request.Method.POST, url, params, parser, callback, tag);
    }

    /**
     * 执行网络请求
     *
     * @param url
     * @param params   post时请求的参数 get时为null
     * @param parser
     * @param callback
     * @param method
     */
    private void invoke(final int method, final String url,
                        final RequestParams params,
                        final Net.Parser<T> parser,
                        final Net.Callback<T> callback,
                        final Object tag) {
        StringRequest request = new StringRequest(method, url,
                new Response.Listener<String>() {
                    public void onResponse(String response) {
                        onNetResponse(parser, callback, response);
                    }
                }, new Response.ErrorListener() {
            public void onErrorResponse(VolleyError error) {
                error.printStackTrace();
                onError(callback, Net.DEF_ERR_MSG);
            }
        }) {
            @Override
            protected Map<String, String> getParams()
                    throws AuthFailureError {
                if (params != null) return params.get();
                return super.getParams();
            }
        };

        VolleyManager.getInstance(mContext).add(request, tag);
    }

    @Override
    public void cancel(Object tag) {
        VolleyManager.getInstance(mContext).cancel(tag);
    }
}

public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Net.init(new VolleyStack(this));
//        Net.init(new OkHttpStack());
//        Net.init(new OkHttpHeaderStack());
    }
}
```

更多内容请操作实例代码和博客：[http://blog.csdn.net/qibin0506/article/details/50127223](http://blog.csdn.net/qibin0506/article/details/50127223)
