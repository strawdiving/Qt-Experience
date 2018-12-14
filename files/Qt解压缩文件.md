解压文件

解压（jsonObject  JSON object， jsonDocBytes Raw bytes of JSON document，  sizeKey key which holds byte size，bytesKey key which holds compressed bytes，decompressedBytes Returned decompressed bytes

- Validate decompressed size key, 即jsonObject是否包含sizeKey，读取size值即为decompressedSize解压缩大小
- 以“bytesKey":"分隔，选择分隔符后面的部分，再以”分隔，摘出value部分双引号里的内容
- 将 decompressed size作为首个4字节，这是qUncompress routine（例程）所必需的
- 将摘出的双引号里的内容转换为Utf8编码，返回其解码副本，进行qUncompress，解压出来的内容存入decompressedBytes
- 验证decompressedBytes的大小和decompressedSize是否匹配

