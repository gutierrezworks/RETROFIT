# RETROFIT

Interface for retrofit

## Features
- ApiService -> Create retrofit builder

## Getting Started

### Add dependencies

```bash
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    // Retrofit with Scalar Converter
    implementation("com.squareup.retrofit2:converter-scalars:2.9.0")
```

### Usage

ApiService:

```bash
private val BASE_URL = "https://www.alphavantage.co/"
private val API_KEY = "V27PW7M6ZUFOEGO3"
private val FUNCTION = "TIME_SERIES_DAILY"
private val retrofit = Retrofit.Builder()
    .addConverterFactory(ScalarsConverterFactory.create())
    .baseUrl(BASE_URL)
    .build()

interface StockApiService {

     @GET("query")
     suspend fun getStockData(
         @Query("apikey") apiKey: String = API_KEY,
         @Query("function") function: String = FUNCTION,
         @Query("symbol") symbol: String,
     ): String
}

object StockApi {
    // Only create it when need it with by lazy
    val retrofitService: StockApiService by lazy { retrofit.create(StockApiService::class.java) }
}
```

StockMainViewModel

```bash
class StockMainViewModel: ViewModel() {

    init {
        getStockData()
    }

    fun getStockData() {
        viewModelScope.launch {
            val listResult = StockApi.retrofitService.getStockData(symbol = "NVDA")
            Log.d("listResult", listResult)
        }
    }
}
```

