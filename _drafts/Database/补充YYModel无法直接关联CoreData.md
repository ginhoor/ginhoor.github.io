# 补充YYModel无法直接关联CoreData

我封装了一个简单的BaseModel类供大家参考
[DEMO下载地址](https://github.com/ginhoor/CEBaseYYModel)
*****
JSON转Model是一个很长常见，又非常繁琐的技术方案，好在现在有很多第三方库支援，让我们不用重复造轮子。博主一直都喜欢用Mantle框架，无奈Mantle框架的model定义过于繁琐，尤其是类型转换。于是就转投了YYModel的怀抱，在此感谢下YYModel的作者（[YYModel](https://github.com/ibireme/YYModel)）。

YYModel有一个一直未实现的，就是Model和CoreData的关联转换。于是借鉴Mantle的思路，我自己造了个轮子，虽然不是快，胜在能用，也还有不少不足，希望大家能帮我补充提点下

思路是这样的，Model实现YYModel协议后已经有了JSON转换功能，MagicalRecording实现了从JSON转NSManagedObject，现在再增加上NSManagedObject转Model的功能，就齐活了。

为了实现低耦合，选择协议的方式来实现，首先定义一个协议
```objective-c
@protocol CEYYModelCoreData <NSObject>

// 关联NSManagedObjectEntity的名字
+ (NSString *)managedObjectEntityName;

// 指明 NSManagedObjectEntity 中表示关系的字段
+ (NSDictionary <NSString *, id> *)managedRelationshipPropertyMapper;

// 指明 NSManagedObjectEntity 中集合字段的类型
+ (NSDictionary <NSString *, id> *)managedRelationshipContainerPropertyMapper;

@end
```
实现参考
```objective-c
+ (NSString *)managedObjectEntityName
{
    return [NSString stringWithFormat:@"%@Managed",NSStringFromClass([self class])];
}

+ (NSDictionary <NSString *, id> *)managedRelationshipPropertyMapper
{
    return @{@"auditStatus":[CEAuditStatusEnum class],
	@"loanApplicationFileList":[NSArray class],
	@"vehicleList":[NSArray class]};
}

+ (NSDictionary <NSString *, id> *)managedRelationshipContainerPropertyMapper
{
    return @{@"loanApplicationFileList":[CELoanFile class],
             @"vehicleList":[CEVehicle class]};
}
```
这个协议的作用是描述Model和NSManagedObjectEntity之间的转化关系。
接下去就是如何转换的问题了，这里我碰到了一个问题：NSManagedObjectEntity存储如SQLite的时候，CoreData会为这条数据添加一个自增的主键id，但是这个id无法在NSManagedObjectEntity结构体中获得，所以我添加了一个NSString类型的UUID来作为这条数据的唯一标识符。

下面是进行转换
```objective-c
- (void)setValuesByManagedObj:(NSManagedObject *)obj
{
    // 获得Entity中的字段名
    NSArray *attributesKeys = [[[obj entity] attributesByName] allKeys];
    // 根据字段名获得对应值
    NSDictionary *attributeDic = [obj dictionaryWithValuesForKeys:attributesKeys];
    // 将值设置给Model
    [self yy_modelSetWithJSON:attributeDic];
    
    // 获得Entity的关联关系
    NSArray<NSString *> *relationshipsKeys = [[[obj entity] relationshipsByName] allKeys];
    // 判断关联关系是否存在
    if (relationshipsKeys.count > 0) {
        // 获取关联关系的对应转换
        NSDictionary *relationshipDic = [(id<CEYYModelCoreData>)[self class] managedRelationshipPropertyMapper];
        
        [relationshipsKeys enumerateObjectsUsingBlock:^(NSString * _Nonnull key, NSUInteger idx, BOOL * _Nonnull stop) {
            if ([relationshipDic.allKeys containsObject:key]) {
                Class relationshipClass = relationshipDic[key];
                
                //判断是否为容器类型
                if ([relationshipClass isSubclassOfClass:[NSArray class]] ||
                    [relationshipClass isSubclassOfClass:[NSSet class]] ||
                    [relationshipClass isSubclassOfClass:[NSOrderedSet class]]
                    ) {
                    // 获取关联关系中容器的对应转换
                    NSDictionary *relationshipContainerDic = [(id<CEYYModelCoreData>)[self class] managedRelationshipContainerPropertyMapper];
                    
                    if ([relationshipContainerDic.allKeys containsObject:key]) {
                        
                        Class relationshipContainerClass = relationshipContainerDic[key];
                        // 获得Entity中的容器内容
                        NSArray <NSManagedObject *> *managedObjList = [obj valueForKey:key];
                        // 将关系中的元素转换成对应类型
                        id result = [managedObjList bk_map:^id(NSManagedObject *managedObj) {
                            CEBaseYYModel *containerObj = [[relationshipContainerClass alloc] init];
                            [containerObj setValuesByManagedObj:managedObj];
                            
                            return containerObj;
                        }];

                        NSString *newKey = key;
                        //默认转换key为对应关系的Key，如果使用YYModel定义了kKey转换关系，则会被替换成YYModel中的Key
                        if ([self respondsToSelector:@selector(modelCustomPropertyMapper)]) {
                            NSDictionary *modelCustomPropertyMapper = [(id<YYModel>)[self class] modelCustomPropertyMapper];
                            if ([modelCustomPropertyMapper.allValues containsObject:key]) {
                                newKey = modelCustomPropertyMapper.allKeys[[modelCustomPropertyMapper.allValues indexOfObject:key]];
                            }
                        }
                        // 将转换好的元素设置到Model中
                        [self setValue:result forKey:newKey];
                    }
                } else {
                    // 从Entity中获取对应值设置给Model
                    NSManagedObject *managedObj = [obj valueForKey:key];
                    CEBaseYYModel *containerObj = [[relationshipClass alloc] init];
                    [containerObj setValuesByManagedObj:managedObj];
                    [self setValue:containerObj forKey:key];
                }
            }
        }];
    }
}
```
其中重要思路是先获得NSManagedObject中的字段与关联关系，先转换字段，然后根据Model实现CEYYModelCoreData协议中的关联关系定义来转换容器类型字段