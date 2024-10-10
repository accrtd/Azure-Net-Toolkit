Get value from Azure Storage Table in a safe way

```csharp
using System.Text;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;

namespace AzureHelpers;

public class AzureTable
{
    public async Task<T> GetCountSafelyAsync(string azureStorageConnectionString, string tablePartitionKey, string containerName)
    {
        var client = new BlobContainerClient(azureStorageConnectionString, "lck");
        var blob = client.GetBlobClient($"/{containerName}.lck");
        // Checking lease container
        if (!(await client.ExistsAsync()))
        {
            // Creating {containerName} lck container
            await client.CreateIfNotExistsAsync();
            var blobHttpHeader = new BlobHttpHeaders
            {
                ContentType = "application/octet-stream"
            };
            using var stream = new MemoryStream(Encoding.UTF8.GetBytes("sthsththtstht"));
            await blob.UploadAsync(stream,
                    new BlobUploadOptions
                    {
                        HttpHeaders = blobHttpHeader
                    });
        }
        // Acquire lease
		{
			// Retry acquire lease
			var blc = blob.GetBlobLeaseClient();
			Boolean leaseSet = false;
			for (int i = 0; i < 3; i++)
			{
				try
				{
					await blc.AcquireAsync(TimeSpan.FromSeconds(15));
					leaseSet = true;
				}
				catch (Exception ex)
				{
					_logger.LogError(ex.Message);
				}
				if (leaseSet)
					break;
				System.Threading.Thread.Sleep(1000);
			}
			if (!leaseSet)
				throw new Exception("Azure table lease is set!");
		}

        try
        {
            // Get value from Azure Table;
			// .....
        }
        catch (Exception e)
        {
            throw new Exception(e.Message);
        }
        finally
        {
            // Release lease
            await blc.ReleaseAsync();
        }
        return .....;
    }
}
```
