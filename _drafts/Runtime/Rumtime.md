```obj-c
+ (void)createClass
{
    // 创建一个继承NSObject的类，名字为mClass，类末尾字节分配为0；
    Class mClass = objc_allocateClassPair([NSObject class],"mClass",0);
    
    // 为mClass添加一个NSString类型的变量personName。
    // 最后一位参数“@”表示，这个变量指向的是一个对象。
    /*   c A char
     i An int
     s A short
     l A long，l is treated as a 32-bit quantity on 64-bit programs.
     q A long long
     C An unsigned char
     I An unsigned int
     S An unsigned short
     L An unsigned long
     Q An unsigned long long
     f A float
     d A double
     B A C++ bool or a C99 _Bool
     v A void
     * A character string (char *)
     @ An object (whether statically typed or typed id)
     # A class object (Class)
     : A method selector (SEL)
     [array type] An array
     {name=type...} A structure
     (name=type...) A union
     bnum A bit field of num bits
     ^type A pointer to type
     ? An unknown type (among other things, this code is used for function pointers)
     */
    class_addIvar(mClass, "personName", sizeof(NSString *), 0, "@");
    // 添加一个方法，“v@:@”表示“v”：void返回值，“@”：self，“:”selector，“@”：对象类型参数
    class_addMethod(mClass, @selector(changePersonName:), (IMP)changePersonName, "v@:@");
    
    objc_registerClassPair(mClass);
    
    // 初始化对象
    id mPerson = [[mClass alloc] init];
    NSLog(@"mPerson:%@",mPerson);
    NSLog(@"1.personName：%@",[mPerson valueForKey:@"personName"]);
    
    [mPerson setValue:@"Tom" forKey:@"personName"];
    NSLog(@"2.personName：%@",[mPerson valueForKey:@"personName"]);
    
    [mPerson changePersonName:@"Roy"];
    NSLog(@"3.personName：%@",[mPerson valueForKey:@"personName"]);
    
    NSLog(@"****************");
    
    // 或者
    id mPerson2 = objc_msgSend(mClass, @selector(alloc));
    mPerson2 = objc_msgSend(mPerson2, @selector(init));
    NSLog(@"mPerson2：%@",mPerson2);
    NSLog(@"2.1.personName：%@",objc_getAssociatedObject(mPerson2, @"personName"));
   
    objc_setAssociatedObject(mPerson2, @"personName", @"Jim", OBJC_ASSOCIATION_COPY_NONATOMIC);
    NSLog(@"2.2.personName：%@",objc_getAssociatedObject(mPerson2, @"personName"));
    
    objc_msgSend(mPerson2, @selector(changePersonName:),@"Sam");
    NSLog(@"2.3.personName：%@",objc_getAssociatedObject(mPerson2, @"personName"));
    
    Ivar p_ivar = class_getInstanceVariable(mClass, "personName");
    NSString *personName = object_getIvar(mPerson2, p_ivar);
    NSLog(@"2.4.personName：%@",personName);
    
}

void changePersonName(id self, SEL cmd, NSString *newPersonName) {
    //获得类中的personName成员变量
    Ivar p_ivar = class_getInstanceVariable([self class], "personName");
    
    NSString *personName = object_getIvar(self, p_ivar);
    NSLog(@"personName-before->%@",personName);
    
    object_setIvar(self, p_ivar, [newPersonName copy]);
    NSLog(@"newPersonName-->%@",newPersonName);

    personName = object_getIvar(self, p_ivar);
    NSLog(@"personName-after->%@",personName);
    
    
}
// 必须实现方法，不然方法不会被加入到method_list
- (void)changePersonName:(NSString *)newPerson
{
    
}


+ (void)createProperty
{
    Class mClass = objc_allocateClassPair([NSObject class], "testClass", 0);
    objc_registerClassPair(mClass);
    
    /*
     R The property is read-only (readonly).
     C The property is a copy of the value last assigned (copy).
     & The property is a reference to the value last assigned (retain).
     N The property is non-atomic (nonatomic).
     G<name> The property defines a custom getter selector name. The name follows the G (for example, GcustomGetter,).
     S<name> The property defines a custom setter selector name. The name follows the S (for example, ScustomSetter:,).
     D The property is dynamic (@dynamic).
     W The property is a weak reference (__weak).
     P The property is eligible for garbage collection.
     t<encoding> Specifies the type using old-style encoding.
     */
    //设置类型
    objc_property_attribute_t attribute1;
    attribute1.name = "T";
    attribute1.value = @encode(NSString *);
    
    // 设置nonatomic
    objc_property_attribute_t attribute2 = {"N",""};
    // 设置copy
    objc_property_attribute_t attribute3 = {"C",""};
    // 设置对应的ivar名称
    objc_property_attribute_t attribute4 = {"V","_personName"};
    
    objc_property_attribute_t attributes[] = {attribute1,attribute2,attribute3,attribute4};
    
    class_addProperty(mClass, "personName", attributes, 4);
    
    
    unsigned int count;
    objc_property_t *list = class_copyPropertyList(mClass, &count);
    
    for (int i = 0; i < count; i++) {
        objc_property_t property = list[i];
        const char *str = property_getName(property);
        
        NSLog(@"%s",str);
    }
    
}
```