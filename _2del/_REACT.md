## React (основные требования)

### Базовые концепции и архитектура

<details>
<summary>Объясните концепцию Virtual DOM в React и как она способствует производительности</summary>

> Отличный вопрос! Давайте разберем концепцию Virtual DOM подробно, поскольку это одна из ключевых особенностей React.

## Что такое Virtual DOM?

**Virtual DOM (виртуальный DOM)** — это легковесное представление реального DOM в памяти JavaScript. Это **JavaScript-объект**, который содержит структуру и свойства элементов интерфейса, но не может напрямую манипулировать элементами на экране[1][2].

```javascript
// Пример Virtual DOM объекта
{
  type: 'div',
  props: {
    id: 'container',
    className: 'main-content'
  },
  children: [
    {
      type: 'h1',
      props: { className: 'title' },
      children: ['Hello World']
    },
    {
      type: 'button',
      props: { onClick: handleClick },
      children: ['Click me']
    }
  ]
}
```

## Как работает Virtual DOM?

### Процесс работы состоит из трех фаз:

**1. Render Phase (Фаза рендеринга)**

- React создает новое представление Virtual DOM при изменении состояния компонента
- Вызывается метод `render()` для генерации нового дерева Virtual DOM[3]

**2. Diffing Algorithm (Алгоритм сравнения)**

- React сравнивает новое Virtual DOM дерево с предыдущим снимком
- Определяются минимальные изменения, необходимые для обновления[4][5]

**3. Commit Phase (Фаза применения)**

- Применяются только необходимые изменения к реальному DOM
- Обновления группируются (batching) для оптимизации производительности[3]

## Как Virtual DOM улучшает производительность?

### 1. **Минимизация операций с реальным DOM**

Реальный DOM содержит сотни свойств для каждого элемента (align, title, lang, translate, dir, hidden, accessKey и т.д.)[6]. Virtual DOM — это простой JavaScript объект, манипуляции с которым значительно быстрее[7].

### 2. **Batch Updates (Групповые обновления)**

React собирает множественные изменения состояния в одну операцию обновления, избегая ненужных перерендеров[8][9].

### 3. **Эффективный алгоритм сравнения**

React использует эвристический O(n) алгоритм вместо стандартного O(n³), основанный на двух допущениях[9]:

- Элементы разных типов создают разные деревья
- Разработчик может указать стабильные элементы через `key` prop

### 4. **Избежание полной перерисовки**

Вместо обновления всего DOM дерева, React обновляет только измененные узлы[1][2].

## Процесс Reconciliation (Согласования)

**Reconciliation** — это процесс, который React использует для определения изменений между Virtual DOM деревьями[10]:

```javascript
// Было
<div>
  <Counter value={1} />
</div>

// Стало
<div>
  <Counter value={2} />
</div>

// React обновит только prop 'value', сохранив компонент
```

### Правила сравнения:

1. **Разные типы элементов** → полная замена поддерева[11]
2. **Одинаковые типы** → обновление только измененных атрибутов[4]
3. **Списки элементов** → использование `key` для оптимального сопоставления[11]

## Практический пример

```javascript
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// При добавлении нового пользователя:
// Virtual DOM: создается новое дерево с дополнительным <li>
// Diffing: React определяет, что добавлен один элемент
// Real DOM: добавляется только новый <li>, остальные не трогаются
```

## Важные особенности

### React Fiber Architecture

Современный React использует **Fiber** — усовершенствованную архитектуру, которая[1]:

- Разбивает работу на небольшие управляемые части
- Позволяет приостанавливать и возобновлять задачи
- Приоритизирует важные обновления (клики, анимации)

### Ограничения оптимизации

Важно понимать, что Virtual DOM не всегда быстрее прямых манипуляций с DOM. При грамотном ванильном JavaScript можно достичь лучшей производительности[8]. Основная ценность Virtual DOM — в **упрощении разработки** сложных приложений.

## Ключевые выводы для собеседования:

1. **Virtual DOM** — это концепция оптимизации, а не волшебная пуля
2. Производительность достигается через **минимизацию операций с реальным DOM**
3. **Алгоритм reconciliation** — ключ к эффективности React
4. Понимание принципов работы помогает писать более оптимизированные компоненты
5. `key` prop критически важен для производительности списков

</details>

<details>
  <summary>В чем разница между классовыми и функциональными компонентами? Когда какие использовать?</summary>

Отличный вопрос! Это одна из ключевых тем для понимания эволюции React. Давайте разберем подробно.

## Основные различия

### **Классовые компоненты**

```javascript
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      loading: true,
    };
  }

  componentDidMount() {
    this.fetchUser();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  fetchUser = async () => {
    this.setState({ loading: true });
    const user = await api.getUser(this.props.userId);
    this.setState({ user, loading: false });
  };

  render() {
    const { user, loading } = this.state;

    if (loading) return <div>Loading...</div>;

    return (
      <div>
        <h1>{user.name}</h1>
        <p>{user.email}</p>
      </div>
    );
  }
}
```

### **Функциональные компоненты с хуками**

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  const fetchUser = useCallback(async () => {
    setLoading(true);
    const userData = await api.getUser(userId);
    setUser(userData);
    setLoading(false);
  }, [userId]);

  useEffect(() => {
    fetchUser();
  }, [fetchUser]);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Ключевые различия

### **1. Синтаксис и читаемость**

**Функциональные компоненты:**[1][2]

- Более простой и лаконичный синтаксис
- Меньше boilerplate кода
- Нет необходимости в `this` контексте
- Более функциональный подход к программированию

**Классовые компоненты:**[1]

- Более многословный синтаксис
- Необходимость в конструкторе и методах lifecycle
- Работа с `this` контекстом может вызывать путаницу

### **2. Управление состоянием**

**Функциональные:** `useState` хук[3]

```javascript
const [count, setCount] = useState(0);
const [user, setUser] = useState(null);
```

**Классовые:** `this.state` и `this.setState`[1]

```javascript
this.state = { count: 0, user: null };
this.setState({ count: this.state.count + 1 });
```

### **3. Lifecycle методы**

**Функциональные:** `useEffect` хук[2]

```javascript
// componentDidMount + componentDidUpdate
useEffect(() => {
  fetchData();
}, [dependency]);

// componentWillUnmount
useEffect(() => {
  return () => {
    cleanup();
  };
}, []);
```

**Классовые:** Отдельные методы[1]

```javascript
componentDidMount() { /* логика */ }
componentDidUpdate(prevProps) { /* логика */ }
componentWillUnmount() { /* очистка */ }
```

## Преимущества функциональных компонентов

### **1. Производительность**[2][3]

- Более легковесные по сравнению с классами
- Лучшая оптимизация со стороны React
- Встроенные возможности мемоизации (`useMemo`, `useCallback`)

### **2. Переиспользование логики**[2][4]

```javascript
// Кастомный хук для переиспользования логики
function useApi(endpoint) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(endpoint)
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      });
  }, [endpoint]);

  return { data, loading };
}

// Использование в разных компонентах
function UserList() {
  const { users, loading } = useApi("/api/users");
  // ...
}

function PostList() {
  const { posts, loading } = useApi("/api/posts");
  // ...
}
```

### **3. Упрощенное тестирование**[2][3]

- Функции легче тестировать, чем классы
- Меньше моков и сложной настройки
- Кастомные хуки можно тестировать изолированно

### **4. Лучшая совместимость с TypeScript**[5]

```typescript
interface UserProfileProps {
  userId: string;
  onUserLoad?: (user: User) => void;
}

const UserProfile: React.FC<UserProfileProps> = ({ userId, onUserLoad }) => {
  // TypeScript автоматически выводит типы для useState
  const [user, setUser] = useState<User | null>(null);
  // ...
};
```

## Когда использовать классовые компоненты

### **1. Error Boundaries** (пока единственный случай)[6][7]

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log("Error caught:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

### **2. Legacy кодовые базы**[5][8]

- При работе с существующими проектами
- Постепенная миграция может быть более практичной
- Нет необходимости переписывать работающий код

## Современные рекомендации (2025)

### **React команда официально рекомендует:**[9][8]

1. **Использовать функциональные компоненты** для всех новых проектов
2. **Хуки заменяют** большинство возможностей классовых компонентов
3. **Классовые компоненты не deprecated**, но не развиваются активно

### **В индустрии:**[10][11]

- 95%+ новых React проектов используют функциональные компоненты
- Большинство библиотек и туториалов фокусируются на хуках
- Компании предпочитают функциональный подход при найме

## Практические советы для собеседования

### **Что говорить:**

1. **"Предпочитаю функциональные компоненты"** с обоснованием
2. **Понимаю оба подхода** и могу работать с legacy кодом
3. **Знаю когда использовать классы** (Error Boundaries)
4. **Понимаю эволюцию React** и причины появления хуков

### **Пример хорошего ответа:**

> "В современной разработке я использую функциональные компоненты с хуками, так как они обеспечивают более чистый код, лучшую переиспользуемость логики через кастомные хуки и упрощают тестирование. Однако я понимаю классовые компоненты и могу с ними работать в legacy проектах. Классы все еще необходимы для Error Boundaries, но в остальном хуки покрывают все потребности современной разработки."

## Заключение

**Функциональные компоненты с хуками** — стандарт современной React разработки. Они проще, производительнее и более функциональны. **Классовые компоненты** остаются актуальными только для Error Boundaries и работы с legacy кодом[7][8].

</details>
<details>
  <summary>
  Что такое JSX и какие у него преимущества?
  </summary>

> Отличный фундаментальный вопрос! JSX — одна из ключевых особенностей React, поэтому важно понимать его глубоко.

## Что такое JSX?

**JSX (JavaScript XML)** — это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобный код прямо внутри JavaScript файлов[1][2]. JSX не является ни JavaScript, ни HTML — это специальный синтаксис, который **транспилируется** в обычный JavaScript.

### Простой пример:

```javascript
// JSX
const element = <h1>Hello, World!</h1>;

// Транспилируется в:
const element = React.createElement("h1", null, "Hello, World!");
```

## Как работает JSX?

### Процесс трансформации:

**1. Написание JSX**[1]

```javascript
const greeting = (
  <div className="container">
    <h1>Welcome, {userName}!</h1>
    <p>Today is {new Date().toLocaleDateString()}</p>
  </div>
);
```

**2. Транспиляция через Babel**[3]

```javascript
const greeting = React.createElement(
  "div",
  { className: "container" },
  React.createElement("h1", null, "Welcome, ", userName, "!"),
  React.createElement("p", null, "Today is ", new Date().toLocaleDateString())
);
```

**3. Создание Virtual DOM элементов**[1]
React использует эти `createElement` вызовы для построения Virtual DOM дерева.

## Основные преимущества JSX

### **1. Читаемость и интуитивность**[4][5]

**JSX:**

```javascript
function UserCard({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

**Без JSX:**

```javascript
function UserCard({ user }) {
  return React.createElement(
    "div",
    { className: "user-card" },
    React.createElement("img", { src: user.avatar, alt: user.name }),
    React.createElement("h2", null, user.name),
    React.createElement("p", null, user.email)
  );
}
```

### **2. Встроенная безопасность**[6][5]

JSX **автоматически экранирует** значения, защищая от XSS атак:

```javascript
const userInput = "<script>alert('hack')</script>";
const element = <div>{userInput}</div>; // Безопасно! Выведется как текст
```

### **3. Лучшие сообщения об ошибках**[4][6]

React предоставляет более точные и понятные ошибки при использовании JSX, что упрощает отладку.

### **4. Интеграция с JavaScript выражениями**[1][3]

```javascript
function TodoList({ todos, filter }) {
  return (
    <ul>
      {todos
        .filter(
          (todo) =>
            filter === "all" || todo.completed === (filter === "completed")
        )
        .map((todo) => (
          <li key={todo.id} className={todo.completed ? "completed" : ""}>
            {todo.text}
          </li>
        ))}
    </ul>
  );
}
```

### **5. Поддержка компонентной архитектуры**[7][8]

JSX естественно интегрируется с компонентным подходом React:

```javascript
function App() {
  return (
    <div>
      <Header />
      <Navigation />
      <MainContent>
        <Sidebar />
        <ArticleList />
      </MainContent>
      <Footer />
    </div>
  );
}
```

## Ключевые особенности JSX

### **1. Динамический контент через `{}`**[1][3]

```javascript
const name = "Alice";
const element = <h1>Hello, {name}!</h1>;
```

### **2. Атрибуты в camelCase**[3][5]

```javascript
// HTML: class, for, tabindex
// JSX: className, htmlFor, tabIndex
const input = <input className="form-field" tabIndex={1} />;
```

### **3. Самозакрывающиеся теги**[3]

```javascript
const image = <img src="photo.jpg" alt="Description" />;
const lineBreak = <br />;
```

### **4. Условный рендеринг**[3]

```javascript
const element = isLoggedIn ? (
  <WelcomeMessage user={currentUser} />
) : (
  <LoginForm />
);
```

### **5. Фрагменты**[3]

```javascript
// React.Fragment или короткий синтаксис <>
return (
  <>
    <h1>Title</h1>
    <p>Description</p>
  </>
);
```

## Современные возможности JSX

### **TypeScript интеграция**[9]

```typescript
interface Props {
  title: string;
  children: React.ReactNode;
}

const Modal: React.FC<Props> = ({ title, children }) => (
  <div className="modal">
    <h2>{title}</h2>
    {children}
  </div>
);
```

### **Inline стили как объекты**[3]

```javascript
const styles = {
  color: "blue",
  fontSize: "18px",
  fontWeight: "bold",
};

const element = <h1 style={styles}>Styled heading</h1>;
```

## Является ли JSX обязательным?

**Нет!** JSX не обязателен для React[7][5], но **настоятельно рекомендуется** потому что:

- Упрощает разработку сложных интерфейсов
- Улучшает читаемость кода
- Обеспечивает лучший developer experience
- Стал индустриальным стандартом

## Производительность

JSX **не влияет на runtime производительность**[10] — он полностью транспилируется на этапе сборки. Финальный JavaScript код работает так же быстро, как если бы вы писали `React.createElement` вручную.

## Практические советы для собеседования

### **Что подчеркнуть:**

1. **JSX — это синтаксический сахар** над `React.createElement`
2. **Безопасность по умолчанию** — автоматическое экранирование
3. **Лучший DX** (developer experience) — читаемость и отладка
4. **Интеграция с инструментами** — TypeScript, ESLint, Prettier
5. **Не обязателен**, но стал стандартом индустрии

### **Хороший ответ на собеседовании:**

> "JSX — это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобную разметку прямо в JavaScript коде. Основные преимущества: улучшенная читаемость, встроенная защита от XSS, лучшие сообщения об ошибках и естественная интеграция с компонентной архитектурой React. JSX транспилируется в React.createElement вызовы, поэтому не влияет на производительность runtime. Хотя он не обязателен, JSX стал индустриальным стандартом благодаря значительному улучшению developer experience."

</details>
<details>
  <summary>Объясните жизненный цикл компонентов React и современные хуки эквиваленты</summary>

> Отлично! Это фундаментальный вопрос, который демонстрирует глубокое понимание React. Давайте подробно разберем жизненный цикл компонентов и их современные эквиваленты в хуках.

## Фазы жизненного цикла React компонентов

React компоненты проходят через **три основные фазы**[1][2]:

### **1. Mounting (Монтирование)** 🏗️

Компонент создается и вставляется в DOM

### **2. Updating (Обновление)** 🔄

Компонент перерендеривается при изменении props или state

### **3. Unmounting (Размонтирование)** 🗑️

Компонент удаляется из DOM

## Классовые компоненты: Lifecycle методы

### **Mounting Phase**

```javascript
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    // 1. Инициализация state
    this.state = { user: null, loading: true };
    console.log("1. Constructor");
  }

  componentDidMount() {
    // 2. Вызывается ПОСЛЕ первого рендера
    console.log("2. componentDidMount");
    this.fetchUser();
  }

  render() {
    // 3. Возвращает JSX для отображения
    console.log("3. Render");
    return (
      <div>{this.state.loading ? "Loading..." : this.state.user?.name}</div>
    );
  }
}
```

### **Updating Phase**

```javascript
componentDidUpdate(prevProps, prevState) {
  // Вызывается ПОСЛЕ каждого обновления
  console.log('componentDidUpdate');

  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(); // Перезагрузка при изменении userId
  }
}

shouldComponentUpdate(nextProps, nextState) {
  // Оптимизация: можно предотвратить лишние рендеры
  return nextProps.userId !== this.props.userId;
}
```

### **Unmounting Phase**

```javascript
componentWillUnmount() {
  // Очистка ресурсов перед удалением
  console.log('componentWillUnmount');
  clearInterval(this.timer);
  this.subscription?.unsubscribe();
}
```

## Функциональные компоненты: Хуки эквиваленты

### **useEffect - универсальный заменитель**[3][4]

`useEffect` **объединяет три lifecycle метода** в одном хуке:

- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

### **1. componentDidMount эквивалент**

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // componentDidMount - выполняется ОДИН раз после монтирования
  useEffect(() => {
    console.log("Эквивалент componentDidMount");
    fetchUser();
  }, []); // Пустой массив зависимостей = выполнить один раз

  const fetchUser = async () => {
    setLoading(true);
    const userData = await api.getUser(userId);
    setUser(userData);
    setLoading(false);
  };

  return <div>{loading ? "Loading..." : user?.name}</div>;
}
```

### **2. componentDidUpdate эквивалент**[5]

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  // componentDidUpdate - выполняется при изменении userId
  useEffect(() => {
    console.log("Эквивалент componentDidUpdate");
    fetchUser();
  }, [userId]); // Выполнится при изменении userId

  // Можно отслеживать несколько зависимостей
  useEffect(() => {
    console.log("Выполнится при изменении userId или theme");
  }, [userId, theme]);

  return <div>{user?.name}</div>;
}
```

### **3. componentWillUnmount эквивалент**[6]

```javascript
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);

    // Cleanup функция = componentWillUnmount
    return () => {
      console.log("Эквивалент componentWillUnmount");
      clearInterval(interval);
    };
  }, []); // Cleanup выполнится при размонтировании

  return <div>Count: {count}</div>;
}
```

### **4. Комбинированный useEffect**[7]

```javascript
function DataComponent({ id }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    // componentDidMount + componentDidUpdate
    console.log("Загружаем данные для ID:", id);

    const subscription = api.subscribe(id, setData);

    // componentWillUnmount
    return () => {
      console.log("Очищаем подписку");
      subscription.unsubscribe();
    };
  }, [id]); // Перезапуск при изменении id

  return <div>{data?.title}</div>;
}
```

## Полное сравнение методов и хуков

### **Таблица соответствий**[4][5]

| **Классовый метод**       | **Хук эквивалент**                        | **Описание**                           |
| ------------------------- | ----------------------------------------- | -------------------------------------- |
| `constructor()`           | `useState()`                              | Инициализация состояния                |
| `componentDidMount()`     | `useEffect(() => {}, [])`                 | Выполняется после монтирования         |
| `componentDidUpdate()`    | `useEffect(() => {}, [deps])`             | Выполняется при изменении зависимостей |
| `componentWillUnmount()`  | `useEffect(() => { return cleanup }, [])` | Очистка перед размонтированием         |
| `shouldComponentUpdate()` | `React.memo()` + `useMemo()`              | Оптимизация рендеринга                 |

### **shouldComponentUpdate эквивалент**[6]

```javascript
// Классовый компонент
class ExpensiveComponent extends React.Component {
  shouldComponentUpdate(nextProps) {
    return nextProps.data !== this.props.data;
  }
}

// Функциональный эквивалент
const ExpensiveComponent = React.memo(
  ({ data }) => {
    return <div>{data}</div>;
  },
  (prevProps, nextProps) => {
    // true = не перерендеривать, false = перерендерить
    return prevProps.data === nextProps.data;
  }
);

// Или с useMemo для вычислений
function ExpensiveComponent({ data }) {
  const expensiveValue = useMemo(() => {
    return computeExpensiveValue(data);
  }, [data]); // Пересчитывается только при изменении data

  return <div>{expensiveValue}</div>;
}
```

## Важные отличия useEffect от lifecycle методов

### **1. Время выполнения**[8][9]

```javascript
// componentDidMount выполняется ДО paint браузера
componentDidMount() {
  // Блокирует отрисовку до завершения
  document.body.style.backgroundColor = 'red';
}

// useEffect выполняется ПОСЛЕ paint браузера
useEffect(() => {
  // НЕ блокирует отрисовку, выполняется асинхронно
  document.body.style.backgroundColor = 'red';
}, []);

// Для синхронного выполнения используйте useLayoutEffect
useLayoutEffect(() => {
  // Выполняется ДО paint, как componentDidMount
  document.body.style.backgroundColor = 'red';
}, []);
```

### **2. Множественные эффекты**[2]

```javascript
// В классах - один метод на всё
componentDidMount() {
  this.fetchUser();
  this.setupEventListeners();
  this.startTimer();
}

// С хуками - логическое разделение
function MyComponent() {
  // Отдельный эффект для пользователя
  useEffect(() => {
    fetchUser();
  }, []);

  // Отдельный эффект для событий
  useEffect(() => {
    const handleResize = () => {};
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  // Отдельный эффект для таймера
  useEffect(() => {
    const timer = setInterval(() => {}, 1000);
    return () => clearInterval(timer);
  }, []);
}
```

## Практический пример: рефакторинг с классов на хуки

### **Было (класс):**

```javascript
class UserDashboard extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      posts: [],
      loading: true,
      error: null,
    };
  }

  componentDidMount() {
    this.loadUserData();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.loadUserData();
    }
  }

  componentWillUnmount() {
    this.cleanup();
  }

  loadUserData = async () => {
    try {
      this.setState({ loading: true });
      const [user, posts] = await Promise.all([
        api.getUser(this.props.userId),
        api.getUserPosts(this.props.userId),
      ]);
      this.setState({ user, posts, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  cleanup = () => {
    // Очистка подписок
  };
}
```

### **Стало (хуки):**

```javascript
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const loadUserData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const [userData, postsData] = await Promise.all([
        api.getUser(userId),
        api.getUserPosts(userId),
      ]);
      setUser(userData);
      setPosts(postsData);
    } catch (err) {
      setError(err);
    } finally {
      setLoading(false);
    }
  }, [userId]);

  useEffect(() => {
    loadUserData();

    return () => {
      // Очистка подписок
    };
  }, [loadUserData]);

  // Остальной JSX...
}
```

## Советы для собеседования

### **Что подчеркнуть:**

1. **useEffect объединяет три lifecycle метода** в один хук
2. **Dependency array контролирует** когда эффект выполняется
3. **Cleanup функция** заменяет componentWillUnmount
4. **Множественные useEffect** для логического разделения
5. **Время выполнения отличается** - useEffect асинхронный

### **Хороший ответ:**

> "Жизненный цикл React компонентов состоит из трех фаз: mounting, updating и unmounting. В классовых компонентах для каждой фазы есть специальные методы: componentDidMount, componentDidUpdate, componentWillUnmount. В функциональных компонентах хук useEffect заменяет все эти методы. Dependency array определяет когда эффект выполняется: пустой массив [] эквивалентен componentDidMount, массив с зависимостями - componentDidUpdate, а cleanup функция - componentWillUnmount. Преимущество хуков в том, что можно создавать множественные эффекты для логического разделения кода."

</details>
<details>
  <summary>Как работает унидирекциональный поток данных в React?[2]</summary>

Отличный вопрос! Унидирекциональный поток данных — это **фундаментальная архитектурная концепция React**, которая отличает его от многих других фреймворков. Давайте разберем подробно.

## Что такое унидирекциональный поток данных?

**Унидирекциональный поток данных (Unidirectional Data Flow)** — это архитектурный паттерн, при котором данные движутся строго **в одном направлении**: от родительских компонентов к дочерним через **props**[1][2]. Дочерние компоненты **не могут напрямую изменять** данные, которые они получают.

### Визуальное представление:

```
┌─────────────┐
│   Parent    │  ──► State (источник истины)
│ Component   │
└─────┬───────┘
      │ props ↓ (данные вниз)
┌─────▼───────┐
│    Child    │  ──► Получает данные, но не может их изменить
│ Component   │
└─────────────┘
      │ callbacks ↑ (события вверх)
      ▼
┌─────────────┐
│   Parent    │  ──► Обновляет state
│ Component   │
└─────────────┘
```

## Принцип "Props Down, Events Up"

### **1. Props Down (Данные вниз)**[1][3]

```javascript
// Родительский компонент
function App() {
  const [user, setUser] = useState({ name: "Alice", age: 25 });
  const [posts, setPosts] = useState([]);

  return (
    <div>
      <UserProfile user={user} /> {/* Данные передаются вниз */}
      <PostList posts={posts} /> {/* через props */}
    </div>
  );
}

// Дочерний компонент
function UserProfile({ user }) {
  // user доступен только для чтения
  // Компонент НЕ МОЖЕТ изменить user напрямую
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

### **2. Events Up (События вверх)**[3][4]

```javascript
// Родительский компонент
function App() {
  const [count, setCount] = useState(0);

  // Функция-обработчик, которая может изменить состояние
  const incrementCount = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <Counter
        count={count} // Данные вниз
        onIncrement={incrementCount} // Callback вверх
      />
    </div>
  );
}

// Дочерний компонент
function Counter({ count, onIncrement }) {
  return (
    <div>
      <p>Count: {count}</p>
      {/* Дочерний компонент "просит" родителя изменить данные */}
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
}
```

## Практический пример: Todo приложение

```javascript
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", completed: false },
    { id: 2, text: "Build an app", completed: true },
  ]);

  // Все изменения состояния происходят в родителе
  const addTodo = (text) => {
    const newTodo = { id: Date.now(), text, completed: false };
    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  return (
    <div>
      {/* Данные передаются вниз, callbacks передаются вниз */}
      <AddTodoForm onAdd={addTodo} />
      <TodoList todos={todos} onToggle={toggleTodo} onDelete={deleteTodo} />
    </div>
  );
}

function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo} // Данные вниз
          onToggle={onToggle} // События вверх
          onDelete={onDelete} // События вверх
        />
      ))}
    </ul>
  );
}

function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li>
      <span
        style={{ textDecoration: todo.completed ? "line-through" : "none" }}
        onClick={() => onToggle(todo.id)} // Вызываем callback
      >
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
}
```

## Сравнение с двунаправленным потоком данных

### **React (Унидирекциональный)**[5][6]

```javascript
// Контролируемый компонент
function LoginForm() {
  const [email, setEmail] = useState("");

  return (
    <div>
      <input
        value={email} // State → UI
        onChange={(e) => setEmail(e.target.value)} // UI → State (через event)
      />
      <p>Email: {email}</p>
    </div>
  );
}
```

### **Angular/Vue (Двунаправленный)**[7]

```html
<!-- Vue.js - автоматическая синхронизация -->
<input v-model="email" />
<!-- Автоматическая двусторонняя связь -->
<p>{{ email }}</p>

<!-- Angular -->
<input [(ngModel)]="email" />
<!-- Two-way binding -->
<p>{{ email }}</p>
```

## Преимущества унидирекционального потока

### **1. Предсказуемость**[1][8]

```javascript
// Легко понять, откуда приходят данные и как они изменяются
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);

  // Единственное место, где изменяется user
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Ясно видно, что user может изменить только fetchUser
  return <UserProfile user={user} />;
}
```

### **2. Легкость отладки**[8][4]

- **Единый источник истины** — состояние хранится в одном месте
- **Четкий поток данных** — легко проследить, откуда пришли данные
- **Контролируемые изменения** — все изменения происходят через определенные функции

### **3. Тестируемость**[8]

```javascript
// Легко тестировать - просто передаем props и проверяем результат
test("TodoItem renders correctly", () => {
  const todo = { id: 1, text: "Test todo", completed: false };
  const onToggle = jest.fn();

  render(<TodoItem todo={todo} onToggle={onToggle} />);

  expect(screen.getByText("Test todo")).toBeInTheDocument();
});
```

## Lifting State Up (Поднятие состояния)

Когда несколько компонентов должны разделять одно состояние, его **поднимают к ближайшему общему родителю**[9][3]:

```javascript
// Плохо - дублированное состояние
function App() {
  return (
    <div>
      <TemperatureInput scale="c" /> {/* Свое состояние */}
      <TemperatureInput scale="f" /> {/* Свое состояние */}
    </div>
  );
}

// Хорошо - общее состояние в родителе
function App() {
  const [temperature, setTemperature] = useState("");
  const [scale, setScale] = useState("c");

  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={scale === "c" ? temperature : toCelsius(temperature)}
        onTemperatureChange={(temp) => {
          setTemperature(temp);
          setScale("c");
        }}
      />
      <TemperatureInput
        scale="f"
        temperature={scale === "f" ? temperature : toFahrenheit(temperature)}
        onTemperatureChange={(temp) => {
          setTemperature(temp);
          setScale("f");
        }}
      />
    </div>
  );
}
```

## Управление сложным состоянием

### **Context API для глубокой передачи данных**[1]

```javascript
// Создание контекста
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Header />
      <MainContent />
      <Footer />
    </UserContext.Provider>
  );
}

// Использование в глубоко вложенном компоненте
function UserProfile() {
  const { user } = useContext(UserContext); // Прямой доступ без prop drilling

  return <div>{user?.name}</div>;
}
```

### **Redux для централизованного управления**[1]

```javascript
// Все изменения через actions и reducers
const store = createStore(rootReducer);

// Компонент получает данные из store
function TodoList() {
  const todos = useSelector((state) => state.todos);
  const dispatch = useDispatch();

  const handleAddTodo = (text) => {
    dispatch({ type: "ADD_TODO", payload: text }); // Унидирекциональный поток
  };

  return (
    <div>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  );
}
```

## Важные принципы

### **1. Immutability (Неизменяемость)**[1]

```javascript
// Плохо - мутация состояния
const addTodo = (text) => {
  todos.push({ id: Date.now(), text }); // Прямая мутация
  setTodos(todos);
};

// Хорошо - создание нового состояния
const addTodo = (text) => {
  setTodos([...todos, { id: Date.now(), text }]); // Новый массив
};
```

### **2. Single Source of Truth**[8]

```javascript
// Состояние хранится в одном месте
function App() {
  const [appState, setAppState] = useState({
    user: null,
    todos: [],
    theme: "light",
  });

  // Все изменения через централизованные функции
  const updateUser = (user) => {
    setAppState((prev) => ({ ...prev, user }));
  };

  return (
    <StateContext.Provider value={{ appState, updateUser }}>
      <MainApp />
    </StateContext.Provider>
  );
}
```

## Советы для собеседования

### **Что подчеркнуть:**

1. **"Props down, events up"** — основной принцип React
2. **Предсказуемость и отладка** — главные преимущества
3. **Lifting state up** — решение для общего состояния
4. **Immutability** — важность неизменяемости данных
5. **Context API/Redux** — решения для сложных случаев

### **Хороший ответ:**

> "Унидирекциональный поток в React означает, что данные всегда текут в одном направлении: от родительских компонентов к дочерним через props. Дочерние компоненты не могут напрямую изменять полученные данные, вместо этого они используют callback функции для уведомления родителей о необходимости изменений. Это следует принципу 'props down, events up'. Основные преимущества: предсказуемость поведения, простота отладки, четкое разделение ответственности. Для сложных случаев используются Context API или внешние библиотеки управления состоянием как Redux, но они сохраняют унидирекциональность."

</details>

### React Hooks

<details>
  <summary>Объясните значение React Hooks в современной разработке и чем они отличаются от классовых компонентов[1]</summary>
</details>
<details>
  <summary>Расскажите про useState, useEffect, useContext и их практическое применение[2][3]</summary>
</details>
<details>
  <summary>Что такое useCallback, useMemo и React.memo? Когда их использовать?[1][6]</summary>
</details>
<details>
  <summary>Как создать кастомный хук? Приведите пример[3][6]</summary>
</details>
<details>
  <summary>Объясните правила хуков и почему их важно соблюдать[3]</summary>
</details>

### Управление состоянием

<details>
  <summary>Как вы управляете состоянием приложения в React? Какие преимущества использования библиотек управления состоянием?[1]</summary>
</details>
<details>
  <summary>Сравните useState vs useReducer - когда что использовать?[3][6]</summary>
</details>
<details>
  <summary>Когда использовать Context API vs Redux vs Zustand?[1][7][8]</summary>
</details>
<details>
  <summary>Как избежать prop drilling в React приложениях?[2][3]</summary>
</details>

### Производительность и оптимизация

<details>
  <summary>Опишите ваш опыт оптимизации производительности React приложений. Какие инструменты и стратегии вы использовали?[1][6]</summary>
</details>
<details>
  <summary>Как предотвратить ненужные ре-рендеры в React?[3][6][9]</summary>
</details>
<details>
  <summary>Что такое code splitting и как его реализовать в React?[1][6]</summary>
</details>
<details>
  <summary>Как реализовать lazy loading компонентов?[6][9]</summary>
</details>
<details>
  <summary>Объясните концепцию мемоизации в React и когда ее применять[1][6]</summary>
</details>

### Продвинутые паттерны

<details>
  <summary>Объясните паттерны Higher Order Components (HOC) и Render Props[6]</summary>
</details>
<details>
  <summary>Что такое Compound Components и когда их использовать?[6]</summary>
</details>
<details>
  <summary>Как работают React Portals и для чего они нужны?[3]</summary>
</details>
<details>
  <summary>Что такое React Fragments и зачем они нужны?[2][10]</summary>
</details>
