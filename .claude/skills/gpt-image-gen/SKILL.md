---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt. השתמש בסקיל הזה בכל פעם שצריך להפיק תמונה מטקסט — שולח את ה-prompt ל-API ושומר את התמונה כקובץ PNG. משמש בעיקר את יובל (מעצב התמונות).
---

# gpt-image-gen — יצירת תמונות דרך OpenAI Images API

סקיל זה אחראי על דבר אחד: לקבל `prompt` (ונתיב פלט), לשלוח אותו ל-OpenAI Images API,
ולשמור את התמונה שחזרה כקובץ PNG.

## המודל — `gpt-image-2`

⚠️ **אל תשנה את שם המודל. אל תציע אלטרנטיבות.**

- `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI, שיצא ב-21 באפריל 2026.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אל** תחליף ל-`dall-e-3`, `gpt-image-1`, או כל מודל אחר. השתמש בדיוק ב-`gpt-image-2`.
- אם הקריאה ל-API נכשלת — הבעיה היא כמעט תמיד ב-`OPENAI_API_KEY` (חסר/לא תקין)
  או ב-parameters של הבקשה, **לא** בשם המודל. אל "תתקן" את שם המודל.

## דרישות מוקדמות

- מפתח `OPENAI_API_KEY` מוגדר בקובץ `.env` בשורש הפרויקט.
- `curl` זמין (קיים ב-Git Bash). `jq` מומלץ אך לא חובה — יש fallback ל-python.

## איך מריצים

הסקיל רץ דרך Bash (Git Bash על Windows). השלבים:

1. **טען את המפתח מ-`.env`** (אל תכתוב את המפתח inline בשום מקום):

```bash
set -a
source .env
set +a
```

2. **שלח את הבקשה ושמור את התמונה.** החלף את `<the prompt>` ואת `<output-path>`:

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > /tmp/gpt-image-response.json
```

3. **חלץ ושמור את ה-PNG.** מסלול ראשי עם `jq`:

```bash
jq -r '.data[0].b64_json' /tmp/gpt-image-response.json | base64 --decode > "<output-path>.png"
```

**Python fallback** (כש-`jq` לא מותקן ב-Git Bash):

```bash
python -c "import json,base64,sys; d=json.load(open('/tmp/gpt-image-response.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"
```

> אם `python` לא נמצא, נסה `python3`.

## פרמטרים נתמכים בגוף הבקשה

| שדה | ערך ברירת מחדל | הערות |
|------|----------------|--------|
| `model` | `gpt-image-2` | **קבוע — לא לשנות** |
| `prompt` | — | תיאור התמונה (חובה) |
| `size` | `1024x1024` | גודל התמונה |
| `quality` | `medium` | איכות הרינדור |
| `output_format` | `png` | פורמט הפלט |

## אימות

לאחר הריצה ודא שהקובץ נוצר וגודלו גדול מ-0:

```bash
test -s "<output-path>.png" && echo "OK: $(wc -c < "<output-path>.png") bytes" || echo "FAILED: empty or missing file"
```

## טיפול בשגיאות

- **שגיאת אימות / 401** → בדוק ש-`OPENAI_API_KEY` קיים ותקין ב-`.env`.
- **קובץ פלט ריק (0 bytes)** → בדוק את `/tmp/gpt-image-response.json` להודעת שגיאה מה-API
  (למשל `jq '.error' /tmp/gpt-image-response.json`); כנראה בעיית parameters או מכסה.
- **אל תסיק** משגיאה כלשהי שצריך להחליף את שם המודל `gpt-image-2`.
