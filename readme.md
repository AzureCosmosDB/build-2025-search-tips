
# Microsoft Build 2025, DEM567: Tips for fast vector and full-text search with Azure Cosmos DB

This repo contains some practical tips and best practices for optimizing and tuning vector, full-text, and hybrid search in Azure Cosmos DB. Learn how to achieve efficient data ingestion, balance speed with accuracy in search queries, and fine-tune indexing for your specific workload needs. The repo includes detailed examples in C#, complete with direct links to code snippets, as well as an interactive Jupyter notebook containing ready-to-use examples and sample data to help you get started quickly.

## Inserting documents with vectors

### General  Practices
- **Provision enough throughput (RU/s).** This will help ensure you have enough capacity to ingest and index vectors quickly and won't get throttled. Azure Cosmos DB will automatically create new physical partitions as you scale in incremets of 10,000 RU/s. Having more physical partitions can help ingest rate. 
- **Enable bulk execution.** Setting `AllowBulkExecution=true` in the Azure Cosmos DB .NET or Java SDK will leverage bulk execution for more efficient operations.
- **Increase retry attempts and retry wait time**. This will help in case your ingestion client is getting throttled.
- **Use the Azure Cosmos DB Spark connector.** For very large workloads, consider using the spark connector for scale-out ingestion. 

### Example Code
```csharp
CosmosClientOptions cosmosClientOptions = new()
{
    ConnectionMode = ConnectionMode.Direct,
    AllowBulkExecution = bulkExecution,
    // SDK will handle throttles and also wait for the amount of time the service tells it to wait and retry after the time has elapsed.
    // See: https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/how-to-migrate-from-bulk-executor-library
    MaxRetryAttemptsOnRateLimitedRequests = 100,
    MaxRetryWaitTimeOnRateLimitedRequests = TimeSpan.FromSeconds(600)
};
```

🔗 [Source Code Reference](https://github.com/AzureCosmosDB/VectorIndexScenarioSuite/blob/340052db7028cd227d901f27d13b041043d18032/src/Scenario.cs#L168C1-L169C1)

---

## Vector search queries

### Best Practices
- **Co-locate your client application and the Azure Cosmos DB account**  for optimal performance.
- **Configure concurrency**. Use `MaxConcurrency` in `QueryRequestOptions` to control parallelism. Setting this to -1 will optimize the number of concurrent operations for your available compute. 
- **Tune searchlistsize**. An integer specifying the size of the search list when conducting a vector search on the DiskANN index. Increasing this may improve accuracy, but also increase latency. Lowering this 
- 

### Example Code
```csharp
var requestOptions = new QueryRequestOptions 
{ 
    MaxConcurrency = maxConcurrancy 
};

FeedIterator<IdWithSimilarityScore> queryResultSetIterator = 
    this.CosmosContainerForQuery.GetItemQueryIterator<IdWithSimilarityScore>(
        queryDefinition,
        requestOptions);
```

🔗 [Source Code Reference](https://github.com/AzureCosmosDB/VectorIndexScenarioSuite/blob/340052db7028cd227d901f27d13b041043d18032/src/BigANNBinaryEmbeddingScenarioBase.cs#L225C1-L226C1)

## Next Steps

- [Learn about vector search in Azure Cosmos DB](https://aka.ms/CosmosVectorSearch)
- [Azure Cosmos DB AI Samples](https://aka.ms/AzureCosmosDB/Gallery/AI)
- [Try the VectorIndexScenarioSuite code featured in DEM567](https://github.com/AzureCosmosDB/VectorIndexScenarioSuite)
- [Azure Cosmos DB search announcements at Microsoft Build 2025](https://aka.ms/Build25/CosmosDBFTS)
- [Read our recent whitepaper on DiskANN & Azure Cosmos DB](https://aka.ms/CosmosDB/VectorSearch)