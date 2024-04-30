---
description: Статья о тестировании кода в котором используется React Query
---

# Тестирование React Query

Вопросы по теме тестирования возникают довольно часто в контексте React Query, поэтому я постараюсь ответить на некоторые из них здесь. Я думаю, что одной из причин этого является то, что тестирование "умных" компонентов (также называемых [контейнерными компонентами](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)) не является самым простым вопросом. С появлением хуков этот подход был в значительной степени устаревшим. Теперь поощряется непосредственное использование хуков там, где они нужны, вместо того, чтобы делать в основном произвольное разделение и передачу пропсов вниз.

Я считаю, что это в целом очень хорошее улучшение для соседства и читаемости кода, но теперь у нас есть больше компонентов, которые потребляют зависимости вне "просто пропсов".

Они могут использовать `useContext`. Они могут использовать `useSelector`. Или они могут использовать `useQuery`.

Технически эти компоненты больше не являются чистыми, потому что их вызов в разных средах приводит к разным результатам. При их тестировании вам нужно тщательно настроить окружающие среды, чтобы все работало.

## Создание моков сетевых запросов {#mocking-network-requests}

Поскольку React Query - это библиотека управления состоянием асинхронного сервера, ваши компоненты вероятно будут делать запросы к бэкенду. При тестировании этот бэкенд недоступен для передачи данных, и даже если он доступен, вы, вероятно, не хотите делать ваши тесты зависимыми от этого.

Есть множество статей о том, как мокировать данные с помощью Jest. Вы можете мокировать свой клиент API, если у вас есть один. Вы можете мокировать fetch или axios напрямую. Я могу только подтвердить то, что написал Kent C. Dodds в своей статье [Stop mocking fetch](https://kentcdodds.com/blog/stop-mocking-fetch):

Используйте [Mock Service Worker](https://mswjs.io/) от [@ApiMocking](https://twitter.com/ApiMocking)

Это может быть вашим единственным источником истины, когда речь идет о мокировании ваших API:

- работает в Node для тестирования,
- поддерживает REST и GraphQL,
- есть [аддон для storybook](https://storybook.js.org/addons/msw-storybook-addon), так что вы можете писать истории для ваших компонентов, использующих `useQuery`
- работает в браузере в режиме разработки, и вы по-прежнему увидите запросы, отправленные в инструментах разработчика браузера,
- работает с cypress, аналогично фикстурам.

С нашим сетевым уровнем все в порядке, мы можем начать говорить о специфичных для React Query вещах, на которые нужно обратить внимание:

## QueryClientProvider {#queryclientprovider}

Всякий раз, когда вы используете React Query, вам нужен `QueryClientProvider` и передать ему `queryClient` - сосуд, который содержит `QueryCache`. Кеш, в свою очередь, будет содержать данные ваших запросов.

Я предпочитаю предоставлять каждому тесту свой собственный `QueryClientProvider` и создавать `new QueryClient` для каждого теста. Таким образом, тесты полностью изолированы друг от друга. Другой подход может заключаться в очистке кеша после каждого теста, но мне нравится минимизировать общее состояние между тестами. В противном случае вы можете получить неожиданные и непостоянные результаты, если запускаете тесты параллельно.

### Для кастомных хуков {#for-custom-hooks}

Если вы тестируете пользовательские хуки, я уверен, что вы используете [react-hooks-testing-library](https://react-hooks-testing-library.com/). Это самый простой способ тестировать хуки. С помощью этой библиотеки мы можем обернуть наш хук в [обертку](https://react-hooks-testing-library.com/reference/api#wrapper), которая представляет собой компонент React, оборачивающий тестируемый компонент при рендеринге. Я считаю, что это идеальное место для создания QueryClient, потому что он будет выполнен один раз для каждого теста:

```jsx title="wrapper"
const createWrapper = () => {
  // ✅ создает новый QueryClient для каждого теста
  const queryClient = new QueryClient()
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

test('my first test', async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper(),
  })
})
```

### Для компонентов {#for-components}

Если вы хотите протестировать компонент, который использует хук `useQuery`, вам также нужно обернуть этот компонент в `QueryClientProvider`. Небольшая обертка вокруг `render` из [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/) кажется хорошим выбором. Посмотрите, как React Query делает это [внутренне для своих тестов](https://github.com/tannerlinsley/react-query/blob/ead2e5dd5237f3d004b66316b5f36af718286d2d/src/react/tests/utils.tsx#L6-L17).

## Выключите повторные запросы {#turn-off-retries}

Это одна из наиболее распространенных "ловушек" с React Query и тестированием: Библиотека по умолчанию использует три повтора с экспоненциальной задержкой, что означает, что ваши тесты вероятно завершатся по таймауту, если вы хотите протестировать ошибочный запрос. Самый простой способ отключить повторы - снова через `QueryClientProvider`. Рассмотрим расширенный пример выше:

```jsx title="no-retries" hl_lines="2-9"
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        // ✅ выключает повторные запросы
        retry: false,
      },
    },
  })

  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

test("my first test", async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })
}
```

Это установит значения по умолчанию для всех запросов в дереве компонентов как работающие "без повторов". Важно знать, что это сработает только в том случае, если ваш текущий `useQuery` не имеет явно установленных повторов. Если у вас есть запрос, который хочет 5 повторов, он всё равно будет иметь преимущество, потому что значения по умолчанию принимаются только как резервные.

### setQueryDefaults {#setquerydefaults}

Лучший совет, который я могу вам дать для этой проблемы: не устанавливайте эти параметры непосредственно в `useQuery`. Постарайтесь использовать и переопределять значения по умолчанию насколько это возможно, и если вам действительно нужно что-то изменить для конкретных запросов, используйте [queryClient.setQueryDefaults](https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientsetquerydefaults).

Так, например, вместо установки повторов в `useQuery`:

```jsx title="not-on-useQuery"
const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}

function Example() {
  // 🚨 you cannot override this setting for tests!
  const queryInfo = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    retry: 5,
  })
}
```

Установите его следующим образом:

```jsx title="setQueryDefaults" hl_lines="9-10"
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
    },
  },
})

// ✅ only todos will retry 5 times
queryClient.setQueryDefaults(['todos'], { retry: 5 })

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

Здесь все запросы будут повторяться два раза, только запрос todos будут повторяться пять раз, и у меня все еще есть возможность выключить это для всех запросов в моих тестах 🙌.

### ReactQueryConfigProvider {#reactqueryconfigprovider}

Конечно, это работает только для известных ключей запросов. Иногда вам действительно хочется установить некоторые конфигурации для подмножества дерева компонентов. В v2 у React Query был ReactQueryConfigProvider именно для этого случая использования. Вы можете добиться того же самого в v3 с помощью нескольких строк кода:

```jsx title="ReactQueryConfigProvider"
const ReactQueryConfigProvider = ({ children, defaultOptions }) => {
  const client = useQueryClient()
  const [newClient] = React.useState(
    () =>
      new QueryClient({
        queryCache: client.getQueryCache(),
        muationCache: client.getMutationCache(),
        defaultOptions,
      })
  )

  return (
    <QueryClientProvider client={newClient}>
      {children}
    </QueryClientProvider>
  )
}
```

Вы можете увидеть это в действии в этом [примере CodeSandbox](https://codesandbox.io/s/react-query-config-provider-v3-lt00f).

## Всегда дожидайтесь выполнения запроса {#always-await-the-query}

Поскольку React Query асинхронен по своей природе, при запуске хука вы не получите результат немедленно. Обычно он будет в состоянии загрузки и без данных для проверки. [Утилиты для работы с асинхронными операциями](https://react-hooks-testing-library.com/reference/api#async-utilities) из библиотеки react-hooks-testing-library предлагают множество способов решения этой проблемы. В самом простом случае мы можем просто дождаться, пока запрос перейдет в состояние успеха:

```jsx title="waitFor" hl_lines="21-24"
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  })
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

test("my first test", async () => {
  const { result, waitFor } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })

  // ✅ wait until the query has transitioned to success state
  await waitFor(() => result.current.isSuccess)

  expect(result.current.data).toBeDefined()
}
```

!!!danger "Обновление"
    В версии [@testing-library/react v13.1.0](https://github.com/testing-library/react-testing-library/releases/tag/v13.1.0) также появился новый метод [renderHook](https://testing-library.com/docs/react-testing-library/api/#renderhook), который вы можете использовать. Однако он не возвращает свою собственную утилиту `waitFor`, поэтому вам придется использовать утилиту, которую можно импортировать из [@testing-library/react](https://testing-library.com/docs/dom-testing-library/api-async/#waitfor). API немного отличается, поскольку оно не позволяет возвращать логическое значение, но ожидает вместо этого `Promise`. Это означает, что мы должны немного адаптировать наш код:

    ```jsx title="new-render-hook" hl_lines="8-11"
    import { waitFor, renderHook } from '@testing-library/react'

    test("my first test", async () => {
      const { result } = renderHook(() => useCustomHook(), {
        wrapper: createWrapper()
      })

      // ✅ return a Promise via expect to waitFor
      await waitFor(() => expect(result.current.isSuccess).toBe(true))

      expect(result.current.data).toBeDefined()
    }
    ```

## Собирая все вместе {#putting-it-all-together}

Я создал репозиторий, в котором все это хорошо сочетается: mock-service-worker, react-testing-library и упомянуая обертка. В нем содержатся четыре теста - базовые тесты на ошибку и успех для пользовательских хуков и компонентов. Взгляните здесь: [https://github.com/TkDodo/testing-react-query](https://github.com/TkDodo/testing-react-query)

<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/testing-react-query](https://tkdodo.eu/blog/testing-react-query)</small>
