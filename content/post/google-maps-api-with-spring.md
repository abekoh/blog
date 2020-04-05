---
title: "Google Maps APIをSpringで使ってみる"
date: 2020-04-05T23:43:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "Java", "Kotlin", "Spring", "Google Maps" ]
---

Google Mapsの場所情報が使えるPlaces APIを使ったアプリケーションを作ってみたく、実装ためしたメモ。

## Google Maps API
単にGoogle Maps APIといってもその用途によって、ドキュメントや料金体系が別れている。

ドキュメント一覧はここで、目的に合ったAPIを選ぶ。
https://developers.google.com/maps/documentation

料金についてはここ。記事を書いている時点で、モバイル向けマップ表示は無料だったり、ある道路の制限速度を取得するAPIは1000件あたり$20と細かい。1ヶ月$200までは無料で使える。  
https://cloud.google.com/maps-platform/pricing/

今回は[Places API for Web](https://developers.google.com/places/web-service/intro)を使ってみる。指定した場所周辺の飲食店一覧や、その飲食店のレビュー情報なんかが取得できる。

Places APIの中でも、場所を探す、場所の詳細情報を取得する、その周辺の写真を取得する、で料金体系が違うので注意。

## APIの有効化

ここに書いてあるとおり順にやってく。  
https://developers.google.com/places/web-service/get-api-key

手順通り進めたら、API Keyが発行できる。このキーをクエリパラメータに含めるなどして扱う。

発行できたらcurlつかってテスト。
```bash
curl "https://maps.googleapis.com/maps/api/place/nearbysearch/json?key={replace your API Key}&location=35.6812362,139.7649361&radius=100&language=ja&keyword=ramen"
```
上記は[Nearby Search](https://developers.google.com/places/web-service/search#PlaceSearchRequests)を使って、東京駅周辺半径100mで、キーワード"ramen"でお店を探すサンプル。

ドキュメントに書かれているとおり、API Keyには利用元の制限をかけるべき。IPアドレスやリファラなどで設定可能。

## 実装

Java向け公式のクライアントがあったのでそれ利用。

https://github.com/googlemaps/google-maps-services-java

依存を追加。
```kotlin
// build.gradle.kts
dependencies {
    implementation("com.google.maps:google-maps-services:0.11.0")
}
```

使い方は次のような感じ。
```java
// https://github.com/googlemaps/google-maps-services-java/blob/master/README.md より
GeoApiContext context = new GeoApiContext.Builder()
    .apiKey("AIza...")
    .build();
GeocodingResult[] results =  GeocodingApi.geocode(context,
    "1600 Amphitheatre Parkway Mountain View, CA 94043").await();
Gson gson = new GsonBuilder().setPrettyPrinting().create();
System.out.println(gson.toJson(results[0].addressComponents));
```

`GeoApiContext`にAPI Keyを詰めてインスタンス化、それを`GeocodingApi.geocode()`といったstatic methodの引数に詰めて実行できる。

Springで実装するとき、このcontextをどう扱えばよいか。やっぱりDIしてしまうのが後々別のクラスでもAPI呼び出したいってとき便利なので、Bean登録する。
```kotlin
@Configuration
class GeoApiConfig {
    @Bean
    fun createGeoApiConfig(@Value("\${google.api.api-key}") apiKey: String): GeoApiContext {
        return GeoApiContext.Builder()
                .apiKey(apiKey)
                .build()
    }
}
```

application.ymlやコマンドライン引数などで`google.api.api-key`を設定しておくこと。

そして実際に使うところはこのような感じ。Kotlinだと`@Autowired`を省略できたりする。
```kotlin
@Service
class PlaceApiService(private val context: GeoApiContext) {

    companion object {
        const val langCode = "ja"
    }

    override fun getLocationFromKeyword(keyword: String): PlacesSearchResponse {
        val request = PlacesApi
                .textSearchQuery(context, keyword)
                .language(langCode)
        try {
            return request.await()
        } catch (e: Exception) {
            throw RuntimeException("Some error occured.")
        }
    }
}
```
勿論エラーハンドリングはもう少し考えたほうなおよし。

## まとめ

Spring向けってわけではないJavaライブラリを扱うとき、うまくBean登録してあげるのが肝かなーと思った次第です。