# Moduli 05: Status Codes & Response Models 🛡️

<span class="kenyan-tag">Moduli 05</span>

A good developer wa backend haishii tu kwenye kuandika code inayofanya kazi; anahakikisha kwamba:
1. API yake inarudisha **HTTP Status Codes sahihi** zinazofahamika internationally (mfano: `201 Created` badala ya `200 OK` kila mahali).
2. Data privacy (kama vile passwords za watumiaji au internal API keys) **does not leak** at all kwa client kupitia responses.

Kwa module hii, tutaona venye tutaweka status codes kwa urahisi na jinsi ya kutumia **`response_model`** kuprotect data privacy zako.

---

## 1. HTTP Status Codes za Msingi Ni Zipi? 🚥

HTTP Status Codes ni number ya digits tatu ambazo server hutuma kumwambia client kilichotokea:

| Code Range | Maana | Mifano Maarufu |
| :--- | :--- | :--- |
| **2xx (Success)** | Kazi imefanikiwa bila matatizo | `200 OK` (kusoma data), `201 Created` (kuunda kitu kipya), `204 No Content` (kufuta kitu). |
| **3xx (Redirection)** | Client aende URL nyingine | `301 Moved Permanently`, `307 Temporary Redirect`. |
| **4xx (Client Error)** | Error inatoka kwa aliyetuma request | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`. |
| **5xx (Server Error)** | Error iko ndani ya server yako | `500 Internal Server Error` (bug kwenye code yako au database imezimika). |

---

## 2. Kuweka Status Code Kwenye Endpoint 🏷️

Kwa FastAPI, default status code ya kila endpoint huwa ni `200 OK`. Lakini unapotengeneza kitu kipya (kama kufungua account ya user au kuweka order), standard ya kimataifa ni kurudisha **`201 Created`**.

FastAPI inakupea module ya `status` iliyo na majina yote safi ya status codes ili usilazimike kumemorize namba:

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class Order(BaseModel):
    customer_name: str
    amount: float
    item: str

# Tumia status_code=status.HTTP_201_CREATED:
@app.post("/orders/", status_code=status.HTTP_201_CREATED)
def place_order(order: Order):
    return {
        "order_id": 9842,
        "status": "confirmed",
        "details": order.model_dump()
    }
```

Client akituma request hii, browser au app yako itapokea status code ya `201 Created` badala ya `200 OK`.

---

## 3. Siri ya `response_model` (Kuficha Data za Siri) 🔐

Fikiria user anajiandikisha kwa website yako:
1. Client anapaswa kutuma: `username`, `email`, na `password`.
2. Server inapaswa kuhifadhi password (baada ya kui-hash).
3. **LAKINI server haipaswi kurudisha password hiyo kamwe kwa response JSON!** Hata ikiwa hashed, kurudisha password ni security flaw kubwa sana.

Hapa ndipo **`response_model`** inakuwa your hero:

```python
from fastapi import FastAPI, status
from pydantic import BaseModel, EmailStr

app = FastAPI()

# 1. Schema ya data inayoingia kutoka kwa client (Ina password):
class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: str | None = None

# 2. Schema ya data inayorudishwa kwa client (Haina password kabisa!):
class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None

# Tunatumia response_model=UserOut:
@app.post("/register/", response_model=UserOut, status_code=status.HTTP_201_CREATED)
def register_user(user: UserIn):
    # Hapa unai-save kwa database (pamoja na password iliyofanyiwa hashing):
    # fake_db_save(user)
    
    # Hata tukirudisha ile object nzima yenye password:
    return user
```

### What is happening here?
* Angalia kwa makini: Ndani ya function tulifanya `return user` (ambayo ina `password` ndani yake).
* **LAKINI** kwa sababu tuliweka `response_model=UserOut`:
  FastAPI inachukua data hiyo, inaikagua dhidi ya `UserOut`, inaitupa password jalalani, na inamtumia client JSON safi yenye `username`, `email`, na `full_name` pekee!
* Hii inazuia kabisa uwezekano wa developer kusahau na ku-leak private data  kwa user.

---

## 4. `response_model_exclude_unset` (Kuondoa Default Values Zisizotumika) 🧹

Wakati mwingine model yako inakuwa na option fields nyingi za  zenye default values (kama `tax: float = 0.0` au `discount: float | None = None`).

Ukitaka JSON inayorudishwa iwe na zile fields tu ambazo user aliziweka mwenyewe:

```python
class Item(BaseModel):
    name: str
    price: float
    description: str | None = None
    tax: float = 0.0

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
def get_clean_item(item_id: int):
    # Ikiwa 'description' haikuwekwa, haitatokea kwa output JSON kabisa
    return Item(name="Maziwa ya Mala", price=120.0)
```

Output itakuwa fupi na safi:
```json
{
  "name": "Maziwa ya Mala",
  "price": 120.0
}
```

---

## Summary ya Module ya 05 📌
* Tumia **`status_code=status.HTTP_201_CREATED`** au **`HTTP_204_NO_CONTENT`** kufuata correct standards ya REST APIs.
* **`response_model`** inahakikisha data inayorudi kwa client inachujwa na kuendana na schema safe.
* Hutenganisha schemas za kuingiza data (`UserIn`) na za kutoa data (`UserOut`) ili kulinda sensitive information kama passwords.
* Swagger UI (`/docs`) inatumia `response_model` kutengeneza mifano sahihi ya response ya API.

Kwa module inayofuata, tutaangalia jinsi ya kushughulikia makosa na hitilafu za kibiashara kwa kutumia **`HTTPException`** bila kuruhusu server yako ku-crash!

👉 Endelea na **[Module 06: Error Handling & HTTPException](06_error_handling_and_exceptions.md)**.
