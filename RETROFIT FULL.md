# RETROFIT

Interface for retrofit

## Features
- ApiService -> Create retrofit builder

## Getting Started

### Add dependencies

```bash
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

MainActivity

```bash
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            StockTrackerTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    val viewModel: StockMainViewModel = StockMainViewModel()
                    Greeting(
                        name = "Android",
                        stockUiState = viewModel.stockUiState,
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier, stockUiState: StockUiState) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
    when (stockUiState) {
        is StockUiState.Loading -> "LoadingScreen(modifier = modifier.fillMaxSize())"
        is StockUiState.Success -> "ResultScreen(stockUiState.photos, modifier = modifier.fillMaxWidth())"
        is StockUiState.Error -> "ErrorScreen( modifier = modifier.fillMaxSize())"
    }
}
```

Data

```bash
@Serializable
data class Bar(
    @SerialName("o") val opening: String,
    @SerialName("c") val close: String,
    @SerialName("t") val time: String,
    @SerialName("v") val volume: String
)
```

