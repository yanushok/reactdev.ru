---
description: Статья об использований мутаций для изменения данных в React Query
---

# Осваиваем мутации в React Query

## Что такое мутации? {#what-are-mutations}
## Сходства с useQuery {#similarities-to-usequery}
## Отличия от UseQuery {#differences-to-usequery}
## Привязка мутаций к запросам {#tying-mutations-to-queries}
### Инвалидизация {#invalidation}
### Прямые обновления {#direct-updates}
## Optimistic updates {#optimistic-updates}
### Пример {#example}
## Распространенные проблемы {#common-gotchas}
### ожидаемые промисы {#awaited-promises}
### Mutate или MutateAsync {#mutate-or-mutateasync}
### Мутации принимают только один аргумент для переменных {#mutations-only-take-one-argument-for-variables}
### Некоторые коллбэки могут не сработать {#some-callbacks-might-not-fire}







```ts title="pre-filtering" hl_lines="17-26"
type State = 'all' | 'open' | 'done'
type Todo = {
  id: number
  state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`)
  return response.data
}

export const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],
    queryFn: () => fetchTodos(state),
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>([
        'todos',
        'all',
      ])
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? []

      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/mastering-mutations-in-react-query](https://tkdodo.eu/blog/mastering-mutations-in-react-query)</small>
