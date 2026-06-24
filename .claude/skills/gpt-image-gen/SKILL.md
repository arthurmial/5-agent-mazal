---
skill: gpt-image-gen
description: >
  מייצר תמונה בעזרת OpenAI Images API (מודל gpt-image-2) ושומר אותה כ-PNG.
  מקבל: prompt (טקסט), output_path (נתיב לשמירה).
  מחזיר: נתיב התמונה שנשמרה, או הודעת שגיאה.
---

# gpt-image-gen — יצירת תמונות עם OpenAI

סקיל זה עוטף את OpenAI Images API ומייצר תמונה בודדת לפי פרומפט.

**מודל: `gpt-image-2`** — אל תשנה לשם אחר. זהו מודל אמיתי וקיים של OpenAI (יצא 21 באפריל 2026). אם יש שגיאה, הבעיה היא ב-API key או בפרמטרים, לא בשם המודל.

---

## דרישות מוקדמות

משתנה סביבה `OPENAI_API_KEY` חייב להיות מוגדר ב-`.env`.

---

## שימוש

הסקיל מקבל שני פרמטרים:
- `prompt` — תיאור מפורט של התמונה (אנגלית עדיפה לתוצאות טובות יותר)
- `output_path` — נתיב מלא לשמירת קובץ ה-PNG (לדוגמה: `yuval/outputs/2026-06-24-hero.png`)

---

## הפעלה — שלב 1: טעינת OPENAI_API_KEY

```bash
export $(grep -v '^#' .env | xargs)
if [ -z "$OPENAI_API_KEY" ]; then
  echo "ERROR: OPENAI_API_KEY is not set. Add it to your .env file."
  exit 1
fi
```

---

## הפעלה — שלב 2: קריאה ל-API ושמירה לקובץ זמני

```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"model\":\"gpt-image-2\",\"prompt\":\"PROMPT_HERE\",\"size\":\"1024x1024\",\"quality\":\"medium\",\"output_format\":\"png\"}" \
  > /tmp/gpt-image-response.json
```

---

## הפעלה — שלב 3: פענוח base64 ושמירת PNG

**דרך ראשית (jq):**

```bash
jq -r '.data[0].b64_json' /tmp/gpt-image-response.json | base64 --decode > "OUTPUT_PATH_HERE"
```

**גיבוי (Python) — אם jq אינו מותקן:**

```bash
python -c "
import json, base64
with open('/tmp/gpt-image-response.json') as f:
    data = json.load(f)
b64 = data['data'][0]['b64_json']
with open('OUTPUT_PATH_HERE', 'wb') as out:
    out.write(base64.b64decode(b64))
"
```

---

## הפעלה — שלב 4: ולידציה

```bash
if [ -f "OUTPUT_PATH_HERE" ] && [ -s "OUTPUT_PATH_HERE" ]; then
  echo "SUCCESS: Image saved to OUTPUT_PATH_HERE"
else
  echo "ERROR: File not created or is empty."
  cat /tmp/gpt-image-response.json
  exit 1
fi
```

---

## טיפול בשגיאות

| שגיאה | משמעות | פתרון |
|-------|--------|--------|
| `OPENAI_API_KEY is not set` | המפתח חסר | הוסף ל-.env |
| `invalid_api_key` | מפתח שגוי | בדוק את ערך OPENAI_API_KEY |
| `billing_hard_limit_reached` | חשבון חסום | בדוק חשבון OpenAI |
| קובץ ריק / לא נוצר | כשל בפענוח | בדוק את /tmp/gpt-image-response.json |
| `model_not_found` | שגיאת שם מודל | לא צפוי — בדוק הגדרות חשבון OpenAI |

---

## הערות

- הפרומפט צריך להיות באנגלית לתוצאות מיטביות; תרגם פרומפטים בעברית לפני שליחה.
- קובץ הפלט הזמני `/tmp/gpt-image-response.json` ניתן למחיקה לאחר הריצה.
