
---

# תבנית עיצוב: Iterator (איטרטור)

**Iterator** היא תבנית עיצוב **התנהגותית (Behavioral)**.

## 🎯 מטרה ובעיה נפתרת

**הבעיה:** איך לאפשר דרך אחידה וסטנדרטית לעבור על האיברים של אוסף (כמו רשימה, עץ, או כל מבנה נתונים אחר), **מבלי לחשוף את המבנה הפנימי של האוסף**?

אם הקוד שלך (הלקוח) ייגש ישירות למערך הפנימי של רשימה, או יתחיל לנווט ידנית בצמתים של עץ, הוא יהיה תלוי חזק במימוש הספציפי, מה שמפר את עקרונות **[[OCP]]** ו-**[[DIP]]**.

**הפתרון:** האוסף (`Aggregate`) מספק "אובייקט איטרטור". לאובייקט הזה יש ממשק פשוט, בדרך כלל עם שתי מתודות:

- `hasNext()`: בודקת אם יש עוד איברים לעבור עליהם.
    
- `next()`: מחזירה את האיבר הבא בסדרה.
    

זהו בדיוק המנגנון שמאפשר ללולאת ה-**for-each** ב-Java לעבוד.

## 🏗️ מבנה התבנית (UML)
![](iterator-uml.png)
(מתוך מצגת OOPDE-W07Lec13 שקף 10)

---

## ⌨️ דוגמת קוד: מעבר סדור (In-Order) על עץ חיפוש בינארי (BST)

זוהי דוגמה קלאסית שממחישה את כוחה של התבנית.

**הבעיה:**

- יש לנו מבנה נתונים מורכב `BinarySearchTree`, הבנוי מ-`Node` (צמתים) עם בן שמאלי וימני.
    
- אנחנו רוצים לאפשר ללקוח לעבור על כל המספרים בעץ בסדר עולה (In-Order).
    
- **דרישה קריטית:** הלקוח לא אמור לדעת כלום על `Node` או רקורסיה. הוא רק רוצה לעבור על האיברים כאילו היו רשימה פשוטה.
    

השאיפה (קוד הלקוח):

אנחנו רוצים שהלקוח יוכל לכתוב לולאת for-each פשוטה:

```Java
BinarySearchTree myTree = ...;
for (Integer num : myTree) {
    System.out.println(num);
}
```

**הפלט הרצוי (מעבר סדור):**

```Java
2
5
8
```

### שלב 1: מימוש `Iterable` (ה"חוזה")

כדי שלולאת `for-each` תעבוד על אובייקט ב-Java, המחלקה שלו חייבת לחתום על "חוזה" שהיא יודעת לספק איטרטור. החוזה הזה הוא הממשק `Iterable<T>`.

אם לא היית כותב `implements Iterable<Integer>`, לולאת ה-`for-each` הייתה נכשלת בקומפילציה.

### שלב 2: מימוש מתודת `iterator()` (ה"מפעל")

לממשק Iterable יש מתודה אחת בלבד שחייבים לממש: iterator().

המתודה הזו היא בעצם "מפעל" קטן (Factory). כשלולאת ה-for-each מתחילה, היא קוראת למתודה זו פעם אחת כדי לקבל "עובד מומחה" (אובייקט Iterator) שיודע איך לבצע את המעבר בפועל.

```Java
// BinarySearchTree.java
public class BinarySearchTree implements Iterable<Integer> {
    private Node root;
    // ... (קוד של הוספה, מחיקה וכו') ...

    @Override
    public Iterator<Integer> iterator() {
        // יוצרים ומחזירים את ה"עובד המומחה" שיודע לבצע מעבר סדור
        return new InOrderIterator(root); 
    }
}
```

### שלב 3: מימוש ה-`Iterator` עצמו (ה"עובד המומחה")

עכשיו אנחנו צריכים לממש את האובייקט `InOrderIterator`. הוא חייב לממש את הממשק `Iterator<T>`, שדורש שתי מתודות: `hasNext()` ו-`next()`.

כאן נמצאת כל המורכבות הלוגית. ה-`InOrderIterator` שלנו יצטרך לנהל את המעבר הרקורסיבי על העץ (בדרך כלל באמצעות מחסנית - `Stack`) כדי לדעת איזה צומת להחזיר בכל קריאה ל-`next()`.


```Java
// InOrderIterator.java (יכולה להיות מחלקה פנימית)
private class InOrderIterator implements Iterator<Integer> {

    private Stack<Node> stack = new Stack<>();

    // הבנאי "מתכונן" ומגיע לצומת השמאלי ביותר (הקטן ביותר)
    public InOrderIterator(Node root) {
        pushLeftChildren(root);
    }

    @Override
    public boolean hasNext() {
        return !stack.isEmpty();
    }

    @Override
    public Integer next() {
        if (!hasNext()) throw new NoSuchElementException();
        
        Node currentNode = stack.pop(); // שולפים את האיבר הנוכחי להחזרה
        
        // אחרי שהחזרנו את עצמנו, אנחנו מתכוננים לאיבר הבא:
        // (הולכים ימינה פעם אחת, ואז שמאלה עד הסוף)
        pushLeftChildren(currentNode.right); 
        
        return currentNode.value;
    }

    // מתודת עזר פנימית לניהול המחסנית
    private void pushLeftChildren(Node node) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
    }
}
```

---

## 📚 שאלות ממבחנים

- **שאלות קוד פתוח:** אף שזו תבנית בתדירות נמוכה (פעמיים במבחנים), היא מופיעה בשאלות מימוש הדורשות להפוך אוסף לניתן למעבר באמצעות `for-each` (כלומר, מימוש `Iterable<T>`).
    
- **הרצאה 26 (שקף 24):** שאלה פתוחה. נתונה מחלקה `Album` עם רשימת שירים. המטלה היא להוסיף קוד כדי לאפשר `for-each` על אלבום.
    
- **תשפ"ד, א, ב, שאלה 1:** שאלה אמריקאית על מעקב אחר לוגיקה של `Iterator` מותאם אישית למחלקת `Range`.
    
- **תשפ"ב ב א שאלה 3א:** (מימוש `Iterable` עבור `Album`).
    
- **תשפ"ד א ב שאלה 4:** (הרחבת `Iterator` ל-`RetraceableArrayList`).
    
---
**קודם:** [[Strategy]]
**הבא:** [[יחידה 2 עקרונות העיצוב (SOLID)]]

**חזרה ל:** [[תבניות התנהגות]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]