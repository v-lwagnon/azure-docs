---
title: Quickstart - Send a query to the API in Python - Bing Local Business Search
titleSuffix: Azure Cognitive Services
description: Use this quickstart to start using the Bing Local Business Search API in Python.
services: cognitive-services
author: aahill
ms.author: aahi
manager: nitinme
ms.date: 05/12/2020
ms.topic: quickstart
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.custom: devx-track-python, mode-api
---
# Quickstart: Send a query to the Bing Local Business Search API in Python

> [!WARNING]
> Bing Search APIs are moving from Cognitive Services to Bing Search Services. Starting **October 30, 2020**, any new instances of Bing Search need to be provisioned following the process documented [here](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Bing Search APIs provisioned using Cognitive Services will be supported for the next three years or until the end of your Enterprise Agreement, whichever happens first.
> For migration instructions, see [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Use this quickstart to learn how to send requests to the Bing Local Business Search API, which is an Azure Cognitive Service. Although this simple application is written in Python, the API is a RESTful Web service compatible with any programming language capable of making HTTP requests and parsing JSON.

This example application gets local response data from the API for a search query.

## Prerequisites

* An Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services/)
* [Python](https://www.python.org/) 2.x or 3.x.
* Once you have your Azure subscription, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="Create a Bing Search resource"  target="_blank">create a Bing Search resource </a> in the Azure portal to get your key and endpoint. After it deploys, click **Go to resource**.

## Run the complete application

The following example gets localized results, which are implemented in the following steps:
1. Declare variables to specify the endpoint by host and path.
2. Specify the query parameter. 
3. Define the search function that creates the request and adds the `Ocp-Apim-Subscription-Key` header.
4. Set the `Ocp-Apim-Subscription-Key` header. 
5. Make the connection and send the request.
6. Print the JSON results.

The complete code for this demo is as follows:

```python
import http.client, urllib.parse
import json

# Replace the subscriptionKey string value with your valid subscription key.

> [!WARNING]
> Bing Search APIs are moving from Cognitive Services to Bing Search Services. Starting **October 30, 2020**, any new instances of Bing Search need to be provisioned following the process documented [here](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Bing Search APIs provisioned using Cognitive Services will be supported for the next three years or until the end of your Enterprise Agreement, whichever happens first.
> For migration instructions, see [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
subscriptionKey = 'YOUR-SUBSCRIPTION-KEY'

host = 'api.cognitive.microsoft.com'
path = '/bing/v7.0/localbusinesses/search'

query = 'restaurant in Bellevue'

params = '?q=' + urllib.parse.quote (query) + '&mkt=en-us'

def get_local():
    headers = {'Ocp-Apim-Subscription-Key': subscriptionKey}
    conn = http.client.HTTPSConnection (host)
    conn.request ("GET", path + params, None, headers)
    response = conn.getresponse ()
    return response.read ()

result = get_local()
print (json.dumps(json.loads(result), indent=4))

```

## Next steps
- [Local Business Search Java quickstart](local-search-java-quickstart.md)
- [Local Business Search C# quickstart](local-quickstart.md)
- [Local Business Search Node.js quickstart](local-search-node-quickstart.md)
