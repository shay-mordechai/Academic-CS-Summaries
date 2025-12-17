
---

## 3. LSP: Liskov Substitution Principle (עקרון ההחלפה של ליסקוב)

[[SRP]] | [[OCP]] | [[LSP]] | [[ISP]] | [[DIP]]

**הגדרה:** תתי טיפוסים חייבים להיות **ניתנים להחלפה התנהגותית** עבור טיפוסי הבסיס שלהם. קליינט המשתמש ב-reference למחלקת בסיס, **לא צריך לדעת לאיזה תת-אובייקט הוא שייך**.

עקרון LSP דורש שהמחלקות היורשות יקיימו את ההתנהגות שהלקוחות מצפים לה ממחלקת הבסיס, והוא נועד להבטיח שהקוד יהיה **פחות שביר** (Non-fragile).

### כלי ניתוח מרכזי למבחן: Design by Contract

הדרך העיקרית לבחון LSP היא באמצעות **Design by Contract** ("עיצוב מונחה חוזה"). על פי Bertrand Meyer, פונקציה דורסת במחלקה יורשת (Subtype) חייבת לעמוד בכללים הבאים ביחס לפונקציה הנדרסת במחלקת הבסיס (Supertype):

1. **תנאי קדם (Preconditions):** תנאי קדם בפונקציה דורסת רשאים רק **להיחלש** (או להישאר שווים). המשמעות היא שהטווח המותר לקלט בתת-מחלקה חייב להיות לפחות זהה (ואפשר רחב יותר) לזה של מחלקת האב.
    
2. **תנאי בתר (Postconditions):** תנאי בתר בפונקציה דורסת רשאים רק **להתחזק** (או להישאר שווים). ההבטחה לגבי הפלט או השגיאות המובטחות בתת-מחלקה חייבת להיות לפחות זהה לזו של מחלקת האב.
    

### דוגמה קלאסית להפרת LSP: מלבן וריבוע

הפרת LSP נבחנת בצורה הטובה ביותר באמצעות דוגמת **Rectangle** (מלבן) ו-**Square** (ריבוע):

- **הבעיה:** אובייקט מסוג `Square` **אינו עקבי התנהגותית** עם ההתנהגות של אובייקט מסוג `Rectangle`.
    
- **הפרה בפועל:** קליינט שמקבל אובייקט `r` מסוג `Rectangle` עשוי לקרוא ל-`r.setWidth(5)` ול-`r.setHeight(4)` ולצפות ששטח המלבן יהיה 20 (`r.getArea() == 20`). אם מועבר אובייקט מסוג `Square`, הקריאה לכל אחת מפונקציות ה-`set` תשנה גם את המימד השני, והתוצאה הסופית תהיה שטח שונה מ-20, **מה שמפר את ציפיות הקליינט**.
    
- **המסקנה:** מבחינה התנהגותית, **ריבוע אינו מלבן**, וההתנהגות היא המהות האמיתית של התכנה.
    

**LSP בשאלות מבחן:** בדיקות LSP מופיעות בשאלות רב-ברירה תוך התייחסות ישירה לכללי Design by Contract.

---

## 📚 שאלות ממבחנים

- **תשפ"ג ב ב שאלה 3:** (מימוש `sendEmail` ששומר על LSP באמצעות תנאי קדם/בתר).
    
- **תשפ"ד ב ב שאלה 4:** (בדיקת LSP במחלקות GrantDegree1/2/3).
    
- **תשפ"ב א ב שאלה 6:** (בדיקת LSP ב-`RealNumber`/`ComplexNumber`).
    
- **תשפ"ג א א שאלה 2:** (הפרת LSP ב-`ElectronicDuck`/`Duck` בגלל תנאי בתר).
    
![[Pasted image 20251106124133.png]]
---

קודם: [[OCP]]

הבא: [[ISP]]

חזרה ל: [[יחידה 2 עקרונות העיצוב (SOLID)]]

תוכן עניינים ראשי: [[Summeries/עיצוב/תוכן עניינים]]