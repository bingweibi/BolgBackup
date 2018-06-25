---
title: Retrofit2.0教程
date: 2018-05-23 10:18:59
tags: Retrofit2.0
---

# 文章来自于
> https://blog.csdn.net/qq_26787115/article/details/53034267

# 准备工作

```
    //retrofit2.0
    implementation "com.squareup.retrofit2:retrofit:2.3.0"
    implementation "com.squareup.retrofit2:converter-gson:2.3.0"
    implementation 'com.squareup.retrofit2:adapter-rxjava:2.3.0'

    //RxJava
    implementation 'io.reactivex:rxjava:1.3.0'
    //RxAndroid
    implementation 'io.reactivex:rxandroid:1.2.1'

    //okhttp3.0
    implementation "com.squareup.okhttp3:okhttp:3.9.1"

    //Gson
    implementation "com.google.code.gson:gson:2.8.4"
```

<!--more-->
# 定义接口
>在官方文档中，第一句话就是"Retrofit turns your HTTP API into a Java Interface"

我们的项目调用的接口来自于Gank的接口，例如：http://gank.io/api/data/Android/10/1
返回的Json数据：
<center>![](http://p2g00vblr.bkt.clouddn.com/GankApi%E8%BF%94%E5%9B%9EJson%E6%95%B0%E6%8D%AE.png)</center>

## 定义一个接口类

```
public interface GankApi {
    /**
     * @return Android 信息
     */
    @GET("api/data/Android/10/1")
    Call<GankBean> getAndroidInfo();
}
```

### 分析接口的定义
http://gank.io/api/data/Android/10/1
<center>![](http://p2g00vblr.bkt.clouddn.com/%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%AE%9A%E4%B9%89)</center>

在这里我们将接口分为两部分`baseUrl`和`接口定义`，在上面的接口定义中，使用GET请求，直接请求了`api/data/Android/10/1`，这样我们就能和后面定义的`baseUrl`拼接成完整的接口了。

# 基本请求
> 官方文档中 The Retrofit class generates an implementation of the GithubService interface.

意思就是直接把Retrofit的接口变成一个类
- 创建一个Retrofit
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://gank.io")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
```
- 通过retrofit的create方法创建接口对象
```
GankApi api = retrofit.create(GankApi.class);
```
- 直接调用接口方法，返回一个Call
```
Call<GankBean> call = api.getAndroidInfo();
```
- 异步请求
```
call.enqueue(new Callback<GankBean>() {
            @Override
            public void onResponse(Call<GankBean> call, Response<GankBean> response) {
             GankBean.ResultsBean bean = response.body().getResults().get(0);
                mTextView.setText(
                        "_id:"+bean.get_id()+"\n"
                                + "createdAt：" + bean.getCreatedAt() + "\n"
                                + "desc：" + bean.getDesc() + "\n"
                                + "images:" + bean.getImages() + "\n"
                                + "publishedAt:" + bean.getPublishedAt() + "\n"
                                + "source" + bean.getSource() + "\n"
                                + "type:" + bean.getType() + "\n"
                                + "url: " + bean.getUrl() + "\n"
                                + "who:" + bean.getWho());
            }

            @Override
            public void onFailure(Call<GankBean> call, Throwable t) {

            }
        });
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/Screenshot_2018-05-11-11-13-09.png)</center>

基本请求就是这样，配置做好，根据api定义好接口，配合Gson。请求完了就可以直接获得值了。

# Get的动态参数

举个栗子
我们都知道url大多数都是拼接的，像我们查询天气的接口，就是动态传一个城市的名字。

> http://op.juhe.cn/onebox/weather/query?cityname=深圳&key=您申请的KEY

默认cityname是深圳，后面就是我们需要拼接的

1. WeatherApi
```
public interface WeatherApi {
    @GET("onebox/weather/query?cityname=深圳")
    Call<WeatherDataBean> getWeather(@Query("key")String kry);
}
```
这里可以看到，在Get的时候还是把连接的后半段传进去，但是在这里最后拼接的时候是一个key，所以在传入参数的前面加上`Query`
2. 代码
```
//        WeatherDataApi
//        Get的动态参数
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://op.juhe.cn/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        WeatherApi api = retrofit.create(WeatherApi.class);
        Call<WeatherDataBean> call = api.getWeather("4ea58de8a7573377cec0046f5e2469d5");
//        异步
        call.enqueue(new Callback<WeatherDataBean>() {
            @Override
            public void onResponse(Call<WeatherDataBean> call, Response<WeatherDataBean> response) {
                String info = response.body().getResult().getData().getRealtime().getWeather().getInfo();
                mTextView.setText("深圳天气: " + info);
            }

            @Override
            public void onFailure(Call<WeatherDataBean> call, Throwable t) {

            }
        });
```
3. 运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/Screenshot_2018-05-11-17-03-15.png)</center>

# Get参数请求
举个栗子
>http://gank.io/api/data/Android/10/1

例如我们的接口是直接传参的
> //后面三个参数
 Android可接受参数  | Android | iOS | 休息视频 | 福利 | 前端 | App
 count 最大 50
 page  是页数
 
 这种类型又该如何操作呢？
 
 - 假设我们直接传入的参数是page
```
public interface Gank2Api {
    @GET("api/data/Android/10/{page}")
    Call<GankBean> getAndroidInfo(@Path("page") int page);
}
```
在这里使用大括号做占位符，然后使用Path关键字，接着继续写代码
```
//        Get参数请求
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://gank.io/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        Gank2Api api = retrofit.create(Gank2Api.class);
        api.getAndroidInfo(1).enqueue(new Callback<GankBean>() {
            @Override
            public void onResponse(Call<GankBean> call, Response<GankBean> response) {
                mTextView.setText(response.body().getResults().get(0).getDesc());
            }

            @Override
            public void onFailure(Call<GankBean> call, Throwable t) {

            }
        });
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/Screenshot_2018-05-11-17-13-25.png)</center>

# Get参数拼接
```
public interface WeatherApi2 {
    @GET("onebox/weather/query?")
    Call<WeatherDataBean> getWeather(@QueryMap Map<String, String> params);
}
```
Get后面传入的参数并没有像cityname一类的参数，但是有一个QueryMap,传入的是键值对
```
//      Get参数拼接
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://op.juhe.cn/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        WeatherApi2 api = retrofit.create(WeatherApi2.class);
        Map<String, String> params = new HashMap<>();
        params.put("cityname","深圳");
        params.put("key","4ea58de8a7573377cec0046f5e2469d5");
        api.getWeather(params).enqueue(new Callback<WeatherDataBean>() {
            @Override
            public void onResponse(Call<WeatherDataBean> call, Response<WeatherDataBean> response) {
                mTextView.setText(response.body().getResult().getData().getRealtime().getWeather().getInfo());
            }

            @Override
            public void onFailure(Call<WeatherDataBean> call, Throwable t) {

            }
        });
```
运行结果
<center>![](http://p2g00vblr.bkt.clouddn.com/Screenshot_2018-05-11-17-19-21.png)</center>

# Post
```
import retrofit2.Call;
import retrofit2.http.Body;
import retrofit2.http.POST;

public interface PostApi {

    @POST("user/new")
    Call<Result> postUser(@Body User user);
}
```
这里POST的地址和之前的get也是一样的，这里返回一个Result是我们自己定义的结果类，Body是表示参数，我需要一个User，那我们的User就是
```
public class User {

    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```
public class Result {

    private int yes;
    private int no;

    public int getYes() {
        return yes;
    }

    public void setYes(int yes) {
        this.yes = yes;
    }

    public int getNo() {
        return no;
    }

    public void setNo(int no) {
        this.no = no;
    }
}
```
调用
```
User user = new User();
    user.setId(1);
    user.setName("lgl");
    api.postUser(user).enqueue(new Callback<Result>() {
        @Override
        public void onResponse(Call<Result> call, Response<Result> response) {
            if (response.body().getYes() == 0) {
                Toast.makeText(MainActivity.this, "成功", Toast.LENGTH_SHORT).show();
            }
        }

        @Override
        public void onFailure(Call<Result> call, Throwable t) {

        }
    });
```
我在这里传了一个user进去，细节不过就是id和name，如果请求成功，那就返回0，失败就是1，这里服务端定义，这样我们就POST完成了

# Post提交表单
```
@POST("user/edit")
Call<Result> editUser(@Field("id") int id, @Field("name") String name);
```
直接调用方法
```
api.editUser(1, "liuguilin").enqueue(new Callback<Result>() {
        @Override
        public void onResponse(Call<Result> call, Response<Result> response) {
            if (response.body().getYes() == 0) {
                Toast.makeText(MainActivity.this, "成功", Toast.LENGTH_SHORT).show();
            }
        }

        @Override
        public void onFailure(Call<Result> call, Throwable t) {

        }
    });
```

# RxJava + Retrofit2.0
举个栗子
登录，登录成功后获取到user_id，再去请求用户信息，这里应该是两个请求对吧，我们先去写好接口，这里我们先用常规的方法去获取：
```
    @POST("user/login")
    Call<User> login(@Field("username") String user, @Field("password") String password);

    @GET("user/info")
    Call<User> getUser(@Query("id") String id);
```
这里的两个接口，一个是登录，传参用户名和密码，还有一个是用id去查找用户信息的
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("baseUrl")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
 api = retrofit.create(PostApi.class);
```
这里我们需要增加addCallAdapterFactory为我们后面的Rx做准备，然后我们调用两次
```
api.login("liuguilin", "748778890").enqueue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        if (response.isSuccessful()) {
            String id = response.body().getUser_id();
            api.getUser(id).enqueue(new Callback<User>() {
                @Override
                public void onResponse(Call<User> call, Response<User> response) {

                    Toast.makeText(MainActivity.this, "id:" +
                                    response.body().getId()
                                    + "name:" + response.body().getName(),
                            Toast.LENGTH_SHORT).show();
                }

                @Override
                public void onFailure(Call<User> call, Throwable t) {

                }
            });
        }
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {

    }
});
```
RxJava写法
重新定义两个接口
```
    @POST("user/login")
    rx.Observable<User> loginForRX(@Body User user);

    @GET("user/info")
    rx.Observable<User> getUserForRX(@Query("id") String id);
```
这里的是伪代码，注意看使用方法
```
api.loginForRX(new User("liuguilin", "748778890")).flatMap(new Func1<User, Observable<User>>() {
    @Override
    public Observable<User> call(User user) {
        return api.getUser(user.getUser_id());
    }
}).subscribe(new Action1<User>() {
    @Override
    public void call(User user) {
        Toast.makeText(MainActivity.this, "name:" + user.getName(), Toast.LENGTH_SHORT).show();
    }
});
```