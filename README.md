# Mwongozo Kamili wa FastAPI (Kenyan Edition 🇰🇪)

<span class="kenyan-tag">Toleo Rasmi la Kiswahili cha Kenya</span>

Karibu kwenye **Mwongozo Kamili wa FastAPI** ulioandikwa kwa **Kiswahili cha Kenya bila "deep translation"**. 

Hapa hatutafsiri keywords za Python kuwa Kiswahili kigumu cha vitabu kama *"safu"*, *"kigezo"*, au *"kizigeuzi cha data"*. Badala yake, tunaelezea jinsi ya kujenga backend APIs zenye kiwango cha kimataifa kwa lugha ya kawaida tunayoongea maofisini na kwenye tech meetups za Nairobi.

---

## FastAPI Ni Nini na Kwanini Iko Moto? 🔥

**FastAPI** ni framework ya kisasa ya Python inayotumika kujenga RESTful APIs na web backends. Ni moja ya frameworks zinazokua kwa kasi zaidi duniani (ikitumiwa na makampuni kama Microsoft, Netflix, na Uber).

### Sababu Kuu 4 Zinazofanya Ma-dev Kuipenda:
1. **Kasi ya Juu Sana (High Performance):** FastAPI inashindana kwa spidi na frameworks za **Node.js** na **Go** kwa sababu imejengwa juu ya `Starlette` na inatumia Python `async/await` (asynchronous programming).
2. **Automatic Interactive Documentation:** Mara tu unapoandika endpoint yako, FastAPI inakutengenezea web pages mbili za testing kiotomatiki:
   - **Swagger UI (`/docs`):** Ambapo unaweza ku-test kila endpoint na kuona response zake live kwenye browser.
   - **ReDoc (`/redoc`):** Documentation safi ya kusoma format na data schemas.
3. **Data Validation ya Kiotomatiki (Pydantic):** Huwezi kupata data chafu kwenye database yako. Client akituma string badala ya integer au akisahau field muhimu, FastAPI inakataa mara moja na kurudisha error message safi ya `422 Unprocessable Entity` bila wewe kuandika `if/else` ndefu.
4. **Intuitive & Easy to Learn:** Inategemea **Python Standard Type Hints** pekee. Hakuna syntax ngeni ya kujifunza — ukiijua Python ya kawaida, tayari unaijua FastAPI!

---

## Muundo wa Moduli za Mafunzo 📚

Mwongozo huu umegawanywa katika moduli 9 zilizopangwa kimantiki:

* **[Moduli 01: Utangulizi & Setup ya Mazingira](01_introduction_and_setup.md)** — Virtual environments, installation, endpoint ya kwanza (`@app.get("/")`), na kuendesha server kwa `uvicorn`.
* **[Moduli 02: Path Parameters & Type Hints](02_path_parameters_and_types.md)** — Kupokea IDs kwenye URL, type hints, na jinsi validation inavyozuia bugs.
* **[Moduli 03: Query Parameters & Filtering](03_query_parameters.md)** — Kufanya pagination (`?skip=0&limit=10`), optional parameters, na default values.
* **[Moduli 04: Request Body & Pydantic Models](04_request_body_and_pydantic.md)** — Kupokea JSON payloads (POST requests), data validation kwa `BaseModel`, na nested models.
* **[Moduli 05: Status Codes & Response Models](05_status_codes_and_response_models.md)** — Kutumia HTTP status codes sahihi (`201`, `200`, `204`) na kuficha sensitive data (kama passwords) kwa `response_model`.
* **[Moduli 06: Error Handling & HTTPException](06_error_handling_and_exceptions.md)** — Jinsi ya ku-raise errors za maana (`404 Not Found`, `400 Bad Request`) badala ya server ku-crash na `500`.
* **[Moduli 07: Dependency Injection (`Depends`)](07_dependency_injection.md)** — Silaha kuu ya FastAPI: ku-inject database sessions, user authentication, na shared logic.
* **[Moduli 08: CORS & Kuunganisha Frontend](08_cors_and_frontend_integration.md)** — Kurekebisha browser CORS errors ili React, Vue, au Flutter ziongee na API yako.
* **[Moduli 09: Real-World Project: M-Pesa Callback & Store API](09_real_world_mpesa_api_project.md)** — Mradi kamili wa vitendo wa kujenga e-commerce CRUD na M-Pesa payment webhook endpoint.

---

## Mahitaji ya Kuanza (Prerequisites) 🛠️

Ili ufaidike kikamilifu na mwongozo huu, unahitaji:
- **Python 3.10 au zaidi** ikiwa imewekwa kwenye computer yako.
- Code editor yoyote (tunapendekeza **VS Code** au **Cursor**).
- Hamu ya kujenga APIs za kisasa bila stress!

👉 Anza sasa na **[Moduli 01: Utangulizi & Setup ya Mazingira](01_introduction_and_setup.md)**.
