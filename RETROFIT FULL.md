# RETROFIT

Interface for retrofit

## Features
- ApiService -> Create retrofit builder

## Getting Started

### Add dependencies

```bash
plugins {
    id("org.jetbrains.kotlin.plugin.serialization") version "1.9.10"
}

// Retrofit
implementation("com.squareup.retrofit2:retrofit:2.9.0")
// Kotlin serialization
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
// Retrofit with Kotlin serialization Converter
implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
implementation("com.squareup.okhttp3:okhttp:4.11.0")

// OkHttp3 Interceptor
implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
```

### Usage

ApiService:

```bash
private val BASE_URL = "https://data.alpaca.markets/v2/"
private val API_KEY = "PKUFKA91DA8Z0GAH0M9F"
private val API_KEY_SECRET = "jAXOzTcsRe4kgDp8cRuUL3ahOd0abcaYcHpiBXmb"

private val _client =
    OkHttpClient.Builder()
        .addInterceptor(HttpLoggingInterceptor()
            .apply { level = HttpLoggingInterceptor.Level.BODY })
        .build()

private val json = Json {
    ignoreUnknownKeys = true // Allows parsing even if there are unknown keys in the JSON response
    isLenient = true         // Makes the parser more lenient
}

private val retrofit = Retrofit.Builder()
    .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
    .client(_client)
    .baseUrl(BASE_URL)
    .build()

interface StockApiService {

     @GET("stocks/bars")
     suspend fun getStockData(
         @Header("APCA-API-KEY-ID") apiKey: String = API_KEY,
         @Header("APCA-API-SECRET-KEY") apiKeySecret: String = API_KEY_SECRET,
         @Query("symbols") symbol: String,
         @Query("timeframe") timeframe: String,
         @Query("start") start: String,
         @Query("sort") sort: String = "desc",
         ): Stocks
}

object StockApi {
    // Only create it when need it with by lazy
    val retrofitService: StockApiService = retrofit.create(StockApiService::class.java)
}
```

StockMainViewModel

```bash
sealed interface StockUiState {
    data class Success(val bars: Stocks) : StockUiState
    object Error : StockUiState
    object Loading : StockUiState
}

class StockMainViewModel: ViewModel() {

    // To make it state:
    // var stockUiState: StockUiState = StockUiState.Loading
    var stockUiState: StockUiState by mutableStateOf(StockUiState.Loading)

    init {
        getStockData("NVDA", "1D", "2025-06-25")
    }

    fun getStockData(symbol: String, timeframe: String, start: String) {
        viewModelScope.launch {
            stockUiState = try {
                val listResult = StockApi.retrofitService.getStockData(symbol = symbol, timeframe = timeframe, start = start)
                Log.d("listResult", listResult.toString())
                StockUiState.Success(listResult)
            } catch (e: Exception) {
                Log.e("listResult", e.stackTraceToString())
                StockUiState.Error
            }
        }
    }
}
```

StockScreen

```bash
@Composable
fun StockMain(
    stockUiState: StockUiState,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(0.dp)
    ) {
    when (stockUiState) {
        is StockUiState.Loading -> LoadingScreen()
        is StockUiState.Success -> ResultScreen(stockUiState.stocks)
        is StockUiState.Error -> ErrorScreen()
    }
}

@Composable
fun ResultScreen(
    stocks: Stocks,
    modifier: Modifier = Modifier
) {
    Text(
        text = stocks.bars.get("NVDA").toString()
    )
}

@Composable
fun LoadingScreen(
    modifier: Modifier = Modifier
) {
    Text(
        text = "Loading"
    )
}

@Composable
fun ErrorScreen(
    modifier: Modifier = Modifier
) {
    Text(
        text = "Error"
    )
}
```

Data

```bash
@Serializable
data class Stocks(
    @SerialName("bars") val bars: Map<String, List<Bar>>,
    @SerialName("next_page_token") val nextPageToken: String? = null
)

@Serializable
data class Bar(
    @SerialName("o") val opening: Double,
    @SerialName("c") val close: Double,
    @SerialName("t") val time: String,
    @SerialName("v") val volume: Long,
    @SerialName("h") val high: Double,
    @SerialName("l") val low: Double,
    @SerialName("n") val transactions: Long,
    @SerialName("vw") val volumeWeightedPrice: Double
)
```

