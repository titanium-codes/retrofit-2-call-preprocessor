Retrofit 2 generic response adapter
=========
[![GitHub license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/titanium-codes/LocGetter/blob/master/LICENSE)
[ ![Download](https://api.bintray.com/packages/titanium-codes/Android/preprocessor/images/download.svg) ](https://bintray.com/titanium-codes/Android/preprocessor/_latestVersion)

Overview
--------
Retrofit 2 call adapter used to wrap your call adapter.  
As soon as request finishes(or not) you can get access to response and add some logic just before transferring it further.

Install
-------
#### Gradle

**Step 1.** Add it in your root build.gradle at the end of repositories:
```
allprojects {
    repositories {
        jcenter()
    }
}
```
**Step 2.** Add the dependency
```
compile 'codes.titanium:preprocessor:1.1.2'
```

Basic Usage
-----------

Main features:
* Process every entity(or even range of entities) you want before main consumer
* Add generic error handler
* Ignore requests you don't want to preprocess
* Versatile and simple API
* Works with any call adapter(RxJava1, Rxjava2 etc)
* Easy setup

**Example:**  
Create some Preprocessors
```
Preprocessor<Observable<TestEntity>> preprocessor = new Preprocessor<Observable<TestEntity>>() {
      @Override
      public Observable<TestEntity> preprocess(Observable<TestEntity> source) {
        //preprocess observable here
        return source.doOnNext(testEntity -> {
          //save to database
          //some business logic
        });
      }
    };
```
Add all of them into list:
```
List<Preprocessor> preprocessors = new ArrayList();
preprocessors.add(firstEntityPreprocessor);
preprocessors.add(secondEntityPreprocessor);
```
Init retrofit client:
```
 Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("http://example.com")
        .client(client)
        .addCallAdapterFactory(PreprocessAdapter.create(/*Any your call adapter*/, preprocessors))
        .build();
```

What happens next?  
All entities with this type will be processed before main consumer.

Preprocess ignore
-----------
Add annotation to api call that you don't want to preprocess and thats all.  
**Example:**

```
public interface YourAwesomeApiService {
  @GET("/cookies")
  Observable<List<Cookies>> getCookies();

  //All preprocessors related to Cookies will be ignored here
  @PUT("/cookie")
  @PreprocessIgnore
  Observable<Cookie> editCookie(@Body EditCookiesProtocol protocol);
}
```

Wildcard preprocessors
---
You can use wildcards and multiple preprocessors with same signature to preprocess wide spectrum of types  
**Examples:**  
Preprocessor for every request that makes one retry in case of error:
```
Preprocessor<Observable<?>> retryPreprocessor = new Preprocessor<Observable<?>>() {
      @Override
      public Observable<?> preprocess(Observable<?> source) {
        return source.retry(1);
      }
    };
```
 Preprocessor that saves all cookies and its subclass inside cache:
```
Cache cache = dependenciesProvider.getCache();
Preprocessor<Observable< extends Cookie>> cookiePreprocessor = new Preprocessor<Observable<? extends Cookie>>() {
      @Override
      public Observable<? extends Cookie> preprocess(Observable<? extends Cookie> source) {
        return source.doOnNext(cookie -> cache.SaveCookie(cookie));
      }
    };
```

Proguard
---
If you are using proguard add this into your proguard-rules.pro
```
# Premapper
-keepclasseswithmembernames class * implements codes.titanium.premapper.Preprocessor {
    public ** preprocess(**);
}
```

Todo
------------
* Preprocessor priority

Release notes
-------------
### 1.0.0
> * Initial release
### 1.1.0
> * Removed unnecessary abstract method from Preprocessor
> * Can wrap any call adapter
> * Added optional ignore for premap
> * Added support of multiple preprocessors
> * Added support of wildcard preprocessors
### 1.1.1
> * Preprocessor lazy initialization
### 1.1.2
> * Fixed bug causing undefined behavior with kotlin preprocessors