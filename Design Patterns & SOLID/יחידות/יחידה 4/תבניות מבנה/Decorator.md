
---

# תבנית עיצוב: Decorator (קַשְׁטָן)

**Decorator** היא תבנית עיצוב **מבנית (Structural)**. זוהי תבנית חשובה מאוד שלרוב נשאלת במבחנים, במיוחד בשילוב עם תבנית [[Composite]].

## 🎯 מטרה ובעיה נפתרת

המטרה של Decorator היא **להוסיף אחריות או התנהגות לאובייקט בצורה דינאמית (בזמן ריצה)**, מבלי להשפיע על התנהגות של אובייקטים אחרים מאותה מחלקה.

התבנית פותרת את הבעיה של "פיצוץ" תתי-מחלקות (class explosion) שהיה נגרם משימוש בירושה לכל שילוב אפשרי של תכונות. Decorator מאפשרת הרכבה של פונקציונליות באמצעות **עטיפה (Composition)** של אובייקטים, ובכך שומרת על עיקרון **[[OCP]]** (פתוח להרחבה, סגור לשינוי).

### אנלוגיה: קישוט סוכה

- **הבסיס (ConcreteComponent):** יש לך `סוכה בסיסית`.
    
- **הקישוטים (ConcreteDecorators):** אתה רוצה "לעטוף" אותה בשכבות של קישוטים: `שרשרת נייר`, `אורות`, `פוסטר אושפיזין`.
    
- **התוצאה:** כל "עטיפה" (קישוט) מוסיפה התנהגות חדשה (למשל, `decorate()`), וניתן לשלב אותן בכל סדר.
    

## 🏗️ מבנה התבנית (UML)

1. **Component (רכיב):** ממשק משותף לכולם (למשל, `Beverage`), המגדיר את הפעולות (`getCost()`).
    
2. **ConcreteComponent (רכיב קונקרטי):** המימוש הבסיסי של האובייקט (למשל, `SimpleCoffee`).
    
3. **Decorator (קשטן):** מחלקה מופשטת שמממשת את ה-`Component` וגם **מחזיקה רפרנס** לאובייקט `Component` אחר (האובייקט הנעטף).
    
4. **ConcreteDecorator (קשטן קונקרטי):** מחלקות שיורשות מה-`Decorator` (כמו `Milk`, `Sugar`). כל אחת מהן מוסיפה את ההתנהגות הייחודית שלה (מחיר נוסף, תיאור נוסף) לפני או אחרי שהיא קוראת לפעולה של האובייקט שהיא עוטפת.
    

---

## ☕ דוגמת קוד: מערכת בית קפה

נניח שאנו מתכננים מערכת לקופה של בית קפה עם משקאות בסיס (קפה) ותוספות אופציונליות (חלב, סוכר).

### 1. הממשק המשותף (Component)

מגדיר את "החוזה" - מה כל משקה (בסיסי או מקושט) חייב לממש.

Java

```
// Beverage.java
public interface Beverage {
    double getCost();       // מתודה להחזרת המחיר
    String getDescription(); // מתודה להחזרת התיאור
}
```

### 2. הרכיב הבסיסי (ConcreteComponent)

האובייקט המקורי שעליו "יתלבשו" התוספות.

Java

```
// SimpleCoffee.java
public class SimpleCoffee implements Beverage {
    @Override
    public double getCost() {
        return 10.0; // מחיר קבוע לקפה בסיסי
    }
    @Override
    public String getDescription() {
        return "Simple Coffee"; // תיאור בסיסי
    }
}
```

### 3. הקשטן המופשט (Abstract Decorator)

זוהי המחלקה שמחזיקה את "הקסם". היא גם "משקה" (`implements Beverage`) וגם "עוטפת משקה" (`protected Beverage wrappedBeverage`).

Java

```
// BeverageDecorator.java
public abstract class BeverageDecorator implements Beverage {
    // 1. "העטיפה" - כל קישוט מחזיק רפרנס למשקה שהוא עוטף
    protected Beverage wrappedBeverage; 

    public BeverageDecorator(Beverage beverage) {
        this.wrappedBeverage = beverage;
    }

    // 2. מימוש ברירת מחדל: פשוט מעביר את הקריאה הלאה
    @Override
    public double getCost() {
        return wrappedBeverage.getCost();
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription();
    }
}
```

### 4. הקשטנים הקונקרטיים (ConcreteDecorators)

כל תוספת יורשת מה-Decorator המופשט ומוסיפה את הלוגיקה שלה.

Java

```
// Milk.java
public class Milk extends BeverageDecorator {
    public Milk(Beverage beverage) {
        super(beverage);
    }

    @Override
    public double getCost() {
        // מחזיר את מחיר המשקה העטוף + מחיר התוספת
        return wrappedBeverage.getCost() + 2.0;
    }

    @Override
    public String getDescription() {
        // מחזיר את תיאור המשקה העטוף + הוספת טקסט
        return wrappedBeverage.getDescription() + ", with Milk";
    }
}
```

Java

```
// Sugar.java
public class Sugar extends BeverageDecorator {
    public Sugar(Beverage beverage) {
        super(beverage);
    }

    @Override
    public double getCost() {
        return wrappedBeverage.getCost() + 1.0;
    }

    @Override
    public String getDescription() {
        return wrappedBeverage.getDescription() + ", with Sugar";
    }
}
```

### 5. שימוש בלקוח (Client)

הלקוח מרכיב את המוצר באופן דינמי באמצעות עטיפות מקוננות.

Java

```
public class Main {
    public static void main(String[] args) {
        // מתחילים עם קפה פשוט
        Beverage myCoffee = new SimpleCoffee();
        
        // מקשטים אותו דינמית בחלב
        myCoffee = new Milk(myCoffee);

        // ומקשטים גם בסוכר
        myCoffee = new Sugar(myCoffee);

        // הפלט: "Simple Coffee, with Milk, with Sugar"
        System.out.println(myCoffee.getDescription());
        
        // הפלט: 13.0 (10 + 2 + 1)
        System.out.println(myCoffee.getCost());
    }
}
```

כאשר קוראים ל-`getCost()`, הקריאה "מחלחלת" פנימה:

1. `Sugar` שואל את `Milk`: "מה המחיר שלך?"
    
2. `Milk` שואל את `SimpleCoffee`: "מה המחיר שלך?"
    
3. `SimpleCoffee` עונה: "10".
    
4. `Milk` מקבל "10", מוסיף 2 (המחיר שלו), ומחזיר "12".
    
5. `Sugar` מקבל "12", מוסיף 1 (המחיר שלו), ומחזיר את התשובה הסופית: "13".
    

---

## 🤝 למה Decorator מגיע לרוב עם "חברים"?

המטרה של Decorator היא "לעטוף" אובייקט קיים. במבחנים, לרוב האובייקט שצריך לעטוף הוא בעצמו חלק מתבנית אחרת.

- **שילוב עם [[Composite]] (הנפוץ ביותר):**
    
    - **הבעיה:** נתון מבנה היררכי (כמו `Playlist` ו-`Song`).
        
    - **השילוב:** מבקשים להוסיף יכולת חדשה לאובייקט בודד (`Song`) או לקבוצה שלמה (`Playlist`) בצורה שקופה.
        
    - **דוגמה:** (הרצאה 26, שקפים 54-60) צריך להוסיף צליל לפעולת ה-`draw` של צורות גיאומטריות. ניתן "לקשט" `Triangle` בודד או `Geometries` שלם, והלקוח לא ירגיש בהבדל.
        
- **שילוב עם [[Adapter]]:**
    
    - **הבעיה:** נתון אובייקט עם ממשק לא מתאים.
        
    - **השילוב:** קודם כל "מתאמים" אותו עם [[Adapter]] כדי שיתאים למערכת, ואז "מקשטים" את המתאם ביכולות נוספות.
        
    - **דוגמה:** (תשפ"ג א ב 7) שאלת מערכת הנשק. קודם התאמנו את ה-`Missile` עם Adapter, ורק אז קישטנו אותו עם `TelescopicAim` ו-`Silencer`.
        
- **שילוב עם [[Factory]]:**
    
    - **הבעיה:** צריך ליצור אובייקט מקושט, אבל שילוב הקישוטים נקבע באופן דינמי.
        
    - **השילוב:** בונים [[Factory]] שמקבל בקשה (למשל, מחרוזת) ומחזיר את האובייקט הבסיסי כשהוא כבר עטוף בכל ה-Decorators הנכונים.
        
    - **דוגמה:** (מעבדה 9) שאלת השש בש. ה-Factory מקבל מחרוזת כמו "Analyzer TrashTalker" ומחזיר אובייקט `SheshBeshPlayer` שכבר עטוף בשני הקישוטים.
        

**שורה תחתונה למבחן:** כשאתה רואה שאלה שדורשת "להוסיף יכולות אופציונליות באופן דינמי", תחשוב מיד **Decorator**. השלב הבא הוא לשאול את עצמך: "על מה אני מפעיל את ה-Decorator הזה?"

## קשר ל-AOP

תכנות מונחה היבטים (**Aspect Oriented Programming - AOP**) נחשב לפיתוח של מושג תבנית Decorator.

---

## 📚 שאלות ממבחנים

### תבנית Decorator

- **מעבדה 9 (שקף 4):** שאלה אמריקאית קלאסית על זיהוי התבנית (מערכת חלונות עם תוספות).
    
- **מבחן תשפ"ד, א, א, שאלה 9:** שאלה אמריקאית דומה על מערכת להכנת כרזות עם תוספות.
    
- **מעבדה 9 (שקפים 6-16):** שאלה פתוחה ממבחן. שחקן שש בש ו"שכלולים" להתנהגות שלו (`Analyzer`, `TrashTalker`).
    
- **הרצאה 16 (שקפים 26-27):** שאלה פתוחה על מימוש Decorator למערכת ניווט.
    

### שילוב Composite + Decorator

- **הרצאה 26 (שקפים 54-60):** שאלה פתוחה. נתונה היררכיה של צורות גיאומטריות (`Geometry` ו-`Geometries`). המטלה היא להוסיף יכולת להשמיע צליל בעת ציור, מבלי לשנות את המחלקות הקיימות.
    

### אזכורים נוספים

- **תשפ"ג א א שאלה 1:** (זיהוי Decorator בתרשים משולב).
    
- **תשפ"ד ב א שאלה 9:** (סידור תבניות: Composite, Decorator, Proxy).
    
- **תשפ"א ב א שאלה 3א:** (שש בש, הוספת יכולות TrashTalker/Analyzer).
    
- **תשפ"ד ב א שאלה 5:** (הוספת הנפשות לצורות).
    
- **תשפ"ה ב ב שאלה 13:** (חישוב מספר השילובים האפשריים עם Decorators).
    
- **תשפ"ד א א שאלה 9:** (מערכת כרזות, 2 תוספות).
    
---
**קודם:** [[Adapter]]
**הבא:** [[Composite]]

**חזרה ל:** [[תבניות מבנה (Structural Patterns)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]
