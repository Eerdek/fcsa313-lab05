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
