### 简介
　　在2013年Google I/O大会上推出了一个新的网络通信框架——Volley。Volley可是说是把AsyncHttpClient和Universal-Image-Loader的优点集于了一身，既可以像AsyncHttpClient一样非常简单地进行HTTP通信，也可以像Universal-Image-Loader一样轻松加载网络上的图片
 
### 配置方法
　　将 [jar包 ][1]导入工程即可

### 请求String类型数据

 1. 得到RequestQueue对象，Volley所有的请求都是通过这个请求队列发起的

``` java
RequestQueue mQueue = Volley.newRequestQueue(context);  
```


 2. 创建一个StringRequest，指定请求方式，URL，和监听

``` java
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
		new Response.Listener<String>() {
			@Override
			public void onResponse(String response) {
			   //得到结果，自动切回主线程
			}
		}, 
		new Response.ErrorListener() {
			@Override
			public void onErrorResponse(VolleyError error) {
		}
});
```

 3. 将request压入队列中即可
 
``` java
mQueue.add(stringRequest); 
```

### 请求json数据

 1. 得到RequestQueue对象，和上面一样
 2. 创建JsonObjectRequest 
 
``` java
JsonObjectRequest jsonObjectRequest = new JsonObjectRequest("http://m.weather.com.cn/data/101010100.html", null,  
        new Response.Listener<JSONObject>() {  
            @Override  
            public void onResponse(JSONObject response) {  
                Log.d("TAG", response.toString());  
            }  
        }, 
		new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                Log.e("TAG", error.getMessage(), error);  
            }  
        });  
```

 3. 将request压入队列中，和上面一样

  [1]: http://download.csdn.net/detail/sinyu890807/7152015