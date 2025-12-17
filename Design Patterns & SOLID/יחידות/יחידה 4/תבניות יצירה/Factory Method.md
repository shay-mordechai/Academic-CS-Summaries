![](factory-method-diagram.webp)
הרעיון הוא להעביר את האחריות ליצירת האובייקט אל **תתי-המחלקות (Subclasses)**.

- **מבנה:** מחלקת הבסיס (היוצר, Creator), שהיא לרוב מופשטת, מגדירה מתודה מופשטת ליצירת אובייקט. מתודה זו היא ה-**Factory Method**.
- **מימוש:** כל מחלקה יורשת (Concrete Creator), כגון `RoadLogistics` או `SeaLogistics`, **דורסת** את ה-Factory Method ומחליטה איזה אובייקט קונקרטי (Product) ליצור ולהחזיר (כמו `Truck` או `Ship`).
- **קביעת הטיפוס:** הטיפוס הקונקרטי נקבע לפי **סוג המפעל שיצרת** (למשל, `RoadLogistics`), ולא לפי פרמטר חיצוני.


דוגמת הלוגיסטיקה מהמצגת (שקף 37):
![](Summeries/עיצוב/media/media/image87.png)

היררכיית המוצרים - Truck ו-Ship מממשים (implements) את הממשק Transport.

היררכיית היוצרים - RoadLogistics ו-SeaLogistics יורשות (extend) מהמחלקה המופשטת Logistics. (אין בציור)

- יש לנו ממשק Transport (המוצר) ומחלקות Truck ו-Ship (מוצרים קונקרטיים).
    
- יש לנו מחלקה מופשטת Logistics (היוצר). היא מכילה לוגיקה עסקית (planDelivery) שיודעת שהיא צריכה להשתמש באובייקט מסוג Transport, אבל היא לא יודעת איך ליצור אותו. לכן, היא מגדירה מתודה מופשטת: createTransport().
    
- יש לנו שתי מחלקות יורשות:
    

- RoadLogistics מממשת את createTransport כך שיחזיר new Truck().
    
- SeaLogistics מממשת את createTransport כך שיחזיר new Ship().
    

הלוגיקה ב-planDelivery (שנכתבה במחלקת Logistics המופשטת) פשוט קוראת ל-createTransport() ומקבלת את האובייקט הנכון לעבוד איתו, בלי לדעת אם זה Truck או Ship.

בקוד הראשי:
```Java
Logistics mylogistics;
if (customerNeedsSeaShipping) {
    mylogistics = new SeaLogistics();
} else {
    mylogistics = new RoadLogistics();
}

// The business logic runs, and inside, it calls createTransport()
// Polymorphism ensures the correct version (from SeaLogistics or RoadLogistics) is called
mylogistics.planDelivery();
```
הקוד הראשי קורא ל-planDelivery - מכאן והלאה הוא "עיוור".

אצל Logistics:
```Java
// " היוצר" המופשט (The Abstract Creator)
public abstract class Logistics {

    // 1. המטרה העסקית הראשית. היא לא יודעת איזה כלי רכב ייווצר.
    public void planDelivery() {
        // כדי לקבל אובייקט כלשהו קוראת ל-Factory Method
        Transport t = createTransport();

        // משתמשת באובייקט שנוצר דרך הממשק המשותף שלו
        t.deliver();
    }

    // 2. זוהי מתודה מופשטת - ה-Factory Method.
    // כל תת-מחלקה תהיה חייבת לממש אותה ולהחליט איזה אובייקט ליצור.
    public abstract Transport createTransport();
}
```

הקוד הראשי לא מחליט "אני רוצה משאית" או "אני רוצה ספינה". במקום זאת, הוא מחליט "אני צריך לוגיסטיקה יבשתית" או "אני צריך לוגיסטיקה ימית".

מה קורה בתוך planDelivery()? המתודה הזו, שכתובה במחלקת האב Logistics, היא כללית לחלוטין. היא אומרת:

- "אני צריך כלי תחבורה כלשהו. אני לא יודע איזה."
    
- "אני אקרא למתודה createTransport() כדי לקבל אחד."
    
- "אחרי שקיבלתי אותו, אני אשתמש בו כדי לבצע את המשלוח (transport.deliver())."
    

בעזרת פולימורפיזם, בזמן הריצה יקבע אם createTransport יצור סירה או יצור משאית.  
(בדומה לFactory Class שבזמן הריצה נקבע לפי הקלט מהמשתמש)

### Factory Method מול Factory Class  
מי אחראי על לוגיקת היצירה?

- Factory Class: היצירה מרוכזת במחלקה אחת, והלקוח אומר לה מה הוא רוצה (למשל, עם פרמטר String).
    

- היא נקבעת לפי פרמטר שאתה מעביר למתודת היצירה.
    

- Factory Method: היצירה מבוזרת בין תתי-מחלקות. הלקוח בוחר את תת-החלקה המתאימה, והיא כבר יודעת איזה אובייקט ליצור.
    

- היא נקבעת לפי סוג ה"מפעל" שיצרת מלכתחילה.
    

השיטה של Factory Method נחשבת לגמישה יותר ועומדת טוב יותר בעיקרון פתוח-סגור (OCP), כי כדי להוסיף מוצר חדש (למשל, Airplane), אתה פשוט יוצר מחלקת AirLogistics חדשה ולא צריך לשנות את הקוד הקיים.

### הבעיה ב-Factory Method

כמו שראינו בדוגמת Logistics, מחלקת האב (Logistics) אחראית גם על הלוגיקה העסקית (planDelivery) וגם על הגדרת המתודה ליצירה (createTransport).

- הבעיה:  Factory Method עלולה להפר SRP (כי מחלקת היוצר (Creator) מחזיקה גם לוגיקה עסקית וגם את מתודת היצירה).
    
- יתרון: עומד טוב יותר ב-OCP כיוון שהוספת מוצר חדש דורשת יצירת תת-מחלקה חדשה בלבד, ללא שינוי קוד קיים.
