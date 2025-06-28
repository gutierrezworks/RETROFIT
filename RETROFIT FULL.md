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
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.5.1")
    // Retrofit with Kotlin serialization Converter
    implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
    implementation("com.squareup.okhttp3:okhttp:4.11.0")
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
    val retrofitService: StockApiService = retrofit.create(StockApiService::class.java)
}
```

StockMainViewModel

```bash
//Create a mutable state StockUiState that will be observed by the UI and
// change on call requests
sealed interface StockUiState {
    data class Success(val photos: String) : StockUiState
    object Error : StockUiState
    object Loading : StockUiState
}

class StockMainViewModel: ViewModel() {

    // To make it state:
    // var stockUiState: StockUiState = StockUiState.Loading
    var stockUiState: StockUiState by mutableStateOf(StockUiState.Loading)

    init {
        getStockData()
    }

    fun getStockData() {
        viewModelScope.launch {
            stockUiState = try {
                val listResult = StockApi.retrofitService.getStockData(symbol = "NVDA")
                Log.d("listResult", listResult)
                StockUiState.Success(listResult)
            } catch (e: Exception) {
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

DATA

```bash
@Serializable
data class Bar(
    @SerialName("o") val opening: String,
    @SerialName("c") val close: String,
    @SerialName("t") val time: String,
    @SerialName("v") val volume: String
)
```

