---
draft: false
title: "Full-text search on Firebase with MeiliSearch for Semantic Search."
snippet: "Semantic search is an advanced data searching technique that enhances the accuracy and relevance of search results by understanding the contextual meaning and intent behind user queries."
image: {
    src: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*TgGa1-ABTK7leq0dwVFn_A.png",
    alt: "firebase search"
}
publishDate: "Sep 6, 2021"
category: "Semantic Search"
author: "Steven Ogwal"
tags: [firebase, gcp, query, devops]
---

## What is Semantic Search?
Semantic search is an advanced data searching technique that enhances the accuracy and relevance of search results by understanding the contextual meaning and intent behind user queries.

## The challenge
Full-text search has always been a challenge with firebase because Cloud Firestore doesn’t support native indexing or search for text fields in documents. Additionally, downloading an entire collection to search for fields client-side isn’t practical. Therefore, to enable full text search of your Cloud Firestore data, we have to use a dedicated third-party search service.

- Elastic (High cost)
- Algolia (High cost)
- Meilisearch (Affordable) — Self Hosted

## What is MeiliSearch
MeiliSearch is a RESTful search API. It aims to be a ready-to-go solution for everyone who wants a fast and relevant search experience for their end-users.

All of MeiliSearch’s features are provided right out of the box, and can be easily configured.

### The solution using MeiliSearch & Firebase Cloud Functions
Run MeiliSearch
There are many easy ways to download and run a MeiliSearch instance.

For example, if you use Docker:

``` docker
docker pull getmeili/meilisearch:latest
docker run -it --rm -p 7700:7700 getmeili/meilisearch:latest ./meilisearch --master-key=masterKey
```

NB: you can also download MeiliSearch from Homebrew or APT.

Adding records with Cloud Functions
With npm:

```
npm install meilisearch
```

Cloud functionsindex.js:

``` js
const functions = require("firebase-functions");
const { MeiliSearch } = require('meilisearch');
const admin = require('firebase-admin');
admin.initializeApp();
const client = new MeiliSearch({
  host: '<http://127.0.0.1:7700>',
  apiKey: 'masterKey',
});
// Add the search index every time a secret post is written.
exports.onCreateSecret = functions.firestore.document('secrets/{id}')
.onCreate((snap, context) => {
	// Create a document
	const documents = [
            {
                id: snapshot.id,
                secret: snapshot.data().secret,
                authorName: snapshot.data().authorName,
                authorUid: snapshot.data().authorUid,
                timestamp: new       Date(timestamp.seconds*1000).toLocaleDateString()
            }
       ];
		
	// Write to the meilisearch index
      let response = await client.index('secrets').addDocuments(documents);
      console.log(response); // => { "updateId": 0 }
});
```

### Installation in Flutter
You can install the meilisearch package by adding a few lines into pubspec.yaml file.

[meilisearch](https://pub.dev/packages/meilisearch) | Dart Package


Searching for records
``` dart
// a function to search with meilisearch
Future<void> searchSecrets(String _query) async {
  try {
    var client = MeiliSearchClient('<http://127.0.0.1:7700>', 'masterKey');
    var index = client.index('secrets');
    var results = await index.search(_query);
    if (results.hits!.length == 0) {
      print('No results found');
    }else{
	print(results.hits);
    }
  } catch (e) {
    print('Error: $e');
  }
}
```
## Deploying
Since meilisearch is self-hosted, This gives you the freedom of control. Meilisearch can be deployed anywhere of your choice (docker-containers).

Here is a guide on how to deploy.

https://www.meilisearch.com/docs/guides/deployment/digitalocean#part-1-deploy-meilisearch-on-a-droplet

For my project, i used Qovery which offered a free plan at $0.

[Qovery](https://www.qovery.com/) is a platform that combines the power of Kubernetes, the reliability of AWS, and the simplicity of Heroku to deploy your applications in the Cloud.

## Conclusion
In conclusion, this approach not only resolves the challenges posed by traditional Firebase searches but also empowers developers to maintain control over their data infrastructure.

As demonstrated, semantic search stands out as an affordable and effective solution for implementing full-text search among others, thereby enriching the user experience in applications. This case study serves as a valuable reference for developers seeking to implement similar functionalities in their projects, highlighting the potential of combining innovative technologies to meet user needs effectively.

