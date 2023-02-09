# Динамические роуты

Для создания динамического роута в путь страницы необходимо добавить `[param]`.

Рассмотрим страницу `pages/post/[pid].js`:

```js
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { pid } = router.query;

  return <p>Пост: {pid}</p>;
}
```

Роуты `/post/1`, `/post/abc` и т. д. будут совпадать с `pages/post/[pid].js`. Совпавший параметр передается странице как параметр строки запроса, объединяясь с другими параметрами.

Например, для роута `/post/abc` объект `query` будет выглядеть так:

```js
{ "pid": "abc" }
```

А для роута `/post/abc?foo=bar` так:

```js
{ "pid": "abc", "foo": "bar" }
```

Параметры роута перезаписывают параметры строки запроса, поэтому объект query для роута `/post/abc?pid=123` будет выглядеть так:

```js
{ "pid": "abc" }
```

Для роутов с несколькими динамическими сегментами `query` формируется точно также. Например, страница `pages/post/[pid]/[cid].js` будет совпадать с роутом `/post/123/456`, а `query` будет выглядеть так:

```js
{ "pid": "123", "cid": "456" }
```

Навигация между динамическими роутами на стороне клиента обрабатывается с помощью `next/link`:

```js
import Link from 'next/link';

export default function Home() {
  return (
    <ul>
      <li>
        <Link href="/post/abc">
          Ведет на страницу `pages/post/[pid].js`
        </Link>
      </li>
      <li>
        <Link href="/post/abc?foo=bar">
          Также ведет на страницу `pages/post/[pid].js`
        </Link>
      </li>
      <li>
        <Link href="/post/123/456">
          <a>
            Ведет на страницу `pages/post/[pid]/[cid].js`
          </a>
        </Link>
      </li>
    </ul>
  );
}
```

## Перехват всех путей

Динамические роуты могут быть расширены для перехвата всех путей посредством добавления многоточия (`...`) в квадратные скобки. Например `pages/post/[...slug].js` будет совпадать с `/post/a`, `/post/a/b`, `/post/a/b/c` и т. д.

Обратите внимание: вместо `slug` можно использовать любое другое название, например, `[...param]`.

Совпавшие параметры передаются странице как параметры строки запроса (`slug` в данном случае) со значением в виде массива. Например, `query` для `/post/a` будет иметь такую форму:

```js
{ "slug": ["a"] }
```

А для `/post/a/b` такую:

```js
{ "slug": ["a", "b"] }
```

Роуты для перехвата всех путей могут быть опциональными — для этого параметр необходимо обернуть еще в одни квадратные скобки (`[[...slug]]`).

Например, `pages/post/[[...slug]].js` будет совпадать с `/post`, `/post/a`, `/post/a/b` и т. д.

Основное отличие между обычным и опциональным перехватчиками состоит в том, что с опциональным перехватчиком совпадает роут без параметров (`/post` в нашем случае).

Примеры объекта `query`:

```js
{ } // GET `/post` (пустой объект)
{ "slug": ["a"] } // `GET /post/a` (массив с одним элементом)
{ "slug": ["a", "b"] } // `GET /post/a/b` (массив с несколькими элементами)
```

Обратите внимание на следующие особенности:

- статические роуты имеют приоритет над динамическими, а динамические — над перехватчиками, например:
- `pages/post/create.js` — будет совпадать `/post/create`
- `pages/post/[pid].js` — будут совпадать `/post/1`, `/post/abc` и т. д., но не с `/post/create`
- `pages/post/[...slug].js` — будут совпадать с `/post/1/2`, `/post/a/b/c` и т. д., но не с `/post/create` и `/post/abc`
- страницы, обработанные с помощью автоматической статической оптимизации, будут гидратированы без параметров роутов, т. е. `query` будет пустым объектом (`{}`). После гидратации будет запущено обновление приложения для заполнения `query`.