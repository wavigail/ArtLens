# ArtLens — מפרט מערכת (System Specification)

## תיאור כללי

**ArtLens** הוא אתר סטטי (Static Website) שמאפשר למשתמשים לתאר חפץ שמסקרן אותם באמצעות שלוש מילות מפתח, ולגלות יצירות אמנות תואמות מתוך אוסף של מעל 300,000 יצירות במוזיאון The Art Institute of Chicago.

האתר משלב שלושה רכיבים מרכזיים:
1. **שימוש ב-API חיצוני** — חיפוש יצירות אמנות דרך Art Institute of Chicago API (חינמי, CC0)
2. **שימוש ביכולת מכשיר** — גישה למצלמת הטלפון באמצעות HTML5 Camera Input
3. **זיהוי תמונה ב-AI** — ניתוח התמונה שצולמה באמצעות Hugging Face BLIP (image captioning) וחיפוש אוטומטי לפי התיאור

## קהל יעד

מורים ותלמידי מגמת אמנויות העיצוב, גיל 16 ומעלה.

---

## ארכיטקטורה

### סוג הפרויקט
אתר סטטי — קובץ HTML יחיד. ללא framework, ללא build tools, ללא server-side.

### מבנה קבצים
```
ArtLens/
├── index.html        ← האתר (HTML + CSS + JS inline)
└── docs/
    └── spec.md       ← מסמך זה
```

### טכנולוגיות
| רכיב | טכנולוגיה |
|---|---|
| Markup | HTML5, RTL |
| Styling | CSS3 (Custom Properties, Flexbox, Media Queries) |
| Logic | Vanilla JavaScript (ES2017+, async/await) |
| Fonts | Google Fonts — Noto Serif Hebrew, Assistant |
| API | Art Institute of Chicago Public API |
| Camera | HTML5 `<input type="file" accept="image/*" capture="environment">` |
| AI | Hugging Face Inference API — BLIP (Salesforce/blip-image-captioning-large) |

---

## API — Art Institute of Chicago

### Base URL
```
https://api.artic.edu/api/v1
```

### Endpoints בשימוש

#### 1. חיפוש יצירות
```
GET /artworks/search?q={query}&limit=10&fields={fields}
```
**פרמטרים:**
- `q` — מחרוזת חיפוש באנגלית (מתורגמת מעברית, ראו סעיף תרגום)
- `limit` — מספר תוצאות מקסימלי (10)
- `fields` — שדות נדרשים:
  - `id` — מזהה יצירה
  - `title` — שם היצירה
  - `artist_display` — שם האמן
  - `date_display` — תאריך
  - `medium_display` — חומר וטכניקה
  - `image_id` — מזהה תמונה ל-IIIF
  - `color` — נתוני צבע דומיננטי (HSL)
  - `style_title` — סגנון אמנותי
  - `classification_title` — סיווג
  - `material_titles` — רשימת חומרים

#### 2. יצירה בודדת (עבור Fallback)
```
GET /artworks/{id}?fields={fields}
```

#### 3. תמונות (IIIF)
```
https://www.artic.edu/iiif/2/{image_id}/full/843,/0/default.jpg
```

### CORS
ה-API מחזיר `Access-Control-Allow-Origin: *` — תומך בקריאות ישירות מהדפדפן.

### רישיון
CC0 (Public Domain). ללא צורך ב-API key.

---

## זרימת משתמש (User Flow)

```
┌─────────────┐
│  מסך פתיחה  │
│  Hero + Logo │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│ צילום חפץ       │  ← אופציונלי, HTML5 Camera
│ (לא חובה)       │
└──────┬──────────┘
       │
       ▼
┌─────────────────────────┐
│ 3 שדות מילות מפתח:      │
│ 1. מה אתם רואים?        │  ← חפץ
│ 2. איזו אווירה?         │  ← מצב רוח
│ 3. איזו צורה?           │  ← צורה
│ + כפתורי דוגמה (Chips)   │
└──────┬──────────────────┘
       │
       ▼ "גלו יצירה"
       │
  ┌────┴────┐
  │ API Call │
  └────┬────┘
       │
  ┌────┴────────┐
  │             │
  ▼             ▼
נמצאה        לא נמצאה
תוצאה       תוצאה
  │             │
  ▼             ▼
┌────────┐  ┌──────────────┐
│ תוצאה  │  │ הודעת Fallback│
│ עם     │  │ + יצירה      │
│ תמונה  │  │ רנדומלית     │
│ פלטה   │  └──────────────┘
│ פרטים  │
└────────┘
       │
       ▼
  "חיפוש חדש"
```

---

## רכיבי ממשק (UI Components)

### 1. Hero Header
- רקע כהה (`#1A1A1A`)
- לוגו SVG — זכוכית מגדלת בצורת מסגרת תמונה
- Tagline: "Discover by Design"
- טקסט מבוא בעברית

### 2. Camera Section
- כפתור צילום עם אייקון מצלמה
- הודעת הרשאה מרגיעה
- Preview של התמונה שצולמה
- לא חובה — רכיב UX בלבד

### 3. Search Fields
שלושה שדות טקסט חופשיים עם כפתורי דוגמה (Chips):

| שדה | תיאור | דוגמאות |
|---|---|---|
| מה אתם רואים? | חפץ | כיסא עץ, כוס זכוכית, טבעת כסף |
| איזו אווירה? | מצב רוח | דרמטי, נוסטלגי, שקט ורך |
| איזו צורה? | צורה | עגול וחלק, חד וגיאומטרי, מאסיבי |

כל Chip מכיל `data-en` עם תרגום לאנגלית לחיפוש ב-API.

### 4. Result Card
- **ספירת תוצאות** — "נמצאו X יצירות דומות" (מספר אמיתי מה-API)
- **תוצאה ראשית** — תמונת היצירה (IIIF, 843px רוחב), כותרת, אמן, תאריך
- פלטת צבעים (4 עיגולים, מחושבת מנתוני ה-API)
- כפתור "עוד על היצירה" — פותח פאנל עם: חומר וטכניקה, סגנון, חומרים, סיווג
- קישור לאתר המוזיאון
- **עוד תוצאות** — עד 3 כרטיסים נוספים עם תמונה מוקטנת, כותרת ואמן. לחיצה מציגה את היצירה המלאה

### 5. Fallback System
כשאין תוצאה תואמת, מוצגת:
- הודעה הומוריסטית בעברית (אחת מ-4 אפשרויות רנדומליות)
- יצירה מאוצרת מראש (ID: 14763, 49714, 76395, 104463)

---

## מנגנון תרגום (Hebrew → English)

ה-API מבין רק אנגלית. שני מנגנוני תרגום:

### 1. Chip Translation
כל כפתור דוגמה מכיל `data-en` עם ערכים באנגלית:
```html
<button class="chip" data-field="f-obj" data-en="wooden chair">כיסא עץ</button>
```
כשמשתמש לוחץ על Chip, השאילתה נבנית מה-`data-en`.

### 2. Dictionary Fallback
כשמשתמש מקליד בחופשיות, מילון `dict` מתרגם כל מילה בעברית לאנגלית:
```javascript
const dict = {
  'כיסא': 'chair',
  'עץ': 'wood wooden',
  'דרמטי': 'dramatic bold',
  // ...כ-50 ערכים
};
```

---

## מצלמה (Camera Capability)

### מימוש
```html
<input type="file" accept="image/*" capture="environment">
```

### התנהגות
1. לחיצה על "צלמו חפץ" פותחת את מצלמת המכשיר
2. התמונה מוצגת כ-Preview באתר
3. התמונה נשלחת ל-Hugging Face BLIP API לניתוח אוטומטי
4. ה-AI מחזיר תיאור טקסטואלי באנגלית (למשל: "a wooden chair in a dark room")
5. התיאור משמש כשאילתת חיפוש אוטומטית ב-API של המוזיאון
6. אם ה-AI לא זמין, המשתמש יכול לתאר ידנית בשדות למטה

### AI — Hugging Face BLIP
```
POST https://api-inference.huggingface.co/models/Salesforce/blip-image-captioning-large
Authorization: Bearer {HF_TOKEN}
Body: image blob
Response: [{ "generated_text": "a description of the image" }]
```
- **מודל**: BLIP (Bootstrapping Language-Image Pre-training)
- **חינמי**: Hugging Face Free Tier
- **Token**: נדרש Access Token מסוג Read

---

## עיצוב (Design System)

### פלטת צבעים
| שם | ערך | שימוש |
|---|---|---|
| Black | `#1A1A1A` | Hero, כפתורים ראשיים, טקסט |
| Mustard | `#C8960A` | Accent, קישורים, מצב פעיל |
| Off-white | `#F5F0EB` | רקע תוכן |
| White | `#FFFFFF` | שדות קלט, פאנלים |
| Red | `#C43A31` | שמור לנגיעות דגש |
| Border | `#E0D8CE` | גבולות, הפרדות |
| Muted | `#8A8580` | טקסט משני |

### טיפוגרפיה
| שימוש | Font | Weight |
|---|---|---|
| כותרות, Labels | Noto Serif Hebrew | 400, 700 |
| Body, כפתורים | Assistant | 300, 400, 500, 600 |

### Responsive
- Mobile-first
- Breakpoint: 480px
- גודל מינימלי: 320px
- RTL layout מלא

---

## מגבלות ידועות

1. **תרגום חלקי** — המילון מכיל כ-50 מילים. מילים שלא במילון עוברות כמו שהן (עשויות לעבוד אם דומות לאנגלית).
2. **תלות באינטרנט** — האתר דורש חיבור אינטרנט פעיל לקריאות API.
3. **AI תלוי בשירות חיצוני** — ניתוח התמונה דורש חיבור ל-Hugging Face. אם השירות לא זמין, אפשר לתאר ידנית.
4. **API Limit** — מגבלת קריאות: 60 בקשות לדקה (מספיק לשימוש רגיל).
5. **אנגלית בתוצאות** — שמות יצירות, אמנים ופרטים מגיעים באנגלית (כך הם ב-API).

---

## רישיון ומקורות

- **API**: Art Institute of Chicago — [api.artic.edu](https://api.artic.edu) — CC0
- **Fonts**: Google Fonts — Open source
- **קוד**: פרויקט לימודי במסגרת מגמת אמנויות העיצוב, 2027
