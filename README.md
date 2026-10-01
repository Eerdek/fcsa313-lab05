# Лаборатори №5 — API систем тест: Postman ба Newman

**Оюутан:** Эрдэнэбаяр
**Оюутны код:** B232270136
**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

---

## 1. Ажлын орчин

```
$ node -v
v24.18.0

$ newman -v
6.2.2
```

| Зүйл | Утга |
|---|---|
| Collection хэрэгсэл | Postman Collection v2.1 формат (`lab05-collection.json`) |
| Ажиллуулагч | Newman 6.2.2 (Apache 2.0, нээлттэй эх) |
| Сервер | Багшийн өгсөн `server.js` — цэвэр Node.js `http`, нэмэлт сангүй |
| ҮС | Windows 11 Pro (26200), Git Bash (exit code-ийг `${PIPESTATUS[0]}`-оор авсан) |
| Бай (target) | `http://localhost:3000` — `{{baseUrl}}` collection variable |

---

## 2. Даалгавар 1 — Анхны хүсэлт ба oracle

`S1` хавтас: `PUT /students/B231234567` (active, `["CS201"]`) → `PUT /courses/CS313`
(`prerequisites: ["CS201"]`) → `POST /registrations`. Oracle-ууд: статус `201`, `result = "OK"`,
мөн `registrationID` нь эерэг **тоо** эсэх (яг утгыг нь шалгахгүй — сервер санах ойд
хуримтлуулдаг тул дахин ажиллуулахад өөрчлөгдөнө).

---

## 3. Даалгавар 2 — Тест дизайн (Лекц 5-ын 5 алхам)

### 3.1. Бие даан тестлэгдэх функц

`POST /registrations` — оюутныг хичээлд бүртгэх. Оролт: JSON бие `{studentID, courseID}` ба
серверийн төлөв (оюутны бичлэг, хичээлийн бичлэг). Гаралт: HTTP статус + `result`
(+ `registrationID` эсвэл `missing`).

### 3.2. Сонголтууд ба төлөөлөх утгууд (эквивалент анги)

| # | Сонголт (параметр / орчны нөхцөл) | Эквивалент анги | Төлөөлөх утга (тестэд хэрэглэсэн) | Хүлээгдэх үр дүнд нөлөө |
|---|---|---|---|---|
| C1 | studentID-гийн хүчинтэй байдал | **идэвхтэй** (`status: "active"`) | `B231234567` | үргэлжлүүлнэ |
| | | **идэвхгүй** (`status: "inactive"`) | `B231234003` | `ERROR_INACTIVE_STUDENT` |
| | | **байхгүй** (PUT хийгдээгүй) | `B000000002` | `ERROR_NO_STUDENT` |
| | | [хязгаар] status орхисон → анхдагч `active` | `B231234014`, бие `{}` | үргэлжлүүлнэ |
| C2 | Оюутны үзсэн хичээлүүд (coursesTaken) | **урьдач нөхцөлийг хангана** | `["CS201"]` | үргэлжлүүлнэ |
| | | **хангахгүй — огт үзээгүй** | `[]` | `ERROR_PREREQUISITES` |
| | | **хангахгүй — зарим нь үзсэн** | `["CS101"]` (шаардлага CS101+CS201) | `ERROR_PREREQUISITES`, `missing:["CS201"]` |
| C3 | courseID-гийн хүчинтэй байдал | **байгаа** | `CS313` | үргэлжлүүлнэ |
| | | **байхгүй** | `NOPE404` | `ERROR_NO_COURSE` |
| C4 | Хичээлийн урьдач нөхцөл (prerequisites) | **бүгдийг үзсэн** | `["CS201"]` vs `["CS201"]` | `201 OK` |
| | | **огт үзээгүй** | `["CS201"]` vs `[]` | `ERROR_PREREQUISITES` |
| | | **зарим нь** | `["CS101","CS201"]` vs `["CS101"]` | `ERROR_PREREQUISITES` |
| | | [хязгаар] урьдач нөхцөлгүй | `[]` | `201 OK` (coursesTaken хоосон ч) |
| C5 | Хүсэлтийн биеийн хэлбэр | **бүрэн, зөв JSON** | `{"studentID":…,"courseID":…}` | дээрх логик |
| | | [хязгаар] талбар дутуу | `{"studentID":"B231234010"}` | `400 ERROR_BAD_REQUEST` |
| | | [хязгаар] талбар байгаа ч хоосон мөр | `{"studentID":"","courseID":…}` | `400 ERROR_BAD_REQUEST` |
| | | [хязгаар] буруу JSON | `{"studentID": "B231234012", "courseID": "CS313"` (хаалт дутуу) | `400 ERROR_BAD_JSON` |

**Боломжгүй / утгагүй хослолууд.** C2 ба C4 нь зөвхөн *оюутан идэвхтэй* ба *хичээл байгаа*
үед л утгатай: оюутан байхгүй эсвэл хичээл байхгүй үед "урьдач нөхцөл хангана/хангахгүй"
гэдэг сонголт огт үнэлэгддэггүй. Мөн C5-ийн 400 ангиуд (буруу JSON, талбар дутуу) нь
C1–C4-ийн бүх хослолыг "дарна" — сервер өгөгдлийн санд хандахаас өмнө буцаадаг тул тэдгээрийг
C1–C4-тэй хослуулах шаардлагагүй. Иймд бүрэн декарт үржвэр (3×3×2×4×4 = 288) биш, лекцийн
стратегиар доорх 15 спецификацийг сонгосон.

### 3.3. Спецификацийн хүснэгт

| ID | Ангилал | Оюутан (C1, C2) | Хичээл (C3, C4) | Хүсэлтийн бие | Хүлээгдэх статус | Хүлээгдэх `result` | Нэмэлт oracle |
|---|---|---|---|---|---|---|---|
| S1 | Happy path | active, `["CS201"]` | `CS313`, `["CS201"]` | бүрэн | **201** | `OK` | `registrationID` эерэг тоо |
| S2 | Алдааны анги | байхгүй | байгаа, `["CS201"]` | бүрэн | **200** | `ERROR_NO_STUDENT` | `registrationID` байхгүй |
| S3 | Алдааны анги | **inactive**, `["CS201"]` | байгаа, `["CS201"]` | бүрэн | **200** | `ERROR_INACTIVE_STUDENT` | `registrationID` байхгүй |
| S4 | Алдааны анги | active, `["CS201"]` | **байхгүй** | бүрэн | **200** | `ERROR_NO_COURSE` | `registrationID` байхгүй |
| S5 | Алдааны анги | active, `[]` (огт үзээгүй) | байгаа, `["CS201"]` | бүрэн | **200** | `ERROR_PREREQUISITES` | `missing = ["CS201"]` |
| S6 | Алдааны анги (зарим нь) | active, `["CS101"]` | байгаа, `["CS101","CS201"]` | бүрэн | **200** | `ERROR_PREREQUISITES` | `missing = ["CS201"]` |
| S7 | Давхар алдаа | **inactive** | **байхгүй** | бүрэн | **200** | `ERROR_INACTIVE_STUDENT` | — |
| S8 | Давхар алдаа | **байхгүй** | **байхгүй** | бүрэн | **200** | `ERROR_NO_STUDENT` | — |
| S9 | Хязгаар | active, `[]` | байгаа, `[]` (урьдач нөхцөлгүй) | бүрэн | **201** | `OK` | `registrationID` эерэг тоо |
| S10 | Хязгаар | active | — | `courseID` дутуу | **400** | `ERROR_BAD_REQUEST` | — |
| S11 | Хязгаар | — | байгаа | `studentID: ""` | **400** | `ERROR_BAD_REQUEST` | — |
| S12 | Хязгаар | — | — | буруу JSON | **400** | `ERROR_BAD_JSON` | — |
| S13 | Нэмэлт: гаралтын шалгалт | active, `["CS201"]` | байгаа, `["CS201"]` | бүрэн | **201** → GET **200** | — | GET массивт POST-ийн ID-тай мөр зөв `studentID`/`courseID`-тэй |
| S14 | Хязгаар (анхдагч утга) | PUT бие `{}` → active, `[]` | PUT бие `{}` → `[]` | бүрэн | **201** | `OK` | `registrationID` эерэг тоо |
| S15 | Ажиглалт (давтан бүртгэл) | active, `[]` | байгаа, `[]` | бүрэн, 2 удаа | **201**, **201** | — | 2 дахь ID > 1 дэх ID |

S7, S8 нь заавар болон тодруулгад "аль алдаа түрүүлэх нь бичигдээгүй" гэсэн хэсгийг тестээр
тогтоох зорилготой. Серверийн код дээр оюутны шалгалт хичээлийн шалгалтаас өмнө байгаа тул
оюутантай холбоотой алдаа түрүүлнэ гэж таамагласан бөгөөд тестээр баталгаажуулав (6-р хэсэг).

---

## 4. Даалгавар 3 — Collection (`lab05-collection.json`)

Спецификаци бүр collection-д **тусдаа хавтас** (S1 … S15) бөгөөд хавтас бүр:

1. өөрийн **setup** `PUT /students/:id`, `PUT /courses/:id` хүсэлтүүдийг өөрөө хийнэ — тест
   бүр өөр ID (`B231234003`, `CS313-S3` …) ашигладаг тул өөр тестийн өгөгдлөөс хамаардаггүй
   (test independence). PUT нь бичлэгийг дахин бичдэг тул давтан ажиллуулахад аюулгүй;
2. `POST /registrations` хийнэ;
3. **статус** ба **утга** (`result`, `missing`, `registrationID`-ийн төрөл)-ын oracle-уудыг шалгана.

`registrationID`-ийн яг утгыг хаана ч шалгаагүй: S1/S9/S14-д "эерэг тоо", S13-д POST-ийн
хариунаас авсан ID-гаар GET жагсаалтаас мөр хайна, S15-д хоёр ID-г хооронд нь харьцуулна.
`{{baseUrl}}` = `http://localhost:3000` collection variable.

---

## 5. Даалгавар 4 — Newman-аар автоматжуулах

Exit code-ийг `| tee`-ийн дараах `$?` биш, `${PIPESTATUS[0]}`-оор (newman-ийн өөрийнх) авсан.
Exit code бүрийг мөн `results/exit-*.txt` файлд хадгалсан.

### 5.1. PASS — `results/newman-pass.txt`

```bash
newman run lab05-collection.json --color off 2>&1 | tee results/newman-pass.txt
echo "exit=${PIPESTATUS[0]}"      # → exit=0
```

| Мөр | executed | failed |
|---|---|---|
| requests | 38 | 0 |
| test-scripts | 17 | 0 |
| **assertions** | **41** | **0** |

**Тестийн (assertion-ы) тоо = 41**, failed = 0, **exit code = 0**.
15 спецификаци → 15 хавтас; 38 хүсэлт = setup PUT-ууд + POST-ууд + S13-ийн GET.

### 5.2. FAIL — `lab05-collection-fail.json` → `results/newman-fail.txt`

`lab05-collection-fail.json` нь үндсэн collection-ий **тусдаа хуулбар** бөгөөд зөвхөн нэг
oracle-ийг зориуд буруу болгосон: S1-ийн `pm.response.to.have.status(201)` → `status(200)`.
Үндсэн `lab05-collection.json` зөв хэвээр.

```bash
newman run lab05-collection-fail.json --color off 2>&1 | tee results/newman-fail.txt
echo "exit=${PIPESTATUS[0]}"      # → exit=1
```

Үр дүн: assertions executed **41**, failed **1**, **exit code = 1**:

```
 1.  AssertionError  Статус код 200
                     expected response to have status code 200 but got 201
                     inside "S1 Идэвхтэй, урьдач нөхцөл хангасан оюутан бүртгүүлэхэд 201 OK / POST /registrations"
```

Энэ бол CI quality gate-ийн зарчим: oracle нэг ч унавал newman 0 биш код буцааж, pipeline-ийг зогсооно.

### 5.3. DOWN — `results/newman-down.txt`

Серверийг зогсоосны дараа (Ctrl+C / процессыг устгав) үндсэн collection-ийг ажиллуулав:

```bash
newman run lab05-collection.json --color off 2>&1 | tee results/newman-down.txt
echo "exit=${PIPESTATUS[0]}"      # → exit=1
```

Гаралтад хүсэлт бүрд `connect ECONNREFUSED 127.0.0.1:3000` мөр гарсан (requests 38 / failed 38,
assertions 41 / failed 41), **exit code = 1**.

**Интерфейсийн алдаа vs oracle-ийн алдаа:** DOWN үед хүсэлт серверт огт хүрээгүй тул хариу
байхгүй, assertion-ууд "хүлээгдсэн утга буруу" гэж биш хариугүйгээс унасан — энэ бол системтэй
холбогдох **интерфейсийн (холболтын) алдаа**; харин FAIL үед сервер зөв `201` хариу өгсөн боловч
бидний oracle буруу утга (`200`) хүлээсэн тул энэ нь **oracle-ийн алдаа**. Хоёулаа exit=1 буцаадаг
тул CI-д ялгахын тулд гаралтын `ECONNREFUSED` эсвэл `AssertionError` мөрийг харах шаардлагатай.

### 5.4. Гурван ажиллуулалтын exit code

| Ажиллуулалт | Файл | assertions executed / failed | Exit code |
|---|---|---|---|
| PASS | `results/newman-pass.txt` | 41 / 0 | **0** |
| FAIL | `results/newman-fail.txt` | 41 / 1 | **1** |
| DOWN | `results/newman-down.txt` | 41 / 41 (ECONNREFUSED) | **1** |
