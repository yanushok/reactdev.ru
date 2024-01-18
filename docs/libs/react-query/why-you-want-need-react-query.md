---
description: Зачем вам нужен React Query
---

# Зачем вам нужен React Query


## 1. Состояние гонки 🏎 {#1-race-condition-}
## 2. Состояние загрузки 🕐 {#2-loading-state-}
## 3. Пустое состояние 🗑️ {#3-empty-state-️}
## 4. Данные и ошибки не сбрасываются при изменении категории 🔄 {#4-data--error-are-not-reset-when-category-changes-}
## 5. Сработает дважды в режиме StrictMode 🔥🔥 {#5-will-fire-twice-in-strictmode-}
## Бонус: обработка ошибок 🚨 {#bonus-error-handling-}
## 🐛 Баги {#-bugs}
## Бонус: отмена {#bonus-cancellation}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/why-you-want-react-query](https://tkdodo.eu/blog/why-you-want-react-query)</small>
