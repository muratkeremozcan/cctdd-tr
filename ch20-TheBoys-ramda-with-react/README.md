# Ramda ile React

Before starting this section, make sure to go through the [prerequisite](./prerequisite.md) where were mirror `Heroes` to `Boys`.

Bu bölüme başlamadan önce, `Heroes`ı `Boys`a yansıttığımız [önkoşul](https://chat.openai.com/prerequisite.md) bölümünden geçtiğinizden emin olun.

## Neden Fonksiyonel Programlama, neden Ramda?

Eğer EcmaScript'e yapılan son önerilere bir göz atarsak, FP'nin yolunun olduğunu ve yeni JS özelliklerinin [RamdaJS](https://ramdajs.com/docs/) yardımcı programlarına benzer şekilde olacağını fark ederiz. Bir özet için [ES2025'te neler yeni?](https://www.youtube.com/watch?v=FaQA0qU5e6o) başlıklı videoya göz atın. Bugün FP ve Ramda'yı öğrenmenin, en azından önümüzdeki on yıl için geleceğe dayanıklı bir JS beceri seti oluşturacağına inanıyoruz.

Tüm seviyelerde TDD ile oluşturulmuş sağlam testlere sahip bir React uygulamamız ve %100 kod kapsamımız var. Önkoşullarda `Heroes` varlık grubunu `Boys`a yansıttık. Şimdi cesur yeniden yapılandırmalar deneyebilir ve her şeyin hala çalışıp çalışmadığını görebiliriz.

FP üzerine yazılmış çok sayıda kaynak bulunsa da, özellikle modern React ve TypeScript ile FP ve Ramda kullanımına dair gerçek dünya örneklerine yönelik eksiklikler bulunmaktadır. Bu bölümün bu boşluğu doldurmasını umuyoruz.

## Fonksiyonel Programlama JS kaynakları

Bu çalışmayı ilham kaynağı olarak kullanılan kaynaklar şunlardır. Bu bölümden yararlanmak için FP konusunda akıcı olmanız gerekmez, ancak daha sonra bilginizi tamamlamayı tercih ederseniz, bu kaynakları şiddetle önerilir.

- [Functional Light JS](https://www.manning.com/books/functional-light-javascript "https://www.manning.com/books/functional-light-javascript")
- [FP in JS](https://www.amazon.com/Functional-Programming-JavaScript-functional-techniques-ebook-dp-B09781W9HY/dp/B09781W9HY/ref=mt_other?_encoding=UTF8&me=&qid= "https://www.amazon.com/Functional-Programming-JavaScript-functional-techniques-ebook-dp-B09781W9HY/dp/B09781W9HY/ref=mt_other?_encoding=UTF8&me=&qid=")
- [Prof. Frisbie's Mostly Adequate Guide to FP](https://mostly-adequate.gitbook.io/mostly-adequate-guide/ "https://mostly-adequate.gitbook.io/mostly-adequate-guide/")
- [FP patterns with Ramda](https://www.educative.io/courses/functional-programming-patterns-with-ramdajs/YQV9QG6gqz9)
- [JS Allonge](https://leanpub.com/javascriptallongesix/read "https://leanpub.com/javascriptallongesix/read")
- [Composing Software](https://leanpub.com/composingsoftware)
- [Functional JavaScript](https://www.amazon.com/gp/product/1449360726)

## Ramda paketini yükleyin

```bash
yarn add ramda ramda-adjunct
yarn add -D @types/ramda
```

İlerleyen bölümlerde Ramda kullanarak pratik örnekler sunacağız ve bu bilgileri 3 Boys bileşenine uygulayacağız; `BoyDetail.tsx`, `BoyList.tsx` ve `Boys.tsx`.

## [`partial`](https://ramdajs.com/docs/#partial)

`n` bağımsız değişkenli herhangi bir işlev oluşturun ve bunu `partial` ile sarın. Önceden paketlenmiş olan bağımsız değişkenleri bildirin ve geri kalan bağımsız değişkenler beklenir. Basit bir şekilde anlatmak gerekirse; beş parametreli bir işleve sahipseniz ve bağımsız değişkenlerin üçünü sağlarsanız, son ikisini bekleyen bir işlevle karşılaşırsınız.

```typescript
// standalone example, copy paste anywhere
import { partial, partialObject, partialRight } from "ramda";

//// example 1
// create any function
const multiply = (a: number, b: number) => a * b;
// wrap it with partial, declare the pre-packaged args
const double = partial(multiply, [2]);

// the function waits for execution,
// executes upon the second arg being passed
double; // [λ]
double(3); // 6

//// example 2
// create any function
const greet = (
  salutation: string,
  title: string,
  firstName: string,
  lastName: string
) => salutation + ", " + title + " " + firstName + " " + lastName + "!";

// wrap it with partial, declare the pre-packaged args
const sayHello = partial(greet, ["Hello"]);
// the function waits for 3 arguments to be passed before executing
sayHello; // [λ]
sayHello("Ms", "Jane", "Jones"); // Hello, Ms Jane Jones!

// another variety, has 2 pre-packaged args
const sayHelloMs = partial(greet, ["Hello", "Ms"]);
// waits for 2 more args before executing
sayHelloMs; // // [λ]
sayHelloMs("Jane", "Jones"); // Hello, Ms Jane Jones!
```

`partial` React dünyasında nerede uygulanabilir? İsimsiz bir işlemin, bir bağımsız değişkenle adlandırılmış bir işlev döndürdüğü herhangi bir olay işleyici. Aşağıdaki iki örnek aynıdır:

```typescript
// src/boys/BoyDetail.tsx

const handleCancel = () => navigate("/boys");
const handleCancel = partial(navigate, ["/boys"]);
```

`handleCancel`, tıklama olayını bekleyecektir - `onClick={handleCancel}` - ve `/boys` rotasına yönlendirecektir.

Partial'ın benzer uygulamaları `Boys.tsx` dosyasında bulunabilir.

```typescript
// src/boys/Boys.tsx
import { partial } from "ramda";

const addNewBoy = () => navigate("/boys/add-boy");
const addNewBoy = partial(navigate, ["/boys/add-boy"]);

const handleRefresh = () => navigate("/boys");
const handleRefresh = partial(navigate, ["/boys"]);
```

## [`curry`](https://ramdajs.com/docs/#curry)

Hiçbir FP tanıtımı, curry'siz tamamlanmış sayılmaz. Burada basit bir örnek üzerinde duracağız ve daha önce klasik curry'i nasıl kullandığımızı hatırlayacağız.

```typescript
// standalone example, copy paste anywhere
import { compose, curry } from "ramda";

const add = (x: number, y: number) => x + y;

const result = add(2, 3);
result; //? 5

// What happens if you don’t give add its required parameters?
// What if you give too little?
// add(2) // NaN

// So we curry: return a new function per expected parameter
const addCurried = curry(add); // just wrap the function in curry
const addCurriedClassic = (x: number) => (y: number) => x + y;

addCurried(2); // waits to be executed
addCurried(2)(3); //? 5
addCurriedClassic(2); // waits to be executed
addCurriedClassic(2)(3); //? 5

const add3 = (x: number, y: number, z: number) => x + y + z;
const greetFirstLast = (greeting: string, first: string, last: string) =>
  `${greeting}, ${first} ${last}`;

const add3CurriedClassic = (x: number) => (y: number) => (z: number) =>
  x + y + z;
const greetCurriedClassic =
  (greeting: string) => (first: string) => (last: string) =>
    `${greeting}, ${first} ${last}`;
const add3CurriedR = curry(add3); // just wrap the function
const greetCurriedR = curry(greetFirstLast); // just wrap the function

add3CurriedClassic(2)(3); // waiting to be executed
add3CurriedClassic(2)(3)(4); //?
// add3CurriedClassic(2, 3, 4) // doesn't work

greetCurriedClassic("hello"); // waiting to be executed
greetCurriedClassic("hello")("John")("Doe"); //?
// greetCurriedClassic('hello', 'John', 'Doe') // doesn't work

add3CurriedR(2)(3); // waiting to be executed
add3CurriedR(2)(3)(4); //? 9
// the advantage of ramda is flexible arity
add3CurriedR(2, 3, 4); //?

greetCurriedR("hello", "John"); // waiting to be executed
greetCurriedR("hello")("John")("Doe"); //? hello, John Doe
// flexible arity with ramda!
greetCurriedR("hello", "John", "Doe"); //? hello, John Doe
```

Curry, React dünyasında nasıl kullanılır? `Heroes.tsx`, `HeroesList.tsx`, `VillianList.tsx` `VillainList.tsx` bileşenlerinde curry'i kullandığımızı hatırlayacaksınız. O zaman Ramda curry'i kullanmadık, çünkü o zaman çok karmaşık olurdu. İsteğe bağlı olarak şimdi onları değiştirebilirsiniz.

```tsx
// src/heroes/Heroes.tsx
import { curry } from "ramda";

// currying: the outer fn takes our custom arg and returns a fn that takes the event
const handleDeleteHero = (hero: Hero) => () => {
  setHeroToDelete(hero);
  setShowModal(true);
};
// we can use Ramda curry instead, we have to pass the unused event argument though
const handleDeleteHero = curry((hero: Hero, e: React.MouseEvent) => {
  setHeroToDelete(hero);
  setShowModal(true);
});
```

```tsx
// src/heroes/HeroList.tsx
import { curry } from "ramda";

// currying: the outer fn takes our custom arg and returns a fn that takes the event
const handleSelectHero = (heroId: string) => () => {
  const hero = deferredHeroes.find((h: Hero) => h.id === heroId);
  navigate(
    `/heroes/edit-hero/${hero?.id}?name=${hero?.name}&description=${hero?.description}`
  );
};
// we can use Ramda curry instead, we have to pass the unused event argument though
const handleSelectHero = curry(
  (heroId: string, e: MouseEvent<HTMLButtonElement>) => {
    const hero = deferredHeroes.find((h: Hero) => h.id === heroId);
    navigate(
      `/heroes/edit-hero/${hero?.id}?name=${hero?.name}&description=${hero?.description}`
    );
  }
);
```

`BoysList.tsx` bileşenlerinde Ramda curry'i kullanalım.

```tsx
// src/boys/BoyList.tsx
import { curry } from "ramda";

// currying: the outer fn takes our custom arg and returns a fn that takes the event
const handleSelectBoy = (boyId: string) => () => {
  const boy = deferredBoys.find((b: Boy) => b.id === boyId);
  navigate(
    `/boys/edit-boy/${boy?.id}?name=${boy?.name}&description=${boy?.description}`
  );
};
// we can use Ramda curry instead, we have to pass the unused event argument though
const handleSelectBoy = curry(
  (boyId: string, e: MouseEvent<HTMLButtonElement>) => {
    const boy = deferredBoys.find((b: Boy) => b.id === boyId);
    navigate(
      `/boys/edit-boy/${boy?.id}?name=${boy?.name}&description=${boy?.description}`
    );
  }
);
```

## [`ifElse`](https://ramdajs.com/docs/#ifElse)

3 işlev alır; yargı, doğru-sonuç, yanlış-sonuç. Klasik if else veya üçlü operatöre göre avantajı, işlev ifadeleriyle iç içe geçmiş ifadeler veya üçlü değerlerle çok daha yaratıcı hale gelebiliriz. Basit bir örnek ile açıklanabilir:

```typescript
// standalone example, copy paste anywhere
const hasAccess = false; // toggle this to see the difference

// classic if else
function logAccessClassic(hasAccess: boolean) {
  if (hasAccess) {
    return "Access granted";
  } else {
    return "Access denied";
  }
}
logAccessClassic(hasAccess); // Access granted | Access denied

// ternary operator
const logAccessTernary = (hasAccess: boolean) =>
  hasAccess ? "Access granted" : "Access denied";
logAccessTernary(hasAccess); // // Access granted | Access denied

// Ramda ifElse
// The advantage is that you can package the logic away into function
// we can get very creative with function expressions
// vs conditions inside an if statement, or a ternary value
const logAccessRamda = ifElse(
  (hasAccess: boolean) => hasAccess,
  () => "Access granted",
  () => "Access denied"
);

logAccessRamda(hasAccess); // Access granted | Access denied
```

`ifElse` React dünyasında nerede uygulanabilir? Klasik if else ifadeleri veya üçlü operatörlerle ifade etmesi zor olan karmaşık koşullu mantık bulunan her yerde. `BoyDetail.tsx`'deki örnek basit olsa da, Ramda çalışmak icin uygun.

```typescript
// src/boys/BoyDetail.tsx
import { ifElse } from "ramda";
import { isTruthy } from "ramda-adjunct";

const handleSave = () => (name ? updateBoy(boy as Boy) : createBoy(boy as Boy));

const handleSave = ifElse(
  () => isTruthy(name),
  () => updateBoy(boy as Boy),
  () => createBoy(boy as Boy)
);
```

Yukarıdaki değişikliklerin ardından `BoyDetail.tsx` bileşeni burada. `HeroDetail.tsx` ile yan yana kıyaslanarak, Ramda kullanımı ve klasik dizi yöntemleri arasındaki fark görülebilir.

```tsx
// src/boys/BoyDetail.tsx
import { useState, ChangeEvent } from "react";
import { useNavigate, useParams } from "react-router-dom";
import { FaUndo, FaRegSave } from "react-icons/fa";
import InputDetail from "components/InputDetail";
import ButtonFooter from "components/ButtonFooter";
import PageSpinner from "components/PageSpinner";
import ErrorComp from "components/ErrorComp";
import { useEntityParams } from "hooks/useEntityParams";
import { usePostEntity } from "hooks/usePostEntity";
import { Boy } from "models/Boy";
import { usePutEntity } from "hooks/usePutEntity";
import { partial, ifElse } from "ramda";
import { isTruthy } from "ramda-adjunct";

export default function BoyDetail() {
  const { id } = useParams();
  const { name, description } = useEntityParams();
  const [boy, setBoy] = useState({ id, name, description });
  const { mutate: createBoy, status, error: postError } = usePostEntity("boy");
  const {
    updateEntity: updateBoy,
    isUpdating,
    isUpdateError,
  } = usePutEntity("boy");

  const navigate = useNavigate();

  const handleCancel = partial(navigate, ["/boys"]);

  const handleSave = ifElse(
    () => isTruthy(name),
    () => updateBoy(boy as Boy),
    () => createBoy(boy as Boy)
  );

  const handleNameChange = (e: ChangeEvent<HTMLInputElement>) =>
    setBoy({ ...boy, name: e.target.value });

  const handleDescriptionChange = (e: ChangeEvent<HTMLInputElement>) =>
    setBoy({ ...boy, description: e.target.value });

  if (status === "loading" || isUpdating) {
    return <PageSpinner />;
  }

  if (postError || isUpdateError) {
    return <ErrorComp />;
  }

  return (
    <div data-cy="boy-detail" className="card edit-detail">
      <header className="card-header">
        <p className="card-header-title">{name}</p>
        &nbsp;
      </header>
      <div className="card-content">
        <div className="content">
          {id && (
            <InputDetail name={"id"} value={id} readOnly={true}></InputDetail>
          )}
          <InputDetail
            name={"name"}
            value={name ? name : ""}
            placeholder="e.g. Colleen"
            onChange={handleNameChange}
          ></InputDetail>
          <InputDetail
            name={"description"}
            value={description ? description : ""}
            placeholder="e.g. dance fight!"
            onChange={handleDescriptionChange}
          ></InputDetail>
        </div>
      </div>
      <footer className="card-footer">
        <ButtonFooter
          label="Cancel"
          IconClass={FaUndo}
          onClick={handleCancel}
        />
        <ButtonFooter label="Save" IconClass={FaRegSave} onClick={handleSave} />
      </footer>
    </div>
  );
}
```

## [`pipe`](https://ramdajs.com/docs/#pipe)

`pipe`, Ramda'nın temelidir; soldan sağa işlev bileşimi gerçekleştirmek için kullanılır - [`compose`](https://ramdajs.com/docs/#compose) ise sağdan sola işlev bileşimidir. Basit bir örnek ile açıklanır:

```typescript
// standalone example, copy paste anywhere
import { compose, pipe } from "ramda";

// create a function that composes the three functions
const toUpperCase = (str: string) => str.toUpperCase();
const emphasizeFlavor = (flavor: string) => `${flavor} IS A GREAT FLAVOR`;
const appendIWantIt = (str: string) => `${str}. I want it`;

// note that with Ramda, we can pass the argument later
// and we shape the data through the composition / pipe
const classic = (flavor: string) =>
  appendIWantIt(emphasizeFlavor(toUpperCase(flavor)));
const composeRamda = compose(appendIWantIt, emphasizeFlavor, toUpperCase);
const pipeRamda = pipe(toUpperCase, emphasizeFlavor, appendIWantIt);

classic("chocolate");
composeRamda("chocolate");
pipeRamda("chocolate");
// CHOCOLATE IS A GREAT FLAVOR. I want it
```

React dünyasında `pipe` nerede uygulanabilir? Herhangi bir işlem dizisi uygun olacaktır.

Aşağıdaki 3 çeşit `handleCloseModal` fonksiyonu `Boys.tsx` dosyasından eşdeğerdir.

```typescript
// src/boys/Boys.tsx
import { partial, pipe } from "ramda";

const handleCloseModal = () => {
  setBoyToDelete(null);
  setShowModal(false);
};
const handleCloseModal = pipe(
  () => setBoyToDelete(null),
  () => setShowModal(false)
);
const handleCloseModal = pipe(
  partial(setBoyToDelete, [null]),
  partial(setShowModal, [false])
);
```

`Boys.tsx` dosyasından aşağıdaki 3 çeşit `handleDeleteBoy` fonksiyonu da eşdeğerdir.

```typescript
// src/boys/Boys.tsx
import { partial, pipe } from "ramda";

const handleDeleteBoy = (boy: Boy) => () => {
  setBoyToDelete(boy);
  setShowModal(true);
};
const handleDeleteBoy = (boy: Boy) =>
  pipe(
    () => setBoyToDelete(boy),
    () => setShowModal(true)
  );
const handleDeleteBoy = (boy: Boy) =>
  pipe(partial(setBoyToDelete, [boy]), partial(setShowModal, [true]));
```

`Boys.tsx` dosyasından aşağıdaki 3 çeşit `handleDeleteFromModal` fonksiyonu da eşdeğerdir.

```typescript
// src/boys/Boys.tsx
const handleDeleteFromModal = () => {
  boyToDelete ? deleteBoy(boyToDelete) : null;
  setShowModal(false);
};
const handleDeleteFromModal2 = pipe(
  () => (boyToDelete ? deleteBoy(boyToDelete) : null),
  () => setShowModal(false)
);
const handleDeleteFromModal3 = pipe(
  () => (boyToDelete ? deleteBoy(boyToDelete) : null),
  partial(setShowModal, [false])
);
```

Yukarıdaki değişikliklerin ardından `Boys.tsx` bileşeni burada. Ramda ve klasik dizi yöntemleri arasındaki farkı görmek için `Heroes.tsx` ile karşılaştırılabilir.

```tsx
// src/boys/Boys.tsx
import { useState } from "react";
import { useNavigate, Routes, Route } from "react-router-dom";
import ListHeader from "components/ListHeader";
import ModalYesNo from "components/ModalYesNo";
import ErrorComp from "components/ErrorComp";
import BoyList from "./BoyList";
import BoyDetail from "./BoyDetail";
import { useGetEntities } from "hooks/useGetEntities";
import { useDeleteEntity } from "hooks/useDeleteEntity";
import { Boy } from "models/Boy";
import { partial, pipe } from "ramda";

export default function Boys() {
  const [showModal, setShowModal] = useState<boolean>(false);
  const { entities: boys, getError } = useGetEntities("boys");
  const [boyToDelete, setBoyToDelete] = useState<Boy | null>(null);
  const { deleteEntity: deleteBoy, isDeleteError } = useDeleteEntity("boy");

  const navigate = useNavigate();
  const addNewBoy = partial(navigate, ["/boys/add-boy"]);
  const handleRefresh = partial(navigate, ["/boys"]);

  const handleCloseModal = pipe(
    partial(setBoyToDelete, [null]),
    partial(setShowModal, [false])
  );

  const handleDeleteBoy = (boy: Boy) =>
    pipe(partial(setBoyToDelete, [boy]), partial(setShowModal, [true]));

  const handleDeleteFromModal = pipe(
    () => (boyToDelete ? deleteBoy(boyToDelete) : null),
    partial(setShowModal, [false])
  );

  if (getError || isDeleteError) {
    return <ErrorComp />;
  }

  return (
    <div data-cy="boys">
      <ListHeader
        title="Boys"
        handleAdd={addNewBoy}
        handleRefresh={handleRefresh}
      />
      <div>
        <div>
          <Routes>
            <Route
              path=""
              element={
                <BoyList boys={boys} handleDeleteBoy={handleDeleteBoy} />
              }
            />
            <Route path="/add-boy" element={<BoyDetail />} />
            <Route path="/edit-boy/:id" element={<BoyDetail />} />
            <Route
              path="*"
              element={
                <BoyList boys={boys} handleDeleteBoy={handleDeleteBoy} />
              }
            />
          </Routes>
        </div>
      </div>

      {showModal && (
        <ModalYesNo
          message="Would you like to delete the boy?"
          onNo={handleCloseModal}
          onYes={handleDeleteFromModal}
        />
      )}
    </div>
  );
}
```

## Dizi yöntemleriyle arama filtresini Ramda'ya yeniden düzenleme

Bu örnekte, `BoyList.tsx` içindeki arama filtresi mantığını, tipler ve verilerle birlikte bağımsız bir TS dosyasına çıkaracağız. Veri `heroes` dizisine göre (`id`, `name`, `description` gibi) herhangi bir özellikle filtrelemek ve 1 veya daha fazla nesne göstermek istiyoruz. Bunu başarmak için, `searchExistsC` ve `propertyExistsC` adlı 2 lego-fonksiyon kullanarak `searchPropertiesC`'ye, `C` ise klasik anlamında ulaşırız.

```typescript
// standalone example, copy paste anywhere

const heroes = [
  {
    id: "HeroAslaug",
    name: "Aslaug",
    description: "warrior queen",
  },
  {
    id: "HeroBjorn",
    name: "Bjorn Ironside",
    description: "king of 9th century Sweden",
  },
  {
    id: "HeroIvar",
    name: "Ivar the Boneless",
    description: "commander of the Great Heathen Army",
  },
  {
    id: "HeroLagertha",
    name: "Lagertha the Shieldmaiden",
    description: "aka Hlaðgerðr",
  },
  {
    id: "HeroRagnar",
    name: "Ragnar Lothbrok",
    description: "aka Ragnar Sigurdsson",
  },
  {
    id: "HeroThora",
    name: "Thora Town-hart",
    description: "daughter of Earl Herrauðr of Götaland",
  },
];
const textToSearch = "ragnar";
interface Hero {
  id: string;
  name: string;
  description: string;
}
type HeroProperty = Hero["name"] | Hero["description"] | Hero["id"];

// use the classic array methods to search for properties in a given array of entities

/** returns a boolean whether the entity property exists in the search field */
const searchExistsC = (searchField: string, searchProperty: HeroProperty) =>
  String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
  -1;

/** finds the given entity's property in the search field  */
const propertyExistsC = (searchField: string, item: Hero) =>
  Object.values(item).find((property: HeroProperty) =>
    searchExistsC(searchField, property)
  );

/** given the search field and the entity array, returns the entity 
in which the search field exists */
const searchPropertiesC = (data: Hero[], searchField: string) =>
  [...data].filter((item: Hero) => propertyExistsC(searchField, item));

searchPropertiesC(heroes, textToSearch);
/* 
[{ 
  id: 'HeroRagnar',
  name: 'Ragnar Lothbrok',
  description: 'aka Ragnar Sigurdsson' 
}]
*/
```

İşlevleri legolar gibi bölmüş olsak da, kodu okurken mantığı çok kolay takip etmek mümkün değildir. Bunun nedeni, birden fazla argüman kullanmak zorunda olmak ve verinin bir dizi öğesinden (bir nesne) bir dizi öğesine değişmesi, ayrıca argümanların yerini değiştirmek zorunda olmaktır. Ramda kullanarak kodu daha okunaklı ve kullanışlı hale getirebiliriz.

`.toLowerCase()` ve `toLower()` ile `.indexOf()` ve `indexOf` arasında hızlı bir karşılaştırma yapalım. Veri üzerinde zincirleme yerine, Ramda yardımcıları `indexOf` ve `toLower`, veriyi argüman olarak alan işlevlerdir:

- `someData.toLowerCase()` vs `toLower(someData)`
- `array.indexOf(arrayItem)` vs `indexOf(arrayItem, array)`

```typescript
"HELLO".toLowerCase(); // hello
toLower("HELLO"); // hello
[1, 2, 3, 4].indexOf(3); // 2
indexOf(3, [1, 2, 3, 4]); // 2
```

`searchExistsC`'yi Ramda kullanacak şekilde yeniden düzenleyebiliriz. İşte önce ve sonra yan yana:

```typescript
import { indexOf, toLower } from "ramda";

const searchExistsC = (searchField: string, searchProperty: HeroProperty) =>
  String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
  -1;

const searchExists = (searchField: string, searchProperty: HeroProperty) =>
  indexOf(toLower(searchField), toLower(searchProperty)) !== -1;
```

Aşağıda, `array.find` ile Ramda `find`, `Object.values` ile Ramda `values` arasında hızlı bir karşılaştırma bulunmaktadır. Ramda sürümünde verinin sonunda nasıl geldiğine dikkat edin; bu stil ile daha sonra kullanmak üzere bir argümanı kaydedebiliriz.

```typescript
// data.find(callback)
Object.values(heroes[4]).find(
  (property: HeroProperty) => property === "Ragnar Lothbrok"
);
// find(callback)(data)
find((property: HeroProperty) => property === "Ragnar Lothbrok")(
  values(heroes[4])
);
```

İşte `propertyExistsC`'nin Ramda kullanacak şekilde yeniden düzenlenmesi, önce ve sonra yan yana. Akışı klasik sürüme daha benzer hale getirmek için Ramda sürümünü `pipe` ile güçlendirebiliriz.

```typescript
const propertyExistsC = (searchField: string, item: Hero) =>
  Object.values(item).find((property: HeroProperty) =>
    searchExistsC(searchField, property)
  );

// we wrap the function in curry to allow passing the item argument independently, later
// f(a, b) to curry(f(a, b)) allows us to f(a)(b)
const propertyExists = curry((searchField: string, item: Hero) =>
  find((property: HeroProperty) => searchExists(searchField, property))(
    values(item)
  )
);

// we can take it a step further with pipe
// the data item comes at the end, we take it and pipe through values & find
// this way the flow is more similar to the original function
const propertyExistsNew = curry((searchField: string, item: Hero) =>
  pipe(
    values,
    find((property: HeroProperty) => searchExists(searchField, property))
  )(item)
);
```

Son işlevimiz `searchExistsC` önceki ikisini kullanır. İşte klasik ve Ramda sürümleri yan yana. Ramda sürümünü geliştirerek 1 argüman alacak hale getirebilir ve veriyi, ne zaman kullanılabilir olduğunu sonra alabiliriz.

```typescript
// (2 args) => data.filter(callback)
const searchPropertiesC = (data: Hero[], searchField: string) =>
  [...data].filter((item: Hero) => propertyExistsC(searchField, item));

// (2 args) => filter(callback, data)
const searchProperties = (searchField: string, data: Hero[]) =>
  filter((item: Hero) => propertyExists(searchField, item), [...data]);

// (2 args) => filter(callback)(data)
const searchPropertiesBetter = (searchField: string, data: Hero[]) =>
  filter((item: Hero) => propertyExistsNew(searchField, item))(data);

// (1 arg) => filter(fn), the data can come later and it will flow through
// notice how we eliminated another piece of data; item
const searchPropertiesBest = (searchField: string) =>
  filter(propertyExistsNew(searchField));
```

İşte her yere kopyalanıp yapıştırılabilen tam örnek dosyası:

```typescript
// standalone example, copy paste anywhere

import { indexOf, filter, find, curry, toLower, pipe, values } from "ramda";

const heroes = [
  {
    id: "HeroAslaug",
    name: "Aslaug",
    description: "warrior queen",
  },
  {
    id: "HeroBjorn",
    name: "Bjorn Ironside",
    description: "king of 9th century Sweden",
  },
  {
    id: "HeroIvar",
    name: "Ivar the Boneless",
    description: "commander of the Great Heathen Army",
  },
  {
    id: "HeroLagertha",
    name: "Lagertha the Shieldmaiden",
    description: "aka Hlaðgerðr",
  },
  {
    id: "HeroRagnar",
    name: "Ragnar Lothbrok",
    description: "aka Ragnar Sigurdsson",
  },
  {
    id: "HeroThora",
    name: "Thora Town-hart",
    description: "daughter of Earl Herrauðr of Götaland",
  },
];
const textToSearch = "ragnar";
interface Hero {
  id: string;
  name: string;
  description: string;
}

Object.values(heroes[4]).find(
  (property: HeroProperty) => property === "Ragnar Lothbrok"
);

find((property: HeroProperty) => property === "Ragnar Lothbrok")(
  values(heroes[4])
); //?

type HeroProperty = Hero["name"] | Hero["description"] | Hero["id"];

/** returns a boolean whether the hero properties exist in the search field */
const searchExistsC = (searchField: string, searchProperty: HeroProperty) =>
  String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
  -1;

const propertyExistsC = (searchField: string, item: Hero) =>
  Object.values(item).find((property: HeroProperty) =>
    searchExistsC(searchField, property)
  );

const searchPropertiesC = (data: Hero[], searchField: string) =>
  [...data].filter((item: Hero) => propertyExistsC(searchField, item));

searchPropertiesC(heroes, textToSearch); //?

// rewrite in ramda

const searchExists = (searchField: string, searchProperty: HeroProperty) =>
  indexOf(toLower(searchField), toLower(searchProperty)) !== -1;

const propertyExists = curry((searchField: string, item: Hero) =>
  find((property: HeroProperty) => searchExists(searchField, property))(
    values(item)
  )
);
// refactor propertyExists to use pipe
const propertyExistsNew = curry((searchField: string, item: Hero) =>
  pipe(
    values,
    find((property: HeroProperty) => searchExists(searchField, property))
  )(item)
);

// refactor in better ramda
const searchProperties = (searchField: string, data: Hero[]) =>
  filter((item: Hero) => propertyExists(searchField, item), [...data]);

const searchPropertiesBetter = (searchField: string, data: Hero[]) =>
  filter((item: Hero) => propertyExistsNew(searchField, item))(data);

const searchPropertiesBest = (searchField: string) =>
  filter(propertyExistsNew(searchField));

searchProperties(textToSearch, heroes); //?
searchPropertiesBetter(textToSearch, heroes); //?
searchPropertiesBest(textToSearch)(heroes); //?
```

Bununla birlikte, `searchProperties` ve ona yol açan lego-fonksiyonları Ramda sürümleriyle değiştirebiliriz.

İşte ana değişikliklerin karşılaştırması:

```tsx
// src/boys/BoyList.tsx (before)

/** returns a boolean whether the boy properties exist in the search field */
const searchExists = (searchProperty: BoyProperty, searchField: string) =>
  String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
  -1;

/** given the data and the search field, returns the data in which the search field exists */
const searchProperties = (searchField: string, data: Boy[]) =>
  [...data].filter((item: Boy) =>
    Object.values(item).find((property: BoyProperty) =>
      searchExists(property, searchField)
    )
  );

const handleSearch =
  (data: Boy[]) => (event: ChangeEvent<HTMLInputElement>) => {
    const searchField = event.target.value;

    return startTransition(() =>
      setFilteredBoys(searchProperties(searchField, data))
    );
  };
```

```tsx
// src/boys/BoyList.tsx (after)

/** returns a boolean whether the boy properties exist in the search field */
const searchExists = (searchField: string, searchProperty: BoyProperty) =>
  indexOf(toLower(searchField), toLower(searchProperty)) !== -1;

/** finds the given boy's property in the search field  */
const propertyExists = curry((searchField: string, item: Boy) =>
  pipe(
    values,
    find((property: BoyProperty) => searchExists(searchField, property))
  )(item)
);
/** given the search field and the boy array, returns the boy in which the search field exists */
const searchProperties = (
  searchField: string
): (<P extends Boy, C extends readonly P[] | Dictionary<P>>(
  collection: C
) => C) => filter(propertyExists(searchField));

/** filters the boys data to see if the any of the properties exist in the list */
const handleSearch =
  (data: Boy[]) => (event: ChangeEvent<HTMLInputElement>) => {
    const searchField = event.target.value;
    const searchedBoy = searchProperties(searchField)(data);

    return startTransition(() =>
      setFilteredBoys(searchedBoy as React.SetStateAction<Boy[]>)
    );
  };
```

İşte bileşenin son hali:

```tsx
// src/boys/BoyList.tsx
import { useNavigate } from "react-router-dom";
import CardContent from "components/CardContent";
import ButtonFooter from "components/ButtonFooter";
import { FaEdit, FaRegSave } from "react-icons/fa";
import {
  ChangeEvent,
  MouseEvent,
  useTransition,
  useEffect,
  useState,
  useDeferredValue,
} from "react";
import { Boy } from "models/Boy";
import { BoyProperty } from "models/types";
import {
  indexOf,
  find,
  curry,
  toLower,
  pipe,
  values,
  filter,
  Dictionary,
} from "ramda";

type BoyListProps = {
  boys: Boy[];
  handleDeleteBoy: (boy: Boy) => (e: MouseEvent<HTMLButtonElement>) => void;
};

export default function BoyList({ boys, handleDeleteBoy }: BoyListProps) {
  const deferredBoys = useDeferredValue(boys);
  const isStale = deferredBoys !== boys;
  const [filteredBoys, setFilteredBoys] = useState(deferredBoys);
  const navigate = useNavigate();
  const [isPending, startTransition] = useTransition();

  // needed to refresh the list after deleting a boy
  useEffect(() => setFilteredBoys(deferredBoys), [deferredBoys]);

  const handleSelectBoy = curry(
    (boyId: string, e: MouseEvent<HTMLButtonElement>) => {
      const boy = deferredBoys.find((b: Boy) => b.id === boyId);
      navigate(
        `/boys/edit-boy/${boy?.id}?name=${boy?.name}&description=${boy?.description}`
      );
    }
  );

  /** returns a boolean whether the boy properties exist in the search field */
  const searchExists = (searchField: string, searchProperty: BoyProperty) =>
    indexOf(toLower(searchField), toLower(searchProperty)) !== -1;

  /** finds the given boy's property in the search field  */
  const propertyExists = curry((searchField: string, item: Boy) =>
    pipe(
      values,
      find((property: BoyProperty) => searchExists(searchField, property))
    )(item)
  );
  /** given the search field and the boy array, returns the boy in which the search field exists */
  const searchProperties = (
    searchField: string
  ): (<P extends Boy, C extends readonly P[] | Dictionary<P>>(
    collection: C
  ) => C) => filter(propertyExists(searchField));

  /** filters the boys data to see if the any of the properties exist in the list */
  const handleSearch =
    (data: Boy[]) => (event: ChangeEvent<HTMLInputElement>) => {
      const searchField = event.target.value;
      const searchedBoy = searchProperties(searchField)(data);

      return startTransition(() =>
        setFilteredBoys(searchedBoy as React.SetStateAction<Boy[]>)
      );
    };

  return (
    <div
      style={{
        opacity: isPending ? 0.5 : 1,
        color: isStale ? "dimgray" : "black",
      }}
    >
      {deferredBoys.length > 0 && (
        <div className="card-content">
          <span>Search </span>
          <input data-cy="search" onChange={handleSearch(deferredBoys)} />
        </div>
      )}
      &nbsp;
      <ul data-cy="boy-list" className="list">
        {filteredBoys.map((boy, index) => (
          <li data-cy={`boy-list-item-${index}`} key={boy.id}>
            <div className="card">
              <CardContent name={boy.name} description={boy.description} />
              <footer className="card-footer">
                <ButtonFooter
                  label="Delete"
                  IconClass={FaRegSave}
                  onClick={handleDeleteBoy(boy)}
                />
                <ButtonFooter
                  label="Edit"
                  IconClass={FaEdit}
                  onClick={handleSelectBoy(boy.id)}
                />
              </footer>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Araçlarımız ve testlerimiz, değişikliklerden sonra hala sağlam durumda ve bu, çok yüksek kapsamaya sahip olmanın lüksüdür. Kodumuza cesur, deneysel yeniden düzenlemeler uygulayabildik ve FP ve Ramda kullanarak kodun daha okunaklı ve çalışılabilir hale gelirken hiçbir şeyin gerilemediğinden emin olduk.

Önce ve sonraya bakmak için, bu bölümle ilgili PR'ı şu adreste bulabilirsiniz: https://github.com/muratkeremozcan/tour-of-heroes-react-cypress-ts/pull/110. Ayrıca Boys dosyalarını Heroes veya Villains ile karşılaştırabilirsiniz.
