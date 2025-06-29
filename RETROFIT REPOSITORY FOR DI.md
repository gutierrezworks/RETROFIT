# RETROFIT REPOSITORY W/ MANUAL DI

We want to share the Repository along the Application level.
This allow us to TEST the data, to add different sources later as from where do I get StockData. I could get it from an API, firebase ... The viewModel doesn't care so this allow us scalability.

## SET UP

Create the AppContainer

```bash
interface AppContainer {
    val stocksRepository: StocksRepository
}

class DefaultAppContainer(): AppContainer {

    private val baseUrl = "https://data.alpaca.markets/v2/"

    private val _client =
        OkHttpClient.Builder()
            .addInterceptor(
                HttpLoggingInterceptor()
                .apply { level = HttpLoggingInterceptor.Level.BODY })
            .build()

    private val json = Json {
        ignoreUnknownKeys = true // Allows parsing even if there are unknown keys in the JSON response
        isLenient = true         // Makes the parser more lenient
    }

    private val retrofit = Retrofit.Builder()
        .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
        .client(_client)
        .baseUrl(baseUrl)
        .build()

    private val retrofitService: StockApiService by lazy {
        retrofit.create(StockApiService::class.java)
    }

    override val stocksRepository: StocksRepository by lazy {
        NetworkStocksRepository(retrofitService)
    }
}
```

Define the new Application

```bash
class StockTrackerApplication: Application() {
    lateinit var container: AppContainer
    override fun onCreate() {
        super.onCreate()
        container = DefaultAppContainer()
    }
}
```

Declare it on the manifest

```bash
    <application
        android:name=".StockTrackerApplication"
```

Create the service

```bash
private val API_KEY = "PKUFKA91DA8Z0GAH0M9F"
private val API_KEY_SECRET = "jAXOzTcsRe4kgDp8cRuUL3ahOd0abcaYcHpiBXmb"

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
```

Create repository

```bash
interface StocksRepository {
    suspend fun getStockData(symbol: String, timeframe: String, start: String): Stocks
}

class NetworkStocksRepository(
    private val retrofitService: StockApiService
): StocksRepository {
    override suspend fun getStockData(symbol: String, timeframe: String, start: String): Stocks {
        return retrofitService.getStockData(symbol = symbol, timeframe = timeframe, start = start)
    }
}
```

Update ViewModel

```bash
sealed interface StockUiState {
    data class Success(val stocks: Stocks) : StockUiState
    object Error : StockUiState
    object Loading : StockUiState
}

class StockMainViewModel(
    private val stocksRepository: StocksRepository
): ViewModel() {

    // To make it state:
    // var stockUiState: StockUiState = StockUiState.Loading
    var stockUiState: StockUiState by mutableStateOf(StockUiState.Loading)

    init {
        getStockData("NVDA", "1D", "2025-06-25")
    }

    fun getStockData(symbol: String, timeframe: String, start: String) {
        viewModelScope.launch {
            stockUiState = try {
                val listResult = stocksRepository.getStockData(symbol = symbol, timeframe = timeframe, start = start)
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

Because the Android framework does not allow a ViewModel to be passed values in the constructor when created, we implement a ViewModelProvider.Factory object

Add a Factory to the StockMainViewModel

```bash
companion object {
        val Factory: ViewModelProvider.Factory = viewModelFactory {
            initializer {
                val application = (this[APPLICATION_KEY] as StockTrackerApplication)
                val stocksRepository = application.container.stocksRepository
                StockMainViewModel(stocksRepository = stocksRepository)
            }
        }
    }
```

Update the UI

```bash
Surface(
            modifier = Modifier.fillMaxSize().padding(it)
        ) {
            val viewModel: StockMainViewModel = viewModel(factory = StockMainViewModel.Factory)
            StockMain(
                stockUiState = viewModel.stockUiState,
                contentPadding = it,
            )
        }

```

| ❓ Is Hilt doing the same as the codelab’s Application pattern? | Yes, but automatically and more cleanly. |

It replaces AppContainer with a Dagger/Hilt @Module

It replaces MyViewModel.Factory with Hilt’s @HiltViewModel

It manages lifecycles, scopes, and graph creation for you


