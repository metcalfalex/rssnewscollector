# rssnewscollector
Serverless AWS pipeline to capture news headlines from rss

1. Lambda: Grab rss xml file
2. Lambda: Parse out fields of interest into csv
3. Lambda: Save csv to S3
4. Cloudwatch: Lambda execution scheduling
5. Athena: Query data

```python
import urllib.request
import xml.etree.ElementTree as ET
import boto3
import csv
from datetime import datetime

def lambda_handler(event, context):
    with open("/tmp/justin.csv","w") as f:
        datetimenow = datetime.now()
        writer = csv.writer(f, delimiter='|', quotechar='`')
        urllib.request.urlretrieve("http://www.abc.net.au/news/feed/51120/rss.xml", "/tmp/justin.xml")
        tree = ET.parse("/tmp/justin.xml")
        root = tree.getroot()
        for item in root[0].findall('item'):
            title = item.find('title').text
            link = item.find('link').text
            description = item.find('description').text.strip()
            pubDate = item.find('pubDate').text
            row = 'JustIn', title, link, description, pubDate, datetimenow.strftime("%Y-%m-%d %H:%M:%S")
            writer.writerow(row)
    s3 = boto3.resource('s3')
    outfile = 'abc/justin_' + datetimenow.strftime("%Y%m%d%H%M%S") + '.csv'
    s3.meta.client.upload_file('/tmp/justin.csv', '20170523datalake', outfile)
    return None
```


