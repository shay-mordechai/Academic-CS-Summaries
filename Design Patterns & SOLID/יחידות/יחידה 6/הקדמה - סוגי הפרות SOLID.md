
---

# יחידה 6: מקרי בוחן (Case Studies)

במסגרת מקרי הבוחן, [[יחידה 2 עקרונות העיצוב (SOLID)]], משמשים נקודת מוצא לשינוי עיצובי (Refactoring). המטרה היא לזהות הפרה חמורה במודל ראשוני, ואז להשתמש בתבניות עיצוב (Design Patterns) כדי לתקן אותה.

ההתייחסות היא לרוב לעקרונות המשפיעים ישירות על צימוד (Coupling) ותלות (Dependency) במערכות גדולות.

## 1. מקרה בוחן: מערכת ניטור מזג אוויר (WMS)

זהו נושא קריטי ששווה כ-20 נקודות במבחן. העיצוב הראשוני של WMS הציג הפרות קשות שהובילו לצימוד חזק (Tight coupling).

|**עיקרון מופר**|**תיאור ההפרה במודל הראשוני**|**הפתרון התבניתי (תיקון)**|
|---|---|---|
|**[[DIP]]** (היפוך תלויות)|**המתזמן (`Scheduler`)** (מודול רמה גבוהה) תלוי ישירות במחלקות קונקרטיות של **חיישנים** ו**מסך**.|**[[Observer]]** (ניתוק המסך), **[[Bridge]]** (ניתוק החומרה), **[[Abstract Factory]]** (ניתוק האתחול).|
|**[[OCP]]** (פתיחות/סגירות)|**הוספת חיישן חדש** מחייבת **שינוי** בקוד הקיים של ה-`Scheduler`.|**[[Template Method]]** (מונע שכפול קוד בחיישנים), **[[Observer]]** ו-**[[Bridge]]** (מאפשרים הרחבה).|
|**[[SRP]]** (אחריות יחידה)|**ה-`Scheduler`** עושה יותר מדי: גם מתזמן, גם מנהל חיישנים, וגם מעדכן תצוגה.<br><br>  <br><br>**מחשבון הקיצון** (`HiLoData`) אחראי גם על חישוב וגם על שמירה.|**[[Observer]]** (הוריד מהמתזמן את אחריות עדכון המסך).<br><br>  <br><br>**[[Proxy]]** (הפריד את לוגיקת החישוב מלוגיקת השמירה).|

## 2. מקרה בוחן: טלפון סלולרי

מקרה הבוחן של הטלפון הסלולרי מראה כיצד העיצוב מתפתח מניסיון לחקות מבנה פיזי לניתוח התנהגות.

|**עיקרון מופר**|**תיאור ההפרה במודל הראשוני**|**הפתרון התבניתי (תיקון)**|
|---|---|---|
|**[[ISP]]** (הפרדת ממשקים)|המחלקות `Dialer` ו-`CellularRadio` היו תלויות בממשק `Display` "שמן" (Fat Interface), וכל אחת צריכה רק חלק מהמתודות.|**פיצול ממשקים:** פיצול `Display` לממשקים קטנים יותר (`DialerDisplay` ו-`CellularRadioDisplay`).|
|**[[DIP]]** (היפוך תלויות)|קיימת **תלות ישירה** בין `Button` (כמו `DigitButton`) לבין `Dialer` (החיגן).|**שימוש בממשק מופשט** (כגון `IButtonServer`) לניתוק התלות בין הכפתור למקבל הפקודה.|

---

## 💡 טיפ לחיבור SOLID ותבניות

- **[[DIP]] + [[Bridge]]:** תבנית [[Bridge]] פותרת בעיית [[DIP]].
    
- **[[OCP]] + [[Decorator]]:** תבנית [[Decorator]] מאפשרת להוסיף יכולות בלי לשנות את המחלקות הקיימות (עמידה ב-[[OCP]]).
    
- **[[SRP]] + [[Proxy]]:** תבנית [[Proxy]] מאפשרת להפריד אחריות רוחבית (כמו שמירה לדיסק) מאחריות הליבה של האובייקט.
    
- **[[ISP]]:** תרגול פיצול ממשקים (כמו במקרה הבוחן של האכלנים/שתיינים) הוא ממוקד ונדרש.
    

---

## 💡 פירוט: הפרת DIP במערכת WMS

זוהי שאלה קריטית להבנת עקרון היפוך התלויות (**[[DIP]]**) וכיצד הוא מיושם ב-WMS.

### 1. זיהוי רמות המודולים (גבוהה מול נמוכה)

- **מודול ברמה גבוהה (High-Level Module):** מכיל את הלוגיקה העסקית (Policy).
    
    - **במקרה זה:** המחלקה המופשטת `TemperatureSensor`, כיוון שהיא מכילה את האלגוריתם המשותף של בדיקת שינוי ועדכון (`check()`).
        
- **מודול ברמה נמוכה (Low-Level Module):** מכיל פרטי מימוש טכניים.
    
    - **במקרה זה:** המחלקה `MonitoringScreen`, כיוון שהיא פרט קונקרטי של ממשק משתמש (GUI).
        

### 2. הבעיה: הפרת DIP

עקרון **[[DIP]]** קובע כי "מודולים מופשטים לא צריכים להיות תלויים בפרטים".

- **ההפרה:** המחלקה המופשטת `TemperatureSensor` (רמה גבוהה) במתודה `check()` מכילה קריאה ישירה למחלקה הקונקרטית `itsMonitoringScreen` (רמה נמוכה).
    
- **הקוד הבעייתי:** `itsMonitoringScreen.displayTemperature(lastReading);`
    
- זוהי **תלות ישירה** של ההפשטה בפרט קונקרטי, וזה מפר **[[DIP]]**.
    

### 3. מדוע זו בעיה?

- **התוצאה של ההפרה:** אם נרצה לחבר את החיישן לצרכן עדכונים חדש (כמו לוגר, `Log`), נצטרך **לשנות את הקוד של המחלקה המופשטת `TemperatureSensor`**.
    
- מאחר ששינוי המודול ברמה גבוהה (החיישן) נדרש בגלל שינוי בפרטי המימוש (הוספת צרכן חדש), המערכת מפרה **[[OCP]]** (היא לא סגורה לשינוי).
    

הפתרון לבעיה זו במערכת WMS היה שימוש בתבנית **[[Observer]]**, אשר ניתקה את התלות בין החיישנים לתצוגה.


## 💡 טיפ לחיבור SOLID ותבניות

כאשר לומדים את עקרונות SOLID, חשוב לחבר אותם ישירות לתבניות העיצוב:

- **[[DIP]] + [[Bridge]]:** תבנית [[Bridge]] פותרת [[DIP]].
    
- **[[OCP]] + [[Decorator]]:** תבנית [[Decorator]] מאפשרת להוסיף יכולות בלי לשנות את המחלקות הקיימות (עמידה ב-[[OCP]]).
    
- **[[SRP]] + [[Proxy]]:** תבנית [[Proxy]] מאפשרת להפריד אחריות רוחבית (כמו שמירה לדיסק) מאחריות הליבה של האובייקט.
    
- **[[ISP]]:** תרגול פיצול ממשקים (כמו במקרה הבוחן של האכלנים/שתיינים) הוא ממוקד ונדרש.
    


---

## 📚 שאלות ממבחנים

### [[DIP]]

- תשפ"ד ב א שאלה 5 (הפרת OCP/DIP/SRP בפונקציית `activateNavigationForCars`).
    
- תשפ"ד א ב שאלה 11 (מי צריך להיות תלוי בהפשטות לאחר תיקון DIP).
    

### [[SRP]]

- תשפ"ג ב ב שאלה 2 (בדיקת עקרון SRP במחלקת `Refrigerator`).
    
- תשפ"ד א א שאלה 10 (בדיקת עקרון SRP במחלקת `User`).
    
- תשפ"ד ב א שאלה 5 (הפרת OCP/DIP/SRP בפונקציית `activateNavigationForCars`).
    

### [[ISP]]

- תשפ"ד א א שאלה 7 (הפרת ISP בממשק `MedicalWorker`).
    
- תשפ"ב א א שאלה 2 (תיקון עיצוב `Eater`/`Drinker` באמצעות ISP).
    

---

### מקרה הבוחן הבא:

- [[מערכת ניטור מזג אוויר (WMS)]]
    

### לתוכן עניינים:

[[Summeries/עיצוב/תוכן עניינים]]