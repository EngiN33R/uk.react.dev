---
title: Підтримуємо чистоту компонентів
---

<Intro>

Деякі функції в JavaScript є *чисті.* Чисті функції лише виконують обчислення й більше нічого. Пишучи компоненти як чисті функції, ви маєте змоги уникнути цілого ряду збиваючих з пантелику помилок та непередбачуваних поведінок по мірі розвитку вашого коду. Але щоб мати змоги повноцінно користатися цими перевагами, є деякі правила, яких треба дотримуватися.

</Intro>

<YouWillLearn>

* Що таке чистота і як вона допомагає уникати помилок
* Як дотримуватися чистоти компонентів, виносячи зміни з фази рендеру
* Як користатися Strict Mode для пошуку помилок у компонентах

</YouWillLearn>

## Чистота: Компоненти як формули {/*purity-components-as-formulas*/}

В інформатиці (зокрема у світі функціонального програмування), [чиста функція](https://uk.wikipedia.org/wiki/%D0%A7%D0%B8%D1%81%D1%82%D0%B0_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D1%96%D1%8F) є функція з такими характеристиками:

* **Вона не сує носа в чужі справи.** Вона не зміняє ніякі об'екти або змінні, що існували перед тим як вона була викликана.
* **Однакові входи, однакові виходи.** Якщо дані однакові входи, чиста функція завжди поверне однаковий результат.

З одним прикладом чистих функцій ви мабуть уже знайомі: формули в математиці.

Подивимося на таку формулу: <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>.

Якщо <Math><MathI>x</MathI> = 2</Math>, тоді <Math><MathI>y</MathI> = 4</Math>. Завжди. 

Якщо <Math><MathI>x</MathI> = 3</Math>, тоді <Math><MathI>y</MathI> = 6</Math>. Завжди. 

Якщо <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> не рівнятиметься іноді <Math>9</Math>, або <Math>–1</Math>, або <Math>2.5</Math>, залежно від часу доби чи стану фондового ринку. 

Якщо <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> та <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> _завжди_ рівнятиметься <Math>6</Math>. 

Якщо ми перетворимо цю формулу на функцію JavaScript, вона виглядатиме так:

```js
function double(number) {
  return 2 * number;
}
```

У прикладі вище, `double` є **чиста функція.** Якщо їй передати `3`, вона поверне `6`. Завжди.

React побудований саме на цій концепції. **React припускає, що кожен написаний компонент є чиста функція.** Це означає, що компоненти React, які ви пишете, мушуть повертати однаковий JSX, якщо дані однакові входи:

<Sandpack>

```js App.js
function Recipe({ drinkers }) {
  return (
    <ol>    
      <li>Закип'ятити {drinkers} стаканів води.</li>
      <li>Додати {drinkers} ложок чаю і {0.5 * drinkers} ложок спецій.</li>
      <li>Додати {0.5 * drinkers} стаканів молока й сахару на смак.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Рецепт Spiced Chai</h1>
      <h2>Для двох</h2>
      <Recipe drinkers={2} />
      <h2>Для групи</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

</Sandpack>

Коли до `Recipe` передається `drinkers={2}`, компонент повертає JSX, що містить текст `2 cups of water`. Завжди. 

Якщо передається `drinkers={4}`, він повертає JSX, що містить текст `4 cups of water`. Завжди.

Так само як і математична формула. 

Про компоненти можна міркувати як про рецепти: якщо їх дотримуватись і не додавати нових інгредієнтів під час процесу приготування, кожного разу вийде та сама страва. Саме ця «страва» є JSX, який компонент передає до React для [рендеру.](/learn/render-and-commit)

<Illustration src="/images/docs/illustrations/i_puritea-recipe.png" alt="Рецепт чаю для x людей: взяти x стаканів води, додати x ложок чаю, 0.5x ложок спецій та 0.5x стаканів молока" />

## Побічні ефекти: (не)навмисні наслідки {/*side-effects-unintended-consequences*/}

Процес рендеру React завжди мусить бути чистий. Компоненти мушуть лише *повертати* JSX і не *міняти* об'екти або змінні, що існували перед рендером — це зробить їх нечистими!

Ось компонент, що порушує це правило:

<Sandpack>

```js
let guest = 0;

function Cup() {
  // Погано: міняємо існуючу заздалегідь змінну!
  guest = guest + 1;
  return <h2>Чашка чаю для гостя №{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

Цей компонент читає та переписує змінну `guest`, що була оголошена зовні. Це означає, що **з кожним викликом компоненту він повертатиме різний JSX!** Більш того, якщо _інші_ компоненти читатимуть `guest`, вони також повертатимуть різний JSX залежно від того, коли відбувся їх рендер! Це непередбачувано.

Повертаючись до формули <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>, тепер навіть якщо <Math><MathI>x</MathI> = 2</Math>, ми не можемо бути впевненими, що <Math><MathI>y</MathI> = 4</Math>. Наші тести можуть провалитися, наші користувачі будуть збиті з пантелику, літаки попадають з неба — очевидно, як саме це призведе до заплутаних помилок!

Цей компонент можна виправити, якщо [передавати `guest` як проп](/learn/passing-props-to-a-component):

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Чашка чаю для гостя №{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

Тепер компонент є чистий, бо JSX, що він повертає, залежить лише від пропу `guest`.

Взагалі не слід покладатися на фіксований порядок рендеру компонентів. Немає різниці, чи викликається <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> перед або після <Math><MathI>y</MathI> = 5<MathI>x</MathI></Math>: обидві формули дадуть результат незалежно одна від одної. Таким самим чином кожному компоненту під час рендеру слід «думати про себе» й ні залежити від інших компонентів, ні намагатися координувати свій рендер з ними. Рендер — як екзамен у школі: кожен компонент має обчислювати свій JSX сам!

<DeepDive>

#### Виявлення нечистих обчислювань за допомогою StrictMode {/*detecting-impure-calculations-with-strict-mode*/}

Хоча ви, можливо, ще не використовували їх усі, у React є три види входів, які можна зчитувати під час рендеру: [пропси](/learn/passing-props-to-a-component), [стан](/learn/state-a-components-memory) та [контекст.](/learn/passing-data-deeply-with-context) До цих входів слід завжди ставитися як до даних лише для читання.

Якщо у відповідь на введення користувача треба щось *змінити*, слід [встановити стан](/learn/state-a-components-memory), замість того щоб переписувати змінну. Не слід змінювати вже існуючі змінні та об'екти під час рендеру.

React пропонує «строгий режим» (Strict Mode), у якому він викликає функцію кожного компоненту двічі під час розробки. **Викликаючи функції компонентів двічі, Strict Mode допомагає знайти компоненти, що порушують ці правила.**

Зверніть увагу, як у первісному прикладі відображалися пункти «гість №2», «гість №4» та «гість №6» замість «гість №1», «гість №2» та «гість №3». Первісна функція була нечиста, тому подвійний виклик її ламає. Але виправлений чистий варіант працює навіть якщо функція кожного разу викликається двічі. **Чисті функції виконують лише обчислення, тому подвійний виклик нічого не змінить** — так само як подвійний виклик `double(2)` не змінює повертаємий результат, а подвійне вирішення <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> не змінює значення <MathI>y</MathI>. Однакові входи, однакові виходи. Завжди.

Strict Mode не діє в продакшні, тому він не сповільнюватиме застосунок для користувачів. Щоб увімкнути Strict Mode, можна загорнути кореневий компонент у вузол `<React.StrictMode>`. Деякі фреймворки вже роблять це із самого початку.

</DeepDive>

### Локальні мутації: Невеличка таємниця компоненту {/*local-mutation-your-components-little-secret*/}

У прикладі вище проблема полягала у тому, що компонент міняв *вже існуючу* змінну під час рендеру. Це часто зветься **«мутацією»**, щоб звучало трішки страшніше. Чисті функції не мутують змінні поза межами функції та об'екти, що були створені перед викликом — це робить їх нечистими!

Однак **міняти змінні та об'екти, що ви *тільки що* створили під час рендеру, не заборонено.** У цьому прикладі створюється порожній масив `[]`, що призначується до змінної `cups`, а потім до нього додаються за допомогою `push` дванадцять чашок:

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Чашка чаю для гостя №{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

</Sandpack>

Якщо б змінна `cups` або масив `[]` були створені поза функцією `TeaGathering`, це була б величезна проблема! Виконувалася б зміна *вже існуючого* об'екту через додавання до масиву нових елементів.

Однак проблеми немає, адже змінна та масив створені *під час самого рендеру* всередині `TeaGathering`. Жодний рядок коду поза `TeaGathering` не знатиме, що там відбулося. Це називається **локальною мутацією** — свого роду невеличка таємниця компоненту.

## Де _можна_ викликати побічні ефекти {/*where-you-_can_-cause-side-effects*/}

Хоча функціональне програмування дуже сильно покладається на концепцію чистоти, щось десь _має_ зазнати змін. Адже це і є суть програмування! Ці зміни — оновлення екрану, початок анімації, зміна даних — називаються **побічними ефектами.** Вони є те, що відбувається _«на стороні»_, а не під час рендеру.

У React **побічним ефектам, як правило, місце в [обробниках подій.](/learn/responding-to-events)** Обробники подій є функції, які React викликає, коли виконується якась дія — наприклад, коли користувач тисне на кнопку. І хоча обробники подій визначаються *всередині* компоненту, вони не виконуються *під час* рендеру! **Тому обробники подій не мають бути чистими.**

Якщо жоден з обробників подій не пасує й більше нема варіантів, куди можна додати потрібний побічний ефект, його можна додати до повертаємого JSX за допомогою виклику [`useEffect`](/reference/react/useEffect) в компоненті. Так React зможе виконати цей ефект пізніше, після рендеру, коли побічні ефекти не заборонені. **Однак цей підхід слід застосовувати як останній засіб.**

Коли це можливо, намагайтесь виразити логіку одним лише рендером. Цей підхід спроможен вирішити на диво багато різних завданнь!

<DeepDive>

#### Чому для React важлива чистота? {/*why-does-react-care-about-purity*/}

Для написання чистих функцій потрібно сформувати звичку і дотримуватися дісципліни. Але вони також відкривають чудові можливості:

* Компоненти зможуть працювати в іншому середовищі — наприклад, на сервері! Оскільки вони повертають однаковий результат для однакових входів, один компонент може обробляти одразу декілька запитів користувачів.
* За допомогою [пропуску рендеру](/reference/react/memo) компонентів, для яких не змінилися входи, можна покращити їх продуктивність. Це безпечно, адже чисті функції завжди повертають однакові результати, тому їх можна безпечно кешувати.
* Якщо деякі дані змінюються під час рендеру глибокого дерева компонентів, React може почати рендер знову, не витрачаючи зайвого часу на завершення застарілого рендеру. Завдяки чистоті рендер можна безпечно припинити в будь-який час.

Кожна нова особливість React, яку ми розроблюємо, так чи інакше користується чистотою. Від завантаження даних до анімацій і продуктивності, дотримання концепції чистоти компонентів розкриває всю силу парадигми React.

</DeepDive>

<Recap>

* Компонент має бути чистим, тобто:
  * **Не сувати носа у чужі справи.** Він не має зміняти ніякі об'екти або змінні, що існували перед рендером.
  * **Однакові входи, однакові виходи.** Якщо дані однакові входи, компонент завжди має повертати однаковий JSX.
* Рендер може статися будь-коли, тому компоненти мають не залежити від порядку рендера один одного.
* Не слід мутувати будь-які з входів, які компоненти використовують для рендеру. Сюди входять пропси, стан та контекст. Для оновлення екрана слід [встановлювати стан](/learn/state-a-components-memory) замість мутації вже існуючих об'єктів.
* Слід намагатися виразити логіку компоненту безпосередньо в повертаємому JSX. Коли щось потрібно «змінити», зазвичай це слід зробити в обробнику подій. Як останній засіб також можна використати `useEffect`.
* Написання чистих функцій потребує практики, але воно розкриває силу парадигми React.

</Recap>


  
<Challenges>

#### Ремонт годинника {/*fix-a-broken-clock*/}

Цей компонент намагається встановити CSS-клас елемента `<h1>` на `"night"` у час між опівночі та шостою годиною ранку, та на `"day"` у всі інші часи. Однак він не працює. Чи ви можете виправити компонент?

Своє рішення ви можете перевірити тимчасовою зміною часового поясу. Коли поточний час знаходиться між опівночі та шостою годиною ранку, кольори годинника мають бути інвертовані!

<Hint>

Рендер є *обчисленням*, йому не слід щось «робити». Чи можна виразити цю саму ідею інакше?

</Hint>

<Sandpack>

```js Clock.js active
export default function Clock({ time }) {
  let hours = time.getHours();
  if (hours >= 0 && hours <= 6) {
    document.getElementById('time').className = 'night';
  } else {
    document.getElementById('time').className = 'day';
  }
  return (
    <h1 id="time">
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

<Solution>

Цей компонент можна виправити, обчислюючи значення `className` та додаючи його в результат рендеру:

<Sandpack>

```js Clock.js active
export default function Clock({ time }) {
  let hours = time.getHours();
  let className;
  if (hours >= 0 && hours <= 6) {
    className = 'night';
  } else {
    className = 'day';
  }
  return (
    <h1 className={className}>
      {time.toLocaleTimeString()}
    </h1>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

У цьому прикладі побічний ефект (модифікація DOM) був взагалі не потрібен. Треба було всього лише повертати JSX.

</Solution>

#### Виправити поламаний профіль {/*fix-a-broken-profile*/}

Два компоненти `Profile` відрендерені з боків з різними даними. Натисніть «Collapse» у першому профілі, а потім натисніть «Expand». Як можна помітити, обидва профілі тепер показуть ту ж саму людину. Це є помилка.

Знайдіть причину помилки та виправте її.

<Hint>

Код з помилкою знаходиться в `Profile.js`. Уважно прочитайте його від початку до кінця!

</Hint>

<Sandpack>

```js Profile.js
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}

function Header() {
  return <h1>{currentPerson.name}</h1>;
}

function Avatar() {
  return (
    <img
      className="avatar"
      src={getImageUrl(currentPerson)}
      alt={currentPerson.name}
      width={50}
      height={50}
    />
  );
}
```

```js Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Collapse' : 'Expand'}
      </button>
      {open && children}
    </section>
  );
}
```

```js App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  )
}
```

```js utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

<Solution>

Проблема полягає в тому, що компонент `Profile` переписує вже існуючу змінну `currentPerson`, а компоненти `Header` і `Avatar` читають її значення. Це робить *всі три компоненти* нечистими, а їх поведінку — непередбачуваною.

Щоб виправити помилку, видаліть змінну `currentPerson`. Натомість передайте всю інформацію з `Profile` до `Header` і `Avatar` пропсами. Вам буде потрібно додати проп `person` до обох компонентів і передати його до самого низу.

<Sandpack>

```js Profile.js active
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

export default function Profile({ person }) {
  return (
    <Panel>
      <Header person={person} />
      <Avatar person={person} />
    </Panel>
  )
}

function Header({ person }) {
  return <h1>{person.name}</h1>;
}

function Avatar({ person }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={50}
      height={50}
    />
  );
}
```

```js Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Collapse' : 'Expand'}
      </button>
      {open && children}
    </section>
  );
}
```

```js App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  );
}
```

```js utils.js hidden
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

Пам'ятайте, що React не надає гарантій, що функції компонентів будуть виконані у фіксованому порядку, тому комунікація між компонентами за допомогою запису до змінних не є можлива. Вся комунікація має виконуватись за допомогою пропсів.

</Solution>

#### Fix a broken story tray {/*fix-a-broken-story-tray*/}

CEO вашої компанії просить вас додати до мобільного застосунку годинника «сторіси», і відмовити не можна. Ви написали компонент `StoryTray`, що приймає перелік `stories` та додає плейсхолдер створення розповіді в кінці.

Ви реалізували плейсхолдер створеня розповіді за допомогою додання фейкової розповіді в кінець масиву `stories`, який отримується як проп. Але цей плейсхолдер чомусь відображається більше одного разу. Виправіть проблему.

<Sandpack>

```js StoryTray.js active
export default function StoryTray({ stories }) {
  stories.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>It is {time.toLocaleTimeString()} now.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

</Sandpack>

<Solution>

Зверніть увагу на те, як «Create Story» додається двічі кожного разу, коли відновляється годинник. Це є підказка, що десь під час рендеру в нас виконується мутація — Strict Mode викликає компоненти двічі, щоб такі помилки були помітніші.

Функція `StoryTray` не є чиста. Викликаючи `push` для отриманого масиву `stories` (що є проп!), вона мутує об'єкт, що був створений *перед* рендером `StoryTray`. Це робить поведінку компонента дуже непередбачуваною.

Найпростіше вирішення проблеми є взагалі не чіпати масив і рендерити «Create Story» окремо:

<Sandpack>

```js StoryTray.js active
export default function StoryTray({ stories }) {
  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
      <li>Create Story</li>
    </ul>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>It is {time.toLocaleTimeString()} now.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Також можна створити _новий_ масив (копію існуючого) перед тим як додавати до нього новий елемент:

<Sandpack>

```js StoryTray.js active
export default function StoryTray({ stories }) {
  // Copy the array!
  let storiesToDisplay = stories.slice();

  // Does not affect the original array:
  storiesToDisplay.push({
    id: 'create',
    label: 'Create Story'
  });

  return (
    <ul>
      {storiesToDisplay.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  {id: 0, label: "Ankit's Story" },
  {id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState([...initialStories])
  let time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>It is {time.toLocaleTimeString()} now.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Це локалізує мутацію та робить функцію чистою. Однак все одно треба бути уважними: наприклад, якщо потрібно змінити ще й якісь із існуючих елементів масиву, потрібно буде склонувати і їх також.

Корисно пам'ятати, які саме операції над масивами їх мутують, а які — ні. Наприклад, `push`, `pop`, `reverse` та `sort` мутують існуючий масив, а ось `slice`, `filter` та `map` створюють новий.

</Solution>

</Challenges>
