
---

## 4. ISP: Interface Segregation Principle (עקרון הפרדת ממשקים)

[[SRP]] | [[OCP]] | [[LSP]] | [[ISP]] | [[DIP]]

**הגדרה:** עיקרון הפרדת הממשקים (ISP) קובע כי **אסור להכריח לקוח להיות תלוי בממשק שהוא לא משתמש בו**. הקליינט צריך להיות חשוף לממשק ו**רק לממשק שהוא זקוק לו**.

### מהות ההפרה ("ממשקים שמנים")

ISP מופר כאשר ממשק אחד הוא "שמן" מדי (Fat Interface), ומכיל יותר מדי מתודות. כתוצאה מכך, מחלקות המממשות אותו נאלצות לספק מימוש (אפילו ריק) עבור מתודות שהן אינן זקוקות להן.

**הפתרון:** הפתרון להפרת ISP הוא **פיצול** ממשקים "שמנים" לממשקים קטנים וספציפיים יותר.

---

## עקרון ISP במקרי בוחן ודוגמאות

עקרון ISP הוא אחד העקרונות, יחד עם [[DIP]], שמניע להוספת ממשקים לעיצוב.

### 1. דוגמה: עובדי רפואה (`MedicalWorker`)

- **הבעיה:** הפרת ISP בממשק `MedicalWorker` כאשר מחלקות כמו `Doctor` ו-`Nurse` נדרשות לממש מתודות שאינן רלוונטיות לשתיהן.
    
- **דוגמה:** אם רופא צריך `writePrescription` ואח/אחות צריך `inject`, לא נכון לכפות על שניהם לממש את שתי הפעולות בממשק משותף אחד.
    

### 2. דוגמה: מערכת הטלפון הסלולרי

- **הבעיה:** מקרה הבוחן של הטלפון הסלולרי הראה הפרת ISP כאשר המחלקות `Dialer` ו-`CellularRadio` היו תלויות במחלקה `Display`. שתי המחלקות היו צריכות **רק חלק** מהמתודות של `Display`.
    
- **התיקון:** פיצול ממשק `Display` לשני ממשקים נפרדים: `DialerDisplay` ו-`CellularRadioDisplay`. המחלקה `Display` מממשת את שני הממשקים, אך כעת `Dialer` תלוי רק ב-`DialerDisplay` ו-`CellularRadio` תלוי רק ב-`CellularRadioDisplay`.
    

### 3. דוגמה: אכלנים ושתיינים

- **הבעיה:** במערכת המכילה `Eater` (אכלנים) ו-`Drinker` (שתיינים) והמחלקה `Silverware` (כלי אכילה), המחלקה `Silverware` הוגדרה עם פעולות `scoop()` ו-`stir()`. האכלנים (`Eater`) משתמשים רק ב-`scoop()`, ואילו השתיינים (`Drinker`) משתמשים רק ב-`stir()`.
    
- **התיקון:** הפרדת הממשק לשני ממשקים: `Scoopable` (עבור האכלנים) ו-`Stirable` (עבור השתיינים).
    

---

## 📚 שאלות ממבחנים

- **תשפ"ד א א שאלה 7:** (הפרת ISP בממשק `MedicalWorker`).
    
- **תשפ"ד ב ב שאלה 4:** (מהו התיקון לבעיית ISP).
    
- **תשפ"ד ב ב שאלה 3:** (ISP/DIP).
    
- **תשפ"ד א ב שאלה 12:** (העקרונות שהובילו לתבנית `Iterator`, כולל ISP).
    

---

קודם: [[LSP]]

הבא: [[DIP]]

חזרה ל: [[יחידה 2 עקרונות העיצוב (SOLID)]]

תוכן עניינים ראשי: [[Summeries/עיצוב/תוכן עניינים]]