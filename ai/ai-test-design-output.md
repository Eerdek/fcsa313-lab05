# AI туслахын гаралт (нэмэлт даалгавар)

Prompt (зөвхөн `server.js`-ийг өгсөн, миний collection/README-г харуулаагүй):

> Энэ API-гийн POST /registrations функцийн сонголтуудыг тодорхойлж, сонголт бүрийг
> эквивалент ангид хуваагаад, ангиудын хослолоор Postman тестүүд үүсгэ

Доорх нь AI-ийн хариуны товчлол (хүснэгт, oracle-ууд нь эх хэвээрээ).

## AI-ийн сонголт ба ангиуд

| Choice | Classes |
|---|---|
| A. studentID in request | A1 present and the student exists; A2 present but the student does not exist; A3 missing, empty or null |
| B. Student status | B1 `active`; B2 not active (`inactive` or `suspended`). Only meaningful when A1 holds. |
| C. courseID in request | C1 present and the course exists; C2 present but the course does not exist; C3 missing, empty or null |
| D. Course prerequisites | D1 none (empty list); D2 all satisfied; D3 partly satisfied; D4 none satisfied |
| E. Request body format | E1 valid JSON; E2 malformed JSON |

## AI-ийн тест кейсүүд

| # | Name | Classes | Expected |
|---|---|---|---|
| T1 | Valid, no prerequisites | A1,B1,C1,D1 | 201 OK + numeric registrationID |
| T2 | Valid, prerequisites all satisfied | A1,B1,C1,D2 | 201 OK |
| T3 | Student does not exist | A2 | 200 ERROR_NO_STUDENT |
| T4 | Missing studentID | A3 | 400 ERROR_BAD_REQUEST |
| T5 | Inactive student | B2 | 200 ERROR_INACTIVE_STUDENT |
| T6 | Course does not exist | C2 | 200 ERROR_NO_COURSE |
| T7 | Missing courseID | C3 | 400 ERROR_BAD_REQUEST |
| T8 | Prerequisites partly met | D3 | 200 ERROR_PREREQUISITES, missing = ["m2"] |
| T9 | Prerequisites none met | D4 | 200 ERROR_PREREQUISITES, missing = ["m1","m2"] |
| T10 | Malformed JSON | E2 | 400 ERROR_BAD_JSON |
| T11 | Priority: missing student beats missing course | A2+C2 | 200 ERROR_NO_STUDENT |
| T12 | Priority: inactive student beats missing prerequisites | B2+D4 | 200 ERROR_INACTIVE_STUDENT |
| T13 | Empty string studentID | A3 | 400 ERROR_BAD_REQUEST |
| T14 | Two registrations get different IDs | A1..D1 twice | 201 both, second ID > first |

AI-ийн oracle-ийн жишээ:

```js
pm.test("status is 201", () => pm.response.to.have.status(201));
pm.test("result OK", () => pm.expect(pm.response.json().result).to.eql("OK"));
pm.test("registrationID is a number", () =>
  pm.expect(pm.response.json().registrationID).to.be.a("number"));
pm.test("content-type is JSON", () =>
  pm.response.to.have.header("Content-Type", /application\/json/));
pm.test("missing lists only m2", () =>
  pm.expect(pm.response.json().missing).to.have.members(["m2"]).and.lengthOf(1));
// setup PUT бүрд:
pm.test("setup OK", () => {
  pm.response.to.have.status(200);
  pm.expect(pm.response.json().result).to.eql("OK");
});
```

AI-ийн өөрийн анхааруулга: "The oracles above follow the code ... Check the Lecture 5 spec to
confirm that is the intended semantics." Мөн тестүүдээ ажиллуулаагүй гэдгээ дурдсан.
