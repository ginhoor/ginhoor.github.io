# 渐进式迁移

### 创建MappingModel

  创建一个迁移文件(.xcmappingmodel)，指定需要迁移版本与目标版本。
  在Mapping中可以使用NSEntityMigrationPolicy实现自定义数据迁移。

### APP启动后，检测数据库是否需要迁移。



##### 1. 调用NSPersistentStoreCoordinator，获取当前Model（NSManagedObjectModel）的MetaData

```obj-c
+（NSDictionary <NSString *，id> *）metadataForPersistentStoreOfType：（NSString *）storeType URL：（NSURL *）url error：（NSError * _Nullable *）error;
```



##### 2. 调用当前NanagedObjectModel，比对MetaData是否兼容

```obj-c
- (BOOL)isConfiguration:(NSString *)configuration 	compatibleWithStoreMetadata:(NSDictionary<NSString *,id> *)metadata;
```



##### 3. 开始渐进式迁移（递归处理）

1. 获取当前NSManagedObjectModel的MetaData，比对当前NSManagedObjectModel，当相同的时候，跳出递归，即迁移完成。
2. 从项目文件中，获取对应当前Model文件（momd），以及Model的所有版本文件（mom），并从Model版本中找出对应当前Model的版本。
3. 从找到对应当前Model的迁移Mapping，以及迁移Mapping的目标版本。
4. 根据当前Model版本和迁移Mapping目标版本创建NSMigrationManager
```obj-c
NSMigrationManager *manager = [[NSMigrationManager alloc] initWithSourceModel:sourceModel destinationModel:destinationModel]
```

5. NSMigrationManager通过Mapping从当前Model版本迁移到目标Model版本

```obj-c
   BOOL didMigrate = [manager migrateStoreFromURL:sourceStoreURL
                                             type:type
                                          options:nil
                                 withMappingModel:mappingModel
                                 toDestinationURL:destinationStoreURL
                                  destinationType:type
                               destinationOptions:nil
                                            error:error];
```
6. 进行递归，回到步骤一，直到当前Model版本与迁移目标Model版本相同。


