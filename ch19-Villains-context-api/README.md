# Context API

Bu bölüme başlamadan önce, uygulamayı Kahramanlar ve Kötüler arasında daha genel hale getirmek için [önbilgi](https://chat.openai.com/prerequisite.md) bölümünden geçtiğinizden emin olun.

Bu bölümde, kahramanları kötülere yansıtacağız ve Context api'yi kötülere uygulayacağız. Context api, değeri bileşen ağacının derinliklerine geçirmemize olanak tanır, bunu her bileşenden açıkça geçirerek yapmamız gerekmez. Bu, bileşen ağacında durumu paylaşmak için context'i kullanmak ile durumu alt bileşenlere prop olarak geçirmek arasında güzel bir karşılaştırma sunacaktır.

`Heroes.tsx`, `heroes` özelliğini `HeroList.tsx`'e aktarır.

`HeroDetail`, `hero` `{id, name, description}` durumunu URL'den `useParams` ve `useSearchParams` ile alır.

Bu, uygulamamızdaki durum yönetiminin sınırlarıdır. Gerçekte context'e ihtiyacımız yoktur, ancak context api ile şeylerin nasıl farklı olabileceğini inceleyeceğiz.

## Kötüler için Context API

Şu anda kahramanlardan kötülere tam bir yansıma elde ettik, işlevsellik ve testler tam olarak aynı şekilde gerçekleştiriliyor. Bununla birlikte, kötüler grubunu değiştirecek ve bunu yaparken Context api'den faydalanacağız.

`Villains.tsx`, `villains` özelliğini `VillainList.tsx`'e aktarır. Bunun yerine Context api'yi kullanarak `villains`'in `Villains.txt` altındaki tüm bileşenlerde kullanılabilir olmasını sağlayacağız.

Context api ile genel adımlar şunlardır:

1.  Context'i oluşturun ve dışa aktarın. Genellikle bu ayrı bir dosyada bulunur ve arabulucu görevi görür.

    ```tsx
    // src/villains/VillainsContext.tsx (the common node)
    import { Villain } from "models/Villain";
    import { createContext } from "react";

    const VillainsContext = createContext<Villain[]>([]);
    export default VillainsContext;
    ```

2.  Alt bileşenlere geçirilecek durumu belirleyin. Orada context'i içe aktarın.

    ```tsx
    // src/villains/Villains.tsx
    import { VillainsContext } from "./VillainsContext";

    // ...

    const { villains, status, getError } = useGetEntity();
    ```

3.  UI'ı context'in Provider bileşeni ile sarın, geçirilecek durumu `value` özelliğine atayın:

    ```tsx
    // src/villains/Villains.tsx (the sharer)

    <VillainsContext.Provider value={villains}>
      <Routes>...</Routes>
    </VillainsContext.Provider>
    ```

4.  Duruma ihtiyaç duyan herhangi bir bileşende, Context API'sini tüketin; `useContext` kancasını ve context nesnesini içe aktarın:

    ```tsx
    // src/villains/VillainList.tsx
    import { useContext } from "react";
    import { VillainsContext } from "./VillainsContext";
    ```

5.  Paylaşılan context ile useContext'i çağırın, bir değişkene atayın:

    ```tsx
    // src/villains/VillainList.tsx (the sharee)
    import { useContext } from "react";
    import { VillainsContext } from "./VillainsContext";
    
    // ..
    
    const villains = useContext(VillainsContext);
    ```

İlk adımı takiben, yeni bir dosyada context'i oluşturuyoruz ve dışa aktarıyoruz:

```tsx
// src/villains/VillainsContext.ts
import { Villain } from "models/Villain";
import { createContext } from "react";

const VillainsContext = createContext<Villain[]>([]);

export default VillainsContext;
```

Örneğimizde, `Villains.tsx` dosyasından, `villains`'i `VillainDetail.tsx`'e geçiriyoruz. `villains`'i `useGetEntity` adlı kancadan alıyoruz. Şu anda `villains`'i bir özellik olarak `VillainDetail.tsx`'e geçiriyoruz ve bunun yerine context api'yi kullanmak istiyoruz. Bu yüzden context'i içe aktarıyoruz ve context sağlayıcıyı, içinde `villains`'e atanmış bir `value` özelliği olan rotalarla sarıyoruz (Adımlar 2, 3).

```tsx
// src/villains/Villains.tsx
import { useState } from "react";
import { useNavigate, Routes, Route } from "react-router-dom";
import ListHeader from "components/ListHeader";
import ModalYesNo from "components/ModalYesNo";
import PageSpinner from "components/PageSpinner";
import ErrorComp from "components/ErrorComp";
import VillainList from "./VillainList";
import VillainDetail from "./VillainDetail";
import { useGetEntities } from "hooks/useGetEntities";
import { useDeleteEntity } from "hooks/useDeleteEntity";
import { Villain } from "models/Villain";
import VillainsContext from "./VillainsContext";

export default function Villains() {
  const [showModal, setShowModal] = useState<boolean>(false);
  const { entities: villains, status, getError } = useGetEntities("villains");
  const [villainToDelete, setVillainToDelete] = useState<Villain | null>(null);
  const { deleteEntity: deleteVillain, isDeleteError } =
    useDeleteEntity("villain");

  const navigate = useNavigate();
  const addNewVillain = () => navigate("/villains/add-villain");
  const handleRefresh = () => navigate("/villains");

  const handleCloseModal = () => {
    setVillainToDelete(null);
    setShowModal(false);
  };

  const handleDeleteVillain = (villain: Villain) => () => {
    setVillainToDelete(villain);
    setShowModal(true);
  };
  const handleDeleteFromModal = () => {
    villainToDelete ? deleteVillain(villainToDelete) : null;
    setShowModal(false);
  };

  if (status === "loading") {
    return <PageSpinner />;
  }

  if (getError || isDeleteError) {
    return <ErrorComp />;
  }

  return (
    <div data-cy="villains">
      <ListHeader
        title="Villains"
        handleAdd={addNewVillain}
        handleRefresh={handleRefresh}
      />
      <div>
        <div>
          <VillainsContext.Provider value={villains}>
            <Routes>
              <Route
                path=""
                element={
                  <VillainList handleDeleteVillain={handleDeleteVillain} />
                }
              />
              <Route path="/add-villain" element={<VillainDetail />} />
              <Route path="/edit-villain/:id" element={<VillainDetail />} />
              <Route
                path="*"
                element={
                  <VillainList handleDeleteVillain={handleDeleteVillain} />
                }
              />
            </Routes>
          </VillainsContext.Provider>
        </div>
      </div>

      {showModal && (
        <ModalYesNo
          message="Would you like to delete the villain?"
          onNo={handleCloseModal}
          onYes={handleDeleteFromModal}
        />
      )}
    </div>
  );
}
```

`VillainsList.tsx` bileşenine context aracılığıyla durumu iletmemiz gerekiyor. Bu nedenle context'i ve React'tan `useContext`'i içe aktarıyoruz. Paylaşılan context'i argüman olarak içeren `useContext`'i çağırıyoruz ve `villains` adlı bir değişkene atıyoruz (Adımlar 4, 5). Şimdi, özelliğin yerine, durumu `VillainsContext`'ten alıyoruz.

```tsx
// src/villains/VillainList.tsx
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
import { useContext } from "react";
import { Villain } from "models/Villain";
import VillainsContext from "./VillainsContext";

type VillainListProps = {
  handleDeleteVillain: (
    villain: Villain
  ) => (e: MouseEvent<HTMLButtonElement>) => void;
};

export default function VillainList({ handleDeleteVillain }: VillainListProps) {
  const villains = useContext(VillainsContext);

  const deferredVillains = useDeferredValue(villains);
  const isStale = deferredVillains !== villains;
  const [filteredVillains, setFilteredVillains] = useState(deferredVillains);
  const navigate = useNavigate();
  const [isPending, startTransition] = useTransition();

  // needed to refresh the list after deleting a villain
  useEffect(() => setFilteredVillains(deferredVillains), [deferredVillains]);

  const handleSelectVillain = (villainId: string) => () => {
    const villain = deferredVillains.find((h: Villain) => h.id === villainId);
    navigate(
      `/villains/edit-villain/${villain?.id}?name=${villain?.name}&description=${villain?.description}`
    );
  };

  type VillainProperty =
    | Villain["name"]
    | Villain["description"]
    | Villain["id"];

  /** returns a boolean whether the villain properties exist in the search field */
  const searchExists = (searchProperty: VillainProperty, searchField: string) =>
    String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
    -1;

  /** given the data and the search field, returns the data in which the search field exists */
  const searchProperties = (searchField: string, data: Villain[]) =>
    [...data].filter((item: Villain) =>
      Object.values(item).find((property: VillainProperty) =>
        searchExists(property, searchField)
      )
    );

  /** filters the villains data to see if the any of the properties exist in the list */
  const handleSearch =
    (data: Villain[]) => (event: ChangeEvent<HTMLInputElement>) => {
      const searchField = event.target.value;

      return startTransition(() =>
        setFilteredVillains(searchProperties(searchField, data))
      );
    };

  return (
    <div
      style={{
        opacity: isPending ? 0.5 : 1,
        color: isStale ? "dimgray" : "black",
      }}
    >
      {deferredVillains.length > 0 && (
        <div className="card-content">
          <span>Search </span>
          <input data-cy="search" onChange={handleSearch(deferredVillains)} />
        </div>
      )}
      &nbsp;
      <ul data-cy="villain-list" className="list">
        {filteredVillains.map((villain, index) => (
          <li data-cy={`villain-list-item-${index}`} key={villain.id}>
            <div className="card">
              <CardContent
                name={villain.name}
                description={villain.description}
              />
              <footer className="card-footer">
                <ButtonFooter
                  label="Delete"
                  IconClass={FaRegSave}
                  onClick={handleDeleteVillain(villain)}
                />
                <ButtonFooter
                  label="Edit"
                  IconClass={FaEdit}
                  onClick={handleSelectVillain(villain.id)}
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

Bileşen testini güncelleyin ve bağlama sırasında context sağlayıcıyı da kullanın.

```tsx
// src/villains/VillainList.cy.tsx
import VillainList from "./VillainList";
import "../styles.scss";
import villains from "../../cypress/fixtures/villains.json";
import VillainsContext from "./VillainsContext";

describe("VillainList", () => {
  it("no villains should not display a list nor search bar", () => {
    cy.wrappedMount(
      <VillainList handleDeleteVillain={cy.stub().as("handleDeleteVillain")} />
    );

    cy.getByCy("villain-list").should("exist");
    cy.getByCyLike("villain-list-item").should("not.exist");
    cy.getByCy("search").should("not.exist");
  });

  context("with villains in the list", () => {
    beforeEach(() => {
      cy.wrappedMount(
        <VillainsContext.Provider value={villains}>
          <VillainList
            handleDeleteVillain={cy.stub().as("handleDeleteVillain")}
          />
        </VillainsContext.Provider>
      );
    });

    it("should render the villain layout", () => {
      cy.getByCyLike("villain-list-item").should(
        "have.length",
        villains.length
      );

      cy.getByCy("card-content");
      cy.contains(villains[0].name);
      cy.contains(villains[0].description);

      cy.get("footer")
        .first()
        .within(() => {
          cy.getByCy("delete-button");
          cy.getByCy("edit-button");
        });
    });

    it("should search and filter villain by name and description", () => {
      cy.getByCy("search").type(villains[0].name);
      cy.getByCyLike("villain-list-item")
        .should("have.length", 1)
        .contains(villains[0].name);

      cy.getByCy("search").clear().type(villains[2].description);
      cy.getByCyLike("villain-list-item")
        .should("have.length", 1)
        .contains(villains[2].description);
    });

    it("should handle delete", () => {
      cy.getByCy("delete-button").first().click();
      cy.get("@handleDeleteVillain").should("have.been.called");
    });

    it("should handle edit", () => {
      cy.getByCy("edit-button").first().click();
      cy.location("pathname").should(
        "eq",
        "/villains/edit-villain/" + villains[0].id
      );
    });
  });
});
```

RTL testini, bağlam sağlayıcıyı kullanarak renderlarken de güncelleyin.

```tsx
// src/villains/VillainList.test.tsx
import VillainList from "./VillainList";
import { wrappedRender, screen, waitFor } from "test-utils";
import userEvent from "@testing-library/user-event";
import { villains } from "../../db.json";
import VillainsContext from "./VillainsContext";

describe("VillainList", () => {
  const handleDeleteVillain = jest.fn();

  it("no villains should not display a list nor search bar", async () => {
    wrappedRender(<VillainList handleDeleteVillain={handleDeleteVillain} />);

    expect(await screen.findByTestId("villain-list")).toBeInTheDocument();
    expect(screen.queryByTestId("villain-list-item-1")).not.toBeInTheDocument();
    expect(screen.queryByTestId("search-bar")).not.toBeInTheDocument();
  });

  describe("with villains in the list", () => {
    beforeEach(() => {
      wrappedRender(
        <VillainsContext.Provider value={villains}>
          <VillainList handleDeleteVillain={handleDeleteVillain} />
        </VillainsContext.Provider>
      );
    });

    const cardContents = async () => screen.findAllByTestId("card-content");
    const deleteButtons = async () => screen.findAllByTestId("delete-button");
    const editButtons = async () => screen.findAllByTestId("edit-button");

    it("should render the villain layout", async () => {
      expect(
        await screen.findByTestId(`villain-list-item-${villains.length - 1}`)
      ).toBeInTheDocument();

      expect(await screen.findByText(villains[0].name)).toBeInTheDocument();
      expect(
        await screen.findByText(villains[0].description)
      ).toBeInTheDocument();
      expect(await cardContents()).toHaveLength(villains.length);
      expect(await deleteButtons()).toHaveLength(villains.length);
      expect(await editButtons()).toHaveLength(villains.length);
    });

    it("should search and filter villain by name and description", async () => {
      const search = await screen.findByTestId("search");

      userEvent.type(search, villains[0].name);
      await waitFor(async () => expect(await cardContents()).toHaveLength(1));
      await screen.findByText(villains[0].name);

      userEvent.clear(search);
      await waitFor(async () =>
        expect(await cardContents()).toHaveLength(villains.length)
      );

      userEvent.type(search, villains[2].description);
      await waitFor(async () => expect(await cardContents()).toHaveLength(1));
    });

    it("should handle delete", async () => {
      userEvent.click((await deleteButtons())[0]);
      expect(handleDeleteVillain).toHaveBeenCalled();
    });

    it("should handle edit", async () => {
      userEvent.click((await editButtons())[0]);
      await waitFor(() =>
        expect(window.location.pathname).toEqual(
          "/villains/edit-villain/" + villains[0].id
        )
      );
    });
  });
});
```

### Özel bir kanca kullanarak bağlamı paylaşma

`VillainsContext`'te oluşturulan bağlam, hakem görevi görüyor. `Villains.tsx` bağlamı, `villains` durumunu alt bileşeni `VillainList`'e iletmek için kullanır. `VillainList` durumunu `VillainsContext`'ten alır. `VillainsContext` yerine, içe aktarmaları azaltacak bir kanca kullanabiliriz.

`src/villains/VillainsContext.ts` dosyasını kaldırın ve bunun yerine `src/hooks/useVillainsContext.ts` adlı bir kanca oluşturun. Kancanın ilk yarısı `VillainsContext` ile aynıdır. Durumu ayarlamak için bir ayarlayıcı ekleriz, böylece durumu alan bileşenler de onu ayarlama yeteneği kazanır. Ayrıca kancanın işlevselliğiyle ilgili durumu ve etkileri kancanın içinde yönetir ve yalnızca bileşenlerin ihtiyaç duyduğu değer(ler)i döndürürüz.

```typescript
// src/hooks/useVillainsContext.ts
import { createContext, useContext, SetStateAction, Dispatch } from "react";
import { Villain } from "models/Villain";

// Context api lets us pass a value deep into the component tree
// without explicitly threading it through every component (2nd tier state management)

const VillainsContext = createContext<Villain[]>([]);
// to be used as VillainsContext.Provider,
// takes a prop as `value`, which is the context/data/state to share
export default VillainsContext;

const VillainsSetContext = createContext<Dispatch<
  SetStateAction<Villain[] | null>
> | null>(null);

// Manage state and effects related to a hook’s functionality
// within the hook and return only the value(s) that components need

export function useVillainsContext() {
  const villains = useContext(VillainsContext);
  const setVillains = useContext(VillainsSetContext);

  return [villains, setVillains] as const;
}
```

`Villains.tsx`'deki tek değişiklik, `VillainsContext`'in içe aktarma konumudur.

```tsx
// src/villains/Villains.tsx
import { useState } from "react";
import { useNavigate, Routes, Route } from "react-router-dom";
import ListHeader from "components/ListHeader";
import ModalYesNo from "components/ModalYesNo";
import PageSpinner from "components/PageSpinner";
import ErrorComp from "components/ErrorComp";
import VillainList from "./VillainList";
import VillainDetail from "./VillainDetail";
import { useGetEntities } from "hooks/useGetEntities";
import { useDeleteEntity } from "hooks/useDeleteEntity";
import { Villain } from "models/Villain";
import VillainsContext from "hooks/useVillainsContext";

export default function Villains() {
  const [showModal, setShowModal] = useState<boolean>(false);
  const { entities: villains, status, getError } = useGetEntities("villains");
  const [villainToDelete, setVillainToDelete] = useState<Villain | null>(null);
  const { deleteEntity: deleteVillain, isDeleteError } =
    useDeleteEntity("villain");

  const navigate = useNavigate();
  const addNewVillain = () => navigate("/villains/add-villain");
  const handleRefresh = () => navigate("/villains");

  const handleCloseModal = () => {
    setVillainToDelete(null);
    setShowModal(false);
  };

  const handleDeleteVillain = (villain: Villain) => () => {
    setVillainToDelete(villain);
    setShowModal(true);
  };
  const handleDeleteFromModal = () => {
    villainToDelete ? deleteVillain(villainToDelete) : null;
    setShowModal(false);
  };

  if (status === "loading") {
    return <PageSpinner />;
  }

  if (getError || isDeleteError) {
    return <ErrorComp />;
  }

  return (
    <div data-cy="villains">
      <ListHeader
        title="Villains"
        handleAdd={addNewVillain}
        handleRefresh={handleRefresh}
      />
      <div>
        <div>
          <VillainsContext.Provider value={villains}>
            <Routes>
              <Route
                path=""
                element={
                  <VillainList handleDeleteVillain={handleDeleteVillain} />
                }
              />
              <Route path="/add-villain" element={<VillainDetail />} />
              <Route path="/edit-villain/:id" element={<VillainDetail />} />
              <Route
                path="*"
                element={
                  <VillainList handleDeleteVillain={handleDeleteVillain} />
                }
              />
            </Routes>
          </VillainsContext.Provider>
        </div>
      </div>

      {showModal && (
        <ModalYesNo
          message="Would you like to delete the villain?"
          onNo={handleCloseModal}
          onYes={handleDeleteFromModal}
        />
      )}
    </div>
  );
}
```

Benzer şekilde, `VillainsList` bileşen testinde yalnızca içe aktarma değişir. `useVillainsContext` adlı kancayı kullanıyoruz. Aynı durum `VillainList.test.tsx` için de geçerlidir.

```tsx
// src/villains/VillainList.cy.tsx
import VillainList from "./VillainList";
import "../styles.scss";
import villains from "../../cypress/fixtures/villains.json";
import VillainsContext from "hooks/useVillainsContext";

describe("VillainList", () => {
  it("no villains should not display a list nor search bar", () => {
    cy.wrappedMount(
      <VillainList handleDeleteVillain={cy.stub().as("handleDeleteVillain")} />
    );

    cy.getByCy("villain-list").should("exist");
    cy.getByCyLike("villain-list-item").should("not.exist");
    cy.getByCy("search").should("not.exist");
  });

  context("with villains in the list", () => {
    beforeEach(() => {
      cy.wrappedMount(
        <VillainsContext.Provider value={villains}>
          <VillainList
            handleDeleteVillain={cy.stub().as("handleDeleteVillain")}
          />
        </VillainsContext.Provider>
      );
    });

    it("should render the villain layout", () => {
      cy.getByCyLike("villain-list-item").should(
        "have.length",
        villains.length
      );

      cy.getByCy("card-content");
      cy.contains(villains[0].name);
      cy.contains(villains[0].description);

      cy.get("footer")
        .first()
        .within(() => {
          cy.getByCy("delete-button");
          cy.getByCy("edit-button");
        });
    });

    it("should search and filter villain by name and description", () => {
      cy.getByCy("search").type(villains[0].name);
      cy.getByCyLike("villain-list-item")
        .should("have.length", 1)
        .contains(villains[0].name);

      cy.getByCy("search").clear().type(villains[2].description);
      cy.getByCyLike("villain-list-item")
        .should("have.length", 1)
        .contains(villains[2].description);
    });

    it("should handle delete", () => {
      cy.getByCy("delete-button").first().click();
      cy.get("@handleDeleteVillain").should("have.been.called");
    });

    it("should handle edit", () => {
      cy.getByCy("edit-button").first().click();
      cy.location("pathname").should(
        "eq",
        "/villains/edit-villain/" + villains[0].id
      );
    });
  });
});
```

```tsx
// src/villains/VillainList.test.tsx
import VillainList from "./VillainList";
import { wrappedRender, screen, waitFor } from "test-utils";
import userEvent from "@testing-library/user-event";
import { villains } from "../../db.json";
import VillainsContext from "hooks/useVillainsContext";

describe("VillainList", () => {
  const handleDeleteVillain = jest.fn();

  it("no villains should not display a list nor search bar", async () => {
    wrappedRender(<VillainList handleDeleteVillain={handleDeleteVillain} />);

    expect(await screen.findByTestId("villain-list")).toBeInTheDocument();
    expect(screen.queryByTestId("villain-list-item-1")).not.toBeInTheDocument();
    expect(screen.queryByTestId("search-bar")).not.toBeInTheDocument();
  });

  describe("with villains in the list", () => {
    beforeEach(() => {
      wrappedRender(
        <VillainsContext.Provider value={villains}>
          <VillainList handleDeleteVillain={handleDeleteVillain} />
        </VillainsContext.Provider>
      );
    });

    const cardContents = async () => screen.findAllByTestId("card-content");
    const deleteButtons = async () => screen.findAllByTestId("delete-button");
    const editButtons = async () => screen.findAllByTestId("edit-button");

    it("should render the villain layout", async () => {
      expect(
        await screen.findByTestId(`villain-list-item-${villains.length - 1}`)
      ).toBeInTheDocument();

      expect(await screen.findByText(villains[0].name)).toBeInTheDocument();
      expect(
        await screen.findByText(villains[0].description)
      ).toBeInTheDocument();
      expect(await cardContents()).toHaveLength(villains.length);
      expect(await deleteButtons()).toHaveLength(villains.length);
      expect(await editButtons()).toHaveLength(villains.length);
    });

    it("should search and filter villain by name and description", async () => {
      const search = await screen.findByTestId("search");

      userEvent.type(search, villains[0].name);
      await waitFor(async () => expect(await cardContents()).toHaveLength(1));
      await screen.findByText(villains[0].name);

      userEvent.clear(search);
      await waitFor(async () =>
        expect(await cardContents()).toHaveLength(villains.length)
      );

      userEvent.type(search, villains[2].description);
      await waitFor(async () => expect(await cardContents()).toHaveLength(1));
    });

    it("should handle delete", async () => {
      userEvent.click((await deleteButtons())[0]);
      expect(handleDeleteVillain).toHaveBeenCalled();
    });

    it("should handle edit", async () => {
      userEvent.click((await editButtons())[0]);
      await waitFor(() =>
        expect(window.location.pathname).toEqual(
          "/villains/edit-villain/" + villains[0].id
        )
      );
    });
  });
});
```

`VillainList.tsx`'de, `Villains.tsx`'de olduğu gibi içe aktarma değişir. Ayrıca, `useContext`'i içe aktarmaya gerek kalmaz. Şimdi `villains`'i de yapılandırıyoruz; `const [villains] = useVillainsContext()`. Gerekirse, bağlamı ayarlamak için kancadan ayarlayıcıyı da alabiliriz.

```tsx
// src/villains/VillainList.tsx
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
import { useVillainsContext } from "hooks/useVillainsContext";
import { Villain } from "models/Villain";

type VillainListProps = {
  handleDeleteVillain: (
    villain: Villain
  ) => (e: MouseEvent<HTMLButtonElement>) => void;
};

export default function VillainList({ handleDeleteVillain }: VillainListProps) {
  const [villains] = useVillainsContext();

  const deferredVillains = useDeferredValue(villains);
  const isStale = deferredVillains !== villains;
  const [filteredVillains, setFilteredVillains] = useState(deferredVillains);
  const navigate = useNavigate();
  const [isPending, startTransition] = useTransition();

  // needed to refresh the list after deleting a villain
  useEffect(() => setFilteredVillains(deferredVillains), [deferredVillains]);

  // currying: the outer fn takes our custom arg and returns a fn that takes the event
  const handleSelectVillain = (villainId: string) => () => {
    const villain = deferredVillains.find((h: Villain) => h.id === villainId);
    navigate(
      `/villains/edit-villain/${villain?.id}?name=${villain?.name}&description=${villain?.description}`
    );
  };

  type VillainProperty =
    | Villain["name"]
    | Villain["description"]
    | Villain["id"];

  /** returns a boolean whether the villain properties exist in the search field */
  const searchExists = (searchProperty: VillainProperty, searchField: string) =>
    String(searchProperty).toLowerCase().indexOf(searchField.toLowerCase()) !==
    -1;

  /** given the data and the search field, returns the data in which the search field exists */
  const searchProperties = (searchField: string, data: Villain[]) =>
    [...data].filter((item: Villain) =>
      Object.values(item).find((property: VillainProperty) =>
        searchExists(property, searchField)
      )
    );

  /** filters the villains data to see if the any of the properties exist in the list */
  const handleSearch =
    (data: Villain[]) => (event: ChangeEvent<HTMLInputElement>) => {
      const searchField = event.target.value;

      return startTransition(() =>
        setFilteredVillains(searchProperties(searchField, data))
      );
    };

  return (
    <div
      style={{
        opacity: isPending ? 0.5 : 1,
        color: isStale ? "dimgray" : "black",
      }}
    >
      {deferredVillains.length > 0 && (
        <div className="card-content">
          <span>Search </span>
          <input data-cy="search" onChange={handleSearch(deferredVillains)} />
        </div>
      )}
      &nbsp;
      <ul data-cy="villain-list" className="list">
        {filteredVillains.map((villain, index) => (
          <li data-cy={`villain-list-item-${index}`} key={villain.id}>
            <div className="card">
              <CardContent
                name={villain.name}
                description={villain.description}
              />
              <footer className="card-footer">
                <ButtonFooter
                  label="Delete"
                  IconClass={FaRegSave}
                  onClick={handleDeleteVillain(villain)}
                />
                <ButtonFooter
                  label="Edit"
                  IconClass={FaEdit}
                  onClick={handleSelectVillain(villain.id)}
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

## Özet ve Çıkarımlar

- Context API, değeri her bileşenden açıkça geçirmeye gerek kalmadan bileşen ağacının derinliklerine geçirmemizi sağlar.
- Özel bir kanca kullanın, durumu ve etkileri kancanın içinde yönetin ve bileşenlerin ihtiyaç duyduğu değerleri döndürün, bu değer ve setValue olabilir.
- `cy.intercept` ve MSW ile, bir ağ isteğinin dışarı çıktığını kontrol ettik ve işlemin bir kancanın çağrılmasına neden olduğunu kontrol ettik. Sonuç olarak, kancaların değiştirilmesinin testler üzerinde veya işlevsellik üzerinde hiçbir etkisi olmadı. İşte bu yüzden biraz daha yüksek bir soyutlama düzeyinde test etmek istiyoruz ve uygulamanın sonuçlarını uygulama ayrıntılarına kıyasla doğrulamak istiyoruz.
