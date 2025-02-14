# Предварительная выборка данных

## Данные страницы верхнего уровня

Есть много способов предварительно получить данные для SWR. Для запросов верхнего уровня настоятельно рекомендуется использовать [`rel="preload"`](https://developer.mozilla.org/ru/docs/Web/HTML/Preloading_content):

```html
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```

Просто поместите его в свой HTML `<head>`. Это просто, быстро и нативно.

Он выполнит предварительную выборку данных при загрузке HTML, даже до того, как начнется загрузка JavaScript. Все ваши входящие запросы на выборку с одним и тем же URL-адресом будут повторно использовать этот результат (включая, конечно, SWR).

## Программная предварительная выборка

SWR предоставляет API `preload` для предварительной загрузки ресурсов программным путем и сохранения результатов в кеше. `preload` принимает в качестве аргументов `key` и `fetcher`.

Вы можете вызывать `preload` даже вне React.

```jsx
import { useState } from 'react'
import useSWR, { preload } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

// Предварительно загрузите ресурс перед рендерингом компонента User ниже,
// это предотвращает потенциальные водопады в вашем приложении.
// Вы также можете начать предварительную загрузку при наведении на кнопку или ссылку.
preload('/api/user', fetcher)

function User() {
  const { data } = useSWR('/api/user', fetcher)
  ...
}

export default function App() {
  const [show, setShow] = useState(false)
  return (
    <div>
      <button onClick={() => setShow(true)}>Показать пользователя</button>
      {show ? <User /> : null}
    </div>
  )
}
```

В дереве рендеринга React `preload` также доступен для использования в обработчиках событий или эффектах.

```jsx
function App({ userId }) {
  const [show, setShow] = useState(false)

  // предзагрузка в эффектах
  useEffect(() => {
    preload('/api/user?id=' + userId, fetcher)
  }, [useId])

  return (
    <div>
      <button
        onClick={() => setShow(true)}
        {/* предзагрузка в колбэках событий */}
        onHover={() => preload('/api/user?id=' + userId, fetcher)}
      >
        Показать пользователя
      </button>
      {show ? <User /> : null}
    </div>
  )
}
```

Вместе с такими техниками, как [предзагрузка страниц](https://nextjs.org/docs/api-reference/next/router#routerprefetch) в Next.js, вы сможете мгновенно загружать как следующую страницу, так и данные.

Во избежание проблем с водопадом, в режиме Suspense вы должны использовать `preload`.

```jsx
import useSWR, { preload } from 'swr'

// следует вызывать перед рендерингом
preload('/api/user', fetcher);
preload('/api/movies', fetcher);

const Page = () => {
  // Приведенные ниже хуки useSWR приостановят рендеринг, но `preload` уже начал запросы к `/api/user` и `/api/movies`,
  // чтобы не возникало проблемы с водопадом.
  const { data: user } = useSWR('/api/user', fetcher, { suspense: true });
  const { data: movies } = useSWR('/api/movies', fetcher, { suspense: true });
  return (
    <div>
      <User user={user} />
      <Movies movies={movies} />
    </div>
  );
}
```

## Предварительное заполнение данных

Если вы хотите предварительно заполнить существующие данные в кеш SWR, вы можете использовать опцию `fallbackData`. Например:

```jsx
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```

Если SWR ещё не получил данные, этот хук вернёт `prefetchedData` в качестве запасного варианта.

Вы также можете настроить это для всех SWR хуков и множественных ключей с помощью `<SWRConfig>` и опцией `fallback`. Смотрите подробности в разделе [Next.js SSG и SSR](/docs/with-nextjs).
