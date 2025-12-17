תחזית מזג אוויר - חלק 3:
### הבעיה:

עד עכשיו, תכננו את כל הרכיבים של המערכת, אבל עדיין לא טיפלנו בשאלה מי יוצר את כולם ואיך מחברים ביניהם. אם לא נעשה זאת נכון, ניצור תלויות חזקות שיפרו את כל העקרונות שעבדנו קשה לשמור עליהם.

הפתרון האלמנטרי והבעייתי (שקפים 9-14): הפתרון הראשון שהמצגת מציעה הוא ליצור מחלקה ראשית, WeatherMonitoringStation, שתהיה Singleton ובבנאי שלה היא תיצור את כל הרכיבים (new Nimbus1_0TemperatureSensor(), new AlarmClock(...) וכו').

הבעיה הייתה אתחול (Initialization). בפתרון הראשוני, מחלקת WeatherMonitoringStation יצרה ישירות מופעים קונקרטיים של רכיבי החומרה (למשל, new Nimbus1_0TemperatureSensor()).

למה זה רע?

- הפרת OCP ו-DIP: המחלקה WeatherMonitoringStation תלויה ישירות במחלקות הקונקרטיות של חומרת Nimbus 1.0. אם נרצה לעבור ל-Nimbus 2.0, נצטרך לפתוח את הקוד שלה ולשנות אותו.
    

---

### התיקון הראשון: תבנית Abstract Factory

שימוש בתבנית Abstract Factory באמצעות ממשק StationToolkit.

- ממשק ה-StationToolkit (ערכת כלים), מגדיר מתודות ליצירת משפחה של רכיבים תואמים (כמו makeTemperatureSensor(), getClock()).
    
- נוצרות מחלקות קונקרטיות כמו Nimbus1_0Toolkit (המפעל הקונקרטי) שמממשות את הממשק ומחזירות אך ורק אובייקטים התואמים לאותה גרסת חומרה.
    
- משנים את WeatherMonitoringStation, במקום ליצור אובייקטים ישירות, הוא יקבל בבנאי שלו StationToolkit וישתמש בו כדי ליצור את כל הרכיבים.
    

- היא נהיית תלויה רק ב-StationToolkit (ההפשטה), ולא ברכיבי החומרה הקונקרטיים. זה מאפשר גמישות מלאה והחלפה של כל משפחת החומרה (גרסה 1.0 ל-2.0) על ידי החלפת Factory קונקרטי אחד.
    
- יש לציין כי ב-WMS, האתחול של ה-Factory הקונקרטי עצמו יכול להתבצע באופן דינמי באמצעות Reflection (טעינת המחלקה לפי שם מחרוזת) כדי להימנע לחלוטין מתלות קשיחה בגרסאות החומרה.
    

המטרה: Abstract Factory משמשת במערכת מזג האוויר כדי לאמת שכל הרכיבים שנוצרים (חיישן, שעון, וכו') שייכים לאותה "משפחה" או גרסת חומרה (למשל, Nimbus 1.0).  
(מבחן תשפד ב א 3, תשפד א ב 3)

  

1. מגדירים "ערכת כלים" (Toolkit): יוצרים ממשק StationToolkit שמגדיר מתודות ליצירת כל רכיב חומרה (makeTemperatureSensor(), getClock() וכו').
    
2. יוצרים מימושים קונקרטיים:
    

- Nimbus1_0Toolkit תממש את הממשק ותחזיר את הרכיבים של Nimbus 1.0.
    
- בעתיד, Nimbus2_0Toolkit תחזיר את הרכיבים של Nimbus 2.0.
    

4. משנים את ה-WeatherMonitoringStation: במקום ליצור אובייקטים ישירות, הוא יקבל בבנאי שלו StationToolkit וישתמש בו כדי ליצור את כל הרכיבים.
    

התוצאה: ה-WeatherMonitoringStation כבר לא תלוי בחומרה הספציפית, אלא רק בממשק המופשט StationToolkit. שיפרנו את המצב, אבל עדיין יש בעיה קטנה...

---

### שימוש ב-Abstract Factory במערכת מזג האוויר (WMS)

ב-WMS, השתמשו ב-Abstract Factory (ממומשת באמצעות `StationToolkit`) כדי לאתחל את רכיבי החומרה (כגון חיישנים ושעון). זה מבטיח שלא יהיה ערבוב בין גרסאות חומרה (לדוגמה, Nimbus 1.0 ו-2.0).

האתחול יכול להתבצע על ידי קריאה למתודת Factory סטטית שיוצרת את המופע הקונקרטי:

```Java
public class AbstractFactory {
    …
    private static AbstractFactory theInstance;
    public static AbstractFactory theFactory(int factoryId){
        if (theInstance == null) {
            if (factoryId == 0) theInstance = new AcmeFactory();
            else theInstance = new HackmeFactory();
        }
        return theInstance;
    }
}
```

**פתרון מתקדם (Singleton עם ירושה):** במקום שהלקוח (כמו `Dishwasher`) יהיה תלוי ב-`factoryId`, ניתן לממש את `AbstractFactory` כ-Singleton עם ירושה, שבה המחלקה המופשטת (Animal/AbstractFactory) מכילה שדה סטטי משותף, והקריאה הראשונה של מחלקה קונקרטית (Dog/AcmeFactory) קובעת את הטיפוס היחיד לכל המערכת.

```Java
// AbstractFactory (מופשט)
public abstract class AbstractFactory {
    // ...
    protected static AbstractFactory theInstance;
    public static AbstractFactory theFactory() {
        assert theInstance != null : "Cannot instantiate an abstract Factory";
        return theInstance;
    }
}
```

במקרה זה, יש רק מקום אחד בקוד (בדרך כלל ב-`main`) שאחראי לאתחל את המופע הקונקרטי.

### הבעיה שנותרה והתיקון השני: Reflection

עדיין, במקום כלשהו בקוד (בדרך כלל ב-main), מישהו צריך להחליט באיזה Toolkit להשתמש וליצור אותו (new Nimbus1_0Toolkit()). כדי למנוע גם את התלות הזו ולהפוך את המערכת לגמישה לחלוטין, המצגת מציגה שימוש ב-Reflection.

הרעיון: במקום לקרוא ל-new, אנחנו מקבלים את שם המחלקה כמחרוזת (למשל, מקובץ הגדרות) ומשתמשים ב-Reflection של Java כדי ליצור מופע שלה באופן דינמי.

```java
Class tkClass = Class.forName("com.cloud.nimbus1.Nimbus1_0Toolkit");
StationToolkit st = (StationToolkit)tkClass.newInstance();
```
התוצאה: עכשיו אפשר לשנות את גרסת החומרה של כל המערכת על ידי שינוי של מחרוזת אחת בקובץ הגדרות, מבלי לקמפל את הקוד מחדש. זוהי רמה גבוהה מאוד של גמישות.

  

שאלות ממבחנים:

בבחינה, Abstract Factory נשאל בהקשר ה-WMS הן כשאלה רב-ברירה (למשל, זיהוי התבנית ששימשה לאתחול תחנת מזג האוויר) (תשפד ב א, תשפג ב ב 10),

 והן כחלק משאלה פתוחה הדורשת את אתחול רכיבי המערכת. (תשפג א א)














## 5. Abstract Factory (בית חרושת מופשט)

Abstract Factory היא תבנית עיצוב **יצירה**.

### מטרה ויישום (אתחול המערכת)

התבנית שימשה לפתרון בעיות **OCP ו-DIP** במחלקת האתחול הראשית, `WeatherMonitoringStation`, שהייתה תלויה בעצמה במחלקות חומרה קונקרטיות (כמו `Nimbus1_0TemperatureSensor`).

- **Goal:** לספק ממשק (`StationToolkit`) ליצירת **"משפחות" של אובייקטים קשורים** (שעון, חיישנים) בלי לציין את הטיפוס הקונקרטי. זה מבטיח שכל הרכיבים שייכים לאותה גרסת חומרה (למשל, Nimbus 1.0).
- **הממשק:** **`StationToolkit`**.
- **המפעל הקונקרטי:** `Nimbus1_0Toolkit` (מומש כסינגלטון), מממש את `StationToolkit` ומחזיר את כל הרכיבים של גרסה 1.0.

### אתחול באמצעות Factory ו-Reflection

כדי להבטיח OCP מלא, נמנעים מלקרוא ל-`new Nimbus1_0Toolkit()` אפילו ב-`main`. במקום זאת, משתמשים ב**Reflection** כדי לטעון את המחלקה הקונקרטית של ה-Toolkit לפי שם (מחרוזת) בזמן ריצה. זה מאפשר לשנות את כל גרסת החומרה על ידי שינוי מחרוזת אחת, ללא קמפול מחדש.

---
**קודם:** [[Bridge]]
**הבא:** [[Proxy]]

**חזרה ל:** [[מערכת ניטור מזג אוויר (WMS)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]