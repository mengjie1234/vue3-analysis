# API详解

## normalizeVNode
child == null 或者child是布尔值，创建并返回Comment类型VNode。

child为数组类型，创建并返回Fragment类型VNnode，child作为子VNode。

child为其他类型，创建并返回Text类型VNode，child作为子VNode。

#### 语法
```typescript
export function normalizeVNode(child: VNodeChild): VNode {
  if (child == null || typeof child === 'boolean') {
    // empty placeholder
    return createVNode(Comment)
  } else if (isArray(child)) { // [h('div, 'one'), two] 数组就是fragment了 normalizeVnode 来进行vnode的转换 其实也就是 type 为 fragment child 为那个数组罢了
    // fragment
    return createVNode(Fragment, null, child)
  } else if (typeof child === 'object') {
    // already vnode, this should be the most common since compiled templates
    // always produce all-vnode children arrays
    return child.el === null ? child : cloneVNode(child)
  } else {
    // strings and numbers
    return createVNode(Text, null, String(child))
  }
}
```

#### 参数（选填）
child，VNodeChild

#### 返回值
VNode类型



## getFallthroughAttrs
遍历attrs，返回，过滤掉键不为'class'或者'style'或者正则匹配为on开头的键值对，并返回。

#### 代码
```typescript
type Data = { [key: string]: unknown }

const getFallthroughAttrs = (attrs: Data): Data | undefined => {
  let res: Data | undefined
  for (const key in attrs) {
    if (key === 'class' || key === 'style' || isOn(key)) {
      ;(res || (res = {}))[key] = attrs[key]
    }
  }
  return res
}
```

#### 返回值
undefined | Data



## defineComponent

在创建组件的时候，使用该方法可以使用typescript的类型检测，参数补充。

#### 代码

```typescript
function defineComponent(options: unknown) {
  return isFunction(options) ? { setup: options } : options
}

interface LegacyOptions<
  Props,
  D,
  C extends ComputedOptions,
  M extends MethodOptions,
  Mixin extends ComponentOptionsMixin,
  Extends extends ComponentOptionsMixin
> {
  // allow any custom options
  [key: string]: any

  // state
  // Limitation: we cannot expose RawBindings on the `this` context for data
  // since that leads to some sort of circular inference and breaks ThisType
  // for the entire component.
  data?: (
    this: CreateComponentPublicInstance<Props>,
    vm: CreateComponentPublicInstance<Props>
  ) => D
  computed?: C
  methods?: M
  watch?: ComponentWatchOptions
  provide?: Data | Function
  inject?: ComponentInjectOptions

  // composition
  mixins?: Mixin[]
  extends?: Extends

  // lifecycle
  beforeCreate?(): void
  created?(): void
  beforeMount?(): void
  mounted?(): void
  beforeUpdate?(): void
  updated?(): void
  activated?(): void
  deactivated?(): void
  beforeUnmount?(): void
  unmounted?(): void
  renderTracked?: DebuggerHook
  renderTriggered?: DebuggerHook
  errorCaptured?: ErrorCapturedHook
}
```

#### 参数

options，Funtion类型或者LegacyOptions类型

#### 返回值

LegacyOptions | { setup: Function }