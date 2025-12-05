# 🚀 Dokumentációs Munkafolyamat Gyorshivatkozó

Ez a dokumentum a Home Lab dokumentáció (MkDocs + Material) szerkesztési, tesztelési és publikálási lépéseit foglalja össze, valamint az ajánlott Git ágkezelést.

---

## 🐍 1. Virtuális Környezet Kezelése (`.venv`)

A munka megkezdése előtt mindig aktiválni kell a Python virtuális környezetet, ami tartalmazza az MkDocs és a Material témát.

| Feladat | Parancs | Megjegyzés |
| :--- | :--- | :--- |
| **Indítás** (Aktiválás) | `source .venv/bin/activate` | Belépés az MkDocs-ot tartalmazó környezetbe. **(Ezt kell futtatni!)** |
| **Leállítás** (Deaktiválás) | `deactivate` | Kilépés a projektkörnyezetből, ha befejezted a munkát. |

---

## 💻 2. Szerkesztés és Helyi Tesztelés

A tartalom szerkesztése a Markdown fájlokban történik. A helyi szerverrel azonnal ellenőrizheted az eredményt.

### Szerkesztő Szerver Indítása

Futtasd ezt a parancsot a projekt gyökerében (miután aktiváltad a `.venv`-t):

```bash
mkdocs serve
```

* **Elérés:** Nyisd meg a böngészőben: `http://127.0.0.1:8000/`
* **Visszajelzés:** Minden fájlmentés után a weboldal automatikusan frissül.

---

## 🔄 3. Publikálás és Verziókezelés (Git Flow)

Ez a folyamat biztosítja, hogy a módosítások először a **`main`** (forrás) ágba kerüljenek, majd onnan a **`gh-pages`** (publikus) ágra.

### 3.1 Ajánlott Folyamat

Ha nem a `main` ágban dolgozol, a publikálás előtt mindig olvaszd be a változásokat a fő ágba:

| Lépés | Parancsok | Magyarázat |
| :--- | :--- | :--- |
| **Visszaváltás** | `git checkout main` | Térj vissza a fő forráságra. |
| **Olvasztás** | `git merge [a-saját-ágad-neve]` | Egyesítsd a módosításokat a főágba. |
| **Feltöltés** | `git push origin main` | Töltsd fel a forrásfájlokat a GitHubra. |
| **Publikálás** | `mkdocs gh-deploy` | **Generálja és feltölti** a generált weboldalt a `gh-pages` ágra. |

---

> 💡 **Tipp:** Ha csak a `main` ágban dolgozol, a folyamat egyszerűsíthető a következőkre:

> 1. `mkdocs serve`
> 2. Szerkesztés
> 3. `git commit -m "Frissítés"`
> 4. `mkdocs gh-deploy`
