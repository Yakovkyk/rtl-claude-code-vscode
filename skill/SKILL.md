---
name: rtl-fix
description: Diagnose and fix RTL (right-to-left) support for Claude Code in VS Code. Use when the user complain about Hebrew/Arabic text appears left-to-right instead of right-to-left, or when the user asks about the RTL extension.
disable-model-invocation: false
allowed-tools: Bash, Read, Glob, Grep, Edit, AskUserQuestion
---

# RTL Fix - תוסף RTL לקלוד קוד

## רקע טכני (לשימוש פנימי - לא להציג למשתמש אלא אם שואל)

תוסף ה-RTL עובד על ידי הזרקת קוד JavaScript לסוף הקובץ `webview/index.js` של תוסף Claude Code.
הבעיה: כשקלוד קוד מתעדכן לגרסה חדשה, הקובץ מוחלף בנקי - וההזרקה נעלמת.
בנוסף, ה-CSS class names של Claude Code הם minified ומשתנים בין גרסאות - לכן צריך לסרוק ולעדכן אותם.

- סקריפט ההתקנה: [scripts/install.ps1](scripts/install.ps1) (מקומי בתוך תיקיית הסקיל)
- קוד ה-RTL להזרקה: [scripts/rtl-claude-code.js](scripts/rtl-claude-code.js) (מקומי בתוך תיקיית הסקיל)
- תיקיית התוספים של VS Code: `~/.vscode/extensions/anthropic.claude-code-*` (Windows: `C:/Users/<username>/.vscode/extensions/anthropic.claude-code-*`)

## איך לנהל את השיחה

השפה: **עברית תמיד**. המשתמש מדבר עברית.

### עיקרון מרכזי: אבחן → תקן → דווח

כשהמשתמש מבקש "תקן RTL" - **פשוט תתקן.** לא לשאול אישור להתקנה עצמה.
לדווח בסוף מה נמצא ומה תוקן. לעצור ולשאול רק אם משהו לא צפוי קורה (סקריפט נכשל, backup חסר, מבנה לא מוכר).

### שלב 1: אבחון (שקט, אוטומטי)

כשהסקיל מופעל:
1. בדוק אילו גרסאות Claude Code קיימות
2. בדוק באילו מהן RTL מותקן (חפש את המחרוזת "RTL Support for Claude Code" ב-webview/index.js)
3. **בדוק תאימות סלקטורים** (שלב 1.5 - ראה למטה)
4. **המשך ישירות לביצוע** - אל תשאל את המשתמש מה לעשות

### שלב 1.5: סריקת תאימות סלקטורים (חובה!)

**זה השלב הקריטי.** ה-class names של Claude Code הם minified ומשתנים בכל גרסה.
חובה לבצע את הבדיקה הזו בכל הרצה, גם אם RTL כבר מותקן.

#### איך לסרוק את ה-class names הנוכחיים:

1. קרא את קובץ ה-backup (הגרסה המקורית ללא הזרקה):
   `webview/index.js.backup` (אם קיים) או `webview/index.js` (אם RTL לא מותקן)

2. חפש את הגדרת ה-CSS module של Ln (מודול הודעות הצ'אט) באמצעות הפקודה:
   ```
   grep -oP 'Ln\s*=\s*\{[^}]{0,500}' <path-to-backup>
   ```
   זה יחזיר משהו כמו:
   ```
   Ln={chatContainer:"ri",messagesContainer:"P",message:"U",userMessageContainer:"N",userMessage:"ai",timelineMessage:"e",...}
   ```

3. חלץ את ה-class names הרלוונטיים:
   - `message` - class של הודעה בודדת (למשל `"U"`)
   - `userMessageContainer` - class של מיכל הודעת משתמש (למשל `"N"`)
   - `timelineMessage` - class של הודעת עוזר (למשל `"e"`)
   - `userMessage` - class של טקסט הודעת משתמש (למשל `"ai"`)

4. חפש גם את ה-CSS modules של הדיאלוגים:

   **מודול AskUserQuestion** (שאלות עם אפשרויות בחירה):
   ```
   grep -oP '\w+\s*=\s*\{[^}]*questionsContainer[^}]{0,500}' <path-to-backup>
   ```
   זה יחזיר משהו כמו:
   ```
   An={questionsContainer:"an",questionBlock:"pn",questionTextLarge:"sn",option:"eo",optionLabel:"kn",optionDescription:"yn",navigationBar:"en",navTab:"Ko",otherInput:"ro",...}
   ```
   חלץ את ה-class names:
   - `questionsContainer` (למשל `"an"`)
   - `questionTextLarge` (למשל `"sn"`)
   - `option` (למשל `"eo"`)
   - `optionLabel` (למשל `"kn"`)
   - `optionDescription` (למשל `"yn"`)
   - `navigationBar` (למשל `"en"`)
   - `navTab` (למשל `"Ko"`)
   - `otherInput` (למשל `"ro"`)

   **מודול Permission** (דיאלוגי אישור פעולות):
   ```
   grep -oP '\w+\s*=\s*\{[^}]*permissionRequestContainer[^}]{0,500}' <path-to-backup>
   ```
   חלץ:
   - `permissionRequestContainer` (למשל `"t"`)
   - `permissionRequestHeader` (למשל `"Co"`)
   - `permissionRequestDescription` (למשל `"a"`)

5. השווה עם הסלקטורים ב-`rtl-claude-code.js`:
   - חפש את הבלוק `chatSelectors` - בדוק אם מתאים ל-Ln class names
   - חפש את הבלוק `dialogSelectors` - בדוק אם מתאים ל-An ו-ai class names
   - **חובה לבדוק את שניהם!**

6. **אם הסלקטורים לא מתאימים** - עדכן את `rtl-claude-code.js`:
   - שנה את הסלקטורים בבלוק `chatSelectors` לפי ה-class names החדשים
   - שנה את הסלקטורים בבלוק `dialogSelectors` לפי ה-class names החדשים
   - הפורמט chatSelectors: `.{message}.{userMessageContainer}` להודעות משתמש, `.{message}.{timelineMessage}` להודעות עוזר, `.{userMessage}` לטקסט
   - עדכן גם את ההערות לציין את הגרסה

#### דוגמה - chatSelectors:
אם מצאת `Ln={...,message:"Q",userMessageContainer:"Z",timelineMessage:"f",userMessage:"bx",...}`:
```javascript
chatSelectors: [
    ...
    '.Q.Z',   // Ln.message + Ln.userMessageContainer
    '.Q.f',   // Ln.message + Ln.timelineMessage
    '.bx',    // Ln.userMessage
    ...
]
```

#### דוגמה - dialogSelectors:
אם מצאת `An={questionsContainer:"xy",questionTextLarge:"ab",...}` ו-`ai={permissionRequestContainer:"cd",...}`:
```javascript
dialogSelectors: {
    questionsContainer: '.xy',    // An.questionsContainer
    questionText: '.ab',          // An.questionTextLarge
    ...
    permissionContainer: '.cd',   // ai.permissionRequestContainer
    ...
}
```

### שלב 2: ביצוע (אוטומטי, בלי לבקש אישור)

#### 2א: עדכון הקוד (אם הסלקטורים השתנו)

ערוך את `scripts/rtl-claude-code.js` בתוך תיקיית הסקיל עם ה-class names החדשים.

#### 2ב: התקנת RTL בתוסף

1. אם RTL כבר מותקן עם סלקטורים ישנים - שחזר מ-backup:
   ```
   cp <path>/webview/index.js.backup <path>/webview/index.js
   ```
2. הרץ את סקריפט ההתקנה (Windows PowerShell):
   ```
   powershell -ExecutionPolicy Bypass -File "$HOME/.claude/skills/rtl-fix/scripts/install.ps1"
   ```
3. בדוק שההזרקה הצליחה (חפש את המחרוזת וודא שהסלקטורים החדשים שם)

### שלב 3: דיווח (תמיד בסוף)

דווח למשתמש סיכום קצר שכולל:
- **מה נמצא** - אילו גרסאות, מה המצב שהיה
- **מה תוקן** - התקנה חדשה / עדכון סלקטורים / שניהם
- **אם הסלקטורים עודכנו** - טבלה קצרה עם השינויים
- **אם הותקן** - הזכר: צריך **להפעיל מחדש את VS Code** (`Ctrl+Shift+P` → `Reload Window`)
- **אם הכל היה תקין ולא נדרש שינוי** - פשוט דווח: "הכל תקין, RTL מותקן ומעודכן בכל הגרסאות"
