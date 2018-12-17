---
title: antd源码解读（一）Form组件解析
date: 2018-04-15 15:53:11
tags:
- antd
- 前端
- react
---

### 引言
看过antd源码的都知道，antd其实是在一组[react-componment](https://github.com/react-component)组件的基础上进行了一层ui封装，本文主要解读antd组件Form的基础组件[react-componment/form](https://github.com/react-component/form)，另外会略过`development`模式下的warning代码。

### Form.create
解读源码首先要从自己最常用的或者感兴趣的入手，首先form组件最主要的还是在`Form.create({options})`这个装饰器入手。找到项目下的文件`createForm.js`，这个文件还是主要主要对`createBaseForm.js`文件进行了一层封装，提供了一些默认配置参数，下看查看`createBaseForm.js`里的createBaseForm方法，改方法主要是一个装饰器作用，包装一个高阶React组件，在props里注入一个值为`formPropName(默认为form)`变量，所有功能在这个变量里完成，主要内容如下
```js
render() {
  const { wrappedComponentRef, ...restProps } = this.props;
  const formProps = {
    [formPropName]: this.getForm(), // 来在 formPropName默认为form，getForm方法来自`createForm.js`
  };
  if (withRef) {
    formProps.ref = 'wrappedComponent';
  } else if (wrappedComponentRef) {
    formProps.ref = wrappedComponentRef;
  }
  const props = mapProps.call(this, {
    ...formProps,
    ...restProps,
  });
  return <WrappedComponent {...props} />;
}
```
在装饰器初始化的时候，Form初始化了一个只属于该组件实例的store，用来存放当前Form组件的一些输入的数据，主要代码如下:
```js
const fields = mapPropsToFields && mapPropsToFields(this.props);  // mapPropsToFields来自于Form.create的配置参数，用来转化来自mobx或者redux等真正的store来源的value，以初始化该Form实例的fieldsStore
this.fieldsStore = createFieldsStore(fields || {});  // createFieldsStore来自于文件`createFieldsStore.js`文件
```
### getFieldDecorator
柯里化函数，通过id与参数声明的输入，返回一个函数以输入组件为入参的函数，通过该函数声明的输入组件与表单Form双向数据绑定。
```js
  ...
  getFieldDecorator(name, fieldOption) {
    const props = this.getFieldProps(name, fieldOption);  // 初始化一个field
    return (fieldElem) => {
      const fieldMeta = this.fieldsStore.getFieldMeta(name);  // 获取变化（Form的onChange）后的field数据
      const originalProps = fieldElem.props;
      fieldMeta.originalProps = originalProps;  // 输入组件初始化时保存的Prop
      fieldMeta.ref = fieldElem.ref;
      return React.cloneElement(fieldElem, {
        ...props,
        ...this.fieldsStore.getFieldValuePropValue(fieldMeta),  // 获取prop属性 value
      });
    };
  }
  ...
```
### getFieldProps
查看函数 `getFieldProps`，主要用来初始化输入组件的props，将特定的函数缓存在内部，如onChange事件，另外初次保存field到store中
```js
  ...
  getFieldProps(name, usersFieldOption = {}) {
    if (!name) {
      throw new Error('Must call `getFieldProps` with valid name string!');
    }
    delete this.clearedFieldMetaCache[name];
    const fieldOption = {
      name,
      trigger: DEFAULT_TRIGGER,
      valuePropName: 'value',
      validate: [],
      ...usersFieldOption, // 用户输入，如rules，initialValue
    };

    const {
      rules,
      trigger,
      validateTrigger = trigger,
      validate,
    } = fieldOption;

    const fieldMeta = this.fieldsStore.getFieldMeta(name);
    if ('initialValue' in fieldOption) {
      fieldMeta.initialValue = fieldOption.initialValue;
    }

    const inputProps = {
      ...this.fieldsStore.getFieldValuePropValue(fieldOption), // 获取输入组件的value，如果没有，返回initialValue
      ref: this.getCacheBind(name, `${name}__ref`, this.saveRef),
    };
    if (fieldNameProp) { // 及value
      inputProps[fieldNameProp] = name;
    }

    const validateRules = normalizeValidateRules(validate, rules, validateTrigger); // 校验规则标准化
    const validateTriggers = getValidateTriggers(validateRules);
    validateTriggers.forEach((action) => {
      if (inputProps[action]) return;
      inputProps[action] = this.getCacheBind(name, action, this.onCollectValidate); // 如果设置了输入校验rules，绑定onChange事件`this.onCollectValidate`
    });

    // make sure that the value will be collect
    if (trigger && validateTriggers.indexOf(trigger) === -1) {
      inputProps[trigger] = this.getCacheBind(name, trigger, this.onCollect); // 如果没有绑定rules校验，绑定默认的onChange事件
    }
    const meta = {
      ...fieldMeta,
      ...fieldOption,
      validate: validateRules,
    };
    this.fieldsStore.setFieldMeta(name, meta);  // 保存field到store中
    if (fieldMetaProp) {
      inputProps[fieldMetaProp] = meta;
    }
    if (fieldDataProp) {
      inputProps[fieldDataProp] = this.fieldsStore.getField(name);
    }
    return inputProps;
  },
  ...
```
### getCacheBind
`getCacheBind`方法，缓存函数，使用bind方法绑定上下文并缓存部分参数，返回一个新的函数，用做onChange及数据校验。
```js
  ...
  getCacheBind(name, action, fn) {
    if (!this.cachedBind[name]) {
      this.cachedBind[name] = {};
    }
    const cache = this.cachedBind[name];
    if (!cache[action]) {
      cache[action] = fn.bind(this, name, action); // 绑定参数并返回
    }
    return cache[action];
  },
  ...
```

### onCollectCommon
在`getFieldProps`方法中看到利用`getCacheBind`方法当无rules的时候绑定了一个`onCollect`方法，onCollect方法主要调用`onCollectCommon`方法，并将得到的结果保存到store。
```js
onCollectCommon(name, action, args) {
  const fieldMeta = this.fieldsStore.getFieldMeta(name);
  if (fieldMeta[action]) {  // 如果getFieldDecorator方法中的参数定义了onChange，则触发改onChange
    fieldMeta[action](...args);
  } else if (fieldMeta.originalProps && fieldMeta.originalProps[action]) { // 如果输入组件绑定了onChange，则触发该onChange
    fieldMeta.originalProps[action](...args);
  }
  const value = fieldMeta.getValueFromEvent ?  // 获取更新后的value，兼容原生组件e.target.value
    fieldMeta.getValueFromEvent(...args) :
    getValueFromEvent(...args);
  if (onValuesChange && value !== this.fieldsStore.getFieldValue(name)) {  // 如果Form.create时用户定义有onValuesChange，则触发
    const valuesAll = this.fieldsStore.getAllValues();
    const valuesAllSet = {};
    valuesAll[name] = value;
    Object.keys(valuesAll).forEach(key => set(valuesAllSet, key, valuesAll[key]));
    onValuesChange(this.props, set({}, name, value), valuesAllSet);
  }
  const field = this.fieldsStore.getField(name);    // 获取合并field，并返回
  return ({ name, field: { ...field, value, touched: true }, fieldMeta });
},
```

### onCollectValidate
在有输入rules的时候`getCacheBind`方法绑定`onCollectValidate`作为onChange事件，该方法做了除了调用了onCollectCommon事件以外，还调用了校验方法`validateFieldsInternal`。

### validateFieldsInternal
该方法主要是从store中获取rules校验规则并标准化后，使用`async-validator`模块进行校验，并把结果保存到store中，本文不做讲解。

### setFields
该方法主要是设置store中的field，因为store的数据是不可观测的数据，不会引起页面的重渲染，该方法也负责调用`forceUpdate()`强制更新页面。
```js
setFields(maybeNestedFields, callback) {
  const fields = this.fieldsStore.flattenRegisteredFields(maybeNestedFields); // 处理field嵌套问题
  this.fieldsStore.setFields(fields);
  if (onFieldsChange) {  // 如果设置有FieldsChange事件监听事件变化，则触发事件
    const changedFields = Object.keys(fields)
      .reduce((acc, name) => set(acc, name, this.fieldsStore.getField(name)), {});
    onFieldsChange(this.props, changedFields, this.fieldsStore.getNestedAllFields());
  }
  this.forceUpdate(callback);  // 强制更新视图
},
```

### 最后
主要方法大概就上面这些，其中流程差不多在每次setFields之前，会在store中存一个field的变化字段`fieldMeta`，在最后强制更新页面的时候，将该变量取出来做处理后覆盖到field，所有数据保存在field中，并提供了一些hock方法如`setFieldsValue`、`validateFields`等方法设置和获取store中的field字段和值。
