---
title: json解析的三种方式
date: 2018-01-20 18:58:01
tags: 
	- Json 
	- Android
	- Gson
---

**Json 是什么？**

json是一种用于进行网络数据传输的格式，使用广泛。在json中有JsonObject和JsonArray两种形式
> 当你看到以 **{** 开始的说明这是一个JsonObject
> 如果是看到以 **[** 开始的说明是一个JsonArray

1. json 数据在键/值对中
2. 数据由逗号分隔
3. 花括号{}保存对象
4. 方括号[]保存数组

**json对象**
JsonObject(json对象)在花括号{}中，json对象包括多个键/值对，例如：
```
{"name":"zhangsan" , "age":25}
```
<!-- more -->

**json数组**
JsonArray(json数组)在方括号[]中，数组也可以包括多个对象，例如：
```
{"student":[{"name":"zhangsan" , "age":12},{"name":"lisi" , "age":13},{"name":"wangwu" , "age":14}]}
```
---
###Android中如何解析json
前面铺垫了这么久，现在才是这篇文章的重点，如何在android中对json数据进行解析。在接下来的解析过程中我们分别使用三种方式对下面的json数据进行解析。

---
**json数据**
```
json1 = {"name":"sam","age":18,"weight":60} //只包含json对象
json2 = [12,13,15] //只包含json数组
json3 = [{"name":"sam","age":18},{"name":"leo","age":19},{"name":"sky", "age":20}] //含有对象的json数组
```

---
**解析的方式**
1. 使用util包中的JsonReader进行解析
2. 使用org.json包中的JSONObject和JSONArray进行解析
3. 使用谷歌的[Gson](https://github.com/google/gson)包进行解析

---
###对json1分别用三种方式进行解析
```
json1 = {"name":"sam","age":18,"weight":60}
```

* JsonReader方式
```
JsonReader reader = new JsonReader(new InputStreamReader(json1));
try{
    reader.beginObject();
    while(reader.hasNext()){
        String keyName = reader.nextName();
        if("name".equals(keyName)){
        String name = reader.nextString();
        }else if("age".equals(keyName)){
            int age = reader.nextInt();
        }else if("weight".equals(keyName)){
            int weight = reader.nextInt();
        }
    }
    reader.endObject();
}
finally{
    reader.close();
}
```
* JSONObject方式和JSONArray方式
```
JSONObject jsonObj = new JSONObject(json1);
String name = jsonObj.optString("name");
int age = jsonObj.optInt("age");
int weight = jsonObj.optInt("weight");
```
* Gson方式
```
Gson gson = new Gson();
Student student = gson.fromJson(json1, Student.class);
```
如果希望采用这种方式进行解析，那么需要把json所有的对象都需要在bean中写出来，而且名字也需要和返回的对象名一样。类似于：
```
private String name;
    private int age;
    private int weight;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }
```
***
###对json2进行解析
```
json2 = [12,13,15]
```
* JsonReader方式
```
JsonReader reader = new JsonReader(new InputStreamReader(json2));
try {
    List<Integer> ages = new ArrayList<Integer>();
    reader.beginArray();
    while (reader.hasNext()) {
        ages.add(reader.nextInt());
    }
    reader.endArray();
} 
finally {
    reader.close();
}
```
* JsonObject和JsonArray方式
```
JSONArray jsonArray = new JSONArray(json2);
for (int i= 0; i < jsonArray.length(); i++) {
    int age = jsonArray.optInt(i); 
}
```
* Gson方式
对于Json数组我们可以解析成数组或者list形式

 * 数组形式
```
Gson gson = new Gson();
int[] ages = gson.fromJson(json2, int[].class);
```

 * list形式
```
Gson gson = new Gson();
List<Integer> ages = gson.fromJson(json2, new TypeToken<List<Integer>>(){}.getType);
```

###对json3解析
```
json3 = [{"name":"sam","age":18},{"name":"leo","age":19},{"name":"sky", "age":20}]
```
* jsonReader方式
```
JsonReader reader = new JsonReader(new InputStreamReader(json3));
try {
    List<Object> ages = new ArrayList<Object>();
    reader.beginArray();
    while (reader.hasNext()) {
        reader.biginObject();
        while(reader.hasNext()){
            String keyName = reader.nextName();
            if("name".equals(keyName)){
            String name = reader.nextString();
            }else if("age".equals(keyName)){
                int age = reader.nextInt();
            }else if("weight".equals(keyName)){
                int weight = reader.nextInt();
            }
        }
        reader.endObject();
    }
    reader.endArray();
} 
finally {
    reader.close();
}
```
* JsonObject和JsonArray方式
```
JSONArray jsonArray = new JSONArray(json3);
for (int i= 0; i < jsonArray.length(); i++) {
    JSONObject jsonObject = jsonArray.optJSONObject(i);
    String name = jsonObject.optString("name");
    int age = jsonObject.optInt("age");
}
```
* Gson方式
* 我们直接解析成list方式
```
Gson gson = new Gson();
List<Student> students = gson.fromJson(json3, new TypeToke<List<Student>>(){}.getType);
```
###总结

在平时的解析过程过程中，我是很少采用第一种方式，更多的是第二种方式和第三种方式相互结合起来.Gson在解析那种对象较少的情况下还是比较方便，如果对象较多，那么写bean的时候，那感觉就不说了，第二种方式其实挺不错的。

---
###最后
附上一个平时解析的真实案例：
json数据：
```
{
"date": "20171210",
"stories": [
{
"images": [
"https://pic1.zhimg.com/v2-f7ef9c4491a77fd0921b39d0ce71ff9c.jpg"
],
"type": 0,
"id": 9660440,
"ga_prefix": "121016",
"title": "《塞尔达传说：旷野之息》获 TGA 2017 年度最佳游戏，名至实归"
},
{
"images": [
"https://pic1.zhimg.com/v2-e88b85b62e17b2f7bcc643c1e2161158.jpg"
],
"type": 0,
"id": 9660342,
"ga_prefix": "121014",
"title": "该用的东西永远找不到、永远乱糟糟，其实厨房收纳有技巧"
},
{
"images": [
"https://pic2.zhimg.com/v2-c8411ecbb96cdaacacc76402fc134a85.jpg"
],
"type": 0,
"id": 9659553,
"ga_prefix": "121012",
"title": "大误 · 在魔法次元你主修什么？"
},
{
"images": [
"https://pic1.zhimg.com/v2-4b403aab5f1728ca8dcc81cb3b8d7bc0.jpg"
],
"type": 0,
"id": 9660290,
"ga_prefix": "121011",
"title": "刚学开车，要是不注意这些经验，怕是要「送人头」的"
},
{
"images": [
"https://pic2.zhimg.com/v2-5163ab7218452fced95c39b427b320a1.jpg"
],
"type": 0,
"id": 9660389,
"ga_prefix": "121010",
"title": "「商业模式」，见过无数次，从来也没人解释这词什么意思"
},
{
"images": [
"https://pic3.zhimg.com/v2-a1d9bc75eb5f833c7dd7a110ece0c6d6.jpg"
],
"type": 0,
"id": 9659495,
"ga_prefix": "121009",
"title": "后来我才知道，这个世上并不存在没有任何「残」「障」的人"
},
{
"images": [
"https://pic3.zhimg.com/v2-85aad5f9d12407b768b72c48d4994686.jpg"
],
"type": 0,
"id": 9660420,
"ga_prefix": "121008",
"title": "本周热门精选 · 不安的时代"
},
{
"images": [
"https://pic1.zhimg.com/v2-3b3598e906c694016ae6b3ad82d7c300.jpg"
],
"type": 0,
"id": 9660432,
"ga_prefix": "121007",
"title": "有哪些设计很棒的学校宿舍楼？"
},
{
"images": [
"https://pic4.zhimg.com/v2-92c3e744296c1785dd8121269be9a5f7.jpg"
],
"type": 0,
"id": 9660422,
"ga_prefix": "121007",
"title": "在微博就能找到梅西的年代，信虫还在用最原始的方法追星"
},
{
"images": [
"https://pic1.zhimg.com/v2-38b60458db353bc0fca4b640e7f974a0.jpg"
],
"type": 0,
"id": 9659975,
"ga_prefix": "121007",
"title": "消失中的线下服装市场：一个时代已远去"
},
{
"images": [
"https://pic3.zhimg.com/v2-cd12dc594ecef927aabf1f1fa98e2bc2.jpg"
],
"type": 0,
"id": 9660412,
"ga_prefix": "121006",
"title": "瞎扯 · 如何正确地吐槽"
}
],
"top_stories": [
{
"image": "https://pic3.zhimg.com/v2-4a898c5475f6476d3b8378165c27634e.jpg",
"type": 0,
"id": 9660420,
"ga_prefix": "121008",
"title": "本周热门精选 · 不安的时代"
},
{
"image": "https://pic2.zhimg.com/v2-0df5956d2c24f8b76f5198227714680d.jpg",
"type": 0,
"id": 9660189,
"ga_prefix": "120916",
"title": "赚了钱，企业家们该不该扶贫？"
},
{
"image": "https://pic4.zhimg.com/v2-557ca00c7e34c6ed672ea5b9e5d395a3.jpg",
"type": 0,
"id": 9660049,
"ga_prefix": "120910",
"title": "把一个人丢进蚊子窝里会死，但却不是因为失血而亡……"
},
{
"image": "https://pic2.zhimg.com/v2-910ace866bcf0a8a370f95f082403489.jpg",
"type": 0,
"id": 9659771,
"ga_prefix": "120909",
"title": "只见人进，不见人归：传说中的「死亡谷」"
},
{
"image": "https://pic3.zhimg.com/v2-15949b00edcf48c610499dfc096b85be.jpg",
"type": 0,
"id": 9659975,
"ga_prefix": "121007",
"title": "消失中的线下服装市场：一个时代已远去"
}
]
}
```
bean:
```
package com.example.bbw.openzz.Model.ZhiHuDaily;


import java.util.List;

/**
 * Created by bbw on 2017/10/26.
 * @author bbw
 */

public class ZhiHuDaily {

    private List<StoryBean> storiesList;

    public List<StoryBean> getStoriesList() {
        return storiesList;
    }

    public void setStoriesList(List<StoryBean> storiesList) {
        this.storiesList = storiesList;
    }

    public static class StoryBean {

        private String title;
        private String images;
        private int id;

        public StoryBean(String title, String images, int id) {
            this.title = title;
            this.images = images;
            this.id = id;
        }

        public String getTitle() {
            return title;
        }

        public void setTitle(String title) {
            this.title = title;
        }

        public String getImages() {
            return images;
        }

        public void setImages(String images) {
            this.images = images;
        }

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }
    }
}

```
解析：
```
 public static List<ZhiHuDaily.StoryBean> handleZhuHuDailyLatest(String responseText) throws JSONException {

            JSONObject jsonObject = new JSONObject(responseText);
            JSONArray jsonArray = jsonObject.getJSONArray("stories");
            List<ZhiHuDaily.StoryBean> storyBeansList = new ArrayList<>();
            for (int i=0;i<jsonArray.length();i++){
                JSONObject newsInJson = jsonArray.getJSONObject(i);
                int id = newsInJson.optInt("id");
                String title = newsInJson.optString("title");
                String image = "";
                if (newsInJson.has("images")) {
                    image = (String) newsInJson.getJSONArray("images").get(0);

                }
                ZhiHuDaily.StoryBean news = new ZhiHuDaily.StoryBean(title,image,id);
                storyBeansList.add(news);
            }
        return storyBeansList;
    }
```