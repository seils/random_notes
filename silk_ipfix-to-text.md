# Convert SiLK-encoded IPFIX records to text for loading into BigQuery

Setup SiLK VM instance based on:
https://tools.netsa.cert.org/confluence/pages/viewpage.action?pageId=23298051

Match all the sensor files by name:
```
declare -a FILENAME_PATTERNS=("ext2ext*" "allin*" "int2int*" "inweb*" "allout*"
"outweb*")
```

Create a file with a list of all the sensor files
```
for p in "${FILENAME_PATTERNS[@]}"; do SENSOR_FILES=`find . -type f -name "$p"`;
echo "$SENSOR_FILES"; done > files.txt
```

Convert from SiLK --> IPFIX
```
rwsilk2ipfix --print-statistics --ipfix-output=cisflows.ipfix --xargs=files.txt
```

Convert from IPFIX --> text
```
yafscii --in cisflows.ipfix --print-header --tabular
```

Compress (Optional, gets ~89% reduction)
```
zip cisflows.zip cisflows.yaf.txt
```
