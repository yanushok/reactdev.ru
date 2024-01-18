---
description: Часто задаваемые вопросы о React Query
---

# Часто задаваемые вопросы о React Query

## How can I pass parameters to refetch? {#how-can-i-pass-parameters-to-refetch}
### Состояния загрузки {#loading-states}
## Why are updates not shown? {#why-are-updates-not-shown}
### 1: Query Keys are not matching {#1-query-keys-are-not-matching}
### 2: The QueryClient is not stable {#2-the-queryclient-is-not-stable}
## Why should I useQueryClient()... {#why-should-i-usequeryclient}
### 1: useQuery uses the hook too {#1-usequery-uses-the-hook-too}
### 2: It decouples your app from the client {#2-it-decouples-your-app-from-the-client}
### 3: You sometimes can't export {#3-you-sometimes-cant-export}
## Why do I not get errors ? {#why-do-i-not-get-errors-}
### Fetch API {#the-fetch-api}
### Логгирование {#logging}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/react-query-fa-qs](https://tkdodo.eu/blog/react-query-fa-qs)</small>
