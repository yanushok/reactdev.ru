---
description: Статья о работе приложения в режиме offline с использованием Reqct Query
---

# Работа offline в React Query

## Проблемы в v3 {#issues-in-v3}
### 1) нет данных в кэше {#1-no-data-in-the-cache}
### 2) отсутствие повторных попыток запроса {#2-no-retries}
### 3) запросы, которым не нужна сеть {#3-queries-that-dont-need-the-network}
## Новый NetworkMode {#the-new-networkmode}
### online {#online}
#### fetchStatus {#fetchStatus}
### always {#always}
### offlineFirst {#offlinefirst}
## Что все это значит для меня? {#what-does-all-of-this-mean-for-me-exactly}







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


<small>:material-information-outline: Источник &mdash; [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)</small>
