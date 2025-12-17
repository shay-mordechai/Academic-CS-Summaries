
---

# יחידה 5: תכנות מתקדם ו-Frameworks

הסיכום יתמקד בטכניקות שמאפשרות גמישות, גנריות והפרדת דאגות.

| **נושא**                                  | **נקודות ממוקדות לסיכום**                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **[[Generics-Streams]]**                  | **Streams:** שליטה בפונקציות עיקריות כמו `map`, `reduce`, `filter`, `collect`.<br><br>  <br><br>תרגול פתרון שאלות הדורשות `groupingBy` וחישוב סטטיסטי (כמו שכיחות או ממוצע).<br><br>  <br>  <br><br>**Generics (Wildcards):** הבנת **PECS** (`Producer Extends`, `Consumer Super`).<br><br>  <br><br>תרגול כתיבת חתימות גנריות מורכבות.                                     |
| **[[DI]] (Dependency Injection)**         | **IoC:** הבנת העיקרון של היפוך שליטה (Framework מנהל את בקרת הזרימה).<br><br>  <br>  <br><br>**DI:** שיטה לאתחול אובייקטים ופתרון תלויות. שליטה בשימוש ב-`@Inject` (Constructor Injection או Field Injection), ושימוש ב-`@Named`, `@Alternative` או `@Produces` לפתרון דו-משמעות באתחול (כאשר יש מספר מימושים לממשק).                                                       |
| **[[AOP]] (Aspect-Oriented Programming)** | **מטרה:** הפרדת דאגות חוצות (Cross-Cutting Concerns) כמו `Log` או אבטחה.<br><br>  <br>  <br><br>**מונחים:** שליטה במושגים **Aspect**, **Pointcut** ו-**Advice**.<br><br>  <br><br>**Pointcut:** תרגול כתיבת ביטויים לזיהוי מתודות (שימוש ב-`execution` וב-`@annotation`).<br><br>  <br><br>**דמיון:** AOP דומה בתפיסה לתבנית **[[Decorator]]** בהוספת פונקציונליות חיצונית. |

---

## 📚 שאלות ממבחנים (תשפ"ד ב' ב' - 3)

אנו עוברים ליחידה 5 ומתמודדים עם שתי שאלות אלה מהמבחן. הן מכסות את הנושאים המרכזיים של היחידה: **[[Generics-Streams]]** ו-**[[AOP]]**.

### פתרון שאלה 3: הכללת הפונקציה `countModes` (Generics + Streams)

הבעיה:

יש לנו פונקציה countModes שעובדת מצוין עבור List<Worker>, אבל היא "תפורה" במיוחד למחלקה Worker ולמתודה getSalary. אנחנו רוצים להפוך אותה לגנרית כדי שתוכל לעבוד על כל רשימה של אובייקטים (כמו Animal) ועל כל פונקציה שמוציאה מהם ערך מספרי (כמו getNumOfLegs).

> "הניחו שנתונה לכם רשימה של `Worker` בשם `workers` ורשימה של `Animal` בשם `animals`. במחלקה `worker` מוגדרת הפונקציה `getSalary` ובמחלקה `Animal` מוגדרת הפונקציה `getNumOfLegs`. נתון קוד שמחשב את מספר השכיחים בהתפלגות השכר בהנתן רשימת `Worker`. טיפוס ההחזרה של הפונקציה הוא `long`. 
> ```Java
> 1. public static long countModes(List<Worker> workers){
> 2.     Collection<Long> salaryFrequencies = workers.stream()
> 3.         .map(Worker::getSalary)
> 4.         .collect(groupingBy(Function.identity(), counting()))
> 5.         .values();
> 6.     long maxFrequency = salaryFrequencies.stream()
> 7.         .max(Long::compare).get();
> 8.     return salaryFrequencies.stream()
> 9.         .filter(n -> n == maxFrequency).count();
> 10. }
> ```

> נרצה להכליל את הקוד הנתון... כיצד יש לשנות את הפונקציה... כתוב קריאה לפונקציה עם `workers` וקריאה לפונקציה עם `animals`."

הפתרון:

הפתרון הוא להשתמש ב-Java Generics. נהפוך את הפונקציה לגנרית ונעביר לה כפרמטר נוסף את הפונקציה שאחראית על חילוץ הערך המספרי.

**הקוד המתוקן:**

**שורה 1 (חתימת הפונקציה):**

- `<T>` מגדיר את הפונקציה כגנרית. היא מקבלת רשימה מכל סוג `T`.
    
- הפרמטר החדש `Function< ? super T, Long> valueExtractor` הוא פונקציה שיודעת לקחת אובייקט מסוג `T` ולהוציא ממנו `Long`.
    
```Java
public static <T> long countModes(List<T> items, Function< ? super T, Long> valueExtractor) {
```

**שורה 3 (הפעלת הפונקציה הגנרית):**

- במקום לקרוא ישירות ל-`Worker::getSalary`, נשתמש בפונקציה הגנרית שקיבלנו כפרמטר.

```Java
.map(valueExtractor)
```

**קריאות לפונקציה:**

- עבור עובדים:
    
    (אנחנו משתמשים בביטוי למדא להעביר את הפונקציה getSalary ומבצעים המרה ל-long)
    
    ```Java
    countModes(workers, worker -> (long) worker.getSalary());
    ```
    
- **עבור חיות:**
    
    ```Java
    countModes(animals, animal -> (long) animal.getNumOfLegs());
    ```
    

---

### פתרון שאלה 2: שימוש ב-AOP (Aspect-Oriented Programming)

הבעיה:

אנחנו צריכים להוסיף שתי התנהגויות "רוחביות" (cross-cutting) מבלי לזהם את הקוד העסקי הקיים:

1. **אבטחה:** לוודא שרק מנהלים (Admin) קוראים לפונקציות מסוימות.
    
2. **תיעוד (Logging):** לתעד קריאות לפונקציות שמחזירות מידע רגיש.
    

> "בחברה מסחרית מעוניינים להעלות את רמת האבטחה... עבור כל פונקציה המסומנת בביאור `@adminOnly`, יש לוודא כי המשתמש שקרא לפונקציה הוא אכן בתפקיד `Admin`, ובמידה ולא, לזרוק חריגה.
> 
> אם פונקציה מקבלת ארגומנט מסוג `User` ומחזירה מחרוזת שמתחילה ב-"Sensitive", יש לכתוב בקובץ מעקב את שם המשתמש, את שם הפונקציה ואת המחרוזת. לצורך כך, הניחו כי יש פונקציה סטטית `writeToLog`..."

הפתרון:

נבנה מחלקת Aspect אחת שתכיל את שני ה-Advices הנדרשים.

**הקוד המלא (דוגמה למימוש):**

```Java
@Aspect
public class SecurityAndLoggingAspect {

    // Pointcut לפונקציות הדורשות הרשאת מנהל
    @Pointcut("@annotation(com.example.annotations.AdminOnly)")
    public void adminOnlyOperation() {}

    // Pointcut לפונקציות שמחזירות מידע רגיש
    @Pointcut("execution(String *(User, ..)) && args(user, ..)")
    public void sensitiveDataOperation(User user) {}

    // 1. Advice לאבטחה (רץ לפני הפונקציה)
    @Before("adminOnlyOperation() && args(user, ..)")
    public void checkAdminSecurity(User user) throws IllegalAccessException {
        if (!user.getRole().equals("Admin")) {
            throw new IllegalAccessException("User " + user.getName() + " is not authorized.");
        }
    }

    // 2. Advice לתיעוד (רץ אחרי שהפונקציה הצליחה)
    @AfterReturning(pointcut = "sensitiveDataOperation(user)", returning = "returnedString")
    public void logSensitiveData(JoinPoint joinPoint, User user, String returnedString) {
        
        if (returnedString != null && returnedString.startsWith("Sensitive")) {
            String methodName = joinPoint.getSignature().getName();
            String logMessage = "User: " + user.getName() + ", Method: " + methodName + ", Data: " + returnedString;
            
            // קריאה לפונקציה הסטטית שסופקה בשאלה
            LogUtility.writeToLog(logMessage); 
        }
    }
}
```

**הסבר הנקודות החשובות:**

- **אבטחה:** השתמשנו ב-`@Before` כדי שהבדיקה תתבצע _לפני_ שהפונקציה המאובטחת רצה. אם המשתמש אינו מנהל, נזרקת חריגה והפונקציה כלל לא תתבצע.
    
- **תיעוד:** השתמשנו ב-`@AfterReturning` כדי לפעול רק _אחרי_ שהפונקציה הסתיימה בהצלחה. כך יש לנו גישה גם לערך המוחזר (`returnedString`) ונוכל לבדוק אם הוא מתחיל במילה "Sensitive".
    

---

## 🧠 צלילה עמוקה: Generics ו-Wildcards

כל הרעיון של Generics ב-Java נוצר כדי למנוע שגיאה מסוג `ClassCastException` – שגיאה שמתרחשת בזמן ריצה. כדי למנוע את זה, הקומפיילר חייב להיות "פסימי".

### שלב 1: תכנות גנרי (Generics)

(מצגת OOPDE-W05Lec09)

#### הבעיה ש-Generics באים לפתור

לפני Generics, רשימה (`List`) יכלה להחזיק כל סוג של `Object`.

**הקוד הישן והבעייתי (שקפים 13-14):**



```Java
List list = new ArrayList();
list.add(new Integer(1));
list.add("Hello"); // הקומפיילר לא מתלונן
// ...
Integer i = (Integer) list.get(1); // יקרוס בזמן ריצה!
```

**הבעיות כאן חמורות:**

1. **סכנת טעויות בזמן ריצה:** הקומפיילר לא היה מזהיר אותך.
    
2. **קוד מסורבל:** היית צריך לכתוב המרות (casting) בכל מקום.
    

#### הפתרון: Generics

Generics מאפשרים לנו להגדיר את טיפוס התוכן של אוסף בזמן ההצהרה.


```Java
List<Integer> list = new ArrayList<Integer>();
list.add(new Integer(1));
// list.add("Hello"); // שגיאת קומפילציה!
Integer i = list.get(0); // אין צורך בהמרה
```

**הרווח העיקרי:** בטיחות בזמן קומפילציה (**Type Safety**).

---

### Wildcards (?)

#### הבעיה: ג'נריקס וירושה לא תמיד עובדים כמו שמצפים

נניח היררכיה: `Dog` יורש מ-`Animal`. היית מצפה שהקוד הבא יעבוד:

Java

```
List<Dog> dogs = new ArrayList<Dog>();
List<Animal> animals = dogs; // <-- שגיאת קומפילציה!
```

**למה?** כי אם זה היה מותר, הקוד הבא היה אפשרי:

Java

```
List<Dog> dogs = new ArrayList<Dog>();
List<Animal> animals = dogs; // נניח שזה מותר
animals.add(new Cat()); // מוסיפים חתול לרשימה של כלבים
Dog d = dogs.get(1); // קורס!
```

כאן נכנסים ה-Wildcards.

#### עיקרון PECS: Producer Extends, Consumer Super

**1. Upper Bounded Wildcard: `? extends Type` ("יצרן")**

- **תחביר:** `List< ? extends Animal>`
    
- **משמעות:** "רשימה של סוג לא ידוע שיורש מ-`Animal`". (יכול להיות `List<Animal>`, `List<Dog>`, או `List<Cat>`).
    
- **הכלל (Producer Extends):** מרשימה כזו אתה יכול רק **לקרוא** (לייצר). הקומפיילר לא ייתן לך **להוסיף** אליהם, כי הוא לא יודע אם זו רשימת כלבים או חתולים.
    
- **שימוש:**
```Java
   `public void printAnimals(List< ? extends Animal> animals)`
```

**2. Lower Bounded Wildcard: `? super Type` ("צרכן")**

- **תחביר:*
```Java
`List< ? super Dog>`
```
- **משמעות:** "רשימה של סוג לא ידוע שהוא `Dog` או מחלקת אב של `Dog`". (יכול להיות `List<Dog>`, `List<Animal>`, או `List<Object>`).
    
- **הכלל (Consumer Super):** לרשימה כזו אתה יכול **להוסיף** (לצרוך) אובייקטים מסוג `Dog` (ויורשיו).
    
- **שימוש:**
```Java
public void addDog(List< ? super Dog> list)`
```
    

#### טבלת סיכום (עיקרון PECS)

|**סוג ה-Wildcard**|**מה זה אומר?**|**פעולת add (כתיבה)**|**פעולת get (קריאה)**|**כלל זיכרון (PECS)**|
|---|---|---|---|---|
|**`? extends Number`**|רשימה של יורשים (Producer)|**אסור** (לא בטוח)|**מותר** (תמיד יתקבל `Number`)|**P**roducer **E**xtends|
|**`? super Number`**|רשימה של אבות (Consumer)|**מותר** (להוסיף `Number` ויורשיו)|**מוגבל** (יתקבל רק `Object`)|**C**onsumer **S**uper|

---

## 🌊 צלילה עמוקה: Java Streams

`Streams` מאפשרים לנו לעבור לסגנון הצהרתי (**declarative**). אנחנו מתארים _מה_ אנחנו רוצים שיקרה, וה-API דואג ל_איך_.

**הדרך הישנה (לולאת `for`):**

```Java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> squaredEvens = new ArrayList<>();
for (Integer n : numbers) {
    if (n % 2 == 0) {
        squaredEvens.add(n * n);
    }
}
```

**הדרך החדשה (עם `Streams`):**

```Java
List<Integer> squaredEvens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());
```

### מבנה של פס ייצור (Pipeline)

1. **מקור (Source):** מאיפה מגיעים הנתונים? (`numbers.stream()`).
    
2. **פעולות ביניים (Intermediate Operations):** שרשרת של פעולות (כמו `filter`, `map`) שלא מופעלות עד השלב הסופי.
    
3. **פעולה סופית (Terminal Operation):** הפעולה שמפעילה את השרשרת (`collect`, `count`).
    

### מה זה ביטוי למדא? (OOPDE-W05Lec10)

ביטוי למדא הוא דרך קצרה לכתוב פונקציה אנונימית.

תחביר: (פרמטרים) -> { גוף הפונקציה }

ב-Java, למדא היא תמיד מימוש של **ממשק פונקציונלי** (ממשק עם מתודה מופשטת אחת):

- **`Predicate<T>` (פרדיקט):**
    
    - **תפקיד:** לבדוק תנאי ולהחזיר `true` או `false`.
        
    - **שימוש ב-Streams:** לסינון (`filter`).
        
    - **דוגמה:** `n -> n > 10`
        
- **`Function<T, R>` (פונקציה):**
    
    - **תפקיד:** להפוך אובייקט מסוג `T` לאובייקט מסוג `R`.
        
    - **שימוש ב-Streams:** להמרה (`map`).
        
    - **דוגמה:** `s -> s.length()`
        
- **`Consumer<T>` (צרכן):**
    
    - **תפקיד:** לבצע פעולה על אובייקט, מבלי להחזיר כלום.
        
    - **שימוש ב-Streams:** לביצוע פעולה על כל איבר (`forEach`).
        
    - **דוגמה:** `s -> System.out.println(s)`
        

---

## 📚 קישורים לשאלות ממבחנים

### [[Generics-Streams]]

- **תשפ"ג ב ב שאלה 1:** מציאת מדינה עם הכי הרבה אזרחים באמצעות Streams.
    
- **תשפ"ד א א שאלה 2:** חישוב שכיח בהתפלגות שכר באמצעות Streams.
    
- **תשפ"ג א א שאלה 3:** מחלקה גנרית `ArrayListMedian` המשתמשת ב-Streams.
    
- **תשפ"ד א ב שאלה 6:** שיפור חתימת הפונקציה `compose` באמצעות גנריות.
    
- **תשפ"ב ב א שאלה 1:** חישוב שטח באמצעות Streams.
    
- **תשפ"ד ב א שאלה 6:** Generics, הגדרה נכונה של תנאים `extends Auditable & Comparable<T>`.
    
- **תשפ"ב ב א שאלה 8:** חתימה נכונה לפונקציית `copy` גנרית: `<? super T>` ו-`<? extends T>`.
    

### [[DI]] (Dependency Injection)

- **תשפ"ד א א שאלה 1:** אתחול `Bicycle` עם גלגלים שונים באמצעות הזרקת `Constructor` ו-`@Named`.
    
- **תשפ"ד ב א שאלה 2:** אתחול `C1/C2` באמצעות `DI` ו-`@Named`/`@Alternative`.
    
- **תשפ"ד א ב שאלה 5:** שימוש ב-`DI`, `Producers` וב-`@Alternative` לאתחול תתי טיפוס שונים.
    
- **תשפ"ב א א שאלה 1:** הגדרת `Inversion of Control`.
    
- **תשפ"ג ב ב שאלה 3:** אתחול `Politician/Voter` עם מפלגות שונות באמצעות `@Produces` ו-`@Qualifier`.
    

### [[AOP]] (Aspect-Oriented Programming)

- **תשפ"ג ב ב שאלה 3:** שליחת הודעת `SMS` באמצעות `@After` ו-`@Pointcut` המשתמש ב-`execution`.
    
- **תשפ"ד ב א שאלה 2:** בקרת הרשאות ותיעוד פעולות רגישות באמצעות `AOP`.
    
- **תשפ"ד ב א שאלה 4:** כתיבת `Pointcut` לפונקציה גנרית עם אנוטציה.
    
- **תשפ"ג ב ב שאלה 10:** שימוש ב-`AOP` למדידת זמנים באמצעות `@annotation(Loggable)`.
    
- **תשפ"ד ב א שאלה 9:** איזה עקרון עיצובי בא לידי ביטוי בפתרון `Cross cutting concern` באמצעות `Aspect`.
    

---

[[מיקוד וסטטיסטיקות]]