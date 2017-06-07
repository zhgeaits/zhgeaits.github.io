---
layout: post
title:  "Learning Android Network Library OkHttp"
date:   2017-02-23 11:00:03
categories: android
supertype: career
type: android
---

现在OkHttp已经发展到了3.6版本了，虽然一出来就已经知道，但是从来没有去学习和使用，只是简单的了解过一下，也曾经在[这篇blog](http://zhgeaits.me/android/2014/09/15/android-http-study-notes.html)里面关于Android网络框架有过一些记录。今天就来学习和使用一下这个框架吧，毕竟，从Android4.4开始，底层的网络已经使用OkHttp和OkIO来代替了HttpURLConnection的默认实现，[提交的历史在这](https://android.googlesource.com/platform/libcore/+/2503556d17b28c7b4e6e514540a77df1627857d0)。

## 1. What's OkHttp

官方[地址在这](http://square.github.io/okhttp/)，还有[GitHub](https://github.com/square/okhttp)。

虽然4.4以后系统使用OkHttp了，但是低端机还有很多，我们还是需要支持的。当年FaceBook的App以前的优化加载图片就是使用了OkHttp，因为OkHttp 支持在糟糕的网络环境下面更快的重试，并且还能利用SPDY协议进行快速的并发网络请求。

在OkHttp的官网上面介绍得比较清楚，反正就是一堆的高性能啊高效啊的好处，然后google一下OkHttp的教程，基本上都是翻译了一下而已。当然，现在已经很多研究源码的资料了，我现在还没有这个研究源码的兴趣呢。

关于网上的教程，比较好的有[IBM的这篇](https://www.ibm.com/developerworks/cn/java/j-lo-okhttp/)，和[稀土挖金的这篇](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html)。既然网上都有教程，我也不搬砖，这里只是记录一些自己的理解和别人没有的东西。

## 2.Volley扩展

关于Volley里面扩展支持使用OkHttp，在这个[Gist](https://gist.github.com/JakeWharton/5616899)上有大家的交流。

## 3.封装使用

对于OkHttp的封装使用，网上的代码一大堆，各有千秋，但是目的都是一个方便好用，当然是对于自己的项目而言。下面是我的封装使用。

首先封装一个参数类：

{% highlight java %}

public abstract class RequestParam {

	// 每个请求不同的参数
	protected Map<String, String> mParams = new HashMap<String, String>();
	// 基本的参数,不参与生成cachekey
	protected Map<String, String> mBase = new HashMap<String, String>();
	// 上传文件参数
	protected FileParam mFileParam;
    // 上传流的参数
    protected StreamParam mStreamParam;

	public static RequestParam defaultParam() {
		return new ZGRequestParam().buildBaseParam();
	}

	public abstract RequestParam buildBaseParam();

	public abstract Map<String, String> mapAllParams();

	public abstract String buildUrlWithQueryString(String url, RequestParam params);

	protected void addBaseParam(String key, String value) {
		if (key == null) {
			return;
		}
		if (value == null) {
			return;
		}
		mBase.put(key, value);
	}

	public void addParam(String key, String value) {
		if (key == null) {
			return;
		}
		if (value == null) {
			return;
		}
		mParams.put(key, value);
	}

	public void addParam(String key, int value) {
		addParam(key, String.valueOf(value));
	}

	public void addParam(String key, long value) {
		addParam(key, String.valueOf(value));
	}

	public void addParam(String key, float value) {
		addParam(key, String.valueOf(value));
	}

	public void addParam(String key, double value) {
		addParam(key, String.valueOf(value));
	}

	public String getParam(String key) {
		if (key == null) {
			return null;
		}
		if (mParams.containsKey(key)) {
			return mParams.get(key);
		}
		return mBase.get(key);
	}

	public void addFileParam(String key, File file, String mediaType) {
		if (key == null) {
			return;
		}
		if (file == null) {
			return;
		}
		mFileParam = new FileParam();
		mFileParam.key = key;
		mFileParam.file = file;
		if (mediaType != null) {
			mFileParam.mediaType = mediaType;
		} else {
			mFileParam.mediaType = "application/octet-stream";
		}
	}

	public FileParam getFileParam() {
		return mFileParam;
	}

    public void addStreamParam(String key, String filename, InputStream inputStream, String mediaType) {
        if (FP.empty(key)) {
            return;
        }
        if (inputStream == null) {
            return;
        }
        mStreamParam = new StreamParam();
        mStreamParam.key = key;
        mStreamParam.filename = filename;
        mStreamParam.inputStream = inputStream;
        if (!FP.empty(mediaType)) {
            mStreamParam.mediaType = mediaType;
        } else {
            mStreamParam.mediaType = "application/octet-stream";
        }
    }

    public StreamParam getStreamParam() {
        return mStreamParam;
    }

	public String makeParamsString() {
		List<String> keyList = new ArrayList<String>();
		Set<String> keySet = mParams.keySet();
		Set<String> baseKeySet = mBase.keySet();
		if (keySet.size() > 0 || baseKeySet.size() > 0) {
			for (String key : keySet) {
				keyList.add(key);
			}
			for (String key : baseKeySet) {
				keyList.add(key);
			}
		} else {
			return "";
		}

		Map<String, String> tempParams = new HashMap<String, String>();
		tempParams.putAll(mParams);
		tempParams.putAll(mBase);
		Collections.sort(keyList);
		String result = toStringByKeyOrder(keyList, tempParams);
		return result;
	}

	private String toStringByKeyOrder(List<String> keyList, Map<String, String> params) {
		if (keyList == null || keyList.size() == 0) {
			return "";
		}
		StringBuilder result = new StringBuilder();
		for (String key : keyList) {
			result.append(key);
			result.append("=");
			result.append(params.get(key));
			result.append("&");
		}
		result.deleteCharAt(result.length() - 1);
		return result.toString();
	}

	public class FileParam {
		public String mediaType;
		public String key;
		public File file;
	}

    public class StreamParam {
        public String mediaType;
        public String key;
        public String filename;
        public InputStream inputStream;
    }
}

{% endhighlight %}

它是一个抽象类，这样设计的原因是，可以通过参数扩展接口，不然局限性就很大了。

下面是一个具体的实现类：

{% highlight java %}

public class ZGRequestParam extends RequestParam {

    @Override
    public RequestParam buildBaseParam() {
        return this;
    }

    @Override
    public String buildUrlWithQueryString(String url, RequestParam params) {
        if(params != null) {
            String paramString = params.makeParamsString();
            if (paramString != null && paramString.length() > 0) {
                if (url.indexOf("?") == -1) {
                    url += "?" + paramString;
                } else {
                    url += "&" + paramString;
                }
            }
        }
        return url;
    }

    @Override
    public Map<String, String> mapAllParams() {
        Map<String, String> tempParams = new HashMap<String, String>();
        tempParams.putAll(mParams);
        tempParams.putAll(mBase);
        return tempParams;
    }
}

{% endhighlight %}

可以看到，我可以根据不同的业务需求扩展不同的参数需求，而不需要改动封装的代码。

下面看看封装的OkHttp代码：

{% highlight java %}

public class OkHttpClientUtils {

    private static OkHttpClientUtils instance = null;
    private static OkHttpClient mOkHttpClient = null;
    private Handler mMainHandler;

    public static OkHttpClientUtils getInstance() {
        if (instance == null) {
            synchronized (OkHttpClientUtils.class) {
                if (instance == null) {
                    instance = new OkHttpClientUtils();
                }
            }
        }
        return instance;
    }

    private OkHttpClientUtils() {
    }

    public void init() {
        mMainHandler = new Handler(Looper.getMainLooper());
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        builder.connectTimeout(12, TimeUnit.SECONDS);
        builder.readTimeout(12, TimeUnit.SECONDS);
        builder.writeTimeout(12, TimeUnit.SECONDS);
        StethoUtils.addOkHttpInterceptor(builder);
        mOkHttpClient = builder.build();
    }

    public String submitRequest(String url, RequestParam requestParam) throws IOException {
        String paramUrl = requestParam.buildUrlWithQueryString(url, requestParam);
        Request request = new Request.Builder().url(paramUrl).build();
        Response response = mOkHttpClient.newCall(request).execute();
        if (response.isSuccessful()) {
            return response.body().string();
        } else {
            throw new IOException("Unexpected code " + response);
        }
    }

    public void submitRequestAsync(String url, RequestParam requestParam, final OnHttpListener listener) {
        String paramUrl = requestParam.buildUrlWithQueryString(url, requestParam);
        Request request = new Request.Builder().url(paramUrl).build();
        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                final IOException exception = e;
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onFailure(exception);
                        }
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String result = response.body().string();
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onResponse(result);
                        }
                    }
                });
            }
        });
    }

    public String submitPostRequest(String url, RequestParam requestParam) throws IOException {
        FormBody.Builder formBuilder = new FormBody.Builder();

        Map<String, String> allParams = requestParam.mapAllParams();
        for (Map.Entry<String, String> entry : allParams.entrySet()) {
            formBuilder.add(entry.getKey(), entry.getValue());
        }

        Request request = new Request.Builder().url(url).post(formBuilder.build()).build();
        Response response = mOkHttpClient.newCall(request).execute();
        if (response.isSuccessful()) {
            return response.body().string();
        } else {
            throw new IOException("Unexpected code " + response);
        }
    }

    public void submitPostRequestAsync(String url, RequestParam requestParam, final OnHttpListener listener) {
        FormBody.Builder formBuilder = new FormBody.Builder();

        Map<String, String> allParams = requestParam.mapAllParams();
        for (Map.Entry<String, String> entry : allParams.entrySet()) {
            formBuilder.add(entry.getKey(), entry.getValue());
        }

        Request request = new Request.Builder().url(url).post(formBuilder.build()).build();

        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                final IOException exception = e;
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onFailure(exception);
                        }
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String result = response.body().string();
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onResponse(result);
                        }
                    }
                });
            }
        });
    }

    public void submitFileRequest(String url, RequestParam requestParam, final OnHttpListener listener) {
        MultipartBody.Builder multiBuilder = new MultipartBody.Builder();

        Map<String, String> allParams = requestParam.mapAllParams();
        for (Map.Entry<String, String> entry : allParams.entrySet()) {
            multiBuilder.addFormDataPart(entry.getKey(), entry.getValue());
        }

        RequestParam.FileParam fileParam = requestParam.getFileParam();
        if (fileParam != null) {
            RequestBody fileBody = RequestBody.create(MediaType.parse(fileParam.mediaType), fileParam.file);
            multiBuilder.addFormDataPart(fileParam.key, fileParam.file.getPath(), fileBody);
        }

        Request request = new Request.Builder().url(url).post(multiBuilder.build()).build();

        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                final IOException exception = e;
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onFailure(exception);
                        }
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                final String result = response.body().string();
                mMainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        if (listener != null) {
                            listener.onResponse(result);
                        }
                    }
                });
            }
        });
    }

    public interface OnHttpListener {
        void onFailure(IOException e);
        void onResponse(String response);
    }

    public interface OnProgressHttpListener extends OnHttpListener {
        void onProgress(long total, long progress);
    }
}

{% endhighlight %}

直接看代码就好了，不要太多的解释，封装也是简单。注意的是，如果请求没有带gzip参数，OkHttp会自动带上的，详细参见：BridgeInterceptor。

## 4.关于拦截器

其实我对拦截器的概念并不陌生，在大学的时候，那时候我开发web，学习Servlet，就写过Filter过滤器了，里面的原理也比较了解，可以看这篇[blog](http://zhgeaits.me/javaweb/2011/03/31/javaweb-servlet-study-notes.html)。当我学习Struts2的时候，真正接触拦截器的概念，并且都是链式的连接器，也开发过代码，原理都比较理解，详细看这篇[blog](http://zhgeaits.me/javaweb/2011/04/28/javaweb-struts-study-notes.html)。

而其实OkHttp的拦截器也是一样的原理，都是链式执行的。它拦截的是请求的头部和响应的头部，具体，它分了两种拦截器，如下图所示：

![alt iterceptor](http://img.blog.csdn.net/20151125170226504?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "Iterceptor")

应用拦截器对于每个HTTP响应都只会调用一次，可以通过不调用Chain.proceed方法来终止请求，也可以通过多次调用Chain.proceed 方法来进行重试。网络拦截器对于调用执行中的自动重定向和重试所产生的响应也会被调用，而如果响应来自缓存，则不会被调用。

当然，我并没有看过源码，并且http网络这块的底层的东西还是很多不够了解的，这点以后在学习补充吧。