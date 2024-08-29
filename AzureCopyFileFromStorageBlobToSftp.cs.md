Copy directly a file to specific directory on (S)FTP.

Libraries:
- [FluentFTP - tested with 2024.1.0](https://github.com/robinrodricks/FluentFTP)
- [SSH.NET - tested with 50.1.0](https://github.com/sshnet/SSH.NET)

```csharp
// usage:
var azureBlobClient = await BlobUtility.GetBlobClientAsync(storageConnectionString, blobPath, blobName);
var customStream = new CustomStream(azureBlobClient, (await azureBlobClient.GetPropertiesAsync()).Value.ContentLength, defaultBufferSize);

// class:
class CustomStream : Stream
{
    private bool _isEndOfStreamReached;
    private bool _ifFirstRunThenAdjustLocalBuffer;
    private BlobClient _blobClient;
    private long _totalFileSize;
    private long _localOffset;
    private long _localBufferSize;

    public AzureBlobFileHandlingStream(BlobClient blobClient, long fileSize, long howManyByteToLoadIntoMemory)
    {
        _ifFirstRunThenAdjustLocalBuffer = true;
        _blobClient = blobClient;
        _totalFileSize = fileSize;
        _localOffset = 0;
        _localBufferSize = howManyByteToLoadIntoMemory;
    }

    public override bool CanSeek => false;

    public override long Length => _totalFileSize;

    public override int Read(byte[] buffer, int offset, int bufferSize)
    {
        if (_isEndOfStreamReached)
            return 0;
        if (_ifFirstRunThenAdjustLocalBuffer)
        {
            if (bufferSize < _localBufferSize)
                _localBufferSize = bufferSize;
            _ifFirstRunThenAdjustLocalBuffer = false;
        }
        byte[] fileByteContent = null;
        var blobContent = _blobClient.Download(new HttpRange(_localOffset, _localBufferSize));
        _localOffset += bufferSize < _localBufferSize ? bufferSize : _localBufferSize;
        if (_localOffset > _totalFileSize)
            _isEndOfStreamReached = true;
        using (var memoryStream = new MemoryStream())
        {
            blobContent.Value.Content.CopyTo(memoryStream);
            fileByteContent = memoryStream.ToArray();
        }
        Array.Copy(fileByteContent, buffer, fileByteContent.Length);
        return fileByteContent.Length;
    }

    public override async Task<int> ReadAsync(byte[] buffer, int offset, int bufferSize, CancellationToken token)
    {
        var task = Task.Run(() => Read(buffer, offset, bufferSize));
        var valueToRet = await task;
        if (token.IsCancellationRequested)
            return 0;
        return valueToRet;
    }

    public override bool CanRead => throw new NotImplementedException();

    public override bool CanWrite => throw new NotImplementedException();

    public override long Position { get => throw new NotImplementedException(); set => throw new NotImplementedException(); }

    public override void Flush() => throw new NotImplementedException();

    public override long Seek(long offset, SeekOrigin origin) => throw new NotImplementedException();

    public override void SetLength(long value) => throw new NotImplementedException();

    public override void Write(byte[] buffer, int offset, int count) => throw new NotImplementedException();
}
```
