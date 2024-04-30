# Документация

**React Query** - библиотека для получения, кэширования, синхронизации и обновления "серверного" состояния в React-приложениях.

## Установка

```bash
yarn add react-query
# или
npm i react-query
```

## Пример

```js
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from 'react-query';

const queryClient = new QueryClient();

export const App = () => (
  <QueryClientProvider>
    <Example />
  </QueryClientProvider>
);

function Example() {
  const { isLoading, error, data } = useQuery(
    'repoData',
    () =>
      fetch(
        'https://api.github.com/repos/tannerlinsley/react-query'
      ).then((response) => response.json())
  );

  if (isLoading) return <p>Загрузка...</p>;

  if (error) return <p>Ошибка: {error.message}</p>;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{' '}
      <strong>✨ {data.stargazers_count}</strong>{' '}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  );
}
```

## Быстрый старт

Ключевыми концепциями `React Query` являются:

- Запросы (queries)
- Мутации (mutations)
- Инвалидация (аннулирование, признание недействительным) запроса (query invalidation), его кэша

```js
import {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
} from 'react-query';

import { getTodos, postTodo } from '../api';

// Создаем клиента
const queryClient = new QueryClient();

const App = () => (
  // Передаем клиента в приложение
  <QueryClientProvider client={queryClient}>
    <Todos />
  </QueryClientProvider>
);

function Todos() {
  // Получаем доступ к клиенту
  const queryClient = useQueryClient();

  // Запрос
  const query = useQuery('todos', getTodos);

  // Мутация
  const mutation = useMutation(postTodo, {
    onSuccess: () => {
      // Инвалидация и обновление
      queryClient.invalidateQueries('todos');
    },
  });

  return (
    <div>
      <ul>
        {query.data.map((todo) => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>

      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now(),
            title: 'Привет',
          });
        }}
      >
        Добавить задачу
      </button>
    </div>
  );
}

render(<App />, document.getElementById('root'));
```

## Инструменты разработчика

По умолчанию инструменты разработчика не включаются в производственную сборку (когда `process.env.NODE_ENV === 'production'`).

Существует два режима добавления инструментов в приложение: плавающий (floating) и встроенный (inline).

### Плавающий режим

```js
import { ReactQueryDevtools } from 'react-query/devtools';

const App = () => (
  <QueryClientProvider client={queryClient}>
    {/* Другие компоненты */}
    <ReactQueryDevtools initialIsOpen={false} />
  </QueryClientProvider>
);
```

#### Настройки

- `initialIsOpen` - если `true`, то панель инструментов по умолчанию будет открыта
- `panelProps` - используется для добавления пропов к панели, например, `className`, `style` и т.д.
- `closeButtonProps` - используется для добавления пропов к кнопке закрытия панели
- `toggleButtonProps` - используется для добавления пропов к кнопке переключения
- `position`: "top-left" | "top-right" | "bottom-left" | "bottom-right" - положение логотипа `React Query` для открытия/закрытия панели

### Встроенный режим

```js
import { ReactQueryDevtoolsPanel } from 'react-query/devtools';

const App = () => (
  <QueryClientProvider client={queryClient}>
    {/* Другие компоненты */}
    <ReactQueryDevtoolsPanel
      style={styles}
      className={className}
    />
  </QueryClientProvider>
);
```

## Важные настройки по умолчанию

- Экземпляры запроса(query instances), созданные с помощью `useQuery` или `useInfiniteQuery` по умолчанию считают кэшированные данные устаревшими (stale)
- Устаревшие запросы автоматически повторно выполняются в фоновом режиме (background) в следующих случаях:
  - Монтирование нового экземпляра запроса
  - Переключения окна в браузере
  - Повторное подключение к сети
  - Когда задан интервал выполнения повторного запроса (refetch interval)
- Результаты запроса, не имеющие активных экземпляров `useQuery`, `useInfiniteQuery` или наблюдателей (observers) помечаются как "неактивные" (inactive) и остаются в кэше
- По умолчанию запросы, помеченные как неактивные, уничтожаются сборщиком мусора через 5 минут
- Провалившиеся запросы автоматически повторно выполняются 3 раза с экспоненциально увеличивающейся задержкой перед передачей ошибки в UI
- Результаты запросов по умолчанию структурно распределяются для определения того, действительно ли данные изменились, и если это не так, ссылка на данные остается неизменной, что способствует стабилизации данных при использовании `useMemo` и `useCallback`

## Запросы

### Основы

Запрос - это декларативная зависимость асинхронного источника данных, связанного с уникальным ключом. Запрос может использоваться с любым основанным на промисе методом (включая методы `GET` и `POST`) получения данных от сервера. Если метод изменяет данные на сервере, то лучше использовать мутации.

Для подписки на запрос в компоненте или пользовательском хуке следует вызвать хук `useQuery` со следующими параметрами:

- Уникальный ключ запроса
- Функция, возвращающая промис, который
  - разрешается данными (resolve data) или
  - выбрасывает исключение (throw error)

```js
import { useQuery } from 'react-query';

function App() {
  const info = useQuery('todos', fetchTodoList);
}
```

Уникальный ключ используется для выполнения повторных запросов, кэширования и распределения запросов в приложении.

Результаты запроса, возвращаемые `useQuery`, содержат всю необходимую информацию о запросе.

```js
const result = useQuery('todos', fetchTodos);
```

Объект `result` содержит несколько важных состояний (states). Запрос может находиться в одном из следующих состояний:

- `isLoading` или `status === 'loading'` - запрос находится на стадии выполнения, данные еще не получены
- `isError` или `status === 'error'` - запрос завершился ошибкой
- `isSuccess` или `status === 'success'` - запрос завершился успешно, данные доступны
- `isIdle` или `status === 'idle'` - запрос отключен

Кроме того, объект `result` содержит следующую информацию:

- `error` - если запрос находится в состоянии `isError`, ошибка доступна через свойство `error`
- `data` - если запрос находится в состоянии `success`, данные доступны через свойство `data`
- `isFetching` - в любом состоянии, если запрос находится на стадии выполнения (включая фоновый повторный запрос) `isFetching` будет иметь значение `true`

Для большинства запросов имеет смысл сначала проверять состояние `isLoading`, затем состояние `isError` и, наконец, использовать данные для рендеринга:

```js
function Todos() {
  const { isLoading, isError, data, error } = useQuery(
    'todos',
    fetchTodoList
  );

  if (isLoading) {
    return <span>Загрузка...</span>;
  }

  if (isError) {
    return <span>Ошибка: {error.message}</span>;
  }

  // На данном этапе мы можем предположить, что `isSuccess === true`
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

Вместо логических значений можно использовать состояние `status`:

```js
function Todos() {
  const { status, data, error } = useQuery(
    'todos',
    fetchTodoList
  );

  if (status === 'loading') {
    return <span>Загрузка...</span>;
  }

  if (status === 'error') {
    return <span>Ошибка: {error.message}</span>;
  }

  // status === 'success'
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

### Ключи запроса

`React Query` осуществляет кэширование запросов на основе ключей. Ключи могут быть любыми уникальными сериализуемыми значениями (строками, массивами, объектами и т.д.).

#### Строковые ключи

При передачи строки в качестве ключа запроса, она преобразуется в массив с единственным элементом. Данный формат может использоваться для:

- Общих ресурсов списка/индекса
- Неиерархических ресурсов

```js
// Список задач
useQuery('todos', ...) // queryKey === ['todos']
```

#### Ключи в виде массива

Когда для описания данных требуется больше информации, в качестве ключа запроса можно передать массив со строкой и любым количеством сериализуемых объектов. Такой формат может использоваться для:

- Иерархических или вложенных ресурсов
  - В этом случае, обычно, передается идентификатор, индекс или другой примитив для определения элемента
- Запросов с дополнительными параметрами
  - В этом случае, как правило, передается объект с дополнительными настройками

```js
// Конкретная задача
useQuery(['todo', 5], ...)
// queryKey === ['todo', 5]

// Конкретная задача в формате "превью"
useQuery(['todo', 5, { preview: true }], ...)
// queryKey === ['todo', 5, { preview: true }]

// Список выполненных задач
useQuery(['todos', { type: 'done' }], ...)
// queryKey === ['todos', { type: 'done' }]
```

#### Ключи хешируются детерменировано

Это означает, что порядок расположения ключей в объекте не имеет значения:

```js
// Эти запросы идентичны
useQuery(['todos', { status, page }], ...)
useQuery(['todos', { page, status }], ...)
useQuery(['todos', { page, status, other: undefined }], ...)
```

А порядок расположения ключей в массиве, напротив, имеет значение:

```js
// Эти запросы не идентичны
useQuery(['todos', status, page], ...)
useQuery(['todos', page, status], ...)
useQuery(['todos', undefined, page, status], ...)
```

_Обратите внимание_, что если функция запроса использует переменную, такая переменная должна включаться в ключ запроса:

```js
function Todos({ todoId }) {
  const result = useQuery(['todos', todoId], () =>
    fetchTodoById(todoId)
  );
}
```

### Функции запроса

Функция запроса может быть любой функцией, возвращающей промис, который, в свою очередь, должен либо разрешаться данными, либо выбрасывать исключение:

```js
useQuery(['todos', todoId], fetchTodoById);
useQuery(['todos', todoId], () => fetchTodoById(todoId));
useQuery(['todos', todoId], async () => {
  const data = await fetchTodoById(todoId);
  return data;
});
```

#### Обработка ошибок

При возникновении ошибки, функция запроса должна выбрасывать исключение, которое сохраняется в состоянии `error` запроса:

```js
const { data, error } = useQuery(
  ['todos', todoId],
  async () => {
    const response = await fetch('/todos/' + todoId);

    if (!response.ok) {
      throw new Error('Что-то пошло не так');
    }

    return await response.json();
  }
);
```

#### Переменные функции запроса

При необходимости, в функции запроса можно получить доступ к переменным, указанным в ключе запроса:

```js
function Todos({ status, page }) {
  const result = useQuery(
    ['todos', { status, page }],
    fetchTodoList
  );
}

// Получаем переменные `key`, `status` и `page` в функции запроса
function fetchTodoList({ queryKey }) {
  const [_key, { status, page }] = queryKey;
  return new Promise();
}
```

#### Использование объекта вместо параметров

Для настройки запроса можно использовать объект:

```js
import { useQuery } from 'react-query';

useQuery({
  queryKey: ['todo', 5],
  queryFn: fetchTodo,
  ...config,
});
```

### Параллельные запросы

"Параллельными" называются запросы, которые выполняются одновременно.

Когда количество запросов остается неизменным, для их параллельного выполнения достаточно указать несколько хуков `useQuery` или `useInfiniteQuery`:

```js
function App () {
  // Эти запросы будут выполнены одновременно
  const usersQuery = useQuery('users', fetchUsers)
  const teamsQuery = useQuery('teams', fetchTeams)
  const projectsQuery = useQuery('projects', fetchProjects)
  ...
}
```

Когда количество запросов меняется от рендеринга к рендерингу, для их параллельного выполнения следует использовать хук `useQueries`. Он принимает массив объектов с настройками запроса и возвращает массив результатов запросов:

```js
function App({ users }) {
  const userQueries = useQueries(
    users.map((user) => ({
      queryKey: ['user', user.id],
      queryFn: () => fetchUserById(user.id),
    }))
  );
}
```

### Зависимые запросы

Начало выполнения зависимых (или последовательных) запросов зависит от окончания выполнения предыдущих запросов. Для реализации последовательного выполнения запросов используется настройка `enabled`:

```js
// Получаем пользователя
const { data: user } = useQuery(
  ['user', email],
  getUserByEmail
);

const userId = user?.id;

// Затем получаем проекты пользователя
const { isIdle, data: projects } = useQuery(
  ['projects', userId],
  getProjectsByUser,
  {
    // Запрос не будет выполняться до получения userId
    enabled: !!userId,
  }
);
```

### Индикаторы получения данных в фоновом режиме

В большинстве случаев для отображения индикатора загрузки во время получения данных достаточно состояния `status === 'loading'`. Но иногда может потребоваться отображать индикатор для запроса, выполняющегося в фоновом режиме. Для этого запросы предоставляют дополнительное логическое значение `isFetching`:

```js
function Todos() {
  const {
    status,
    data: todos,
    error,
    isFetching,
  } = useQuery('todos', fetchTodos);

  return status === 'loading' ? (
    <span>Загрузка...</span>
  ) : status === 'error' ? (
    <span>Ошибка: {error.message}</span>
  ) : (
    <>
      {isFetching ? <p>Обновление...</p> : null}

      <div>
        {todos.map((todo) => (
          <Todo todo={todo} />
        ))}
      </div>
    </>
  );
}
```

Для отображения глобального индикатора загрузки при выполнении любого запроса (в том числе, в фоновом режиме) можно использовать хук `useIsFetching`:

```js
import { useIsFetching } from 'react-query';

function GlobalLoadingIndicator() {
  const isFetching = useIsFetching();

  return isFetching ? (
    <p>Запрос выполняется в фоновом режиме...</p>
  ) : null;
}
```

### Выполнение повторного запроса при фокусировке на окне

Если пользователь покидает приложение и возвращается к устаревшим данным, `React Query` автоматически запрашивает свежие данные в фоновом режиме. Это поведение можно отключить глобально или в отношении конкретного запроса с помощью настройки `refetchOnWindowFocus`.

Глобальное отключение:

```js
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      ...
    </QueryClientProvider>
  );
}
```

Отключение для запроса:

```js
useQuery('todos', fetchTodos, {
  refetchOnWindowFocus: false,
});
```

### Отключение/приостановка выполнения запросов

Для отключения автоматического выполнения запроса используется настройка `enabled`.

При установка значения данной настройки в `false`:

- Если запрос имеет кэшированные данные
  - Запрос будет инициализирован в состоянии `status === 'success'` или `isSuccess`
- Если запрос не имеет таких данных
  - Запрос бует запущен в состоянии `status === 'idle'` или `isIdle`
- Запрос не будет автоматически выполняться при монтировании
- Запрос не будет автоматически выполняться повторно при монтировании или появлении нового экземпляра
- Запрос будет игнорировать вызовы `invalidateQueries` и `refetchQueries` на клиенте запроса (query client), обычно, приводящие к выполнению повторного запроса
- Для ручного выполнения запроса может быть использован метод `refetch`

```js
function Todos() {
  const {
    isIdle,
    isLoading,
    isError,
    data,
    error,
    refetch,
    isFetching,
  } = useQuery('todos', fetchTodoList, {
    enabled: false,
  });

  return (
    <>
      <button onClick={() => refetch()}>
        Получить задачи
      </button>

      {isIdle ? (
        <span>Не готов...</span>
      ) : isLoading ? (
        <span>Загрузка...</span>
      ) : isError ? (
        <span>Ошибка: {error.message}</span>
      ) : (
        <>
          <ul>
            {data.map((todo) => (
              <li key={todo.id}>{todo.title}</li>
            ))}
          </ul>
          <div>
            {isFetching ? 'Выполнение запроса...' : null}
          </div>
        </>
      )}
    </>
  );
}
```

### Повторное выполнение провалившихся запросов

При провале запроса, автоматически выполняется 3 попытки его повторного выполнения.

Это поведение можно изменить как на глобальном уровне, так и на уровне конкретного запроса:

- Установка `retry: false` отключает повторы
- Установка `retry: 6` увеличивает количество повторов до 6
- Установка `retry: true` делает повторы бесконечными
- Устанока `retry: (failureCount, error) => {}` позволяет кастомизировать обработку провала

```js
import { useQuery } from 'react-query';

// Увеличиваем количество повторов для конкретного запроса
const result = useQuery(['todos', 1], fetchTodoListPage, {
  retry: 10, // Перед отображением ошибки будет выполнено 10 попыток
});
```

По умолчанию задержки между повторами составляют 2000, 4000 и 6000 мс. Это поведение можно изменить с помощью настройки `retryDelay`.

### Запросы для пагинации

Для реализации пагинации с помощью `React Query` достаточно включить информацию о странице в ключ запроса:

```js
const result = useQuery(['projects', page], fetchProjects);
```

Тем не менее, если запустить данный пример, то обнаружится, что UI перепрыгивает между состояниями `success` и `loading`, поскольку каждая новая страница расценивается как новый запрос.

Для решения этой проблемы `React Query` предоставляет `keepPreviousData`. Установка данной настройки в значение `true` приводит к следующему:

- Данные последнего успешного запроса остаются доступными во время запроса новых данных
- После получения новых данных старые `data` мягко ими заменяются
- `isPreviousData` позволяет определять, какие данные предоставляются текущим запросом

```js
function Todos() {
  const [page, setPage] = React.useState(0);

  const fetchProjects = (page = 0) =>
    fetch('/api/projects?page=' + page);

  const {
    isLoading,
    isError,
    error,
    data,
    isFetching,
    isPreviousData,
  } = useQuery(
    ['projects', page],
    () => fetchProjects(page),
    { keepPreviousData: true }
  );

  return (
    <div>
      {isLoading ? (
        <span>Загрузка...</span>
      ) : isError ? (
        <span>Ошибка: {error.message}</span>
      ) : (
        <div>
          {data.projects.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </div>
      )}
      <span>Текущая страница: {page + 1}</span>
      <button
        onClick={() =>
          setPage((old) => Math.max(old - 1, 0))
        }
        disabled={page === 0}
      >
        Предыдущая страница
      </button>{' '}
      <button
        onClick={() => {
          if (!isPreviousData && data.hasMore) {
            setPage((old) => old + 1);
          }
        }}
        // Отключаем кнопку, пока следующая страница не будет доступной
        disabled={isPreviousData || !data.hasMore}
      >
        Следующая страница
      </button>
      {isFetching ? <span>Загрузка...</span> : null}{' '}
    </div>
  );
}
```

`keepPreviousData` также работает с хуком `useInfiniteQuery`, что позволяет пользователям продолжать просмотр кэшированных данных при изменении ключей запроса.

### Бесконечные запросы

Для запроса бесконечных списков (например, "загрузить еще" или "бесконечная прокрутка") `React Query` предоставляет специальную версию `useQuery` - `useInfiniteQuery`.

Особенности использования `useInfiniteQuery`:

- `data` - объект, содержащий бесконечные данные:
  - `data.pages` - массив полученных страниц
  - `data.pageParams` - массив параметров страницы, использованных для запроса страниц
- Доступны функции `fetchNextPage` и `fetchPreviousPage`
- Доступны настройки `getNextPageParam` и `getPreviousPageParam`, позволяющие определить наличие данных для загрузки
- Доступно логическое значение `hasNextPage`. Если оно равняется `true`, `getNextPageParam` вернет значение, а не `undefined`
- Доступно логическое значение `hasPreviousPage`. Если оно равняется `true`, `getPreviousPageParam` вернет значение, а не `undefined`
- Доступны логические значения `isFetchingNextPage` и `isFetchingPreviousPage`, позволяющие различать состояние фонового обновления и состояние дополнительной загрузки

#### Примеры

Предположим, что у нас имеется API, возвращающий страницы с тремя `projects` за раз на основе индекса `cursor`, а также сам курсор, который может быть использован для получения следующей группы проектов:

```js
fetch('/api/projects?cursor=0');
// { data: [...], nextCursor: 3}
fetch('/api/projects?cursor=3');
// { data: [...], nextCursor: 6}
fetch('/api/projects?cursor=6');
// { data: [...], nextCursor: 9}
fetch('/api/projects?cursor=9');
// { data: [...] }
```

Располагая этими сведениями, мы можем реализовать UI "Загрузить еще" посредством:

- Ожидания, пока `useInfiniteQuery` выполнить запрос на получение первой порции данных по умолчанию
- Возвращения информации для следующего запроса в `getNextPageParam`
- Вызова функции `fetchNextPage`

```js
import { useInfiniteQuery } from 'react-query';

function Projects() {
  const fetchProjects = ({ pageParam = 0 }) =>
    fetch('/api/projects?cursor=' + pageParam);

  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery('projects', fetchProjects, {
    getNextPageParam: (lastPage, pages) =>
      lastPage.nextCursor,
  });

  return status === 'loading' ? (
    <p>Загрузка...</p>
  ) : status === 'error' ? (
    <p>Ошибка: {error.message}</p>
  ) : (
    <>
      {data.pages.map((group, i) => (
        <React.Fragment key={i}>
          {group.projects.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </React.Fragment>
      ))}
      <div>
        <button
          onClick={() => fetchNextPage()}
          disabled={!hasNextPage || isFetchingNextPage}
        >
          {isFetchingNextPage
            ? 'Загрузка дополнительных данных...'
            : hasNextPage
            ? 'Загрузить еще'
            : 'Больше нечего загружать'}
        </button>
      </div>
      <div>
        {isFetching && !isFetchingNextPage
          ? 'Выполнение запроса...'
          : null}
      </div>
    </>
  );
}
```

По умолчанию переменная, возвращаемая из `getNextPageParam`, передается в функцию запроса. Пользовательская переменная, переданная в `fetchNextPage`, перезаписывает дефолтную:

```js
function Projects() {
  const fetchProjects = ({ pageParam = 0 }) =>
    fetch('/api/projects?cursor=' + pageParam);

  const {
    status,
    data,
    isFetching,
    isFetchingNextPage,
    fetchNextPage,
    hasNextPage,
  } = useInfiniteQuery('projects', fetchProjects, {
    getNextPageParam: (lastPage, pages) =>
      lastPage.nextCursor,
  });

  // Передаем собственный параметр страницы
  const skipToCursor50 = () =>
    fetchNextPage({ pageParam: 50 });
}
```

Двунаправленный бесконечный список можно реализовать с помощью `getPreviousPageParam`, `fetchPreviousPage`, `hasPreviousPage` и `isFetchingPreviousPage`:

```js
useInfiniteQuery('projects', fetchProjects, {
  getNextPageParam: (lastPage, pages) =>
    lastPage.nextCursor,
  getPreviousPageParam: (firstPage, pages) =>
    firstPage.prevCursor,
});
```

Ручное удаление первой страницы:

```js
queryClient.setQueryData('projects', (data) => ({
  pages: data.pages.slice(1),
  pageParams: data.pageParams.slice(1),
}));
```

Ручное удаление значения из конкретной страницы:

```js
const newPagesArray = [];
oldPagesArray?.pages.forEach((page) => {
  const newData = page.data.filter(
    (val) => val.id !== updatedId
  );
  newPagesArray.push({
    data: newData,
    pageParam: page.pageParam,
  });
});
queryClient.setQueryData('projects', (data) => ({
  pages: newPagesArray,
  pageParams: data.pageParams,
}));
```

### Заменители данных запроса

Данные-заменители (placeholder data) позволяют запросам рендерить частичный ("фейковый") контент до получения настоящих данных в фоновом режиме. Это дает результат, похожий на использование настройки `initialData`, но при этом данные не сохраняются в кэше.

Существует два способа помещения данных в кэш для использования в качестве заменителей:

- Декларативный:
  - Предоставление запросу `placeholderData` для заполнения пустого кэша
- Императивный:
  - Предварительное или обычное получение данных с помощью `queryClient` и настройки `placeholderData`

#### Данные-заменители как значение

```js
function Todos() {
  const result = useQuery('todos', () => fetch('/todos'), {
    placeholderData: placeholderTodos,
  });
}
```

#### Данные-заменители как функция

Если процесс получения данных-заменителей является интенсивным или мы не хотим, чтобы он повторялся при каждом рендеринге, тогда в качестве значения `placeholderData` можно мемоизировать значение или передать мемоизированную функцию:

```js
function Todos() {
  const placeholderData = useMemo(
    () => generateFakeTodos(),
    []
  );
  const result = useQuery('todos', () => fetch('/todos'), {
    placeholderData,
  });
}
```

#### Данные-заменители из кэша

В некоторых случаях может потребоваться использовать в качестве данных-заменителей кэшированные результаты другого запроса. Хорошим примером такого случая является поиск кэшированных данных списка запросов поста в блоге для превью поста и последующее использование таких данных в качестве заменителя для запроса на получение конкретного поста:

```js
function Todo({ blogPostId }) {
  const result = useQuery(
    ['blogPost', blogPostId],
    () => fetch('/blogPosts'),
    {
      placeholderData: () => {
        // Используем превью blogPost из запроса 'blogPosts'
        // в качестве заменителя для запроса на получение данного blogPost
        return queryClient
          .getQueryData('blogPosts')
          ?.find((d) => d.id === blogPostId);
      },
    }
  );
}
```

### Начальные данные запроса

Существует несколько способов помещения в кэш начальных данных для запроса:

- Декларативный
  - Предоставление запросу `initialData` для заполнения пустого кэша
- Императивный
  - Предварительное получение данных с помощью `queryClient.prefetchQuery`
  - Ручное помещение данных в кэш с помощью `queryClient.setQueryData`

#### Использование `initialData` для подготовки запроса

Если в нашем приложении имеются начальные данные для запроса, мы можем передать их в запрос с помощью настройки `initialData`, минуя состояние начальной загрузки.

```js
function Todos() {
  const result = useQuery('todos', () => fetch('/todos'), {
    initialData: initialTodos,
  });
}
```

#### `staleTime` и `initialDataUpdatedAt`

По умолчанию `initialData` считаются свежими, как будто они были только что получены. Интерпретация начальных данных зависит от настройки `staleTime` (время устаревания):

- Если наблюдатель запроса был настроен с помощью `initialData` и не было указано `staleTime` (по умолчанию `staleTime` равняется `0`), запрос будет выполнен повторно незамедлительно при монтировании:

```js
function Todos() {
  // initialTodos будут отображены незамедлительно и сразу же будет выполнен повторный запрос на получения задач
  const result = useQuery('todos', () => fetch('/todos'), {
    initialData: initialTodos,
  });
}
```

- Если наблюдатель запроса был настроен с помощью `initialData` и было указано `staleTime` в количестве `1000` мс, данные будут считаться свежими на протяжении указанного времени:

```js
function Todos() {
  // initialTodos будут отображены незамедлительно, но не будут запрошены повторно до регистрации другого события спустя 1000 мс
  const result = useQuery('todos', () => fetch('/todos'), {
    initialData: initialTodos,
    staleTime: 1000,
  });
}
```

- Что если наши начальные данные не совсем свежие? Настройка `initialDataUpdatedAt` позволяет указывать время последнего обновления начальных данных в мс:

```js
function Todos() {
  // initialTodos отображаются незамедлительно, но не будут запрошены повторно до регистрации другого события спустя 1 мин после обновления начальных данных
  const result = useQuery('todos', () => fetch('/todos'), {
    initialData: initialTodos,
    staleTime: 60 * 1000 // 1 минута
    // Это могло произойти 10 секунд или 10 минут назад
    initialDataUpdatedAt: initialTodosUpdatedTimestamp // например, 1608412420052
  })
}
```

#### Функция начальных данных

Если процесс получения начальных данных для запроса является интенсивным или мы не хотим, чтобы он повторялся при каждом рендеринге, то в качестве значения `initialData` можно передать функцию. Эта функция будет выполнена только один раз при инициализации запроса:

```js
function Todos() {
  const result = useQuery('todos', () => fetch('/todos'), {
    initialData: () => {
      return getExpensiveTodos();
    },
  });
}
```

#### Начальные данные из кэша

В некоторых случаях в качестве начальных данных запроса можно использовать кэшированные результаты другого запросаю. Хорошим примером такого случая является поиск кэшированных данных запроса списка задач для конкретной задачи и использование этих данных в качестве начальных для запроса на получение конкретной задачи:

```js
function Todo({ todoId }) {
  const result = useQuery(
    ['todo', todoId],
    () => fetch('/todos'),
    {
      initialData: () => {
        // Используем задачу из запроса 'todos' в качестве начальных данных для запроса на получение данной задачи
        return queryClient
          .getQueryData('todos')
          ?.find((d) => d.id === todoId);
      },
    }
  );
}
```

#### Начальные данные из кэша с `initialDataUpdatedAt`

Для определения того, насколько начальные данные из кэша являются свежими и требуется ли выполнение повторного запроса используется настройка `initialDataUpdatedAt`:

```js
function Todo({ todoId }) {
  const result = useQuery(
    ['todo', todoId],
    () => fetch('/todos'),
    {
      initialData: () =>
        queryClient
          .getQueryData('todos')
          ?.find((d) => d.id === todoId),
      initialDataUpdatedAt: () =>
        queryClient.getQueryState('todos')?.dataUpdatedAt,
    }
  );
}
```

#### Условное получение начальных данных из кэша

Если мы не хотим использовать старый кэш в качестве начальных данных, то можем использовать метод `queryClient.getQueryState` для получения информации, включая `state.dataUpdatedAt`, которую можно использовать для определения того, достаточно ли свежими являются кэшированные данные:

```js
function Todo({ todoId }) {
  const result = useQuery(
    ['todo', todoId],
    () => fetch('/todos'),
    {
      initialData: () => {
        // Получаем состояние запроса
        const state = queryClient.getQueryState('todos');

        // Если запрос существует и содержит данные, не старше 10 секунд...
        if (
          state &&
          Date.now() - state.dataUpdatedAt <= 10 * 1000
        ) {
          // возвращаем конкретную задачу
          return state.data.find((d) => d.id === todoId);
        }

        // В противном случае, возвращаем undefined, что приводит к выполнению запроса на получение задачи от сервера
      },
    }
  );
}
```

### Предварительное получение данных

Если у нас есть возможность предугадывать желания пользователей на получение определенных данных, мы можем выполнить предварительный запрос на их получение и заблаговременно поместить их в кэш:

```js
const prefetchTodos = async () => {
  // Результаты данного запроса будут помещены в кэш как результаты обычного запроса
  await queryClient.prefetchQuery('todos', fetchTodos);
};
```

- Если данные для этого запроса уже имеются в кэше и не аннулированы, запрос выполнен не будет
- Если указано `staleTime`, например, `prefetchQuery('todos', fn, { staleTime: 5000 })` и данные старше, чем значение `staleTime`, запрос будет выполнен
- Если не имеется ни одного экземпляра `useQuery` для предварительного запроса, он будет удален и уничтожен сборщиком мусора по истечении времени, указанного в `cacheTime`

В качестве альтернативы, когда у нас уже имеются данные для запроса, нам не нужно выполнять предварительный запрос на их получение. Мы можем просто использовать метод `setQueryData` клиента запроса для добавления или обновления кэшированного результата запроса по ключу:

```js
queryClient.setQueryData('todos', todos);
```

## Мутации

В отличие от запросов, мутации, обычно, используются для создания/обновления/удаления данных или для выполнения побочных эффектов на сервере. Для этого используется хук `useMutation`.

Пример мутации, добавляющей новую задачу на сервер:

```js
function App() {
  const mutation = useMutation((newTodo) =>
    axios.post('/todos', newTodo)
  );

  return (
    <div>
      {mutation.isLoading ? (
        <p>Добавление задачи...</p>
      ) : (
        <>
          {mutation.isError ? (
            <p>Возникла ошибка: {mutation.error.message}</p>
          ) : null}

          {mutation.isSuccess ? (
            <p>Задача добавлена!</p>
          ) : null}

          <button
            onClick={() =>
              mutation.mutate({
                id: new Date(),
                title: 'Hello World',
              })
            }
          >
            Создать задачу
          </button>
        </>
      )}
    </div>
  );
}
```

Мутация может находиться в одном из следующих состояний:

- `isIdle` или `status === 'idle'` - мутация находится в режиме ожидания или на стадии обновления/сброса
- `isLoading` или `status === 'loading'` - мутация выполняется
- `isError` или `status === 'error'` - при выполнении мутации возникла ошибка
- `isSuccess` или `status === 'success'` - мутация выполнена успешно, данные доступны

Также, в зависимости от состояния мутации, доступна дополнительная информация:

- `error` - если мутация находится в состоянии `isError`, ошибка доступна через свойство `error`
  `data` - если мутация находится в состоянии `success`, данные доступны через свойство `data`

Функция `mutate` принимает переменную или объект.

При совместном использовании с настройкой `onSuccess`, методами `invalidateQueries` и `setQueryData` мутации становятся очень мощным инструментом.

_Обратите внимание_: функция `mutate` является асинхронной. Это означает, что мы не можем использовать ее в качестве колбека в обработчике событий. Для того, чтобы получить доступ к событию в `onSubmit` следует обернуть `mutate` в другую функцию.

```js
// Это работать не будет
const CreateTodo = () => {
  const mutation = useMutation((event) => {
    event.preventDefault();
    return fetch('/api', new FormData(event.target));
  });

  return <form onSubmit={mutation.mutate}>...</form>;
};

// А это будет
const CreateTodo = () => {
  const mutation = useMutation((formData) => {
    return fetch('/api', formData);
  });
  const onSubmit = (event) => {
    event.preventDefault();
    mutation.mutate(new FormData(event.target));
  };

  return <form onSubmit={onSubmit}>...</form>;
};
```

### Сброс состояния мутации

В некоторых случаях требуется очистить `error` или `data` мутации. Для этого используется функция `reset`:

```js
const createTodo = () => {
  const [title, setTitle] = useState('');
  const mutation = useMutation(createTodo);

  const handleSubmit = (e) => {
    e.preventDefault();
    mutation.mutate({ title });
  };

  return (
    <form onSubmit={handleSubmit}>
      {mutation.error && (
        <h5 onClick={() => mutation.reset()}>
          {mutation.error}
        </h5>
      )}
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
      />
      <br />
      <button type="submit">Создать задачу</button>
    </form>
  );
};
```

### Побочные эффекты мутации

`useMutation` содержит некоторые утилиты, позволяющие выполнять побочные эффекты на любой стадии жизненного цикла мутации. Это может быть полезным как для инвалидации и повторного выполнения запроса после мутации, так и для оптимистического обновления:

```js
useMutation(addTodo, {
  onMutate: (vars) => {
    // Произошла мутация

    // Опционально, можно вернуть контекст с данными, которые могут использоваться при отмене изменений, например
    return { id: 1 };
  },
  onError: (error, vars, context) => {
    // Возникла ошибка
    console.log(
      `Отмена изменений для оптимистического обновления с id ${context.id}`
    );
  },
  onSuccess: (data, vars, context) => {
    // Мутация выполнена успешно
  },
  onSettled: (data, error, vars, context) => {
    // Ошибка или успех... неважно
  },
});
```

При возвращении промиса из любого колбека, следующий промис будет ждать разрешения предыдущего:

```js
useMutation(addTodo, {
  onSuccess: async () => {
    console.log('Первый!');
  },
  onSettled: async () => {
    console.log('Второй!');
  },
});
```

При вызове `mutate` можно определять дополнительные колбеки, кроме тех, что определены в `useMutation`. Это может использоваться для запуска побочных эффектов, специфичных для компонента. Поддерживается перезапись таких методов, как `onSuccess`, `onError` и `onSettled`:

```js
useMutation(addTodo, {
  onSuccess: (data, vars, context) => {
    // Первый
  },
  onError: (error, vars, context) => {
    // Первый
  },
  onSettled: (data, error, vars, context) => {
    // Первый
  },
});

mutate(todo, {
  onSuccess: (data, vars, context) => {
    // Второй
  },
  onError: (error, vars, context) => {
    // Второй
  },
  onSettled: (data, error, vars, context) => {
    // Второй
  },
});
```

### Промисы

Для получения промиса, который разрешается в случае успеха или выбрасывает ошибку, следует использовать `mutateAsync` вместо `mutate`. Это может использоваться для создания композиции из побочных эффектов:

```js
const mutation = useMutation(addTodo);

try {
  const todo = await mutation.mutateAsync(todo);
  console.log(todo);
} catch (error) {
  console.error(error);
} finally {
  console.log('Выполнено');
}
```

### Повторное выполнение мутации

По умолчанию `React Query` не запускает повторное выполнение мутации при возникновении ошибки, но это можно изменить с помощью настройки `retry`:

```js
const mutation = useMutation(addTodo, {
  retry: 3,
});
```

### Сохранение мутации

Мутации могут быть сохранены в хранилище с помощью дегидрации/гидрации:

```js
const queryClient = new QueryClient();

// Определяем мутацию "addTodo"
queryClient.setMutationDefaults('addTodo', {
  mutationFn: addTodo,
  onMutate: async (vars) => {
    // Отмена текущих запросов для списка задач
    await queryClient.cancelQueries('todos');

    // Создаем оптимистическую задачу
    const optimisticTodo = {
      id: uuid(),
      title: vars.title,
    };

    // Добавляем ее в список
    queryClient.setQueryData('todos', (oldTodos) => [
      ...oldTodos,
      optimisticTodo,
    ]);

    // Возвращаем контекст с оптимистической задачей
    return { optimisticTodo };
  },
  onSuccess: (result, vars, context) => {
    // Заменяем оптимистическую задачу в списке результатом мутации
    queryClient.setQueryData('todos', (oldTodos) =>
      oldTodos.map((todo) =>
        todo.id === context.optimisticTodo.id
          ? result
          : todo
      )
    );
  },
  onError: (error, vars, context) => {
    // Удаляем оптимистическую задачу из списка
    queryClient.setQueryData('todos', (oldTodos) =>
      oldTodos.filter(
        (todo) => todo.id !== context.optimisticTodo.id
      )
    );
  },
  retry: 3,
});

// Запускаем мутацию в компоненте
const mutation = useMutation('addTodo');
mutation.mutate({ title: 'test' });

// Если выполнение мутации приостановлено, например, по причине отсутствия подключения устройства к сети,
// такая мутация может быть дегидратирована
const state = dehydrate(queryClient);

// Затем данная мутация может быть гидратирована при запуске приложения
hydrate(queryClient, state);

// Продолжаем выполнение приостановленной мутации
queryClient.resumePausedMutation();
```

## Инвалидация запросов

Метод `invalidateQueries` клиента запроса позволяет помечать запроса как устаревшие и потенциально выполнять повторное получение данных:

```js
// Инвалидация всех запросов, находящихся в кэше
queryClient.invalidateQueries();
// Инвалидация всех запросов, ключи которых начинаются с `todos`
queryClient.invalidateQueries('todos');
```

При инвалидации запроса с помощью `invalidateQueries` происходит две вещи:

- Он помечается как устаревший. При этом, настройка `staleTime` в `useQuery` и других хуках перезаписывается
- Если запрос отрендерен с помощью `useQuery` или других хуков, он выполняется повторно в фоновом режиме

### Поиск совпадений с помощью `invalidateQueries`

При использовании таких API, как `invalidateQueries` и `removeQueries` (и других, поддерживающих поиск частичного совпадения), можно осуществлять поиск совпадения по префиксу или искать полного совпадения.

В следующем примере мы используем префикс `todos` для инвалидации любых запросов, ключи которых начинаются с `todos`:

```js
import { useQuery, useQueryClient } from 'react-query';

// Получаем QueryClient из контекста
const queryClient = useQueryClient();

queryClient.invalidateQueries('todos');

// Оба запроса будут аннулированы
const todoListQuery = useQuery('todos', fetchTodoList);
const todoListQuery = useQuery(
  ['todos', { page: 1 }],
  fetchTodoList
);
```

В метод `invalidateQueries` можно передавать переменные для инвалидации конкретных запросов:

```js
queryClient.invalidateQueries(['todos', { type: 'done' }]);

// Данный запрос будет аннулирован
const todoListQuery = useQuery(
  ['todos', { type: 'done' }],
  fetchTodoList
);

// Данный запрос НЕ будет аннулирован
const todoListQuery = useQuery('todos', fetchTodoList);
```

API `invalidateQueries` является очень гибким, поэтому, если мы хотим аннулировать запросы с ключом `todos` без дополнительных переменных или ключей, то можем передать настройку `exact: true`:

```js
queryClient.invalidateQueries('todos', { exact: true });

// Данный запрос будет аннулирован
const todoListQuery = useQuery(['todos'], fetchTodoList);

// Данный запрос НЕ будет аннулирован
const todoListQuery = useQuery(
  ['todos', { type: 'done' }],
  fetchTodoList
);
```

В `invalidateQueries` также можно передавать функцию-предикат. Данная функция будет принимать каждый экземпляр `Query` и возвращать `true` или `false` в зависимости от условия:

```js
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === 'todos' &&
    query.queryKey[1]?.version >= 10,
});

// Данный запрос будет аннулирован
const todoListQuery = useQuery(
  ['todos', { version: 20 }],
  fetchTodoList
);

// Данный запрос также будет аннулирован
const todoListQuery = useQuery(
  ['todos', { version: 10 }],
  fetchTodoList
);

// Данный запрос НЕ будет аннулирован
const todoListQuery = useQuery(
  ['todos', { version: 5 }],
  fetchTodoList
);
```

## Инвалидация запросов с помощью мутаций

Инвалидация запросов - это половина успеха. Вторая половина - знать, когда их следует аннулировать. Обычно, при выполнении мутации, связанные с ней запросы нуждаются в инвалидации и повторном выполнении.

Предположим, что у нас имеется мутация для добавления новой задачи:

```js
const mutation = useMutation(postTodo);
```

После выполнении мутации `postTodo`, нам требуется аннулировать все запросы с ключом `todos` и выполнить их повторно для отображения новой задачи. Для этого мы можем использовать настройку `onSuccess` и функцию `invalidateQueries`:

```js
import { useMutation, useQueryClient } from 'react-query';

const queryClient = useQueryClient();

// Аннулируем все запросы с ключами `todos` и `reminders` после выполнения мутации
const mutation = useMutation(addTodo, {
  onSuccess: () => {
    queryClient.invalidateQueries('todos');
    queryClient.invalidateQueries('reminders');
  },
});
```

Инвалидация может выполняться в любом колбеке хука `useMutation`.

## Выполнение обновлений с помощью данных, возвращаемых мутацией

При выполнении мутаций, обновляющих объект на сервере, в ответ, как правило, возвращается обновленный объект. Вместо повторного выполнения запросов, возвращенный объект можно использовать для обновления существующего запроса с помощью метода `setQueryData` клиента запроса:

```js
const queryClient = useQueryClient();

const mutation = useMutation(editTodo, {
  onSuccess: (data) =>
    queryClient.setQueryData(['todo', { id: 5 }], data),
});

mutation.mutate({
  id: 5,
  name: 'Hello World',
});

// Данный запрос будет обновлен с помощью ответа от успешной мутации
const { status, data, error } = useQuery(
  ['todo', { id: 5 }],
  fetchTodoById
);
```

Логику `onSuccess` можно вынести в переиспользуемую мутацию (пользовательский хук):

```js
const useMutateTodo = () => {
  const queryClient = useQueryClient();

  return useMutation(editTodo, {
    // Обратите внимание: второй аргумент - это объект с переменными, который получает функция `mutate`
    onSuccess: (data, vars) => {
      queryClient.setQueryData(
        ['todo', { id: vars.id }],
        data
      );
    },
  });
};
```

## Оптимистические обновления

Обработчик `onMutate` позволяет вернуть значение, которое затем передается в обработчики `onError` и `onSettled` в качестве последнего аргумента. Это может быть использовано для передачи функции отмены.

### Обновление списка задач при добавлении новой задачи

```js
const queryClient = useQueryClient();

useMutation(updateTodo, {
  // При вызове мутации
  onMutate: async (newTodo) => {
    // Отменяем исходящие запросы (чтобы они не перезаписали оптимистическое обновление)
    await queryClient.cancelQueries('todos');

    // Снимок предыдущего значения
    const prevTodos = queryClient.getQueryData('todos');

    // Оптимистическое обновление
    queryClient.setQueryData('todos', (oldTodos) =>
      oldTodos.concat(newTodo)
    );

    // Возвращаем объект контекста с зафиксированным значением
    return { prevTodos };
  },
  // При провале мутации, используем контекст из onMutate для отмены мутации
  onError: (err, newTodo, context) => {
    queryClient.setQueryData('todos', context.prevTodos);
  },
  // Всегда выполняем повторный запрос
  onSettled: () => {
    queryClient.invalidateQueries('todos');
  },
});
```

### Обновление одной задачи

```js
useMutation(updateTodo, {
  // При вызове мутации
  onMutate: async (newTodo) => {
    // Отменяем исходящие запросы
    await queryClient.cancelQueries(['todos', newTodo.id]);

    // Снимок предыдущего значения
    const prevTodo = queryClient.getQueryData([
      'todos',
      newTodo.id,
    ]);

    // Оптимистическое обновление
    queryClient.setQueryData(
      ['todos', newTodo.id],
      newTodo
    );

    // Возвращаем контекст с предыдущим и новым значениями
    return { prevTodo, newTodo };
  },
  // При провале мутации, используем контекст
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(
      ['todos', context.newTodo.id],
      context.prevTodo
    );
  },
  // Всегда выполняем повторный запрос
  onSettled: (newTodo) => {
    queryClient.invalidateQueries(['todos', newTodo.id]);
  },
});
```

Вместо обработчиков `onError` и `onSuccess` можно использовать функцию `onSettled`:

```js
useMutation(updateTodo, {
  // ...
  onSettled: (newTodo, error, variables, context) => {
    if (error) {
      // ...
    }
  },
});
```

## Отмена запроса

`React Query` предоставляет общий способ отмены запросов с помощью токена отмены или другого API. Для этого необходимо добавить функцию `cancel` к промису, возвращаемому запросом, в котором реализуется логика отмены. Когда запрос становится неактивным или устаревшим, автоматически вызывается функция `promise.cancel`.

### С помощью `axios`

```js
import { CancelToken } from 'axios';

const query = useQuery('todos', () => {
  // Создаем новый токен отмены для данного запроса
  const source = CancelToken.source();

  const promise = axios('/todos', {
    cancelToken: source.token,
  });

  promise.cancel = () => {
    source.cancel('Запрос был отменен');
  };

  return promise;
});
```

### С помощью `fetch`

```js
const query = useQuery('todos', () => {
  // Создаем новый экземпляр `AbortController` для данного запроса
  const controller = new AbortController();
  // Получаем сигнал
  const signal = controller.signal;

  const promise = fetch('/todos', {
    method: 'GET',
    // Передаем сигнал в запрос
    signal,
  });

  // Отменяем запрос
  promise.cancel = () => controller.abort();

  return promise;
});
```

### Ручная отмена

В случае, когда, например, выполнение запроса занимает слишком много времени, мы можем предоставить пользователю возможность отмены такого запроса. Это можно реализовать с помощью вызова `queryClient.cancelQueries(key)`. Если `promise.cancel` является доступным, `React Query` отменит запрос:

```js
const [queryKey] = useState('todos');

const query = useQuery(queryKey, () => {
  const controller = new AbortController();
  const signal = controller.signal;

  const promise = fetch('/todos', {
    signal,
  });

  promise.cancel = () => controller.abort();

  return promise;
});

const queryClient = useQueryClient();

return (
  <button
    onClick={(e) => {
      e.preventDefault();
      queryClient.cancelQueries(queryKey);
    }}
  >
    Отменить
  </button>
);
```

## Фильтры запроса

Некоторые методы `React Query` принимают объект `QueryFilters`. Фильтры запроса - это объект с условиями для поиска совпадения с запросом:

```js
// Отмена всех запросов
await queryClient.cancelQueries();

// Удаление всех неактивных запросов
queryClient.removeQueries('posts', { inactive: true });

// Повторное выполнение всех активных запросов
await queryClient.refetchQueries({ active: true });

// Повторное выполнение всех запросов, ключи которых начинаются с `post`
await queryClient.refetchQueries('posts', { active: true });
```

Объект `QueryFilters` имеет следующие свойства:

- `exact` - поиск точного совпадения с ключом запроса
- `active`- поиск совпадения с активными/неактивными запросами
- `inactive` - поиск совпадения с неактивными/активными запросами
- `stale` - поиск совпадения с устаревшими/свежими запросами
- `fetching` - поиск совпадения с запросами, находящимися на стадии выполнения, или со всеми остальными запросами
- `predicate` - функция-предикат, вызываемая для каждого запроса и возвращающая `true` для найденных запросов
- `queryKey` - ключ для поиска совпадения

## Рендеринг на стороне сервера

`React Query` поддерживает два способа предварительного получения данных на сервере и передачи их клиенту запроса:

- Самостоятельное предварительное получение данных и их передача в качестве `initialData`
  - Подходит для быстрой настройки в простых случаях
  - Имеет некоторые ограничения
- Предварительное выполнение запроса на сервере, дегидрация кэша и регидрация на клиенте
  - Требует дополнительной настройки

### Использование `Next.js`

Рекомендуется использовать `Next.js`, который поддерживает две формы предварительного рендеринга:

- Генерация статического контента, статических сайтов (SSG)
- Рендеринг на стороне сервера (SSR)

#### Использование `initialData`

С помощью `getStaticProps` или `getServerSideProps` из `Next.js` можно передавать запрашиваемые данные в настройку `initialData` хука `useQuery`:

```js
export async function getStaticProps() {
  const posts = await getPosts();
  return { props: { posts } };
}

function Posts(props) {
  const { data } = useQuery('posts', getPosts, {
    initialData: props.posts,
  });

  // ...
}
```

Это минимальная настройка, подходящая для простых случаев, но она имеет некоторые ограничения:

- Вызов `useQuery` в глубоко вложенном компоненте предполагает передачу `initialData` в этот компонент
- Вызов `useQuery` в нескольких местах предполагает передачу `initialData` во все эти места
- Не существует способа определить, когда данные были получены на сервере, поэтому `dataUpdatedAt` и определение того, нуждается ли запрос в повторном выполнении, зависит от времени загрузки страницы

#### Использование гидрации

`React Query` поддерживает предварительное выполнение нескольких запросов на сервере в `Next.js` и дегидрацию этих запросов на клиенте. Это означает, что сервер может предварительно отрендерить необходимую разметку, которая доступна при загрузке страницы и, как только станет доступным JS, `React Query` обновит эти запросы, используя весь функционал библиотеки. Это включает в себя повторное выполнение устаревших запросов.

Для кэширования запросов на сервере и настройки гидрации необходимо сделать следующее:

- Создать новый экземпляр `QueryClient` внутри приложения и в экземпляре `ref`. Это позволяет предотвратить распределение данных между разными пользователями и запросами
- Обернуть компонент приложения в `QueryClientProvider` и передать ему экземпляр клиента
- Обернуть компонент приложения в `Hydrate` и передать ему проп `dehydrateState` из `pageProps`

```js
// App.js
import {
  QueryClient,
  QueryClientProvider,
} from 'react-query';
import { Hydrate } from 'react-query/hydration';

export default function App({ Component, pageProps }) {
  const queryClientRef = React.useRef();
  if (!queryClientRef.current) {
    queryClientRef.current = new QueryClient();
  }

  return (
    <QueryClientProvider client={queryClientRef.current}>
      <Hydrate state={pageProps.dehydratedState}>
        <Component {...pageProps} />
      </Hydrate>
    </QueryClientProvider>
  );
}
```

После этого необходимо выполнить предварительные запросы на получение данных с помощью `getStaticProps` (для SSG) или `getServerSideProps` (для SSR):

- Создаем новый экземпляр `QueryClient` для каждой запрашиваемой страницы. Это позволяет предотвратить распределение данных между разными пользователями и запросами
- Выполняем предварительный запрос с помощью клиентского метода `prefetchQuery` и ждем его завершения
- Используем `dehydrate` для дегидрации кэша запроса и передаем его странице через проп `dehydrateState`

```js
// pages/Posts.js
import { QueryClient, useQuery } from 'react-query';
import { dehydrate } from 'react-query/hydration';

export async function getStaticProps() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery('posts', getPosts);

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
}

function Posts() {
  // Хук `useQuery` можно использовать в любом дочернем компоненте страницы с постами
  const { data } = useQuery('posts', getPosts);

  // Данный запрос не был выполнен заранее на сервере,
  // и будет выполнен на клиенте, паттерны можно смешивать
  const { data: otherData } = useQuery('posts-2', getPosts);

  // ...
}
```

## Дефолтная функция запроса

При создании клиента запроса, ему можно передать функцию для выполнения запроса, которая будет использоваться по умолчанию. Для этого используется настройка `defaultOptions.queries.queryFn` экземпляра `queryClient`:

```js
// Определяем дефолтную функцию запроса, принимающую ключ запроса
const defaultQueryFn = async ({ queryKey }) => {
  const { data } = await axios(
    `https://jsonplaceholder.typicode.com${queryKey[0]}`
  );

  return data;
};

// Передаем ее клиенту запроса
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: defaultQueryFn,
    },
  },
});

const App = () => (
  <QueryClientProvider>
    <MyApp />
  </QueryClientProvider>
);

// После этого просто передаем ключ
function Posts() {
  const { status, data, error, isFetching } = useQuery(
    '/posts'
  );

  // ...
}

// Дефолтную функцию можно отключить
function Post({ postId }) {
  const { status, data, error, isFetching } = useQuery(
    `/posts/${postId}`,
    {
      enabled: !!postId,
    }
  );
}
```

Для перезаписи дефолтной функции достаточно передать в запрос другую функцию.

## Suspense

`React Query` может использоваться совместно с `React Suspense`. Для включения соответствующего режима следует установить значение настройки `suspense` в `true`.

Глобальная настройка:

```js
import {
  QueryClient,
  QueryClientProvider,
} from 'react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      suspense: true,
    },
  },
});

export default () => (
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
);
```

Настройка запроса:

```js
import { useQuery } from 'react-query';

useQuery(queryKey, queryFn, { suspense: true });
```

При включении данного режима состояния `status` и объект `error` становятся недоступными и заменяются соответствующими элементами `React.Suspense` (включая проп `fallback` и предохранители для перехвата ошибок).

Мутации также ведут себя несколько иначе. По умолчанию, вместо генерации переменной `error`, мутация выбрасывает исключение при следующем рендеринге использующего ее компонента для передачи ближайшему предохранителю. Для отключения такого поведения следует установить настройку `useErrorBoundary` в значение `false`. Для отключения всех ошибок следует установить настройку `throwOnError` также в значение `false`.

### Сброс ошибок

Ошибки запросов могут быть сброшены с помощью компонента `QueryErrorResetBoundary` или хука `useQueryErrorResetBoundary`.

При использовании компонента будут сброшены все ошибки, перехваченные предохранителем:

```js
import { QueryErrorResetBoundary } from 'react-query';
import { ErrorBoundary } from 'react-error-boundary';

const App = () => (
  <QueryErrorResetBoundary>
    {({ reset }) => (
      <ErrorBoundary
        onReset={reset}
        fallbackRender={({ resetErrorBoundary }) => (
          <div>
            Возникла ошибка!
            <Button onClick={() => resetErrorBoundary()}>
              Попробовать снова
            </Button>
          </div>
        )}
      >
        <Page />
      </ErrorBoundary>
    )}
  </QueryErrorResetBoundary>
);
```

При использовании хука будут сброшены ошибки из ближайшего `QueryErrorResetBoundary`. Если предохранитель не определен, ошибки будут сброшены глобально:

```js
import { useQueryErrorResetBoundary } from 'react-query';
import { ErrorBoundary } from 'react-error-boundary';

const App: React.FC = () => {
  const { reset } = useQueryErrorResetBoundary();
  return (
    <ErrorBoundary
      onReset={reset}
      fallbackRender={({ resetErrorBoundary }) => (
        <div>
          Возникла ошибка!
          <Button onClick={() => resetErrorBoundary()}>
            Попробовать снова
          </Button>
        </div>
      )}
    >
      <Page />
    </ErrorBoundary>
  );
};
```

## Плагины

_Обратите внимание_: данная технология является экспериментальной и может измениться в будущем.

### persistQueryClient

`persistQueryClient` - это утилита, позволяющая сохранять клиента запроса и его кэш для последующего использования. На сегодняшний день для этого доступен только один плагин - `createLocalStoragePersistor`.

Импорт:

```js
import { persistQueryClient } from 'react-query/persistQueryClient-experimental';
```

Пример использования.

Импортируем функцию `persistQueryClient` и передаем ей экземпляр `QueryClient` (с настройкой `cacheTime`) и интерфейс `Persistor` :

```js
import { persistQueryClient } from 'react-query/persistQueryClient-experimental'
import { createLocalStoragePersistor } from 'react-query/createLocalStoragePersistor-experimental'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      cacheTime: 1000 * 60 * 60 * 24 // 24 часа
    }
  }
})

const localStoragePersistor = createLocalStoragePersistor()

persistQueryClient({
  queryClient
  persistor: localStoragePersistor
})
```

#### API

- `persistQueryClient` - принимает экземпляр `QueryClient` и `persistor`

```js
persistQueryClient({ queryClient, persistor });
```

Настройки:

- `queryClient` - клиент запроса
- `persistor` - интерфейс для сохранения/восстановления кэша
- `maxAge:` - максимальный возраст кэша
- `buster` - уникальная строка, которая может использоваться для принудительной инвалидации кэша

Дефолтные настройки:

```js
{
  (maxAge = 1000 * 60 * 60 * 24), // 24 часа
    (buster = '');
}
```

### createLocalStoragePersistor

Импорт:

```js
import { createLocalStoragePersistor } from 'react-query/createLocalStoragePersistor-experimental';
```

Пример использования.

- Импортируем функцию `createLocalStoragePersistor`
- Создаем новый `localStoragePersistor`
- Передаем его в функцию `persistQueryClient`

```js
import { persistQueryClient } from 'react-query/persistQueryClient-experimental';
import { createLocalStoragePersistor } from 'react-query/createLocalStoragePersistor-experimental';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      cacheTime: 1000 * 60 * 60 * 24, // 24 часа
    },
  },
});

const localStoragePersistor = createLocalStoragePersistor();

persistQueryClient({
  queryClient,
  persistor: localStoragePersistor,
});
```

#### API

- `createLocalStoragePersistor` - принимает опциональный объект с настройками и возвращает `localStoragePersistor`, передаваемый в `persisteQueryClient`

```js
createLocalStoragePersistor(options?)
```

Настройки:

- `localStorageKey` - ключ, используемый для сохранения кэша
- `throttleTime` - "троттлинг" записи значений в хранилище

Дефолтные настройки:

```js
{
  (localStorageKey = `REACT_QUERY_OFFLINE_CACHE`),
    (throttleTime = 1000);
}
```

### broadcastQueryClient

`broadcastQueryClient` - это утилита для распространения (распределения, вещания) и синхронизации состояния клиента запроса между вкладками/окнами браузера одного источника (same origin).

Импорт:

```js
import { broadcastQueryClient } from 'react-query/broadcastQueryClient-experimental';
```

Пример использования.

Импортируем функцию `broadcastQueryClient` и передаем ей экземпляр `QueryClient` и, опционально, устанавливаем `broadcastChannel`:

```js
import { broadcastQueryClient } from 'react-query/broadcastQueryClient-experimental';

const queryClient = new QueryClient();

broadcastQueryClient({
  queryClient,
  broadcastChannel: 'my-app',
});
```

### API

- `broadcastQueryClient` - принимает экземпляр `QueryClient` и, опционально, `broadcastChannel`

```js
broadcastQueryClient({ queryClient, broadcastChannel });
```

Настройки:

- `queryClient` - клиент запроса
- `broadcastChannel` - уникальное название канала, которое будет использоваться для коммуникации между вкладками/окнами

Дефолтные настройки:

```js
{
  broadcastChannel = 'react-query',
}
```

## Основные части API

### useQuery

```js
const {
  data,
  dataUpdatedAt,
  error,
  errorUpdatedAt,
  failureCount,
  isError,
  isFetched,
  isFetchedAfterMount,
  isFetching,
  isIdle,
  isLoading,
  isLoadingError,
  isPlaceholderData,
  isPreviousData,
  isRefetchError,
  isStale,
  isSuccess,
  refetch,
  remove,
  status,
} = useQuery(queryKey, queryFn?, {
  cacheTime,
  enabled,
  initialData,
  initialDataUpdatedAt
  isDataEqual,
  keepPreviousData,
  notifyOnChangeProps,
  notifyOnChangePropsExclusions,
  onError,
  onSettled,
  onSuccess,
  queryKeyHashFn,
  refetchInterval,
  refetchIntervalInBackground,
  refetchOnMount,
  refetchOnReconnect,
  refetchOnWindowFocus,
  retry,
  retryOnMount,
  retryDelay,
  select,
  staleTime,
  structuralSharing,
  suspense,
  useErrorBoundary
})

// можно использовать синтаксис объекта
const result = useQuery({
  queryKey,
  queryFn,
  enabled
})
```

#### Основные настройки

- `queryKey | array` - уникальная строка или массив: ключ для идентификации запроса. Запрос автоматически обновляется при изменении данного ключа (если `enabled` не установлено в `false`)
- `queryFn` - функция для получения данных. Принимает объект `QueryFunctionContext` с переменной `queryKey`. Должна возвращать промис, разрешающийся данными или выбрасывающий исключение
- `enabled` - автоматическое выполнение запроса
- `retry ` - количесто попыток выполнения запроса (`true` - бесконечное количество попыток)
- `retryDelay` - функция, принимающее целое число `retryAttempt` и ошибку и возвращающая задержку, применяемую перед следующей попыткой выполнения запроса в мс. Функция `attempt => Math.min(attempt > 1 ? 2 ** attempt * 1000 : 1000, 30 * 1000)` применяет экспоненциальную задержку. Функция `attempt => attempt * 1000` применяет линейную задержку
- `staleTime` - время в мс, по истечении которого данные считаются устаревшими (`Infinity` - данные всегда будут считаться свежими)
- `cacheTime` - время в мс, в течение которого неиспользуемый/неактивный кэш сохраняется в памяти, после чего такой кэш уничтожается сборщиком мусора (`Infinity` - кэш никогда не будет собран)
- `refetchInterval` - периодическое повторное выполнение запроса (период определяется в мс)
- `notifyOnChangeProps` - компонент будет перерисовываться только при изменении указанных свойств. Например, при указании `['data', 'error']`, компонент будет перерисовываться только при изменении `data` или `error`. `tracked` означает отслеживание доступа к свойствам: компонент будет перерисовываться только при изменении одного из отслеживаемых свойств
- `onSuccess` - функция, вызываемая при успехе запроса. Получает объект `data`
- `onError` - функция, вызываемая при провале запроса. Получает объект `error`
- `onSettled` - функция, вызываемая как при успехе, так и при провале запроса. Получает объекты `data` и `error`
- `select` - функция, используемая для преобразования или выборки данных
- `initialData` - начальное значение для кэша запроса. Если значением является функция, она будет вызвана только один раз при инициализации запроса и должна синхронно возвращать начальные данные. Начальные данные считаются устаревшими, если не установлено `staleTime`
- `initialDataUpdatedAt` - время в мс последнего обновления `initialData`
- `placeholderData` - данные-заменители, используемые до получения настоящих данных и при отсутствии `initialData`
- `keepPreviousData` - сохранение предыдущих данных при запросе новых (при изменении ключа запроса)

### Основные возвращаемые значения

- `status` - возможные значения:
  - `idle` - возможно только при инициализации запроса с `enabled: false` и отсутствии начальных данных
  - `loading` - запрос находится на стадии выполнения
  - `error` - выполнение запроса завершилось ошибкой
  - `success` - выполнение запроса завершилось успешно. Соответствующее свойство `data` - это данные, полученные из успешного запроса, или, при `enabled: false`, начальные данные
- производные логические значения из переменной `status`:
  - `isIdle`
  - `isLoading`
  - `isError`
  - `isSuccess`
- `data` - последние успешно разрешенные данные для запроса (по умолчанию `undefined`)
- `error` - объект ошибки для запроса (по умолчанию `null`)
- `isFetching` - `true`, если запрос находится в процессе выполнения (включая выполнение в фоновом режиме)
- `refetch` - функция для ручного выполнения повторного запроса
  `remove` - функция для удаления запроса из кэша

### useQueries

Хук `useQueries` может использоваться для выполнения нескольких запросов:

```js
const results = useQueries([
  { queryKey: ['post', 1], queryFn: fetchPost },
  { queryKey: ['post', 2], queryFn: fetchPost },
]);
```

Хук принимает массив объектов с настройками запроса и возвращает массив результатов выполнения этих запросов.

### useInfiniteQuery

```js
const {
  fetchNextPage,
  fetchPreviousPage,
  hasNextPage,
  hasPreviousPage,
  isFetchingNextPage,
  isFetchingPreviousPage,
  ...result
} = useInfiniteQuery(
  queryKey,
  ({ pageParam = 1 }) => fetchPage(pageParam),
  {
    ...options,
    getNextPageParam: (lastPage, allPages) =>
      lastPage.nextCursor,
    getPreviousPageParam: (firstPage, allPages) =>
      firstPage.prevCursor,
  }
);
```

#### Настройки

Настройки для `useInfiniteQuery` идентичны настройкам `useQuery`, кроме следующих:

- `queryFn` - функция для получения данных. Принимает объект `QueryFunctionContext` с переменными `queryKey` и `pageParam`. Должна возвращать промис, разрешающийся данными или выбрасывающий исключение. Убедитесь, что возвращаете данные и `pageParam`, если собираетесь использовать последний
- `getNextPageParam` - при получении новых данных запросом, эта функция получает последнюю страницу бесконечного списка данных и массив всех страниц. Функция должна возвращать переменную, которая передается в качестве последнего опционального аргумента в функцию запроса. Возвращает `undefined` при отсутствии следующей страницы
- `getPreviousPageParam` - аналогична функции `getNextPageParam`, но в отношении предыдущей страницы

#### Основные возвращаемые значения

Значения, возвращаемые `useInfiniteQuery`, идентичны значениям, возвращаемым `useQuery`, кроме следующих:

- `data.pages` - массив, содержащий все страницы
- `data.pageParams` - массив, содержащий все параметры страницы
- `fetchNextPage` - функция, позволяющая получать следующую "страницу" результатов. `options.pageParam` позволяет вручную определять параметр страницы вместо использования `getNextPageParam`
- `fetchPreviousPage` - аналогична функции `fetchNextPage`, но в отношении предыдущей страницы
- `hasNextPage` - имеется ли следующая страница для получения
- `hasPreviousPage` - имеется ли предыдущая страница для получения

### useMutation

```js
const {
  data,
  error,
  isError,
  isIdle,
  isLoading,
  isPaused,
  isSuccess,
  mutate,
  mutateAsync,
  reset,
  status,
} = useMutation(mutationFn, {
  mutationKey,
  onError,
  onMutate,
  onSettled,
  onSuccess,
  useErrorBoundary,
});

mutate(variables, {
  onError,
  onSettled,
  onSuccess,
});
```

#### Настройки

- `mutationFn` - функция, выполняющая асинхронную задачу и возвращающая промис. Принимает объект `variables` для мутирования
- `mutationKey` - строка, которая может использовать для наследования дефолтных настроек с помощью `queryClient.setMutationDefaults`, а также для идентификации мутации в инструментах разработчика
- `onMutate` - функция, выполняемая перед мутацией, принимающая те же переменные. Может использоваться для выполнения оптимистических обновлений в надежде, что мутация завершится успешно. В случае провала мутации, возвращаемое этой функцией значение будет передано в `onError` и `onSettled`
- `onSuccess` - функция, выполняемая при успехе мутации. Если возвращается промис, он будет разрешен перед продолжением
- `onError` - функция, выполняемая при провале мутации. Если возвращается промис, он будет разрешен перед продолжением
- `onSettled` - функция, выполняемая как при успехе, так и при провале мутации. Если возвращается промис, он будет разрешен перед продолжением
- `retry` - количество попыток повторного выполнения мутации (`true` - бесконечное количество попыток)
- `retryDelay` - аналогична `retryDelay` запроса
- `useErrorBoundary` - если `true`, ошибки, возникшие при выполнении мутации, выбрасываются на стадии рендеринга и передаются ближайшему предохранителю

#### Возвращаемые значения

- `mutate` - функция мутации, которая может вызываться с переменными для запуска мутации и, опционально, для перезаписи настроек, переданных в `useMutation`
- `mutateAsync` - функция, аналогичная `mutate`, но возвращающая промис
- `status` - возможные значения:
  - `idle`
  - `loading`
  - `error`
  - `success`
- `isIdle`, `isLoading`, `isError`, `isSuccess` - производные логические значения `status`
- `data` - последние успешно разрешенные данные для запроса (по умолчанию `undefined`)
- `error` - объект ошибки для запроса (по умолчанию `null`)
- `reset` - функция для очистки внутреннего состояния мутации

### useIsFetching

`useIsFetching` - опциональный хук, возвращающий количество запросов, выполняемых в фоновом режиме. Может использоваться для глобальных индикаторов загрузки:

```js
import { useIsFetching } from 'react-query';
// Сколько всего запросов выполняется?
const isFetching = useIsFetching();
// Сколько выполняется запросов, ключи которых начинаются с `posts`?
const isFetchingPosts = useIsFetching(['posts']);
```

Настройки:

- `queryKey` - ключ запроса
- `filters` - фильтры запроса

Возвращается количество запросов, выполняемых в фоновом режиме.

### QueryClient

`QueryClient` может использоваться для взаимодействия с кэшем:

```js
import { QueryClient } from 'react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: Infinity,
    },
  },
});

await queryClient.prefetchQuery('posts', fetchPosts);
```

Доступные методы:

- `fetchQuery`
- `fetchInfiniteQuery`
- `prefetchQuery`
- `prefetchInfiniteQuery`
- `getQueryData`
- `setQueryData`
- `getQueryState`
- `invalidateQueries`
- `refetchQueries`
- `cancelQueries`
- `removeQueries`
- `resetQueries`
- `isFetching`
- `getDefaultOptions`
- `setDefaultOptions`
- `getQueryDefaults`
- `setQueryDefaults`
- `getMutationDefaults`
- `setMutationDefaults`
- `getQueryCache`
- `getMutationCache`
- `clear`

#### Настройки

- `queryCache` - кэш запроса, к которому подключен данный клиент
- `mutationCache` - кэш мутации, к которому подключен данный клиент
- `defaultOptions` - настройки по умочанию для всех запросов и мутаций

#### Методы

##### `fetchQuery`

`fetchQuery` - это асинхронный метод, который может использоваться для выполнения и кэширования запроса. Если нам не нужен результат запроса, то вместо `fetchQuery` можно использовать `prefetchQuery`.

Если запрос существует и данные не аннулированы или не старше указанного `staleTime`, то возвращаются данные из кэша. В противном случае, предпринимается попытка получения последних данных.

Разница между `fetchQuery` и `setQueryData` состоит в том, что `fetchQuery` является асинхронным и позволяет убедиться, что для данного запроса с помощью экземпляров `useQuery` не было создано дублирующихся запросов (для отрендеренного запроса во время получения данных).

```js
try {
  const data = await queryClient.fetchQuery(
    queryKey,
    queryFn
  );
} catch (error) {
  console.log(error);
}
```

С помощью `staleTime` можно обеспечить выполнение запроса, когда данные становятся устаревшими:

```js
try {
  const data = await queryClient.fetchQuery(
    queryKey,
    queryFn,
    {
      staleTime: 10000,
    }
  );
} catch (error) {
  console.log(error);
}
```

##### `fetchInfiniteQuery`

`fetchInfiniteQuery` аналогичен `fetchQuery`, но используется для выполнения и кэширования бесконечных запросов:

```js
try {
  const data = await queryClient.fetchInfiniteQuery(
    queryKey,
    queryFn
  );
  console.log(data.pages);
} catch (error) {
  console.log(error);
}
```

##### `prefetchQuery`

`prefetchQuery` - это асинхронный метод, который может использоваться для предварительного выполнения запроса, перед его выполнением или рендерингом с помощью `useQuery`. В целом, данный метод работает так же, как `fetchQuery`.

##### `prefetchInfiniteQuery`

`prefetchInfiniteQuery` аналогичен `prefetchQuery`, но используется для предварительного выполнения и кэширования бесконечных запросов.

##### `getQueryData`

`getQueryData` - синхронная функция для получения существующих кэшированных данных запроса. Если запроса не существует, возвращается `undefined`.

```js
const data = queryClient.getQueryData(queryKey);
```

##### `setQueryData`

`setQueryData` - синхронная функция для незамедлительного обновления кэшированных данных запроса. Если запроса не существует, он создается. Если запрос не используется хуком в течение дефолтных пяти минут `cacheTime`, он уничтожается сборщиком мусора.

```js
queryClient.setQueryData(queryKey, updater);
```

##### `getQueryState`

`getQueryState` - синхронная функция для получения существующего состояния запроса. Если запроса не существует, возвращается `undefined`

```js
const state = queryClient.getQueryState(queryKey);
console.log(state.dataUpdatedAt);
```

##### `invalidateQueries`

Метод `invalidateQueries` используется для инвалидации и обновления одного или нескольких запросов в кэше на основе их ключей или других доступных свойств/состояния. По умолчанию, все совпавшие запросы помечаются как аннулированные и активные запросы повторно выполняются в фоновом режиме.

- Для отключения повторного выполнения активных запросов используется настройка `refetchActive: false`
- Для повторного выполнения неактивных запросов используется настройка `refetchInactive: true`

```js
await queryClient.invalidateQueries(
  'posts',
  {
    exact,
    refetchActive: true,
    refetchInactive: false,
  },
  { throwOnError }
);
```

##### `refetchQueries`

Метод `refetchQueries` используется для повторного выполнения запросов на основе определенных условий.

```js
// Все запросы
await queryClient.refetchQueries();

// Устаревшие запросы
await queryClient.refetchQueries({ stale: true });

// Активные запросы, частично совпадающие с ключом
await queryClient.refetchQueries(['posts'], {
  active: true,
});

// Активные запросы, полностью совпадающие с ключом
await queryClient.refetchQueries(['posts', 1], {
  active: true,
  exact: true,
});
```

##### `cancelQueries`

Метод `cancelQueries` используется для отмены исходящих запросов на основе их ключей или любых других доступных свойств/состояния.

Это может быть полезным при выполнении оптимистических обновлений, когда мы не хотим, чтобы выполняющиеся запросы перезаписывали такие обновления.

```js
await queryClient.cancelQueries('posts', { exact: true });
```

##### `removeQueries`

Метод `removeQueries` используется для удаления запросов из кэша на основе их ключей или любых других доступных свойств/состояния.

```js
queryClient.removeQueries(queryKey, { exact: true });
```

##### `resetQueries`

Метод `resetQueries` используется для сброса запросов в кэше к их начальному состоянию на основе ключей или любых других доступных свойств/состояния.

Данный метод уведомляет подписчиков, в отличие от `clear`, который удаляет подписчиков. Если запрос имеет `initialData`, он будет сброшен к ним. Если запрос является активным, он будет выполнен повторно.

```js
queryClient.resetQueries(queryKey, { exact: true });
```

##### `isFetching`

Метод `isFetching` похож на хук `useIsFetching`. Он возвращает количество запросов, находящихся в процессе выполнения.

```js
if (queryClient.isFetching()) {
  console.log(
    'Как минимум, один запрос находится в процессе выполнения!'
  );
}
```

##### `getDefaultOptions` и `setDefaultOptions`

`getDefaultOptions` возвращает настройки по умолчанию, определенные при создании клиента или с помощью `setDefaultOptions`. `setDefaultOptions` позволяет динамически устанавливать дефолтные настройки для данного клиента.

##### `getQueryDefaults` и `setQueryDefaults`

Аналогичны `getDefaultOptions` и `setDefaultOptions`, но в отношении конкретных запросов.

##### `getMutationDefaults` и `setMutationDefaults`

Аналогичны `getDefaultOptions` и `setDefaultOptions`, но в отношении конкретных мутаций.

##### `getQueryCache` и `getMutationCache`

Данные методы возвращают, соответственно, кэш запроса и кэш мутации, к которым подключен данный клиент.

##### `clear`

Метод `clear` очищает все подключенные кэши.

```js
queryClient.clear();
```

### QueryClientProvider

`QueryClientProvider` используется для подключения и передачи `QueryClient` в приложение:

```js
import {
  QueryClient,
  QueryClientProvider,
} from 'react-query';

const queryClient = new QueryClient();

const App = () => (
  <QueryClientProvider client={queryClient}>
    ...
  </QueryClientProvider>
);
```

### useQueryClient

Хук `useQueryClient` возвращает текущий экземпляр `QueryClient`:

```js
import { useQueryClient } from 'react-query';

const queryClient = useQueryClient();
```

### QueryObserver

`QueryObserver` используется для наблюдения и переключения между запросами:

```js
const observer = new QueryObserver(queryClient, {
  queryKey: 'posts',
});

const unsubscribe = observer.subscribe((result) => {
  console.log(result);
  unsubscribe();
});
```

### InfiniteQueryObserver

`InfiniteQueryObserver` используется для наблюдения и переключения между бесконечными запросами:

```js
const observer = new InfiniteQueryObserver(queryClient, {
  queryKey: 'posts',
  queryFn: fetchPosts,
  getNextPageParam: (lastPage, allPages) =>
    lastPage.nextCursor,
  getPreviousPageParam: (firstPage, allPages) =>
    firstPage.prevCursor,
});

const unsubscribe = observer.subscribe((result) => {
  console.log(result);
  unsubscribe();
});
```

### QueriesObserver

`QueriesObserver` используется для наблюдения за несколькими запросами:

```js
const observer = new QueriesObserver(queryClient, [
  { queryKey: ['post', 1], queryFn: fetchPost },
  { queryKey: ['post', 2], queryFn: fetchPost },
]);

const unsubscribe = observer.subscribe((result) => {
  console.log(result);
  unsubscribe();
});
```
