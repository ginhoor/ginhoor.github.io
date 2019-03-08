
## Form
是一个包含多个表单字段控件的分组容器。

每个单独的表单字段控件都对应一个表单字段，并将表单作为它们共同的祖先（ancestor）、调用`FormState`的方法来进行保存、重置和验证表单中的每个表单字段控件。要获得FormState，可以用`Form.of（context）`传入包含表单的context，或者通过GlobalKey传递给Form构造函数，并调用`GlobalKey.currentState`

## FormField<T> class
表单字段控件

这个控件维护表单字段的状态，只管的在UI上体现更新和验证错误。

当在表单中使用，可以通过`FormState`来查询和操作所有的表单数据。比如每次调用`FormState.save`，将会依次调用FormField的`onSaved`毁掉方法。

如果想要检测表单字段的当前状态，可以使用带有“FormFiled”的GlobalKey。比如一个表单字段依赖另一个表单字段。

使用表单字段控件，表单不是必须的，表单的作用是一次性简单的保存、重置或者验证多个字段。若要在没有表单的情况下使用，传递GlobalKey给构造函数，使用GlobalKey.currentState来保存或重置表单字段控件。

## Flex
## Row
mainAxisSize: MainAxisSize.max
## Expanded
