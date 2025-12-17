
---

# תבנית עיצוב: State (מצב)

State היא תבנית עיצוב התנהגותית (Behavioral).

(מצגת: OOPDE-W07Lec14 - Design patterns - Composite + State.pptx)

## 🎯 מטרה ובעיה נפתרת

**הבעיה:** יש לנו אובייקט שההתנהגות שלו משתנה באופן דרמטי בהתאם למצב הפנימי שלו. הדרך הפשוטה (והנאיבית) לממש את זה היא באמצעות משפטי `if/else` או `switch` ארוכים ומסורבלים בתוך הפונקציות של האובייקט.
```Java
public void handleRequest() {
    if (state == "LOCKED") {
        // do something
        if (condition) {
            state = "UNLOCKED";
        }
    } else if (state == "UNLOCKED") {
        // do something else
        if (condition) {
            state = "LOCKED";
        }
    }
}
```
הקוד הזה הופך מהר מאוד לקשה לתחזוקה ומפר את עיקרון **[[OCP]]** (כל הוספת מצב חדש דורשת שינוי בקוד הקיים).

**הפתרון:** במקום שהאובייקט הראשי ינהל את כל ההתנהגויות בעצמו, אנחנו **"מוציאים" את הלוגיקה של כל מצב למחלקה נפרדת**.

## 🏗️ מבנה התבנית (UML)
![](Summeries/עיצוב/media/media/image86.png)

1. **Context:** זהו האובייקט הראשי שההתנהגות שלו משתנה (למשל, `Context` בדוגמה למטה). הוא מחזיק **רפרנס לאובייקט State נוכחי**.
    
2. **State:** זהו ממשק שמגדיר את הפעולות המשותפות לכל המצבים (למשל, `writeName()`).
    
3. **ConcreteState:** אלו מחלקות נפרדות, אחת לכל מצב אפשרי (למשל, `LowerCaseState`). כל מחלקה כזו מממשת את הלוגיקה הספציפית לאותו מצב. כאשר צריך לעבור למצב אחר, המחלקה הנוכחית אומרת ל-Context לשנות את ה-State שלו לאובייקט State חדש.
    

## 📈 קשר ל-Statecharts (תרשימי מצבים)

**זהו המיקוד העיקרי של נושא זה במבחן.**

תבנית State היא המימוש הקודני הישיר של תרשים מצבים (Statechart) ב-UML.

השאלות הפתוחות בנושא זה (4 מתוך 6 אזכורים במבחנים) דורשות **תרגום ישיר של תרשים מצבים נתון לקוד** המממש את תבנית State (Context, State Interface, Concrete States).

---

## ⌨️ דוגמת קוד (OOPDE-W07Lec14, שקפים 14-18)

**הבעיה:** 
![](Summeries/עיצוב/media/media/image89.png)
(אובייקט `Context` שיודע לכתוב טקסט. ההתנהגות שלו משתנה: לפעמים הוא כותב באותיות קטנות, ולפעמים באותיות גדולות.)
### 1. ה-Context (האובייקט הראשי)

זהו האובייקט שההתנהגות שלו משתנה. הוא מחזיק מצביע למצב הנוכחי, ומאציל (delegates) אליו את ביצוע הפעולות.

```Java
// Context.java
public class Context {
    private State currentState; // המצב הפנימי הנוכחי

    public Context() {
        // מתחילים עם מצב ברירת מחדל
        currentState = new LowerCaseState(); 
    }

    public void setState(State newState) {
        this.currentState = newState;
    }

    public void writeName(String name) {
        // מאציל את העבודה למצב הנוכחי
        currentState.writeName(this, name);
    }
}
```

### 2. ה-State (הממשק המשותף)

זהו הממשק שמגדיר את הפעולה המשותפת לכל המצבים האפשריים.

```Java
// State.java
public interface State {
    // הפעולה המשותפת שמקבלת את הקונטקסט ואת הנתון
    void writeName(Context context, String name);
}
```

### 3. ה-Concrete States (המצבים הקונקרטיים)

אלו המחלקות הנפרדות, אחת לכל מצב. הן מכילות את הלוגיקה הספציפית לאותו מצב, ובסוף הפעולה הן יכולות להגיד ל-Context לעבור למצב הבא.

```Java
// LowerCaseState.java
public class LowerCaseState implements State {
    @Override
    public void writeName(Context context, String name) {
        System.out.println(name.toLowerCase());
        // עוברים למצב הבא
        context.setState(new UpperCaseState());
    }
}
```

```Java
// UpperCaseState.java
public class UpperCaseState implements State {
    private int count = 0; // משתנה פנימי למצב זה בלבד

    @Override
    public void writeName(Context context, String name) {
        System.out.println(name.toUpperCase());
        // אם עברו שתי קריאות, חוזרים למצב הקודם
        if (count == 1) {
            context.setState(new LowerCaseState());
        } else {
            count++;
        }
    }
}
```

### 4. ה-Client (השימוש במערכת)

הלקוח לא מודע בכלל לקיומן של המחלקות `LowerCaseState` או `UpperCaseState`.

```Java
// Main.java
public class Main {
    public static void main(String[] args) {
        Context context = new Context();
        
        context.writeName("Sunday");
        context.writeName("Monday");
        context.writeName("Tuesday");
        context.writeName("Wednesday");
        context.writeName("Thursday");
    }
}
```

**הפלט הצפוי:**

```Java
sunday
MONDAY
TUESDAY
wednesday
THURSDAY
```

ההתנהגות של `context.writeName()` משתנה באופן דינמי בכל קריאה, בהתאם למעברי המצבים שהגדרנו.

---

## 💡 אופטימיזציה: בעיית `new State()` והפתרון (Singleton)

**הבעיה:** יצירת אובייקט `new State()` חדש בכל מעבר מצב (כמו שעשינו בקוד) יכולה להיות בזבזנית ולא יעילה (הרבה הקצאות זיכרון), במיוחד אם המעברים תכופים.

**הפתרון:** אם מחלקות ה-`ConcreteState` הן **חסרות מצב (stateless)** - כלומר, אין להן משתני מופע שמשתנים (כמו ה-`count` שהיה לנו) - אין סיבה ליצור מופע חדש שלהן בכל פעם.

במקרה כזה, הפתרון הנפוץ הוא להפוך את מחלקות ה-State ל-**[[Singleton]]**. יוצרים מופע אחד מכל סוג מצב, ועושים בו שימוש חוזר.

```Java
// דוגמה למימוש Singleton (וריאנט Eager)
public class LowerCaseState implements State {
    // 1. יוצרים מופע יחיד, פעם אחת
    private static final LowerCaseState instance = new LowerCaseState();

    // 2. בנאי פרטי
    private LowerCaseState() {}

    // 3. מספקים נקודת גישה גלובלית
    public static LowerCaseState getInstance() {
        return instance;
    }

    @Override
    public void writeName(Context context, String name) {
        System.out.println(name.toLowerCase());
        // 4. במקום ליצור חדש, משתמשים במופע הקיים של המצב הבא
        context.setState(UpperCaseState.getInstance()); 
    }
}
```

**הערה חשובה:** בדוגמה הספציפית שלנו, `UpperCaseState` _אינו_ חסר מצב (כי הוא מכיל את המשתנה `count`). לכן, במימוש הזה, לא היינו יכולים להפוך _אותו_ ל-Singleton בלי לשנות את הלוגיקה (למשל, להעביר את האחריות על הספירה ל-Context).

---

## 📚 שאלות ממבחנים

- **מעבדה 8 (שקפים 10-16):** שאלה מלאה ממבחן. נתון תרשים מצבים (Statechart) של `SecretKeeper` ונדרש לממש אותו.
    
- **תשפ"ב א א שאלה 5ב:** (מימוש `SecretKeeper`).
    
- **תשפ"ג ב ב שאלה 2:** (תרשים מצבים למנורה, הבהוב).
    
- **תשפ"ג ב ב שאלה 4ב:** (מערכת השכמה בבית חכם, מעבר מצבים מחזורי).
    
- **תשפ"ג א א שאלה 1:** (תרשים מצבים לכספת).
    
---
**קודם:** [[תבניות התנהגות (Behavioral Pattern)]]
**הבא:** [[Visitor]]

**חזרה ל:** [[תבניות התנהגות (Behavioral Pattern)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]