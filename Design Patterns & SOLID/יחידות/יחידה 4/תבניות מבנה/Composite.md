
---

# תבנית עיצוב: Composite (הרכבה)

Composite היא תבנית עיצוב מבנית (Structural).

(מצגת: OOPDE-W07Lec14 - Design patterns - Composite + State.pptx)

## 🎯 מטרה ובעיה נפתרת

המטרה של התבנית היא לאפשר לנו להתייחס לאובייקט בודד ("עלה") ולקבוצה של אובייקטים ("ענף" שמכיל עלים וענפים אחרים) **באותה הדרך האחידה**.

התבנית מטפלת במצב שבו יש לנו אובייקטים המאורגנים במבנה היררכי (כמו עץ), ואנחנו רוצים שהקוד שלנו (הלקוח) יוכל להתייחס לאובייקט בודד ולקבוצה של אובייקטים באותה צורה.

**אנלוגיה קלאסית: מערכת קבצים**

- יש לנו **קבצים** (אובייקטים בודדים / Leaves).
    
- יש לנו **תיקיות** (קבוצות / Composites) שיכולות להכיל קבצים ותיקיות אחרות.
    
- אנו רוצים לבצע את אותה פעולה, למשל `calculateSize()`, גם על קובץ בודד וגם על תיקייה שלמה, מבלי שהקוד שקורא לפעולה יצטרך להבדיל ביניהם (`if (item is File) ... else if (item is Folder) ...`).
    

## 🏗️ מבנה התבנית (UML)
![](decorator-uml.png)

הפתרון, כפי שמוצג במצגת (שקף 6), הוא ליצור ממשק משותף לכל הרכיבים במבנה.

התבנית מורכבת משלושה חלקים עיקריים:

1. **Component (רכיב):** זהו הממשק או המחלקה המופשטת המשותפת לכולם (למשל `Shape`). הוא מגדיר את הפעולות שניתן לבצע על כל אובייקט במבנה (למשל, `draw()`).
    
2. **Leaf (עלה):** אלו האובייקטים הבודדים והבסיסיים במבנה (למשל `Circle`, `Triangle`). הם מממשים את הפעולות מה-Component באופן ישיר.
    
3. **Composite (רכיב מורכב):** אלו האובייקטים המורכבים, "הענפים" בעץ (למשל `Drawing`), שיכולים להכיל בתוכם רשימה של "ילדים" (שיכולים להיות Leafs או Composites אחרים).
    

### ניתוח קשרים (UML)

- **קשר implement:** המחלקה `Drawing` היא גם "סוג של" `Shape` (מממשת את הממשק).
    
- **קשר aggregation/composition:** למחלקה `Drawing` "יש לה" אוסף של `Shape`-ים.
    
- **קשר association:** המחלקה `main` (הלקוח) "משתמשת ב"ממשק `Shape`.
    
- **שמירה על [[DIP]]:** הלקוח תלוי במודל ברמה גבוהה (`Shape`), ולא תלוי במממשים שברמה הנמוכה (`Circle`, `Triangle`, `Drawing`).
    
- **שמירה על [[OCP]]:** אם נרצה להוסיף צורה חדשה (למשל `Square`), לא צריך לשנות קוד קיים אלא פשוט לקבוע שהיא תממש את הממשק `Shape`.
    

---

## ⌨️ דוגמת קוד (צורות גיאומטריות)

### 1. Component (הממשק המשותף)

הממשק המשותף שכל הרכיבים (העלים והענפים) יממשו.

Java

```
// Component
public interface Shape {
    void draw();
}
```

### 2. Leaf (העלים)

האובייקטים הבודדים.

Java

```
// Leaf 1
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("--Drawing a Circle (o)");
    }
}

// Leaf 2
public class Triangle implements Shape {
    @Override
    public void draw() {
        System.out.println("--Drawing a Triangle (△)");
    }
}
```

### 3. Composite (הרכיב המורכב)

זו המחלקה שמייצגת "קבוצה" של אובייקטים. היא מחזיקה רשימה של `Shape`-ים ויודעת להפעיל את הפקודות על כל ה"ילדים" שלה.

Java

```
// Composite
public class Drawing implements Shape {
    private List<Shape> shapes = new ArrayList<>();

    public void addShape(Shape s) {
        shapes.add(s);
    }

    public void removeShape(Shape s) {
        shapes.remove(s);
    }

    @Override
    public void draw() {
        System.out.println("--Drawing a group of shapes:");
        // מאצילה (delegates) את פקודת הציור לכל ה"ילדים" שלה
        for (Shape s : shapes) {
            s.draw();
        }
    }
}
```

### 4. Client (הלקוח)

הלקוח מתייחס לכל האובייקטים (בודדים או מורכבים) באותה צורה, דרך ממשק `Shape`.

Java

```
// Client
public class Main {
    public static void main(String[] args) {
        // יוצרים עלים
        Shape circle1 = new Circle();
        Shape triangle1 = new Triangle();
        
        // יוצרים רכיב מורכב (תיקייה)
        Drawing subDrawing = new Drawing();
        subDrawing.addShape(new Circle());
        subDrawing.addShape(new Triangle());

        // יוצרים רכיב מורכב ראשי
        Drawing mainDrawing = new Drawing();
        mainDrawing.addShape(circle1);
        mainDrawing.addShape(triangle1);
        mainDrawing.addShape(subDrawing); // מכניסים תיקייה לתוך תיקייה

        // הלקוח קורא לפעולה אחת בלבד
        System.out.println("Drawing the main composition:");
        mainDrawing.draw();
    }
}
```

**הפלט הצפוי:**

```
Drawing the main composition:
--Drawing a group of shapes:
----Drawing a Circle (o)
----Drawing a Triangle (△)
----Drawing a group of shapes:
------Drawing a Circle (o)
------Drawing a Triangle (△)
```

---

## 🌊 תרשים רצף (Sequence Diagram) - הרקורסיה

התרשים במצגת (שקף 7) מראה איך קריאה אחת מהלקוח מתפשטת לאורך כל העץ:

1. **הלקוח (main)** מבצע קריאה אחת בלבד: `draw()` על האובייקט הראשי, `mainDrawing` (שהוא Composite).
    
2. **הרכיב המורכב מפיץ:** `mainDrawing` מקבל את הקריאה. הוא עובר על כל ה"ילדים" ברשימה שלו וקורא ל-`draw()` על כל אחד מהם.
    
    - הוא קורא ל-`draw()` על `circle1` (שהוא Leaf) -> מדפיס.
        
    - הוא קורא ל-`draw()` על `triangle1` (שהוא Leaf) -> מדפיס.
        
    - הוא קורא ל-`draw()` על `subDrawing` (שהוא בעצמו Composite).
        
3. **הרקורסיה נמשכת:**
    
    - `subDrawing` מקבל את הקריאה `draw()` ובתורו מעביר אותה לכל הילדים שלו (העיגול והמשולש שבתוכו).
        

הנקודה החשובה: הלקוח ביצע פעולה אחת פשוטה על הרכיב הראשי. המבנה ההיררכי של ה-Composite דאג להפיץ את הפקודה הזו לכל החלקים בעץ באופן שקוף.

---

## 📚 שאלות ממבחנים

תבנית זו נשאלת לרוב בשילוב עם תבניות אחרות.

- ודא שאתה מיומן בניתוח תרשים שבו **Composite** (מבנה עץ) משתלב עם **[[Decorator]]** (הוספת יכולת על הענפים והעלים) (מבחן תשפ"ד ב א).
    
- **חישוב שילובים:** תרגל שאלות הדורשות חישוב מספר השילובים האפשריים של רכיב בסיס עם תוספות אופציונליות (תשפ"ה ב א).
    
- **תשפ"ג א א שאלה 1:** (זיהוי Composite בתרשים משולב).
    
- **תשפ"ד ב א שאלה 9:** (סידור תבניות: Composite, Decorator, Proxy).
    
- **תשפ"ד ב א שאלה 3ב:** (פורטל חדשות/קטגוריות).
    
- **תשפ"ד ב א שאלה 3א:** (פורטל חדשות/מועדונים).
    
- **תשפ"ג ב א שאלה 4א:** (רשת עצבית, מימוש נוירון מורכב).
    
---
**קודם:** [[Decorator]]
**הבא:** [[תבניות התנהגות (Behavioral Pattern)]]

**חזרה ל:** [[תבניות מבנה (Structural Patterns)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]