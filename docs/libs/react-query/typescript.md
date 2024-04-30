---
description: Статья о типизации React Query кода в компонентах
---

# React Query и Typescript

[TypeScript](https://www.typescriptlang.org/) — это 🔥, таково общее мнение среди разработчиков во frontend-сообществе. Многие ожидают, что библиотеки будут написаны на TypeScript или, по крайней мере, предоставят хорошие определения типов. Для меня, если библиотека написана на TypeScript, определения типов являются лучшей документацией. Они никогда не ошибаются, потому что они напрямую отражают реализацию. Я часто смотрю на определения типов, прежде чем читать документацию API.

React Query изначально был написан на JavaScript (v1), а затем был полностью переписан на TypeScript с v2. Это означает, что в настоящее время имеется очень хорошая поддержка для пользователей TypeScript.

Однако есть несколько моментов, с которыми стоит быть осторожным при работе с TypeScript из-за того, насколько динамичен и непринужден React Query. Давайте рассмотрим их по порядку, чтобы сделать ваш опыт работы с библиотекой еще лучше.

## Generics {#generics}

React Query широко использует [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html). Это необходимо, потому что библиотека фактически не извлекает данные для вас, и она не может знать, какого _типа_ будут данные, которые ваш API вернет.

Секция по TypeScript [в официальной документации](https://tanstack.com/query/latest/docs/framework/react/typescript) не очень обширна, и она говорит нам явно указывать Generics, которые ожидает `useQuery` при его вызове:

```ts title="explicit-generics"
function useGroups() {
  return useQuery<Group[], Error>({
    queryKey: ['groups'],
    queryFn: fetchGroups,
  })
}
```

!!!danger "Обновление"
    Документация была обновлена, и в основном теперь она **не** поддерживает этот подход.

С течением времени React Query добавил больше обобщений в хук `useQuery` (теперь их четыре), в основном из-за добавления дополнительного функционала. Приведенный выше код работает и обеспечивает правильный тип для свойства `data` нашего пользовательского хука как `Group[] | undefined`, а также убеждается в том, что свойство `error` имеет тип `Error | undefined`. Однако это может не работать в более сложных случаях, особенно когда требуются другие два обобщения.

### Четыре дженерика {#the-four-generics}

Вот текущее определение хука useQuery:

```ts title="useQuery"
export function useQuery<
  TQueryFnData = unknown,
  TError = unknown,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey
>
```

Здесь происходит много вещей, давайте попробуем разобраться:

- `TQueryFnData`: тип, возвращаемый из `queryFn`. В приведенном выше примере это `Group[]`.
- `TError`: тип ошибок, которые мы ожидаем от `queryFn`. В данном примере это `Error`.
- `TData`: тип, который в конечном итоге будет у нашего свойства `data`. Это актуально только в том случае, если вы используете опцию `select`, поскольку в этом случае свойство `data` может отличаться от того, что возвращает `queryFn`. В противном случае оно будет по умолчанию равно тому, что возвращает `queryFn`.
- `TQueryKey`: тип нашего queryKey, актуально только в случае использования queryKey, передаваемого в ваш queryFn.

Как вы можете видеть, у всех этих обобщений есть значения по умолчанию, что означает, что если вы их не предоставляете, TypeScript будет придерживаться этих типов. Это работает примерно так же, как параметры по умолчанию в JavaScript:

```js title="default-parameters"
function multiply(a, b = 2) {
  return a * b
}

multiply(10) // ✅ 20
multiply(10, 3) // ✅ 30
```

### Вывод типа {#type-inference}

TypeScript работает лучше, если вы позволяете ему самостоятельно выводить (или определять) тип чего-либо. Это не только облегчает написание кода (потому что вам не нужно печатать все типы 😅), но и делает код более читаемым. Во многих случаях это может сделать код в точности похожим на JavaScript. Некоторые простые примеры вывода типов могут быть:

```ts title="type-inference"
const num = Math.random() + 5 // ✅ `number`

// 🚀 и приветствие, и результат приветствия будут строковыми
function greet(greeting = 'ciao') {
  return `${greeting}, ${getName()}`
}
```

Generics, как правило, также могут быть выведены из их использования, что замечательно. Вы также можете предоставить их вручную, но во многих случаях это не обязательно.

```ts title="generic-identity"
function identity<T>(value: T): T {
  return value
}

// 🚨 нет необходимости предоставлять generic
let result = identity<number>(23)

// ⚠️ или объявлять тип переменной
let result: number = identity(23)

// 😎 корректно переводит в `строку`
let result = identity('react-query')
}
```

### Вывод аргумента частичного типа {#partial-type-argument-inference}

...к сожалению, пока не поддерживается в TypeScript (см. [этот открытый вопрос](https://github.com/microsoft/TypeScript/issues/26242)). Это означает, что если вы предоставляете один Generic, вам придется предоставить все остальные. Однако из-за наличия в React Query значений по умолчанию для Generics, мы можем не заметить сразу, что они будут использованы. Результирующие сообщения об ошибках могут быть довольно непонятными. Давайте рассмотрим пример, где это может стать проблемой:

```ts title="default-generics"
function useGroupCount() {
  return useQuery<Group[], Error>({
    queryKey: ['groups'],
    queryFn: fetchGroups,
    select: (groups) => groups.length,
    // 🚨 Type '(groups: Group[]) => number' is not assignable to type '(data: Group[]) => Group[]'.
    // Type 'number' is not assignable to type 'Group[]'.ts(2322)
  })
}
```

Поскольку мы не предоставили третий Generic, вступает в силу значение по умолчанию, которое также является `Group[]`, но мы возвращаем тип `number` из нашей функции `select`. Один из способов исправить это - просто добавить третий Generic:

```ts title="third-generic"
function useGroupCount() {
  // ✅ исправлено
  return useQuery<Group[], Error, number>({
    queryKey: ['groups'],
    queryFn: fetchGroups,
    select: (groups) => groups.length,
  })
}
```

Пока у нас нет вывода аргументов частичного типа, мы должны работать с тем, что у нас есть.

Так в чем же альтернатива?

### Вывести все типы {#infer-all-the-things}

Давайте начнем с того, чтобы вообще не передавать никакие обобщенные типы и дать TypeScript самостоятельно разобраться, что делать. Для этого нам нужно, чтобы у `queryFn` был хороший тип возврата. Конечно, если вы встраиваете эту функцию без явного указания типа возврата, у вас будет `any`, потому что именно это дает `axios` или `fetch`:

```ts title="inlined-queryFn"
function useGroups() {
  // 🚨 данные будут `any`
  return useQuery({
    queryKey: ['groups'],
    queryFn: () =>
      axios.get('groups').then((response) => response.data),
  })
}
```

Если вы (как и я) предпочитаете держать свой слой API отдельно от запросов, вам все равно придется добавить определения типов, чтобы избежать _неявного_ `any`, так что React Query может вывести остальное:

```ts title="inferred-types"
function fetchGroups(): Promise<Group[]> {
  return axios.get('groups').then((response) => response.data)
}

// ✅ данные будут типа `Group[] | undefined`
function useGroups() {
  return useQuery({ queryKey: ['groups'], queryFn: fetchGroups })
}

// ✅ данные будут типа `number | undefined`
function useGroupCount() {
  return useQuery({
    queryKey: ['groups'],
    queryFn: fetchGroups,
    select: (groups) => groups.length,
  })
}
```

Преимущества этого подхода:

- нет необходимости вручную указывать обобщенные типы
- работает для случаев, когда нужны 3-й (select) и 4-й (QueryKey) обобщенные типы
- будет продолжать работать, если будут добавлены дополнительные обобщенные типы
- код менее запутанный / выглядит больше как JavaScript

### А что насчет ошибки? {#what-about-error}

Вы можете спросить, а что насчет ошибки? По умолчанию, без каких-либо обобщенных типов, тип ошибки будет выведен как `unknown`. Это может показаться багом, почему это не `Error`? Но это на самом деле преднамеренно, потому что в JavaScript вы можете бросать _что угодно_ - это не обязательно должен быть тип `Error`:

```js title="totally-legit-throw-statements"
throw 5
throw undefined
throw Symbol('foo')
```

Поскольку React Query не контролирует функцию, возвращающую Promise, он также не может знать, какого типа ошибки она может порождать. Так что тип `unknown` верен. Когда TypeScript позволит пропускать некоторые обобщенные типы при вызове функции с несколькими обобщенными типами (см. [этот вопрос для получения дополнительной информации](https://github.com/microsoft/TypeScript/issues/10571)), мы могли бы обработать это лучше, но на данный момент, если нам нужно работать с ошибками и мы не хотим прибегать к передаче обобщенных типов, мы можем сузить тип с проверкой instanceof:

```jsx title="narrow-with-instanceof"
const groups = useGroups()

if (groups.error) {
  // 🚨 Это не работает, потому что: Object is of type 'unknown'.ts(2571)
  return <div>An error occurred: {groups.error.message}</div>
}

// ✅ проверка instanceOf сужается до типа `Error`
if (groups.error instanceof Error) {
  return <div>An error occurred: {groups.error.message}</div>
}
```

Поскольку нам все равно нужно делать какую-то проверку, чтобы увидеть, есть ли у нас ошибка, проверка с использованием instanceof выглядит совсем не такой уж и плохой и также гарантирует, что у нашей ошибки на самом деле есть свойство message во время выполнения. Это также соответствует тому, что TypeScript планирует в выпуске 4.4, где они введут новый флаг компилятора `useUnknownInCatchVariables`, в котором переменные catch будут иметь тип `unknown` вместо `any` (см. [здесь](https://github.com/microsoft/TypeScript/issues/41016)).

!!!danger "Обновление"
    С версии 4, тип для `Error` по умолчанию устанавливается в `Error` вместо `unknown`. Несмотря на то, что на JavaScript можно бросать что угодно (что делает `unknown` самым правильным типом), почти всегда бросаются Error (или их подклассы). Это изменение упрощает работу с полем ошибки в TypeScript для большинства случаев.

    Кроме того, React Query позволяет регистрировать глобальные ошибки с помощью модульного расширения:

    ```ts title="global-error-registration"
    declare module '@tanstack/react-query' {
      interface Register {
        defaultError: AxiosError
      }
    }
    ```

    С этим вы можете вернуть поведение v4, установив `defaultError: unknown`.


## Сужение типов {#type-narrowing}

Я редко использую деструктуризацию при работе с React Query. Прежде всего, имена, такие как `data` и `error`, довольно универсальны (намеренно такие), поэтому вы, вероятно, все равно переименуете их. Сохранение целого объекта сохранит контекст того, какие данные это или откуда идет ошибка. Это далее поможет TypeScript сузить типы при использовании поля статуса или одного из булевых полей статуса, что не удается сделать при использовании деструктуризации:

```ts title="type-narrowing"
const { data, isSuccess } = useGroups()
if (isSuccess) {
  // 🚨 Здесь данные по-прежнему будут `Group[] | undefined`
}

const groupsQuery = useGroups()
if (groupsQuery.isSuccess) {
  // ✅ groupsQuery.data теперь будет `Group[]`
}
```

Это не имеет никакого отношения к React Query, это просто то, как работает TypeScript. У [@danvdk](https://twitter.com/danvdk) есть [хорошее объяснение](https://twitter.com/danvdk/status/1363614288103964672) этого поведения.

!!!danger "Обновление"
    Отлично! Улучшения в TypeScript 4.6, в частности [анализ управления потоком для деструктурированных различенных объединений](https://devblogs.microsoft.com/typescript/announcing-typescript-4-6/#cfa-destructured-discriminated-unions), сделали такие примеры возможными. Рад, что они постоянно улучшают язык! 🚀

## Безопасность типов с использованием опции enabled {#type-safety-with-the-enabled-option}

Я выражал свою 💖 по поводу опции [`enabled`](practical.md#the-enabled-option-is-very-powerful) с самого начала, но на уровне типов это может быть немного сложным, если вы хотите использовать ее для [зависимых запросов](https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries) и отключить ваш запрос до тех пор, пока некоторые параметры еще не определены:

```ts title="the-enabled-option"
function fetchGroup(id: number): Promise<Group> {
  return axios.get(`group/${id}`).then((response) => response.data)
}

function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: () => fetchGroup(id),
    enabled: Boolean(id),
  })
  // 🚨 Argument of type 'number | undefined' is not assignable to parameter of type 'number'.
  //  Type 'undefined' is not assignable to type 'number'.ts(2345)
}
```

Технически TypeScript прав, `id` может быть `undefined`: опция `enabled` не выполняет уточнение типа. Кроме того, есть способы обойти опцию `enabled`, например, вызвав метод `refetch`, возвращенный из `useQuery`. В этом случае `id` может действительно быть `undefined`.

Лучший способ здесь, если вы не любите оператор [non-null assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#non-null-assertion-operator), — принять, что `id` может быть `undefined`, и отклонить Promise в `queryFn`. Это немного дублирует код, но также является явным и безопасным:

```ts title="explicit-id-check"
function fetchGroup(id: number | undefined): Promise<Group> {
  // ✅ проверьте id во время выполнения, потому что он может быть `undefined`
  return typeof id === 'undefined'
    ? Promise.reject(new Error('Invalid id'))
    : axios.get(`group/${id}`).then((response) => response.data)
}

function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: () => fetchGroup(id),
    enabled: Boolean(id),
  })
}
```

## Optimistic Updates {#optimistic-updates}

Правильная реализация оптимистичных обновлений в TypeScript — это не простая задача, поэтому мы решили добавить это как исчерпывающий [пример](https://tanstack.com/query/latest/docs/framework/react/examples/optimistic-updates-ui) в документацию.

Важная часть: вы должны явно указать тип аргумента `variables`, передаваемого в `onMutate`, чтобы получить наилучший вывод типов. Я полностью не понимаю, почему так происходит, но, кажется, это снова связано с выводом дженерик типов. Взгляните [на этот](https://github.com/tannerlinsley/react-query/pull/1366#discussion_r538459890) комментарий для получения дополнительной информации.

!!!danger "Обновление"
    TypeScript 4.7 внес [улучшения в вывод типов для объектов и методов](https://devblogs.microsoft.com/typescript/announcing-typescript-4-7-beta/#improved-function-inference-in-objects-and-methods), что решает эту проблему. Оптимистичные обновления теперь должны выводить типы для контекста правильно без дополнительных усилий. 🥳

    Кроме того, в React Query v5 появился [новый способ сделать оптимистичные обновления](https://tanstack.com/query/v5/docs/react/guides/optimistic-updates#via-the-ui), который может существенно упростить их написание.

## useInfiniteQuery {#useinfinitequery}

В большинстве случаев типизация `useInfiniteQuery` ничем не отличается от типизации `useQuery`. Заметным «подводным камнем» является то, что значение `pageParam`, которое передается в `queryFn`, имеет тип `any`. В библиотеке это, конечно, можно улучшить, но пока оно any, вероятно, лучше явно определить его:

```ts title="useInfiniteQuery"
type GroupResponse = { next?: number; groups: Group[] }
const queryInfo = useInfiniteQuery({
  queryKey: ['groups'],
  // ⚠️ explicitly type pageParam to override `any`
  queryFn: ({
    pageParam = 0,
  }: {
    pageParam: GroupResponse['next']
  }) => fetchGroups(groups, pageParam),
  getNextPageParam: (lastGroup) => lastGroup.next,
})
```

Если `fetchGroups` возвращает `GroupResponse`, `lastGroup` будет иметь свой тип красиво выведенным, и мы можем использовать тот же тип для определения `pageParam`.

!!!danger "Обновление"
    Это исправлено в версии 5, потому что теперь вам нужно предоставить явный `initialPageParam`, который имеет правильный тип:

    ```ts title="initialPageParam" hl_lines="3-4 7"
    const queryInfo = useInfiniteQuery({
      queryKey: ['groups'],
      // ✅ typed correctly as number
      queryFn: ({ pageParam }) =>
        fetchGroups(groups, pageParam),
      getNextPageParam: (lastGroup) => lastGroup.next,
      initialPageParam: 0,
    })
    ```

## Типизация функции запроса по умолчанию {#typing-the-default-query-function}

Я лично не использую [defaultQueryFn](https://tanstack.com/query/latest/docs/framework/react/guides/default-query-function), но я знаю, что многие люди это делают. Это удобный способ использовать переданный `queryKey` для прямого построения URL-запроса. Если вы встраиваете функцию при создании `queryClient`, тип переданного `QueryFunctionContext` также будет автоматически выведен. TypeScript просто гораздо удобнее, когда вы встраиваете вещи :)

```ts title="defaultQueryFn"
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey: [url] }) => {
        const { data } = await axios.get(`${baseUrl}/${url}`)
        return data
      },
    },
  },
})
```

Это просто работает, однако тип `url` выводится как `unknown`, потому что весь `queryKey` - это массив с типом `unknown`. На момент создания `queryClient` совершенно невозможно гарантировать, как будут построены `queryKeys` при вызове `useQuery`, поэтому здесь React Query может сделать не так уж и много. Это просто природа этой крайне динамичной функции. Это не плохо, потому что это означает, что теперь вам нужно работать с защитой и сужать тип с проверками времени выполнения для работы с ним, например:

```ts title="narrow-with-typeof" hl_lines="5-12"
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey: [url] }) => {
        // ✅ narrow the type of url to string
        // so that we can work with it
        if (typeof url === 'string') {
          const { data } = await axios.get(
            `${baseUrl}/${url.toLowerCase()}`
          )
          return data
        }
        throw new Error('Invalid QueryKey')
      },
    },
  },
})
```

Думаю, это довольно хорошо показывает, почему `unknown` - это такой отличный (и недооцененный) тип по сравнению с `any`. В последнее время это стало моим любимым типом - но это тема для другого блог-поста. 😊

<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-and-type-script](https://tkdodo.eu/blog/react-query-and-type-script)</small>
