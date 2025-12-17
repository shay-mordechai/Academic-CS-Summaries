שני הסיכומים טובים, אבל **הסיכום הראשון ("זה?") הוא המקיף והטוב יותר** ללמידה לקראת מבחן.

**הסיבות:**

1. **הוא משווה תבניות:** הוא מסביר באופן ברור את ההבדלים הקריטיים בין `Strategy` ל-`Command` ובין `Strategy` ל-`Template Method`. אלו שאלות מבחן קלאסיות.
    
2. **הוא מציג אנלוגיות:** האנלוגיה של משחק השח (אגרסיבי/הגנתי) ודוגמת ההנחות (חג/סוף עונה) ממחישות היטב את הרעיון של "משפחת אלגוריתמים ברי החלפה".
    
3. **הוא כולל שאלות מבחן:** הוא מקשר את התיאוריה ישירות לשאלות ספציפיות מהמבחנים (כמו שאלת ה-IRS), מה שהופך את הלמידה לממוקדת.
    

הסיכום השני קצר וקולע, אבל הוא חסר את ההקשר החשוב של ההשוואה לתבניות אחרות ואת הקישור לשאלות המבחן.

לקחתי את הסיכום הראשון (הטוב יותר) וארגנתי אותו מחדש בפורמט Markdown נקי וברור לאובסידיאן.

---

# תבנית עיצוב: Strategy (אסטרטגיה)

**Strategy** היא תבנית עיצוב **התנהגותית (Behavioral)** חשובה.

## 🎯 מטרה ובעיה נפתרת

תבנית Strategy נועדה לפתור מצב שבו למחלקה מסוימת (ה-`Context`) יש פעולה שיכולה להתבצע בכמה דרכים שונות. כלומר, יש **"משפחה של אלגוריתמים"** לאותה משימה.

התבנית מאפשרת להחליף בין האלגוריתמים האלה בקלות, אפילו בזמן ריצה, מבלי לשנות את הקוד של ה-`Context`.

**הבעיה:** הפתרון הנאיבי יהיה לשים `if/else` או `switch` ענק בתוך מחלקת ה-Context כדי לבחור את האלגוריתם. זה מפר את עיקרון **[[OCP]]** (פתוח/סגור), כיוון שהוספת אסטרטגיה חדשה תדרוש שינוי בקוד הקיים.

**הפתרון:** התבנית מציעה להוציא כל אלגוריתם למחלקה נפרדת, ולהשתמש ב**הכלה (Composition)**.

## 🏗️ מבנה התבנית (UML)
![](strategy-uml.png)
1. **Context (הקשר):**
    
    - זוהי המחלקה הראשית שהלקוח עובד איתה (למשל, `Store` או `ChessPlayer`).
        
    - היא מחזיקה רפרנס (באמצעות Composition) לממשק ה-`Strategy`.
        
    - היא חושפת מתודה שהלקוח קורא לה (למשל `checkout()`), אשר בתורה קוראת לאלגוריתם.
        
2. **Strategy (אסטרטגיה):**
    
    - זהו הממשק המשותף שכל האלגוריתמים יממשו (למשל, `Discounter`).
        
    - הוא מגדיר את המתודה שה-Context יקרא לה (למשל, `applyDiscount(double amount)`).
        
3. **ConcreteStrategy (אסטרטגיה קונקרטית):**
    
    - אלו המימושים בפועל של האלגוריתמים השונים (למשל, `PassoverDiscounter`, `AggressiveStrategy`).
        

## ⌨️ קוד דוגמה: מערכת הנחות (מהמצגת)

### 1. Strategy (הממשק)

Java

```
// Discounter.java
public interface Discounter {
    double applyDiscount(double amount);
}
```

### 2. ConcreteStrategy (מימוש האלגוריתם)

Java

```
// PassoverDiscounter.java
public class PassoverDiscounter implements Discounter {
    @Override
    public double applyDiscount(double amount) {
        return amount * 0.5; // 50% הנחת חג
    }
}
```

### 3. Context (המשתמש באסטרטגיה)

Java

```
// Store.java
public class Store {
    private Discounter discounter; // 1. הכלה (Composition)

    // 2. מאפשר להחליף אסטרטגיה בזמן ריצה
    public void setDiscounter(Discounter discounter) {
        this.discounter = discounter;
    }

    public double checkout(double amount) {
        // 3. מאציל את ביצוע האלגוריתם לאסטרטגיה הנוכחית
        return discounter.applyDiscount(amount);
    }
}
```

### 4. Client (הלקוח)

Java

```
// Main.java
public static void main(String[] args) {
    Store store = new Store();
    
    // קובעים אסטרטגיה ראשונה
    store.setDiscounter(new PassoverDiscounter());
    System.out.println(store.checkout(100)); // פלט: 50.0

    // מחליפים אסטרטגיה בזמן ריצה
    store.setDiscounter(new EndOfSeasonDiscounter()); // נניח שזו מחלקה אחרת
    System.out.println(store.checkout(100)); // הפלט ישתנה בהתאם
    
    // אפשר להשתמש גם בלמדא (Java 8+)
    store.setDiscounter(amount -> amount * 0.9); // 10% הנחה
}
```

כפי שניתן לראות, ה-`Store` לא השתנה, אבל ההתנהגות שלו (ה"איך" שהוא מחשב הנחה) השתנתה לחלוטין בזמן ריצה.

---

## 💡 הבדלים מתבניות אחרות (שאלות מבחן קלאסיות)

### Strategy מול Command (תשפ"ג א א שאלה 8)

- **[[Strategy]]** עונה על השאלה "**איך**" לבצע פעולה (יש לה כמה דרכים, `if/else` מורכב).
    
- **[[Command]]** עונה על השאלה "**מה**" לבצע (היא אורזת בקשה אחת באובייקט).
    

### Strategy מול Template Method (תשפ"ב א ב, שאלה פתוחה)

- **[[Strategy]]** משתמש ב**הכלה (Composition)**. האלגוריתם הוא אחריות נפרדת לחלוטין (ממשק) וניתן להחלפה מלאה בזמן ריצה.
    
- **[[Template Method]]** משתמש ב**ירושה**. שלד האלגוריתם קבוע במחלקת האב, ורק חלקים ספציפיים ניתנים לדריסה על ידי מחלקות הבת.
    

---

## 📚 שאלות ממבחנים

- **תשפ"ב א ב שאלה 3ב:** (מערכת רשות המיסים (IRS) שצריכה שיטות דגימה שונות).
    
- **תשפ"ד ב א שאלה 4ב:** (רשת עצבית, נוסחאות שונות לנוירון).
    
- **תשפ"ד ב ב שאלה 8:** (השוואה ל-Command).
    
- **תשפ"ד ב ב שאלה 1:** (הדגמה: `itsI.f()`).
    
- **הרצאה 25 (שקפים 18-19):** (דוגמת ה-Discounter ושימוש ב-Lambda).
    
---
**קודם:** [[Command]]
**הבא:** [[Iterator]]

**חזרה ל:** [[תבניות התנהגות (Behavioral Pattern)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]