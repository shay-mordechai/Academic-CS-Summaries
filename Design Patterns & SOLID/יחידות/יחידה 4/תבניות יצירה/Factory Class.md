
זוהי מחלקה מרכזית שבה יש פונקציה לייצור אובייקטים.

- **מטרה:** בית החרושת מספק אובייקטים שונים שמממשים ממשק משותף (Product). הלקוח משתמש באובייקטים לפי הממשק הכללי, וכך אינו נחשף למימוש הקונקרטי.
- כאשר המחלקה מייצרת אובייקטים מסוגים שונים, היא נקראת **Toolkit**.
- היא מבצעת **כימוס (encapsulation)** לתהליך היצירה, דבר שימושי כאשר תהליך היצירה מורכב או תלוי בגורמי קונפיגורציה או קלט משתמש.



[דוגמת הקוד מהמצגת (שקפים 28-29):]

![](factory-class.png)

תזכורת: הקשר בין ShapeFactory לבין האובייקטים שהוא יוצר (כמו Circle, Rectangle) הוא קשר של תלות (Dependency). בתרשימי UML, נהוג לסמן את התלות הזו באופן ספציפי יותר עם התווית <<creates>>.  
(לכן אמור להיות קו מקוקו לcreates).

**בעיה:** **Factory Class** עלולה להפר את **OCP** (עיקרון פתוח/סגור). הוספת מוצר חדש (כמו צורה חדשה) מחייבת שינוי בקוד ה-Factory (למשל, הוספת `else if`).

### קוד דוגמה בסיסי (ShapeFactory)

```Java
// ShapeFactory.java - The Factory
public class ShapeFactory {
    Scanner scanner = new Scanner(System.in);

    // use getShape method to get object of type shape
    public Shape getShape(String shapeType){
       if(shapeType == null){
          return null;
       }
       if(shapeType.equalsIgnoreCase("CIRCLE")){
          System.out.println("Enter values for the " +
              "center coordinates and the radius");
          double x = scanner.nextDouble();
          double y = scanner.nextDouble();
          double radius = scanner.nextDouble();
          return new Circle(x, y, radius); // יוצר אובייקט קונקרטי
       }
       if(shapeType.equalsIgnoreCase("RECTANGLE")){
          return new Rectangle(...); // קטע קוד חסר במקור, אך הכוונה ברורה
       }
       return null;
    }
}

//Main.java - The Client
public static void main(String[] args) {
	ShapeFactory shapeFactory = new ShapeFactory();
	
	// The client doesn't know about the "Circle" class, only about the factory and the "ShapeFactory"
	Shape shape1 = shapeFactory.getShape("CIRCLE");
	shape1.draw();
	}
```

**קשר UML:** הקשר בין `ShapeFactory` למוצר הקונקרטי (כמו `Circle`) הוא קשר **תלות (Dependency)**, המסומן כ-`<<creates>>`.