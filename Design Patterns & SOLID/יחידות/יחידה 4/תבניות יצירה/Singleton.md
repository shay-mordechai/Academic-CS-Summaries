
---

# תבנית עיצוב: Singleton

**Singleton** היא תבנית עיצוב **יצירה (Creational)**. התבנית נמנית בין התבניות הנשאלות ביותר במבחנים.

## מטרה ובעיה נפתרת

המטרה העיקרית של התבנית היא **להבטיח שלמחלקה מסוימת יהיה מופע (instance) אחד בלבד** בכל המערכת, ולספק נקודת גישה גלובלית ונוחה לאותו מופע יחיד.

המוטיבציה לשימוש בתבנית נובעת משתי סיבות עיקריות:

1. **תיאום בין רכיבים:** כשיש צורך לתאם פעולות בין חלקים שונים של התוכנה שמשתמשים באותו משאב מרכזי (כמו מנהל הגדרות, או במערכת ניטור מזג האוויר, המחלקה `AlarmClock` ממומשת כסינגלטון).
    
2. **חיסכון במשאבים:** כאשר יצירת מופע של המחלקה היא פעולה "יקרה" ואין צורך ביותר ממופע אחד.
    

## מרכיבי מפתח (המימוש הקלאסי)

המימוש הקלאסי מורכב משלושה חלקים מרכזיים:

1. **בנאי מוגן/פרטי:** למניעת יצירת מופעים חיצונית.
    
2. **שדה סטטי פרטי/מוגן:** לשמירת המופע היחיד.
    
3. **מתודה סטטית פומבית (`theInstance`):** נקודת הגישה הגלובלית למופע.
    

### UML
![](singleton-uml.png)
---

## דוגמת שימוש (Reader/Library)

כאשר לקוח (`Reader`) משתמש ב-Singleton, שניהם מקבלים את אותו אובייקט `Library` יחיד בזיכרון.

```Java
public static void main(String[] args) {
    Reader reader1 = new Reader();
    Reader reader2 = new Reader();

    // הקריאה הראשונה יוצרת את מופע הספרייה
    reader1.setItsLibrary(Library.theInstance());

    // הקריאה השנייה מחזירה את המופע הקיים
    reader2.setItsLibrary(Library.theInstance());
}
```

**התוצאה הסופית:** בסוף התהליך, שני האובייקטים הנפרדים, `reader1` ו-`reader2`, מצביעים וחולקים את אותו אובייקט `Library` יחיד. אם `reader1` "שואל" ספר מהספרייה, גם `reader2` יראה שהספר הזה חסר.

- **ניתוח UML:** הקשר בין `Reader` ל-`Library` (דרך המשתנה `itsLibrary`) הוא קישור מכוון (**Directed Association**).
    

---

## וריאנטים של מימוש (ופתרון ל-Thread Safety)

(שקף 16)

יש להכיר את שלושת הווריאנטים (Eager, Lazy, Thread-Safe).

### וריאנט 1: Lazy Initialization (אתחול עצל)

המופע נוצר רק בקריאה הראשונה למתודה, אם הוא עדיין `null`.

```Java
public class Library {
    // 2. משתנה סטטי מוגן/פרטי
    protected static Library instance = null;

    // 1. בנאי מוגן (Protected constructor)
    protected Library() {
        // ...
    }

    // 3. מתודה סטטית פומבית (נקודת גישה)
    public static Library theInstance() {
        if (instance == null) {
            // יצירת המופע רק אם הוא לא קיים (Lazy Initialization)
            instance = new Library();
        }
        return instance;
    }
}
```

- **יתרון:** חיסכון במשאבים. האובייקט לא נוצר אם אף פעם לא משתמשים בו.
    
- **חיסרון (גדול):** לא בטוח לשימוש בסביבה מרובת תהליכונים (Multi-threaded).
    

**הבעיה - "מצב מרוץ" (Race Condition):** אם שני תהליכונים (Threads) שונים קוראים ל-`theInstance()` בדיוק באותו הזמן כש-`instance` עדיין `null`:

1. תהליך א' נכנס ל-`if` ורואה ש-`instance` הוא `null`.
    
2. בדיוק אז, מערכת ההפעלה עוצרת את תהליך א' ונותנת לתהליך ב' לרוץ.
    
3. תהליך ב' נכנס ל-`if` ורואה ש-`instance` עדיין `null`.
    
4. תהליך ב' יוצר מופע חדש.
    
5. תהליך א' חוזר לרוץ ויוצר גם הוא מופע חדש.
    
    תוצאה: יצרנו שני מופעים והפרנו את עקרון ה-Singleton.
    

### וריאנט 2: Thread-Safe Lazy Initialization (הפתרון הנאיבי)

הפתרון הפשוט לבעיית מצב המרוץ הוא להוסיף את המילה `synchronized` לחתימת המתודה. זהו "מנעול" (Mutex) שמבטיח שרק תהליכון אחד יכול להיכנס למתודה בכל רגע נתון.

```Java
public static synchronized Library thInstance() {
    if (instance == null) {
        instance = new Library();
    }
    return instance;
}
```

- **יתרון:** פותר את בעיית ה-thread-safety.
    
- **חיסרון:** יוצר "צוואר בקבוק" בביצועים. `synchronized` היא פעולה יקרה, וכל תהליכון שינסה לגשת למופע (גם אחרי שהוא כבר נוצר!) יצטרך לחכות בתור.
    

### וריאנט 3: Eager Initialization (אתחול מוקדם)

במקום ליצור את המופע "בעצלתיים", אנחנו יוצרים אותו מיד עם טעינת המחלקה. פתרון זה **תמיד בטוח לשימוש מרובה תהליכונים** (Thread-Safe).

```Java
public class EagerSingleton {

    // 1. המופע נוצר מיד, פעם אחת, כשהמחלקה נטענת
    private static final EagerSingleton instance = new EagerSingleton();

    // 2. בנאי פרטי
    private EagerSingleton() {}

    // 3. המתודה פשוט מחזירה את המופע שכבר קיים
    public static EagerSingleton getInstance() {
        return instance;
    }
}
```

- **יתרון:** בטוח לחלוטין לשימוש בסביבה מרובת תהליכונים, ללא צורך ב-`synchronized` יקר.
    
- **חיסרון:** המופע נוצר גם אם לעולם לא נשתמש בו.
    

> **זו התשובה לשאלת המבחן (תשפ"ג א' א' 9): במערכת מרובת תהליכונים שבה הביצועים חשובים, Eager Singleton הוא לרוב הפתרון המועדף והפשוט ביותר.**

---

## וריאנטים מתקדמים

### וריאנט 4: Singleton עם ירושה (שקף 17)

וריאנט זה פותר מצב שבו אנחנו רוצים מופע יחיד של טיפוס בסיס מופשט (abstract), אבל הטיפוס הקונקרטי (Dog או Cat) ייבחר רק פעם אחת בתחילת ריצת התוכנית.

**איך זה עובד?**

1. **המחלקה המופשטת (למשל, `Animal`)**
    
    - מחזיקה את המשתנה הסטטי המשותף: `protected static Animal instance;`.
        
    - יש לה מתודה `public static Animal theInstance()` שרק מחזירה את המופע (וזורקת שגיאה אם הוא `null`).
    
    ```Java
    // Animal.java (Abstract Base Class)
    public abstract class Animal {
        protected static Animal instance;
    
        public static Animal theInstance() {
            assert instance != null : "Cannot instantiate an abstract Animal";
            return instance;
        }
    
        abstract void speak();
    }
    ```
    
2. **המחלקות היורשות (למשל, `Dog`)**
    
    - לכל אחת מתודה `public static Animal theInstance()` (זו לא דריסה כי היא סטטית).
        
    - המתודה הזו "מתחרה": היא בודקת אם `instance` (המשותף) הוא `null`. אם כן, היא יוצרת מופע של _עצמה_ (`new Dog()`) ומציבה אותו ב-`instance`.
    
    ```Java
    // Dog.java (Concrete Subclass)
    public class Dog extends Animal {
        protected Dog() {}
    
        public static Animal theInstance() { // Not an override!
            if (instance == null) {
                instance = new Dog();
            }
            return instance;
        }
        // ... implementation of speak() ...
    }
    ```
    

#### העיקרון: "המרוץ ליצירה"

המחלקה הראשונה שקוראת למתודת `theInstance()` שלה, "זוכה" וקובעת את הטיפוס של המופע היחיד במערכת.

```Java
public static void main(String[] args) {

    Dog.theInstance(); // Dog "wins" and creates the single instance
    // ... later in the code ...
    Animal.theInstance().speak(); // This will call the speak() method of the Dog instance
    Cat.theInstance().speak();   // This ALSO returns the existing Dog object
}
```

### וריאנט 5: Multi-Singleton (שקף 23)

וריאנט המרחיב את התבנית כך שתגביל את מספר המופעים לכמות קבועה (N > 1). זה שימושי לניהול מאגרי משאבים (Pools).

```Java
public class MultiSingleton {
    protected static final int MAX_INSTANCES = 3;
    protected static int currentInstance = 0;
    protected static MultiSingleton[] instances = new MultiSingleton[MAX_INSTANCES];

    private MultiSingleton() { /* ... */ }

    public static MultiSingleton theInstance() {
        if (instances[currentInstance] == null) {
            instances[currentInstance] = new MultiSingleton();
        }
        MultiSingleton result = instances[currentInstance];
        // מעבר בין המופעים בסבב (Round-robin)
        currentInstance = (currentInstance + 1) % MAX_INSTANCES;
        return result;
    }
}
```

_הקוד מחזיר מופע עוקב מהמערך באמצעות פעולת מודולו._

---

## חסרונות וביקורת (Anti-Pattern)

ישנם חסרונות משמעותיים לתבנית, ולכן היא מכונה לעתים "אנטי-תבנית":

1. **התנהגות כמשתנה גלובלי:** המופע היחיד נגיש מכל מקום, מה שמקשה על מעקב ויוצר תלויות נסתרות.
    
2. **פגיעה בבדיקות (Testability):** קשה להחליף את המופע היחיד ב"אובייקט דמה" (Mock) לצורך בדיקות יחידה.
    
3. **הפרת SRP (עקרון האחריות היחידה):** המחלקה אחראית גם על הלוגיקה העסקית שלה וגם על ניהול מחזור החיים של עצמה.
    
4. **קושי בשינוי עתידי:** אם נרצה לבטל את המגבלה על מספר המופעים, נצטרך לשנות הרבה מקומות בקוד.
    

**אלטרנטיבה מודרנית:** הזרקת תלויות (DI) מאפשרת להשתמש בסינגלטון בצורה נקייה יותר, על ידי העברת האחריות על ניהול המופע היחיד ל-Framework חיצוני (למשל עם `@Singleton` ו-`@Inject`).

---
**קודם:** [[תבניות יצירה ( Creational Pattern)]]
**הבא:** [[Factory]]

**חזרה ל:** [[תבניות יצירה ( Creational Pattern)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]