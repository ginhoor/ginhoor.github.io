---
layout: post
title: 'Google Firestore Bundle数据包功能调研'
subtitle: 'Cloud FireStore是Firebase提供的一种托管于云端的NoSQL数据库方案。'
categories: 技术
tags: Firestore Google
---

## 简介

Cloud FireStore是Firebase提供的一种托管于云端的NoSQL数据库方案。

数据结构为Collection（集合）和Document（文档），文档中存储键值对，整个数据库是由小型文档组成的大型集合。

*   支持子集合、复杂分层
*   支持单索引、复合索引
*   支持复合查询
*   支持事务读写
*   支持水平扩展、自动扩容

## Bundle

Bundle是以查询条件为维度，将一系列查询结果缓存在一起的数据体。比如对“overlay_categories”的全表查询，对“overlay_materials”的分页查询。

优势：

*   生成查询快照，避免多次查询Firestore造成的费用消耗。
*   结合CDN，在网络数据查询时直接返回查询快照，加快响应速度。
*   可以作为客户端本地的初始化数据，后续使用API查询时会增量更新数据缓存。
*   Bundle在更新缓存时也会通知addSnapshotListener的监听器，可与线上数据更新逻辑合并处理。

*   Bundle数据生成的缓存与API更新的缓存数据属于同一种缓存，统一管理。

劣势：

*   数据会以近似JSON的格式明文存储，不可存储敏感信息。
*   Bundle数据包的导入需要经历：数据读取，解析，入库的步骤，比较耗时。
*   性能随集合量级上升而下降，不推荐长期离线使用。
*   缓存没有索引，[缓存的查询不如API查询快](https://firebase.blog/posts/2019/08/why-is-my-cloud-firestore-query-slow)。

## 服务端

### SDK生成Bundle数据包

```python
from google.cloud import firestore
from google.cloud.firestore_bundle import FirestoreBundle

db = firestore.Client()
bundle = FirestoreBundle("latest-stories")

doc_snapshot = db.collection("stories").document("news-item").get()
query = db.collection("stories")._query()

# Build the bundle
# Note how `query` is named "latest-stories-query"
bundle_buffer: str = bundle.add_document(doc_snapshot).add_named_query(
    "latest-stories-query", query,
).build()

```

### Cloud Function生成Bundle数据包

#### index.js

运行时 : Node.js 12 入口点 : createBundle

需要添加运行时环境变量
GCLOUD_PROJECT = "ProjectID"
BUCKET_NAME = "xxxx"

```js
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp();

const bundleNamePrefix = process.env.BUNDLE_NAME_PREFIX
const bucketName = process.env.BUCKET_NAME

const collectionNames = [
  "doubleexposure_categories",
  "materials"
]

exports.createBundle = functions.https.onRequest(async (request, response) => {

  // Query the 50 latest stories
  // Build the bundle from the query results
  const store = admin.firestore();
  const bucket = admin.storage().bucket(`gs://${bucketName}`);

  for (idx in collectionNames) {
    const name = collectionNames[idx]
    const bundleName = `${bundleNamePrefix}_${name}_${Date.now()}`;
    const bundle = store.bundle(bundleName);
    const queryData = await store.collection(name).get();

    // functions.logger.log("queryData name:", name);
    // functions.logger.log("queryData:", queryData);
    bundle.add(name, queryData)
    const bundleBuffer = bundle.build();
    bucket.file(`test-firestore-bundle/${bundleName}`).save(bundleBuffer);
  }
  // Cache the response for up to 5 minutes;
  // see https://firebase.google.com/docs/hosting/manage-cache
  // response.set('Cache-Control', 'public, max-age=300, s-maxage=600');
  response.end("OK");
});
```

#### package.json

```json
{
  "name": "createBundle",
  "description": "Uppercaser Firebase Functions Quickstart sample for Firestore",
  "dependencies": {
    "firebase-admin": "^10.2.0",
    "firebase-functions": "^3.21.0"
  },
  "engines": {
    "node": "16"
  }
}
```

## 客户端

### 离线缓存配置

```swift
func setupFirestore() {
        let db = Firestore.firestore()
        let settings = db.settings
        /**
         对于 Android 和 Apple 平台，离线持久化默认处于启用状态。如需停用持久化，请将 PersistenceEnabled 选项设置为 false。
         */
        settings.isPersistenceEnabled = true
        /**
         启用持久化后，Cloud Firestore 会缓存从后端接收的每个文档以便离线访问。
         Cloud Firestore 会为缓存大小设置默认阈值。
         超出默认值后，Cloud Firestore 会定期尝试清理较旧的未使用文档。您可以配置不同的缓存大小阈值，也可以完全停用清理功能
         */
        settings.cacheSizeBytes = FirestoreCacheSizeUnlimited
        /**
         如果您在设备离线时获取了某个文档，Cloud Firestore 会从缓存中返回数据。
         查询集合时，如果没有缓存的文档，系统会返回空结果。提取特定文档时，系统会返回错误。
         */
        db.settings = settings
}
```

### 加载Bundle数据

```swift
// Loads a remote bundle from the provided url.
func fetchRemoteBundle(for firestore: Firestore, from url: URL, completion: @escaping ((Result<LoadBundleTaskProgress, Error>) -> Void)) {
        guard let inputStream = InputStream(url: url) else {
            let error = self.buildError("Unable to create stream from the given url: \(url)")
            completion(.failure(error))
            return
        }

        // The return value of this function is ignored, but can be used for more granular bundle load observation.
        let _ = firestore.loadBundle(inputStream) { (progress, error) in
            switch (progress, error) {
                case (.some(let value), .none):
                    if value.state == .success {
                        completion(.success(value))
                    } else {
                        let concreteError = self.buildError("Expected bundle load to be completed, but got \(value.state) instead")
                        completion(.failure(concreteError))
                    }
                case (.none, .some(let concreteError)):
                    completion(.failure(concreteError))
                case (.none, .none):
                    let concreteError = self.buildError("Operation failed, but returned no error.")
                    completion(.failure(concreteError))
                case (.some(let value), .some(let concreteError)):
                    let concreteError = self.buildError("Operation returned error \(concreteError) with nonnull progress: \(value)")
                    completion(.failure(concreteError))
            }
        }
}
```

### 使用提示

*   Bundle加载一次后就会合并入Firestore缓存，不需要重复导入。

*   Bundle在创建时，会以Query为维度增加Name标签，可以通过QueryName查询导入的Bundle数据集合。

```objective-c
- (void)getQueryNamed:(NSString *)name completion:(void (^)(FIRQuery *_Nullable query))completion
```

*   Bundle由异步加载完成，在加载完成之后才能查到数据，可以用监听器来查询。

```objective-c
- (id<FIRListenerRegistration>)addSnapshotListener:(FIRQuerySnapshotBlock)listener
    NS_SWIFT_NAME(addSnapshotListener(_:))
```

## 性能测试

### 小数据量样本

| 数据大小 | 文档数量 |
| -------- | -------- |
| 22 KB    | 33       |

| 次数 | iPhone 6s 加载耗时 | iPhone XR 加载耗时 |
| ------ | ------- | ------ |
| 第一次 | 1066 ms | 627 ms |
| 第二次 | 715 ms  | 417 ms |
| 第三次 | 663 ms  | 375 ms |
| 第四次 | 635 ms  | 418 ms |
| 第五次 | 683 ms  | 578 ms |

### 大数据量样本

| 数据大小 | 文档数量 |
| -------- | -------- |
| 2.7 M    | 359      |

| 次数 | iPhone 6s 加载耗时 | iPhone XR 加载耗时 |
| ------ | ------- | ------ |
| 第一次 | 4421 ms | 2002 ms |
| 第二次 | 698 ms | 417 ms |
| 第三次 | 663 ms  | 375 ms |
| 第四次 | 635 ms  | 418 ms |
| 第五次 | 683 ms  | 578 ms |

## 参考链接

[Firestore Data Bundles-A new implementation for cached Firestore documents](https://flaming.codes/posts/firestore-data-bundles-for-cached-firebase-cdn)

[Load Data Faster and Lower Your Costs with Firestore Data Bundles!](https://firebase.blog/posts/2021/04/firestore-supports-data-bundles)

[Why is my Cloud Firestore query slow?](https://firebase.blog/posts/2019/08/why-is-my-cloud-firestore-query-slow)