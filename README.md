# 뷰 속성 데코레이터

[![npm](https://img.shields.io/npm/v/vue-property-decorator.svg)](https://www.npmjs.com/package/vue-property-decorator)
[![Build Status](https://travis-ci.org/kaorun343/vue-property-decorator.svg?branch=master)](https://travis-ci.org/kaorun343/vue-property-decorator)

이 라이브러리는 [뷰-클래그-컴포넌트](https://github.com/vuejs/vue-class-component)에 전적으로 의존하고 있습니다, 그러니 이 라이브러리를 사용하기 전에 그것의 README 를 읽어주세요.

## 라이센스

MIT License

## 설치

```bash
npm i -S vue-property-decorator
```

## 사용

7 데코레이터와 1 함수(Mixin)가 있습니다:

* `@Emit`
* `@Inject`
* `@Model`
* `@Prop`
* `@Provide`
* `@Watch`
* `@Component` ([뷰-클래스-컴포넌트](https://github.com/vuejs/vue-class-component)로 **부터** )
* `Mixins` ([뷰-클래스-컴포넌트](https://github.com/vuejs/vue-class-component)에 정의된 기명 `mixins` 헬퍼 함수)

### `@Prop(options: (PropOptions | Constructor[] | Constructor) = {})` 데코레이터

```ts
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) propA!: number
  @Prop({ default: 'default value' }) propB!: string
  @Prop([String, Boolean]) propC: string | boolean
}
```

는 다음과 같다

```js
export default {
  props: {
    propA: {
      type: Number
    },
    propB: {
      default: 'default value'
    },
    propC: {
      type: [String, Boolean]
    },
  }
}
```

**알아두세요:**

* [reflect-metadata](https://github.com/rbuckton/reflect-metadata) 는 이 라이브러리에서 사용되지 않으며 `emitDecoratorMetadata` 설정을 `true` 로 해도 아무 의미 없다.
* 각 프롭의 기본 값은 위의 예시와 같이 정의될 필요가 있다.

### `@Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {})` 데코레이터

```ts
import { Vue, Component, Model } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Model('change', { type: Boolean }) checked!: boolean
}
```

는 다음과 같다

```js
export default {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: {
      type: Boolean
    },
  },
}
```

### `@Watch(path: string, options: WatchOptions = {})` 데코레이터

```ts
import { Vue, Component, Watch } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Watch('child')
  onChildChanged(val: string, oldVal: string) { }

  @Watch('person', { immediate: true, deep: true })
  onPersonChanged(val: Person, oldVal: Person) { }
}
```

는 다음과 같다

```js
export default {
  watch: {
    'child': {
      handler: 'onChildChanged',
      immediate: false,
      deep: false
    },
    'person': {
      handler: 'onPersonChanged',
      immediate: true,
      deep: true
    }
  },
  methods: {
    onChildChanged(val, oldVal) { },
    onPersonChanged(val, oldVal) { }
  }
}
```

### `@Emit(event?: string)` 데코레이터

`@Emit` `$emit` 로 데코레이트된 함수는 그들의 반환 값은 그들의 원본 인수에 따라간다. 반환값이 프로미스라면, emitted 전에 resolved 된다.

If the name of the event is not supplied via the `event` argument, the function name is used instead. In that case, the camelCase name will be converted to kebab-case.

`event` 인자를 통해 지원되지 않는 이벤트 이름이 제공된다면, 함수 이름이 대신 사용된다. 그 경우에, camelCase 이름이 케밥-케이스 로 변환된다.

```ts
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  count = 0

  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  promise() {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```

는 다음과 같다

```js
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    promise() {
      const promise = new Promise(resolve => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })

      promise.then(value => {
        this.$emit('promise', value)
      })
    }
  }
}
```

### `@Provide(key?: string | symbol)` / `@Inject(options?: { from?: InjectKey, default?: any } | InjectKey)` 데코레이터

> provide and inject are primarily provided for advanced plugin / component library use cases. It is NOT recommended to use them in generic application code.
> https://vuejs.org/v2/api/#provide-inject

```ts
import { Component, Inject, Provide, Vue } from 'vue-property-decorator'

const symbol = Symbol('baz')

@Component
export class MyComponent extends Vue {
  @Inject() foo!: string
  @Inject('bar') bar!: string
  @Inject({ from: 'optional', default: 'default' }) optional!: string
  @Inject(symbol) baz!: string
  
  @Provide() foo = 'foo'
  @Provide('bar') baz = 'bar'
}
```

는 다음과 같다

```js
const symbol = Symbol('baz')

export const MyComponent = Vue.extend({

  inject: {
    foo: 'foo',
    bar: 'bar',
    'optional': { from: 'optional', default: 'default' },
    [symbol]: symbol
  },
  data () {
    return {
      foo: 'foo',
      baz: 'bar'
    }
  },
  provide () {
    return {
      foo: this.foo,
      bar: this.baz
    }
  }
})
```

## 보세요 또

[vuex-class](https://github.com/ktsn/vuex-class/)
