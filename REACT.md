## React (основные требования)

### Базовые концепции и архитектура

<details>
<summary>Объясните концепцию Virtual DOM в React и как она способствует производительности</summary>

# Объясните концепцию Virtual DOM в React и как она способствует производительности

Отличный вопрос! Давайте разберем концепцию Virtual DOM подробно, поскольку это одна из ключевых особенностей React.

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
  <summary>Что такое JSX и какие у него преимущества?</summary>

Отличный фундаментальный вопрос! JSX — одна из ключевых особенностей React, поэтому важно понимать его глубоко.

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
  <summary>Объясните жизненный цикл компонентов React и современные хуки эквиваленты[2][5]</summary>
</details>
<details>
  <summary>Как работает унидирекциональный поток данных в React?[2]</summary>
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
