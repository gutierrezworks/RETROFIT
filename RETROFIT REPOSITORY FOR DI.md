# RETROFIT REPOSITORY FOR DI

This allow us to TEST the data, to add different sources later as from where do I get StockData. I could get it from an API, firebase ... The viewModel doesn't care
so this allow us scalability.

## SET UP

1- Create repository

```bash
interface StocksRepository {
    suspend fun getStockData(symbol: String, timeframe: String, start: String): Stocks
}

class NetworkStocksRepository(): StocksRepository {
    override suspend fun getStockData(symbol: String, timeframe: String, start: String): Stocks {
        return StockApi.retrofitService.getStockData(symbol = symbol, timeframe = timeframe, start = start)
    }
}
```

2- Update ViewModel

```bash
fun getStockData(symbol: String, timeframe: String, start: String) {
        viewModelScope.launch {
            stockUiState = try {
                val repository = NetworkStocksRepository()
                val listResult = repository.getStockData(symbol = symbol, timeframe = timeframe, start = start)
                Log.d("listResult", listResult.toString())
                StockUiState.Success(listResult)
            } catch (e: Exception) {
                Log.e("listResult", e.stackTraceToString())
                StockUiState.Error
            }
        }
    }
```
