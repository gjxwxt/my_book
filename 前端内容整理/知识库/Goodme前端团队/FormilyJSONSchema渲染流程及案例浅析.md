![cover_image](https://mmbiz.qpic.cn/sz_mmbiz_jpg/TpB2QHJbiaicGrggbfVEYVNuIwFG2SleOlpibYrvibrV3pElmgKOVyfiaUbGIGIMBicphhkyOVEXMmiaHeFjaDPzqgQlA/0?wx_fmt=jpeg)

#  Formily JSON Schema 渲染流程及案例浅析

原创  漆杰  [ Goodme前端团队 ](javascript:void\(0\);)

__ _ _ _ _

##  前言

Formily 是基于 JSON Schema 的强大表单解决方案，它提供了灵活的表单渲染和数据绑定能力。本文将探讨 Formily JSON Schema
渲染机制和实现细节，帮助我们更好地理解和应用该技术。

##  介绍

Formily 是一个基于 React 的高性能表单解决方案，它提供了 JSON Schema、JSX Schema、纯 JSX
三种开发模式，来描述表单结构和验证规则。

JSON Schema 是一个社区推动的 JSON 文件协议，用于规范 JSON
文件内容。它提供了丰富的属性和约束，可以完整地描述一个表单的各个字段和相关规则。它与平台无关，可以描述任意复杂的数据结构，相比 JSX，JSON
Schema 的描述格式更加紧凑，可读性更好。JSON Schema 在 JSON 的格式上，加入了一些标准化属性，用于描述结构化数据。

##  基本概念

  * 解析 JSON Schema：Formily 解析传入的 JSON Schema 对象，识别其中的字段类型、属性、校验规则等信息。它会根据字段类型选择相应的表单字段组件，并将属性和校验规则应用到相应的组件上。 
    
        const rule = (value) => {  
      if (!value?.length) {  
        return '请至少新增一个';  
      }  
      return true;  
    };  
      
    const formSchema = {  
      type: 'array',  
      title: '明细信息',  
      'x-decorator': 'FormItem',  
      'x-component': 'ArrayTable',  
      'x-validator': rule,  
      // ...  
    }  
    

  * 构建表单树：Formily 根据解析后的 JSON Schema 数据构建表单树，表单树是一个组件树结构，每个节点对应一个表单字段或表单组件。 
    
        import { createForm } from '@formily/core'  
    import { createSchemaField } from '@formily/react-schema-renderer'  
      
    const form = createForm()  
    const SchemaField = createSchemaField({  
      components: {  
        // 定义各个表单组件的渲染方式  
        FormItem,  
        Input,  
        Password,  
        VerifyCode,  
        // ...  
      },  
    })  
      
    const formTree = (  
      <Form form={form}>  
          <SchemaField schema={formSchema} />  
      </Form>;  
    )  
    

  * 表单数据绑定：Formily 通过双向绑定机制将表单数据与表单组件进行关联。当表单组件的值发生变化时，表单数据会自动更新；反之，当表单数据发生变化时，表单组件的值也会更新。这种双向绑定机制使得表单数据与界面保持同步，提供了便捷的数据操作能力。 

  * 表单校验和验证：Formily 根据 JSON Schema 中定义的验证规则对表单数据进行校验和验证。它自动根据字段类型和属性进行验证，并提供实时的验证反馈。 

  * 表单提交和数据处理：一旦用户完成表单填写，您可以使用 Formily 提供的 API 获取表单数据，并进行后续的处理，例如提交到服务器或进行其他业务逻辑操作。Formily 提供了丰富的功能和扩展性，支持自定义表单行为和数据处理逻辑。 

##  渲染流程

在每次进行开发的时候，我们都会使用 ` createSchemaField ` 来声明一个 ` SchemaField ` 来装载我们写好的 `
Schema ` ，那么它是如何实现的呢？ 部分实现如下：

    
    
    export function createSchemaField(options) {  
        if (options === void 0) { options = {}; }  
        function SchemaField(props) {  
            var schema = Schema.isSchemaInstance(props.schema)  
                ? props.schema  
                : new Schema(__assign({ type: 'object' }, props.schema));  
            var renderMarkup = function () {  
                env.nonameId = 0;  
                if (props.schema)  
                    return null;  
                return render(React.createElement(SchemaMarkupContext.Provider, { value: schema }, props.children));  
            };  
            var renderChildren = function () {  
                return React.createElement(RecursionField, __assign({}, props, { schema: schema }));  
            };  
            return (React.createElement(SchemaOptionsContext.Provider, { value: options },  
                React.createElement(SchemaComponentsContext.Provider, { value: lazyMerge(options.components, props.components) },  
                    React.createElement(ExpressionScope, { value: lazyMerge(options.scope, props.scope) },  
                        renderMarkup(),  
                        renderChildren()))));  
        }  
        SchemaField.displayName = 'SchemaField';  
        /// ...  
        return SchemaField;  
    }  
    

我们可以看到，代码相对比较简单，套了几层“壳子”之后，代码最终走向了 ` renderChildren ` 方法中的 ` RecursionField `
。

那么 ` RecursionField ` 究竟是个什么东西？我们接着往下看：

    
    
    export var RecursionField = function (props) {  
        var basePath = useBasePath(props);  
        var fieldSchema = useMemo(function () { return new Schema(props.schema); }, [props.schema]);  
        var fieldProps = useFieldProps(fieldSchema);  
        var renderProperties = function (field) {  
            if (props.onlyRenderSelf)  
                return;  
            var properties = Schema.getOrderProperties(fieldSchema);  
            if (!properties.length)  
                return;  
            return (React.createElement(Fragment, null, properties.map(function (_a, index) {  
                var item = _a.schema, name = _a.key;  
                var base = (field === null || field === void 0 ? void 0 : field.address) || basePath;  
                var schema = item;  
                if (isFn(props.mapProperties)) {  
                    var mapped = props.mapProperties(item, name);  
                    if (mapped) {  
                        schema = mapped;  
                    }  
                }  
                if (isFn(props.filterProperties)) {  
                    if (props.filterProperties(schema, name) === false) {  
                        return null;  
                    }  
                }  
                return (React.createElement(RecursionField, { schema: schema, key: "".concat(index, "-").concat(name), name: name, basePath: base }));  
            })));  
        };  
        var render = function () {  
            if (!isValid(props.name))  
                return renderProperties();  
            if (fieldSchema.type === 'object') {  
                if (props.onlyRenderProperties)  
                    return renderProperties();  
                return (React.createElement(ObjectField, __assign({}, fieldProps, { name: props.name, basePath: basePath }), renderProperties));  
            }  
            else if (fieldSchema.type === 'array') {  
                return (React.createElement(ArrayField, __assign({}, fieldProps, { name: props.name, basePath: basePath })));  
            }  
            else if (fieldSchema.type === 'void') {  
                if (props.onlyRenderProperties)  
                    return renderProperties();  
                return (React.createElement(VoidField, __assign({}, fieldProps, { name: props.name, basePath: basePath }), renderProperties));  
            }  
            return React.createElement(Field, __assign({}, fieldProps, { name: props.name, basePath: basePath }));  
        };  
        if (!fieldSchema)  
            return React.createElement(Fragment, null);  
        return (React.createElement(SchemaContext.Provider, { value: fieldSchema }, render()));  
    };  
    

以上代码看似复杂，其实我们只需要关注两点：

  1. 在第26行， ` RecursionField ` 对自身进行了递归调用。其实，这里也就说明了 ` Formily ` 是如何获取 ` properties ` ，然后根据 ` properties ` 的属性来进行递归渲染的了 

  2. ` render ` 方法中，针对不同的 ` type ` 使用了不同的 ` Field ` 来进行渲染。 

看似这么多个不同的 ` Field ` ，其实他们都有个相同的特点，就是里面都渲染了一层 ` ReactiveField ` 。就比如 `
ObjectField ` ：

    
    
    export var ObjectField = function (props) {  
        var form = useForm();  
        var parent = useField();  
        var field = useAttach(form.createObjectField(__assign({ basePath: parent === null || parent === void 0 ? void 0 : parent.address }, props)));  
        return (React.createElement(FieldContext.Provider, { value: field },  
            React.createElement(ReactiveField, { field: field }, props.children)));  
    };  
    ObjectField.displayName = 'ObjectField';  
    

那么，为什么要着重提到 ` ReactiveField ` ，它有什么魔力呢？

其实 ` ReactiveField ` 才是整个渲染流程里最最重要的核心，照惯例，先上源码。

具体实现如下：

    
    
    var ReactiveInternal = function (props) {  
        var _a;  
        var components = useContext(SchemaComponentsContext);  
        if (!props.field) {  
            return React.createElement(Fragment, null, renderChildren(props.children));  
        }  
        var field = props.field;  
        var content = mergeChildren(renderChildren(props.children, field, field.form), (_a = field.content) !== null && _a !== void 0 ? _a : field.componentProps.children);  
        if (field.display !== 'visible')  
            return null;  
        var getComponent = function (target) {  
            var _a;  
            return isValidComponent(target)  
                ? target  
                : (_a = FormPath.getIn(components, target)) !== null && _a !== void 0 ? _a : target;  
        };  
        var renderDecorator = function (children) {  
            if (!field.decoratorType) {  
                return React.createElement(Fragment, null, children);  
            }  
            return React.createElement(getComponent(field.decoratorType), toJS(field.decoratorProps), children);  
        };  
        var renderComponent = function () {  
            var _a, _b, _c;  
            if (!field.componentType)  
                return content;  
            // props 内容  
            var value = !isVoidField(field) ? field.value : undefined;  
            var onChange = !isVoidField(field)  
                ? function () {  
                    var _a, _b;  
                    var args = [];  
                    for (var _i = 0; _i < arguments.length; _i++) {  
                        args[_i] = arguments[_i];  
                    }  
                    field.onInput.apply(field, __spreadArray([], __read(args), false));  
                    (_b = (_a = field.componentProps) === null || _a === void 0 ? void 0 : _a.onChange) === null || _b === void 0 ? void 0 : _b.call.apply(_b, __spreadArray([_a], __read(args), false));  
                }  
                : (_a = field.componentProps) === null || _a === void 0 ? void 0 : _a.onChange;  
            var onFocus = !isVoidField(field)  
                ? function () {  
                    var _a, _b;  
                    var args = [];  
                    for (var _i = 0; _i < arguments.length; _i++) {  
                        args[_i] = arguments[_i];  
                    }  
                    field.onFocus.apply(field, __spreadArray([], __read(args), false));  
                    (_b = (_a = field.componentProps) === null || _a === void 0 ? void 0 : _a.onFocus) === null || _b === void 0 ? void 0 : _b.call.apply(_b, __spreadArray([_a], __read(args), false));  
                }  
                : (_b = field.componentProps) === null || _b === void 0 ? void 0 : _b.onFocus;  
            var onBlur = !isVoidField(field)  
                ? function () {  
                    var _a, _b;  
                    var args = [];  
                    for (var _i = 0; _i < arguments.length; _i++) {  
                        args[_i] = arguments[_i];  
                    }  
                    field.onBlur.apply(field, __spreadArray([], __read(args), false));  
                    (_b = (_a = field.componentProps) === null || _a === void 0 ? void 0 : _a.onBlur) === null || _b === void 0 ? void 0 : _b.call.apply(_b, __spreadArray([_a], __read(args), false));  
                }  
                : (_c = field.componentProps) === null || _c === void 0 ? void 0 : _c.onBlur;  
            var disabled = !isVoidField(field)  
                ? field.pattern === 'disabled' || field.pattern === 'readPretty'  
                : undefined;  
            var readOnly = !isVoidField(field)  
                ? field.pattern === 'readOnly'  
                : undefined;  
            return React.createElement(getComponent(field.componentType), __assign(__assign({ disabled: disabled, readOnly: readOnly }, toJS(field.componentProps)), { value: value, onChange: onChange, onFocus: onFocus, onBlur: onBlur }), content);  
        };  
        return renderDecorator(renderComponent());  
    };  
    ReactiveInternal.displayName = 'ReactiveField';  
    export var ReactiveField = observer(ReactiveInternal, {  
        forwardRef: true,  
    });  
    

在组件的17行和23行的方法里，分别去获取了 ` x-decorator ` 和 ` x-component ` 所对应的组件来进行渲染，两者稍有不同的是，
` x-component ` 的 ` props `
更为丰富。还有一个需要注意的细节点，在代码的第八行，获取到了所有子项，在25行的判断中，如果当前节点的组件类型为指定时，那么我们就认为他不是一个UI节点，那么就会直接返回它的子项进行后续的解析并渲染。最后，在代码的第70行，将
` Component ` 放入了 ` Decorator ` 中，并将 ` reactive-react ` 提供的 ` observer `
将该组件变成了响应式组件，这也就是解释了为什么 ` formily ` 能做到组件级别的精准刷新，因为他的响应式监听是最小粒度的。

##  实践

我们可以用以下代码简单构建一个表单：

    
    
    {  
     name: {  
      title: '姓名',  
      type: 'string',  
      'x-decorator': 'FormItem',  
      'x-component': 'Input',  
      'x-component-props': {  
       placeholder: '请输入姓名',  
      },  
     },  
     age: {  
      title: '年龄',  
      type: 'number',  
      'x-decorator': 'FormItem',  
      'x-component': 'Input',  
      'x-component-props': {  
       placeholder: '请输入年龄',  
      },  
     },  
     gender: {  
      title: '性别',  
      type: 'number',  
      'x-decorator': 'FormItem',  
      'x-component': 'Radio.Group',  
      enum: [{  
        label: '男',  
        value: 1,  
       },  
       {  
        label: '女',  
        value: 0,  
       },  
      ],  
     },  
    }  
    

效果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGrggbfVEYVNuIwFG2SleOlVYB6vHg1hywo9LDcAPkibHAkFoAbOKuXcsXicEV4fHp5Iopv3f83gSibw/640?wx_fmt=png&from=appmsg)
产品看了一眼：“太丑了！不要每个都占满一行。”于是我们又有了以下代码：

    
    
    {  
        layout: {  
            type: 'void',  
            'x-component': 'Layout.Inline',  
            'x-decorator': 'FormGrid',  
            properties: {  
                name: {  
                    title: '姓名',  
                    type: 'string',  
                    'x-decorator': 'FormItem',  
                    'x-component': 'Input',  
                    'x-component-props': {  
                        placeholder: '请输入姓名',  
                    },  
                },  
                age: {  
                    title: '年龄',  
                    type: 'number',  
                    'x-decorator': 'FormItem',  
                    'x-component': 'Input',  
                    'x-component-props': {  
                        placeholder: '请输入年龄',  
                    },  
                },  
                gender: {  
                    title: '性别',  
                    type: 'number',  
                    'x-decorator': 'FormItem',  
                    'x-component': 'Radio.Group',  
                    enum: [{  
                                label: '男',  
                                value: 1,  
                            },  
                            {  
                                label: '女',  
                                value: 0,  
                            },  
                    ],  
                },  
            },  
        },  
    }  
    

在之前代码的基础上，我们在外层套了一个 x-component 为 Layout.Inline 的节点，效果如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGrggbfVEYVNuIwFG2SleOlexfxib7PgmqgDdRWlUZ7spicFllsvFqbpVcuA5fygjM4lfFoESGwg60w/640?wx_fmt=png&from=appmsg)
这样确实更顺眼一点了。

可是，过了一会儿，后端又来了：我想要把数据的结构从

    
    
    {  
        name: '张三',  
        age: 18,  
        gender: 0  
    }  
    

改成

    
    
    {  
        {  
            nickname: '张三',  
            age: 18  
        },  
        gender: 0  
    }  
    

行吧...，咱也不敢问为什么要这样改。于是我们继续：

    
    
    {  
        layout: {  
            type: 'void',  
            'x-component': 'Layout.Inline',  
            'x-decorator': 'FormGrid',  
            properties: {  
            obj: {  
                type: 'object',  
                properties: {  
                    name: {  
                        title: '姓名',  
                        type: 'string',  
                        'x-decorator': 'FormItem',  
                        'x-component': 'Input',  
                        'x-component-props': {  
                            placeholder: '请输入姓名',  
                        },  
                    },  
                    age: {  
                        title: '年龄',  
                        type: 'number',  
                        'x-decorator': 'FormItem',  
                        'x-component': 'Input',  
                        'x-component-props': {  
                            placeholder: '请输入年龄',  
                        },  
                    },  
                },  
            },  
            gender: {  
                title: '性别',  
                type: 'number',  
                'x-decorator': 'FormItem',  
                'x-component': 'Radio.Group',  
                enum: [{  
                        label: '男',  
                        value: 1,  
                    },  
                    {  
                        label: '女',  
                        value: 0,  
                    },  
                ],  
            },  
            },  
        },  
    }  
    

这次我们在 name 和 age 的外层套了一个名为 obj 的节点，并将 type 设置为 object，并将 name 节点的名称改成了
nickname。验证后，我们发现，表单的数据的结构终于符合预期了。

没多久，产品又双叒叕跑过来了：“我希望性别选择为女性时，不用输入年龄，毕竟女孩的年龄需要保密。”

我：“？？？”

于是，我们继续作以下改动：

    
    
    {  
        layout: {  
            type: 'void',  
            'x-component': 'Layout.Inline',  
            'x-decorator': 'FormGrid',  
            properties: {  
                obj: {  
                    type: 'object',  
                    properties: {  
                        name: {  
                            title: '姓名',  
                            type: 'string',  
                            'x-decorator': 'FormItem',  
                            'x-component': 'Input',  
                            'x-component-props': {  
                                placeholder: '请输入姓名',  
                            },  
                        },  
                        age: {  
                            title: '年龄',  
                            type: 'number',  
                            'x-decorator': 'FormItem',  
                            'x-component': 'Input',  
                            'x-component-props': {  
                                placeholder: '请输入年龄',  
                            },  
                            'x-reactions': {  
                                dependencies: ['gender'],  
                                fulfill: {  
                                    schema: {  
                                        'x-disabled': '{{  $deps[0] === 0 }}',  
                                    },  
                                },  
                            },  
                        },  
                    },  
                },  
                gender: {  
                    title: '性别',  
                    type: 'number',  
                    'x-decorator': 'FormItem',  
                    'x-component': 'Radio.Group',  
                    enum: [{  
                            label: '男',  
                            value: 1,  
                        },  
                        {  
                            label: '女',  
                            value: 0,  
                        },  
                    ],  
                },  
            },  
        },  
    }  
    

相较于以前的代码，我们在 age 节点下加入了 x-reactions 属性，使其能够被 gender 的值影响，来动态变换 disable 的值。

终于，在磕磕绊绊中，我们完成了此次需求。

其实，以上例子虽然简单，但也从侧面反应出，Formily 基于 JSON Schema
的强大表单解决能力。无论是应对表单数据结构的调整，数据字段名的更改，还是组件样式更改和组件联动，都能轻而易举地通过简单配置来化解。

不过在日常项目中，我们所需要经历的场景，大多都复杂得多。

##  复杂场景

###  需求概述

![](https://mmbiz.qpic.cn/sz_mmbiz_png/TpB2QHJbiaicGrggbfVEYVNuIwFG2SleOlAWU4NBKNmpz4eiarzKRDmiaBIME84BChG35rlXPfY16L2GsczhZFTlDg/640?wx_fmt=png&from=appmsg)
如上图， ` 业务对象编码 ` 是一个下拉框 ` Select ` ，它的值改变时，调用接口，将接口返回的数据，应用到其他三个表单数据中。

` 业务对象名称 ` 同理，在它的值改变时，也会调用接口，与业务对象编码行为一致，它会反过来影响业务对象编码的值。

###  需求分析

既然是表单联动，我们首先会想到 x-reactions 属性，我们可以用它来达到组件联动的目的。于是我们心里有了以下的大致结构：

    
    
    {  
        businessObjectCode: {  
            type: 'string',  
            title: '业务对象编码',  
            'x-decorator': 'FormItem',  
            'x-component': 'Select',  
            'x-component-props': {  
                placeholder: '请选择业务对象编码',  
            },  
            required: true,  
            'x-reactions': {  
                dependencies: ['xxx'],  
                fulfill: {  
                    schema: {  
                        'xxx': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
        businessObjectName: {  
            type: 'string',  
            title: '业务对象名称',  
            'x-decorator': 'FormItem',  
            'x-component': 'Select',  
            'x-component-props': {  
                    placeholder: '请选择业务对象名称',  
            },  
            required: true,  
            'x-reactions': {  
                dependencies: ['xxx'],  
                fulfill: {  
                    schema: {  
                        'xxx': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
        ownerName: {  
            type: 'string',  
            title: '货主',  
            'x-decorator': 'FormItem',  
            'x-component': 'Input',  
            required: true,  
            'x-disabled': true,  
            'x-reactions': {  
                dependencies: ['xxx'],  
                fulfill: {  
                    schema: {  
                        'xxx': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
        businessObjectType: {  
            type: 'string',  
            title: '业务对象类型',  
            'x-decorator': 'FormItem',  
            'x-component': 'Input',  
            required: true,  
            'x-disabled': true,  
            'x-reactions': {  
                dependencies: ['xxx'],  
                fulfill: {  
                    schema: {  
                        'xxx': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
    }  
    

我们为每个节点都添加了 x-reactions
属性，使其能够被影响。但是我们的需求并不是简单的通过组件的值去影响其他组件，而是组件的值请求接口后再去影响其他组件。如下图：

以业务对象编码为例

    
    
    graph TD  
      
    Start --> 业务对象编码下拉选择 --> 通过选择到的业务对象编码请求接口内容 --> 将返回的接口数据进行解析 --> 将解析出的数据放如表单中 --> Stop  
    

那么，由此不难想到，要有一个地方放接口请求的逻辑，它能够随着下拉框的数据改变而进行数据请求。所以，简单的 Select
已经不能满足我们的场景了，我们需要自定义一个组件，把这些逻辑都封装到其中。伪代码大致如下：

    
    
    const BusinessObjectSelect: React.FunctionComponent<IBusinessObjectSelectProps> = ({  
      params,  
      onChange,  
    }) => {  
      const handleChange = () => {  
        // 处理接口入参  
        const requestParams = transformParams(params);  
          
        const { response } = service.fetchBusinessObjectData(requestParams);  
          
        // 处理接口数据  
        const data = transformResponse(response);  
        // 将处理完的数据放入onChange  
        onChange(data);  
      };  
      
      return (  
        <Select  
          {...props}  
          labelInValue  
          value={value}  
          onSearch={handleSearch}  
          onChange={handleChange}  
          options={data}  
          defaultActiveFirstOption={false}  
          showArrow={false}  
          filterOption={false}  
        />  
      );  
    };  
    

将组件注册完之后（不懂如何注册详询官方文档），再将 Schema 结构中的组件进行替换，并分别对其依赖属性进行修改。

    
    
    businessObjectCode: {  
        type: 'string',  
        title: '业务对象编码',  
        'x-decorator': 'FormItem',  
        'x-component': 'Select',  
        'x-component-props': {  
            placeholder: '请选择业务对象编码',  
        },  
        required: true,  
            'x-reactions': {  
                dependencies: ['businessObjectName'],  
                fulfill: {  
                    schema: {  
                        'x-component-props.params': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
        businessObjectName: {  
            type: 'string',  
            title: '业务对象名称',  
            'x-decorator': 'FormItem',  
            'x-component': 'Select',  
            'x-component-props': {  
                placeholder: '请选择业务对象名称',  
            },  
            required: true,  
            'x-reactions': {  
                dependencies: ['businessObjectCode'],  
                fulfill: {  
                    schema: {  
                        'x-component-props.params': '{{ $deps[0] }}',  
                    },  
                },  
            },  
        },  
    

然而，出现了新的问题：

  1. 在处理另外两个表单数据（ownerName，businessObjectType）的时候，x-reactions 应该依赖谁。因为无论依赖了其中哪一个，在另一个下拉框数据改变的时候，都无法触发联动了。 

  2. 在提交表单的时候，表单的数据结构被污染了。本应该是 
    
        {  
        businessObjectCode:"",  
        businessObjectName: "",  
        ownerName: "",  
        businessObjectType: "",  
    }  
    

现在却变成了

    
        {  
        businessObjectCode:{  
            // 包含了整个表单数据在内的一大坨数据  
        },  
        businessObjectName: {  
            // 包含了整个表单数据在内的一大坨数据  
        },  
        ownerName: "",  
        businessObjectType: "",  
    }  
    

  3. 现在的表单数据因为联动的需要，x-reactions 中的 dependencies 里的数据（businessObjectCode，businessObjectName）也必须为数据对象，在初始化表单时，可能也得将 businessObjectCode 和 businessObjectName 处理成数据对象。 

  4. 反直觉。在人的直觉中，我们往往希望所见即所得，而现在的方案包含了太多隐式的内容。 

  5. 维护成本。如果后续有需求变更，比如字段调整，表单结构调整，对于维护这段代码的人来说，都是非常难受的。（而我们原本以为改几个字母就好了的呀...） 

当然以上问题，都是可以解决的。比如问题1，在value值改变的时候，我们可以手动触发一下组件的onChange，当然，可能会有一些边界 case
需要考虑。比如问题2和3，我们可以中间做一层数据转换。

但是至此，我们花了太多的成本，甚至以上都是笔者简化过的逻辑和代码。试问，我们使用 Formily
的初衷，不是为了提高我们的效率，减轻我们的心智负担吗？而现在似乎背道而驰。

当然，对于熟悉 Formily 的同学，可能也会想到使用 createForm 中 effects 提供的钩子，去监听组件的值改变，然后再去改变整个
form 的数据。可是一旦表单复杂到一定程度，钩子里的逻辑维护成本也是非常高的，而且，如果像笔者一样，在项目中，大部分场景都使用了 Formily
本身的联动能力去解决，仅针对这种特殊场景，用钩子函数去处理的话，显得不伦不类。但话说回来，这也确实能解决问题，而且比较符合直觉。

所以最终，笔者采用了与此类似的方式，在每个 Select 组件的 onChange 中进行接口请求之后，再使用 form.setValues
来改变整个表单的数据。这样，逻辑似乎更简洁了，也不需要去处理自定义组件里的异常 case，仅仅只需要关注 onChange
以及表单的数据应该怎么赋值。表单数据也干净了，不需要花费额外的转换成本。对于此案例，如果是你，你会怎么处理呢？

##  结论

由此可见，如果仅依靠 Formily 本身的联动能力，在应对某些复杂场景时，是存在一定的局限性的。

另外，在表单交互过程中，JSON Schema 的反复解析可能成为性能瓶颈，特别是当 schema
内容庞大且需要进行复杂的联动和批量更新时，性能问题会更加明显。这是因为 Formily
在内部对状态进行深拷贝，并进行深度遍历的脏检测。虽然这种方式提升了用户体验，但在处理大量数据的情况下，性能问题会显现出来。在这种情况下，考虑屏蔽
Formily 的局部重复渲染，回退到 React 的整树渲染可能是一个解决方案。

然而，任何方案都只能解决部分问题。一个方案可以满足80%的场景，剩下的20%可能需要采用其他方案来处理。因此，在面对问题时，需要根据具体情况选择合适的方案来解决。

##  最后

📚 小茗文章推荐：

  * [ 古茗是如何做前端数据中心的 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485119&idx=1&sn=359c8963c2f89ebfc175a76d1f54faf2&chksm=cfe581b8f89208ae90b04d2b6dd316841ffe7f1ab6389e7186a6974efb9e0e3edc3fb780b53a&scene=21#wechat_redirect)
  * [ 你一定要知道的「React组件」两种调用方式 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247485084&idx=1&sn=a90bb79426534ab5535f2fbb00e711f2&chksm=cfe5819bf892088d2a76a125a62e1b4b2c77a4c241dda4dff5c24e7061a83ed878bd8be0579b&scene=21#wechat_redirect)
  * [ 中后台业务开发（一）「表单原理」 ](http://mp.weixin.qq.com/s?__biz=Mzg4OTkwMTY3Mg==&mid=2247484905&idx=1&sn=88e09add6d0df02ab6217c3f9d6a6b96&chksm=cfe582eef8920bf8279f085b85ac43205d34ef15106a864a172baf318af39588f07ab41eccfe&scene=21#wechat_redirect)

关注公众号「Goodme前端团队」，获取更多干货实践，欢迎交流分享~

  

预览时标签不可点

微信扫一扫  
关注该公众号





****



****



×  分析

  收藏

