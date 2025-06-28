# OKHTTP Interceptor

### Add dependencies

```bash
    // OkHttp3 Interceptor
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
```

### Usage

Adding an interceptor

```bash
private val _client =
    OkHttpClient.Builder()
        .addInterceptor(HttpLoggingInterceptor()
            .apply { level = HttpLoggingInterceptor.Level.BODY })
        .build()

private val retrofit = Retrofit.Builder()
    .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
    .client(_client)
    .baseUrl(BASE_URL)
    .build()
```