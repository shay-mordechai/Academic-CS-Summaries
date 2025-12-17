
---

# תבנית עיצוב: Visitor (מבקר)

**Visitor** היא תבנית עיצוב **התנהגותית (Behavioral)** שנחשבת למתוחכמת וחזקה.

## 🎯 מטרה ובעיה נפתרת

**הבעיה:** נניח שיש לך מבנה של אובייקטים, לרוב מבנה היררכי כמו עץ שבנית עם תבנית [[Composite]] (למשל, עץ של צורות גיאומטריות, או עץ של חלקי מכונה), ואתה רוצה להוסיף **פעולה חדשה** שניתן לבצע על כל האובייקטים במבנה, אבל **מבלי לשנות את הקוד של המחלקות הקיימות** (כדי לא להפר את עיקרון **[[OCP]]**).

**אנלוגיה:** טכנאי (Visitor) שמבקר במכונה מורכבת (Composite).

- המכונה בנויה מחלקים שונים (מנוע, גלגלי שיניים).
    
- הטכנאי יכול לבצע פעולה על כל חלק (למשל, "בדוק תקינות").
    
- אם נרצה להוסיף פעולה חדשה (למשל, "עדכן קושחה"), לא נרצה לפרק את המכונה ולשנות כל חלק. במקום זאת, נשלח "טכנאי עדכון קושחה" חדש שיודע איך לבצע את הפעולה החדשה על כל חלק.
    

## 🏗️ הפתרון: Double Dispatch

הפתרון מבוסס על טכניקה שנקראת **Double Dispatch**. הרעיון הוא להוציא את הלוגיקה של הפעולה החדשה למחלקה חיצונית ונפרדת שנקראת `Visitor`.

### מבנה ב-UML
![](visitor-uml.png)

1. **Element (רכיב):** ממשק משותף לכל האובייקטים במבנה (למשל, `Shape`). מגדיר מתודה `accept(Visitor v)`.
    
2. **ConcreteElement (רכיב קונקרטי):** המחלקות הקונקרטיות במבנה (למשל, `Circle`, `Drawing`). כל אחת מממשת את `accept(Visitor v)` בצורה פשוטה: `v.visit(this);`.
    
3. **Visitor (מבקר):** ממשק שמגדיר מתודת `visit` **נפרדת לכל סוג של ConcreteElement**. לדוגמה: `visit(Circle c)`, `visit(Drawing d)`.
    
4. **ConcreteVisitor (מבקר קונקרטי):** מחלקה שמממשת את ממשק ה-`Visitor`. היא מכילה את הלוגיקה של הפעולה החדשה שרצינו להוסיף, מפוצלת בין מתודות ה-`visit` השונות.
    
5. **ObjectStructure (מבנה האובייקטים):** לרוב זהו ה-`Composite` הראשי, שמחזיק את כל ה-Element-ים ויודע לעבור עליהם (למשל, ברקורסיה) ולהפעיל `accept` על כולם.
    

---

## ⌨️ דוגמת קוד 1: מודמים (מהמצגת)

**הבעיה:** יש לנו היררכיה של מודמים (`HayesModem`, `ZoomModem`). אנחנו רוצים להוסיף פונקציונליות חדשה לכולם ("קבע תצורה עבור Unix") מבלי לשנות את הקוד הקיים שלהם (הפרת OCP).

### 1. Element (ו-ConcreteElement)

מוסיפים את מתודת `accept` לממשק `Modem` ודואגים שכל מחלקה קונקרטית תממש אותה.

```Java
// Element Interface
public interface Modem {
    void dial();
    void send();
    void accept(ModemVisitor v); // 1. דלת הכניסה ל-Visitor
}

// ConcreteElement 1
public class HayesModem implements Modem {
    // ... (מימוש dial, send) ...

    // 2. מימוש Accept
    public void accept(ModemVisitor v) {
        v.visit(this); // קורא ל-visit עם הטיפוס הספציפי "HayesModem"
    }
}

// ConcreteElement 2
public class ZoomModem implements Modem {
    // ... (מימוש dial, send) ...

    // 2. מימוש Accept
    public void accept(ModemVisitor v) {
        v.visit(this); // קורא ל-visit עם הטיפוס הספציפי "ZoomModem"
    }
}
```

### 2. Visitor (ו-ConcreteVisitor)

יוצרים את ה-Visitor שיכיל את הפעולה החדשה.

```Java
// 3. Visitor Interface
public interface ModemVisitor {
    void visit(HayesModem m); // מתודה נפרדת לכל סוג
    void visit(ZoomModem m);
}

// 4. ConcreteVisitor
public class UnixModemConfigurator implements ModemVisitor {
    // 5. הלוגיקה של הפעולה החדשה
    @Override
    public void visit(HayesModem m) {
        // ... קוד תצורה ספציפי למודם Hayes עבור Unix
    }
    
    @Override
    public void visit(ZoomModem m) {
        // ... קוד תצורה ספציפי למודם Zoom עבור Unix
    }
}
```

### 3. Client (השימוש)

הלקוח פשוט יוצר את ה-Visitor (`UnixModemConfigurator`) ושולח אותו "לבקר" בכל המודמים.

```Java
// 6. Client
public class Main {
    public static void main(String[] args) {
        Modem[] modems = { new HayesModem(), new ZoomModem() };
        ModemVisitor unixVisitor = new UnixModemConfigurator();

        for (Modem m : modems) {
            m.accept(unixVisitor); // מנגנון ה-Double Dispatch קורה כאן
        }
    }
}
```

**מה קורה בלולאה (Double Dispatch):**

1. אם `m` הוא `HayesModem`, הוא קורא ל-`unixVisitor.visit(this)` ⬅️ הקומפיילר יודע ש-`this` הוא `HayesModem` ובוחר אוטומטית במתודה `visit(HayesModem m)`.
    
2. אם `m` הוא `ZoomModem`, הוא קורא ל-`unixVisitor.visit(this)` ⬅️ הקומפיילר יודע ש-`this` הוא `ZoomModem` ובוחר אוטומטית במתודה `visit(ZoomModem m)`.
    

בכך הוספנו יכולת חדשה למערכת מבלי לגעת בקוד של המודמים הקיימים.

---

## ⌨️ דוגמת קוד 2: ייצוא צורות ל-XML

**הבעיה:** יש לנו מבנה `Composite` של צורות (`Dot`, `Circle`, `Rectangle`). אנחנו רוצים להוסיף יכולת לייצא כל צורה ל-XML, מבלי להוסיף מתודת `exportXML()` לכל אחת מהן.

### 1. הוספת `accept` ל-Element

```Java
// Element Interface
public interface Shape {
    void draw();
    void accept(Visitor v); // הוספנו את דלת הכניסה
}

// ConcreteElement (Leaf)
public class Dot implements Shape {
    // ...
    @Override
    public void accept(Visitor v) {
        v.visit(this);
    }
}
```

### 2. יצירת ה-Visitor

```Java
// Visitor Interface
public interface Visitor {
    void visit(Dot d);
    void visit(Circle c);
    void visit(Rectangle r);
    // ...
}

// ConcreteVisitor
public class XMLExportVisitor implements Visitor {
    
    @Override
    public void visit(Dot d) {
        // לוגיקה לייצוא נקודה ל-XML
        System.out.println("<dot>...</dot>");
    }

    @Override
    public void visit(Circle c) {
        // לוגיקה לייצוא עיגול ל-XML
        System.out.println("<circle>...</circle>");
    }
    // ... וכן הלאה
}
```

### 3. שימוש ב-Client

```Java
// Client
Shape[] shapes = { new Dot(), new Circle(), new Rectangle() };
Visitor xmlExportVisitor = new XMLExportVisitor();

for (Shape s : shapes) {
    s.accept(xmlExportVisitor);
}
```

---

## 💡 וריאנט מתקדם: Acyclic Visitor

- **הבעיה ב-Visitor הרגיל:** נוצרת **תלות מעגלית**. ה-`Visitor` חייב להכיר את כל ה-`ConcreteElement`-ים, וה-`Element`-ים חייבים להכיר את ה-`Visitor`. התוצאה היא שאם מוסיפים מחלקה חדשה למבנה (למשל, `Star`), חייבים לעדכן את הממשק `Visitor` ואת _כל_ המחלקות שמממשות אותו - **הפרה של OCP**.
    
- **הפתרון של Acyclic Visitor:** במקום ממשק `Visitor` אחד, יוצרים **ממשק Visitor קטן לכל מחלקה קונקרטית** (למשל, `CircleVisitor`, `DrawingVisitor`). ה-`ConcreteVisitor` (למשל, `XmlExportVisitor`) יממש רק את הממשקים של המחלקות שרלוונטיות אליו. זה פותר את התלות המעגלית והופך את המערכת לגמישה יותר להוספת מחלקות חדשות.
    

---

## 📚 שאלות ממבחנים

- **Double Dispatch ו-OCP:** ודא שאתה מבין היטב את מנגנון ה-Double Dispatch. המטרה העיקרית של Visitor היא להוסיף פעולות (כמו ייצוא ל-XML) מבלי לשנות את המחלקות הקיימות, ובכך לעמוד ב-**[[OCP]]** ו-**[[SRP]]** (הפרדת הפעולות מהמבנה). (תשפ"ב א ב 3)
    
- **Acyclic Visitor:** התרשים של Visitor לא מעגלי מופיע בשאלות פתוחות, כגון מימוש מכון רישוי לרכבים. (תשפ"ג ב ב)
    

### אזכורים ספציפיים

- **הרצאה 15 (שקפים 39-44):** שאלה פתוחה (Composite + Visitor) למחיקת ענפים ריקים מעץ רקורסיבי.
    
- **מבחן תשפ"ב, א, א, שאלה 17:** שאלה פתוחה על מימוש Acyclic Visitor למערכת חיישנים.
    
- **מבחן תשפ"ד, א, ב, שאלה 17:** שאלה פתוחה (Composite + Visitor). מידול מערכת מוזיקה (`Song` ו-`Playlist`) ומימוש Visitor לחישוב משך זמן כולל.
    
- **תשפ"ג א א שאלה 5:** (מימוש Visitor לא מעגלי).
    
- **תשפ"ג ב ב שאלה 4:** (Visitor לא מעגלי, מכון רישוי לרכבים).
    
- **תשפ"ה ב ב שאלה 3א:** (מערכת בחינות, הוספת פעולות בדיקה סטטיסטית והעתקות).
    
---
**קודם:** [[State]]
**הבא:** [[Command]]

**חזרה ל:** [[תבניות התנהגות]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]