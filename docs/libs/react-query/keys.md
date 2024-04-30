---
description: Эффективные ключи запросов в React Query
---

# Эффективные ключи запросов в React Query

[Ключи запросов](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys) - очень важное базовое понятие в React Query. Они необходимы, чтобы библиотека могла правильно кешировать ваши данные внутренне и автоматически перезапрашивать их, когда изменяется зависимость от вашего запроса. Наконец, это позволит вам взаимодействовать с кешем запросов вручную при необходимости, например, при обновлении данных после мутации или когда вам нужно вручную инвалидировать некоторые запросы.

Давайте быстро рассмотрим, что означают эти три аспекта, прежде чем показать вам, как я лично организую ключи запросов, чтобы быть способным делать эти вещи более эффективно.

## Кэширование данных {#caching-data}

Внутренне Кеш запросов - это просто объект JavaScript, где ключи представляют собой сериализованные ключи запросов, а значения - ваши данные запроса плюс метаинформация. Ключи хешируются [детерминированным способом](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys#query-keys-are-hashed-deterministically), поэтому вы также можете использовать объекты (однако на верхнем уровне ключи должны быть строками или массивами).

Самая важная часть в том, что ключи должны быть *уникальными* для ваших запросов. Если React Query обнаруживает запись для ключа в кеше, он использует ее. Также обратите внимание, что нельзя использовать один и тот же ключ для `useQuery` и `useInfiniteQuery`. В конце концов, существует только *один* Кеш запросов, и вы бы поделили данные между этими двумя. Это нехорошо, потому что бесконечные запросы имеют фундаментально разную структуру по сравнению с "обычными" запросами.

```ts
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})

// 🚨 это не будет работать
useInfiniteQuery({
  queryKey: ['todos'],
  queryFn: fetchInfiniteTodos,
})

// ✅ выберите что-то другое
useInfiniteQuery({
  queryKey: ['infiniteTodos'],
  queryFn: fetchInfiniteTodos,
})
```

## Автоматическое извлечение информации {#automatic-refetching}

<div style="text-align: center; border-radius: 8px; border: 2px solid #76c2af; color: #76c2af; padding: 10px; font-weight: bold">
  Запросы являются декларативными
</div>

Это *очень* важное понятие, которое нельзя недооценить, и это также нечто, что может потребовать некоторого времени для "усвоения". Большинство людей думают о запросах, особенно о перезапросе, в *императивном* ключе.

У меня есть запрос, который получает некоторые данные. Теперь я нажимаю эту кнопку, и я хочу выполнить перезапрос, но с другими параметрами. Я видел множество попыток, которые выглядят так:

```jsx title="imperative-refetch"
function Component() {
  const { data, refetch } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  // ❓ как передать параметры для повторной выборки ❓
  return <Filters onApply={() => refetch(???)} />
}
```

Ответ: **Вам и не нужно.**

Это не то, для чего предназначен метод `refetch` - он предназначен для перезапроса *с теми же параметрами.*

Если у вас есть какое-то *состояние*, которое изменяет ваши данные, все, что вам нужно сделать, это поместить его в ключ запроса, потому что React Query автоматически запускает перезапрос, когда ключ изменяется. Так что когда вы хотите применить свои фильтры, просто измените *состояние вашего клиента*:

```jsx title="query-key-drives-the-query"
function Component() {
  const [filters, setFilters] = React.useState()
  const { data } = useQuery({
    queryKey: ['todos', filters],
    queryFn: () => fetchTodos(filters),
  })

  // ✅ задайте локальное состояние и позвольте ему управлять запросом
  return <Filters onApply={setFilters} />
}
```

Перерисовка, вызванная обновлением `setFilters`, передаст React Query другой ключ запроса, что заставит его выполнить перезапрос. У меня есть более подробный пример в статье [#1: Практический React Query - Обращение с ключом запроса как с массивом зависимостей](practical.md#treat-the-query-key-like-a-dependency-array).

## Ручное взаимодействие {#manual-interaction}

Ручные взаимодействия с кешем запросов - это момент, когда структура ваших ключей запросов имеет наибольшее значение. Многие из этих методов взаимодействия, такие как [invalidateQueries](https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientinvalidatequeries) или [setQueriesData](https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientsetqueriesdata), поддерживают [фильтры запросов](https://tanstack.com/query/latest/docs/framework/react/guides/filters#query-filters), которые позволяют вам нечетко сопоставлять ваши ключи запросов.

## Эффективные ключи запросов {#effective-react-query-keys}

Обратите внимание, что эти моменты отражают мое личное мнение (как и все на этом блоге, собственно говоря), поэтому не принимайте их как нечто, что вы обязательно должны делать при работе с ключами запросов. Я обнаружил, что эти стратегии лучше всего работают, когда ваше приложение становится более сложным, и они также хорошо масштабируются. Вам определенно не нужно делать это для приложения со списком дел 😁.

### Размещение {#colocate}

Если вы еще не прочитали статью [Maintainability through colocation](https://kentcdodds.com/blog/colocation) от [Kent C. Dodds](https://twitter.com/kentcdodds), пожалуйста, сделайте это. Я не считаю, что хранение всех ваших ключей запросов глобально в `/src/utils/queryKeys.ts` сделает все лучше. Я держу свои ключи запросов рядом с соответствующими запросами, соседствующими в директории функции, примерно так:

```
- src
  - features
    - Profile
      - index.tsx
      - queries.ts
    - Todos
      - index.tsx
      - queries.ts
```

Файл *queries* будет содержать все, что связано с React Query. Обычно я экспортирую только пользовательские хуки, поэтому сами функции запросов, а также ключи запросов, останутся локальными.

### Всегда используйте массивы ключей {#always-use-array-keys}

Да, ключи запросов могут быть также строками, но для единообразия я предпочитаю всегда использовать массивы. React Query в любом случае внутренне преобразует их в массивы, так что:

```ts title="always-use-array-keys"
// 🚨 в любом случае будет преобразовано в ['todos']
useQuery({ queryKey: 'todos' })
// ✅
useQuery({ queryKey: ['todos'] })
```

**Обновление**: С React Query версии 4 все ключи должны быть массивами.

### Структура {#structure}

Структурируйте ваши ключи запросов *от наиболее общих к наиболее конкретным*, с любым количеством уровней детализации между ними, которое вы считаете нужным. Вот как я бы структурировал список задач, который позволяет фильтровать списки, а также просматривать детали:

```js
['todos', 'list', { filters: 'all' }]
['todos', 'list', { filters: 'done' }]
['todos', 'detail', 1]
['todos', 'detail', 2]
```

С такой структурой я могу инвалидировать все, что связано с задачами, с помощью `['todos']`, все списки или все детали, а также нацелиться на конкретный список, если я знаю точный ключ. Обновления из ответов мутации становятся намного более гибкими с этим, потому что вы можете нацелиться на все списки, если это необходимо:

```js title="updates-from-mutation-responses"
function useUpdateTitle() {
  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      // ✅ обновить информацию о делах
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

      // ✅ обновить все списки, содержащие это задание
      queryClient.setQueriesData(['todos', 'list'], (previous) =>
        previous.map((todo) =>
          todo.id === newTodo.id ? newtodo : todo
        )
      )
    },
  })
}
```

Это может не сработать, если структура списков и деталей сильно отличается, поэтому в качестве альтернативы вы также можете, конечно, просто инвалидировать все списки:

```js title="invalidate-all-lists" hl_lines="10-13"
function useUpdateTitle() {
  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

      // ✅ просто аннулирует все списки
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list']
      })
    },
  })
}
```

Если вы знаете, на каком списке вы находитесь в данный момент, например, читая фильтры из URL, и поэтому можете сконструировать точный ключ запроса, вы также можете объединить эти два метода и вызвать `setQueryData` на вашем списке и инвалидировать все остальные:

```js title="combine" hl_lines="14-28"
function useUpdateTitle() {
  // представьте себе пользовательский хук,
  // который возвращает текущие фильтры, сохраненные в url
  const { filters } = useFilterParams()

  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

      // ✅ обновить список, в котором мы сейчас находимся
      queryClient.setQueryData(
        ['todos', 'list', { filters }],
        (previous) =>
          previous.map((todo) =>
            todo.id === newTodo.id ? newtodo : todo
          )
      )

      // 🥳 аннулируйте все списки,
      // но не перевыбирайте активный.
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list'],
        refetchActive: false,
      })
    },
  })
}
```

**Обновление**: В версии 4 `refetchActive` был заменен на `refetchType`. В приведенном выше примере это будет `refetchType: 'none'`, потому что мы не хотим ничего перезапрашивать.

### Используйте фабрики ключей запросов {#use-query-key-factories}

В приведенных выше примерах вы видите, что я много вручную объявлял ключи запросов. Это не только подвержено ошибкам, но и делает изменения в будущем сложнее, например, если вы обнаружите, что хотели бы добавить еще один уровень детализации к вашим ключам.

Поэтому я рекомендую использовать одну фабрику ключей запросов для каждой функциональности. Это просто объект с записями и функциями, которые будут производить ключи запросов, которые вы затем можете использовать в ваших пользовательских хуках. Для приведенной выше структуры пример может выглядеть примерно так:

```ts title="query-key-factory"
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
}
```

Это дает мне много гибкости, поскольку каждый уровень строится на основе другого, но все же является независимо доступным:

```ts title="examples"
// 🕺 удалите все, что связано с фичей todos
queryClient.removeQueries({
  queryKey: todoKeys.all
})

// 🚀 аннулировать все списки
queryClient.invalidateQueries({
  queryKey: todoKeys.lists()
})

// 🙌 предварительная выборка одного todo
queryClient.prefetchQueries({
  queryKey: todoKeys.detail(id),
  queryFn: () => fetchTodo(id),
})
```


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/effective-react-query-keys](https://tkdodo.eu/blog/effective-react-query-keys)</small>
