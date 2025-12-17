
---
# תבנית עיצוב: Adapter (מתאם)

**Adapter** היא תבנית עיצוב **מבנית (Structural)**.

## מטרה ובעיה נפתרת

המטרה של תבנית Adapter היא **לתרגם** בין ממשק קיים של מחלקה (Adapt**ee**) לבין הממשק שהלקוח דורש (Target), מבלי לשנות את הקוד הקיים של המחלקות.

**הבעיה נפתרת:** יש לנו מחלקה קיימת שהיינו רוצים להשתמש בה, אך חתימת המתודות שלה אינה תואמת לזו שהקוד שלנו מצפה לה. המתאם עוטף את האובייקט הלא-מתאים ומספק ממשק תואם.

- השימוש ב-Adapter שומר על **עקרון פתוח/סגור (OCP)** ועל **עקרון היפוך התלויות (DIP)** בכך שהוא מאפשר לשלב קוד קיים במערכת חדשה ללא צורך לשנות את המקור.
- ה-Adapter יכול לשמש, למשל, ליצירת מתאם (`MS_TemperatureObserver`) בין ממשק גנרי (`Observer`) למחלקה קונקרטית (`MonitoringScreen`).

## סוגי מימוש ומבנה UML

קיימים שני סוגים עיקריים למימוש המתאם:
![](Summeries/עיצוב/media/media/image83.png)

|סוג המתאם|מנגנון המימוש|תיאור|
|:--|:--|:--|
|**Object Adapter**|**הרכבה (Composition)**|המתאם (`Adapter`) מחזיק בתוכו מופע של המחלקה הלא-תואמת (`Adaptee`) ומאציל אליו את הקריאות. זהו הסוג הנפוץ והגמיש יותר.|
|**Class Adapter**|**ירושה (Inheritance)**|המתאם יורש מהמחלקה הלא-תואמת (`Adaptee`) וגם מממש את הממשק הרצוי (`Target`). פחות נפוץ ב-Java.|

### תרשים מחלקות (Object Adapter)

בתרשים, הלקוח (`Client`) משתמש בממשק הנדרש (`Target`), והמתאם (`Adapter`) מממש ממשק זה תוך שהוא מחזיק רפרנס למחלקה הלא-תואמת (`Adaptee`).

## דוגמה 1: כפתור ונורה (Object Adapter)
 OOPDE-W03Lec05 שקף 16:
**התרחיש:** מחלקה `Button` רוצה לשלוט על `Lamp` דרך ממשק `IButtonServer` המצפה למתודות `turnOn()` ו-`turnOff()`. המחלקה `Lamp` הקיימת משתמשת במתודות `on()` ו-`off()`.

Button.java (לפני שימוש ב-Adapter)
```Java
public class Button {
    // !הכפתור מכיר ישירות את הנורה הקונקרטית
    private Lamp itsLamp;

    // ...קונסטרוקטור, setters, getters

    // המטודה הראשית שמבצעת את הלוגיקה
    void detect() {
        // (שלא מוצגת כאן) שבודקת את המצב הפיזי של הכפתור
        // קוראים לפונקציה
        boolean isButtonOn = getPhysicalState();

        // הלוגיקה קשורה ישירות ל-Lamp
        if (isButtonOn) {
            itsLamp.on(); // Lamp-קריאה ישירה למטודה הספציפית של
        } else {
            itsLamp.off(); // Lamp-קריאה ישירה למטודה הספציפית של
        }
    }

    // מתודה פיקטיבית לבדיקת מצב הכפתור
    private boolean getPhysicalState() {
        // ... קוד שבודק את החומרה ...
        return true; // לצורך הדוגמה
    }
}
```
### הבעיה: ממשקים לא תואמים

- ה-Button לא יכול לעבוד ישירות עם ה-Lamp כי שמות המתודות לא תואמים. אסור לנו לשנות את הקוד של Lamp בגלל OCP. 
    
- בגלל שButton משתמש במצביע של Lamp בעצמה, יש ביניהם קשר של directed Association. הקוד הזה סובל מהפרת DIP כי יש תלות ב low level - מכשיר חשמלי ספציפי.
    
### הפתרון: Adapter
אנחנו צריכים ליצור "מתאם" שישב באמצע, יממש את הממשק IButtonServer שה-Button מצפה לו, ומאחורי הקלעים יקרא למתודות הנכונות של ה-Lamp.

- מצד אחד, יש לנו מחלקה Button שרוצה לשלוט על איזשהו "שרת" דרך ממשק בשם IButtonServer. הממשק הזה דורש שתי מתודות: turnOn() ו-turnOff().
    
- מצד שני, יש לנו מחלקה קיימת, Lamp (נורה), שמספקת את הפונקציונליות הדרושה, אבל עם שמות מתודות שונים: on() ו-off().
    

יש שתי דרכים לממש את המתאם הזה:

#### 1. Object Adapter (באמצעות הרכבה - Composition)

זו הדרך הנפוצה והגמישה יותר.

IButtonServer הוא הממשק שהלקוח (המחלקה Button) מצפה לעבוד איתו. זהו ה"חוזה" שמגדיר את הפעולות שהכפתור רוצה להפעיל.

הקוד של הממשק מופיע במצגת OOPDE-W03Lec05 בשקף 12:
#### הממשק הנדרש (Target)

```Java
// file: IButtonServer.java
public interface IButtonServer {
    void turnOn();  // הפעולה שהכפתור מצפה לה כדי "להדליק"
    void turnOff(); // הפעולה שהכפתור מצפה לה כדי "לכבות"
}
```

_**ממשק זה נדרש על ידי מחלקת Button**_.

#### המתאם (Adapter)

המתאם `LampAdapter` מממש את `IButtonServer` ומחזיק מופע של `Lamp`.
LampAdapter.java (שקף 19)  
```Java
// File LampAdapter.java
public class LampAdapter implements IButtonServer{
    // המתאם מחזיק מופע של האובייקט הלא מתאים (Adaptee)
    private Lamp itsLamp;

    public LampAdapter(Lamp lamp){
        this.itsLamp = lamp;
    }

    @Override
    public void turnOn() {
        // המימוש של turnOn "מתרגם" לקריאה למתודה הנכונה של Lamp
        itsLamp.on();
    }

    @Override
    public void turnOff() {
        // המימוש של turnOff "מתרגם" לקריאה למתודה הנכונה של Lamp
        itsLamp.off();
    }
}
```

_**שימוש במתאם זה מאפשר למחלקת Button לשלוט על Lamp מבלי שתצטרך להכיר את שמות המתודות on/off**_.

איך משתמשים בו ב client?
```Java
Lamp lamp = new Lamp();
IButtonServer server = new LampAdapter(lamp); // יוצרים את המתאם
Button button = new Button(server); // מעבירים את המתאם ל-Button
button.detect(); עכשיו הכפתור יכול לשלוט בנורה דרך המתאם
```
#### 2. Class Adapter (באמצעות ירושה - Inheritance)

בגישה זו, המתאם יורש מהמחלקה שהוא צריך להתאים (Lamp) וגם מממש את הממשק הרצוי (IButtonServer). גישה זו פחות נפוצה ב-Java כי היא לא תומכת בירושה מרובה ממחלקות.

LampButtonServer.java (שקף 20)
```Java
// המתאם יורש מהלקוח הקיים וגם מממש את הממשק הרצוי
public class LampButtonServer extends Lamp implements IButtonServer {

    // המימוש של מתודות הממשק - קורא ישירות למתודות שירשנו
    @Override
    public void turnOn() {
        on(); // קורא למטודה on ש-Lamp שירשנו מ
    }

    @Override
    public void turnOff() {
        off(); // קורא למטודה off ש-Lamp שירשנו מ
    }
}
```
איך משתמשים בו?
```Java
LampButtonServer server = new LampButtonServer(); // יוצרים ישירות את המתאם (זה גם Lamp)
Button button = new Button(server);
button.detect();
```
שתי הדרכים משיגות את אותה מטרה - לגשר על הפער בין הממשקים. ההבדל הוא בטכניקת המימוש.

דגש: אם נרצה להוסיף מכשיר חשמלי, למשל Fan שbutton ידע להפעיל, ניצור מחלקה FanAdapter שתממש את IButtonServer ותעטוף את ה-Fan, והיא תיראה כך:
```Java
public class FanAdapter implements IButtonServer {
    private Fan itsFan;
    // ... constructor ...

    @Override
    public void turnOn() { itsFan.spin(); }
    @Override
    public void turnOff() { itsFan.stopSpinning(); }
}
```
זו ההוכחה שהתבנית Adapter שומרת על DIP, וזה מה שמאפשר לשמור על OCP.


## דוגמה 2: Bassoon וכלי נגינה (מימוש מודרני)

בגרסאות Java 8 ומעלה, ניתן לייתר את הצורך במחלקת מתאם שלמה (`BassoonAdapter`) באמצעות שימוש ב **ביטויי למדא** או **Method Reference**, וזאת בתנאי שהממשק הנדרש הוא **ממשק פונקציונלי** (מכיל מתודה מופשטת אחת בלבד).

OOPDE-W04Lab04 שקף 20:

תזכורת לבעיה מהמעבדה (שקפים 23-24):

- יש מערכת כלי נגינה (Instrument) עם מתודת playNote(int note).
    
- יש מחלקה חיצונית Bassoon עם מתודה makeSound(int note).
    
- רוצים לשלב את ה-Bassoon במערכת בלי לשנות קוד קיים.
    

הבעיה:

- יש לנו מערכת כלי נגינה - הממשק `Instrument`. כל כלי הנגינה מממשים אותו, ואת המתודה שלו `playNote(int note)`.
    
- יש לנו מחלקת `Player` (נגן) שיודעת לעבוד עם כל אובייקט שמממש `Instrument`.
    
- קנינו מחלקה חיצונית בשם `Bassoon`. היא עושה מה שאנחנו צריכים, אבל המתודה שלה נקראת `makeSound(int note)`.
    
- אסור לנו לשנות את הקוד של `Instrument`, `Player`, או `Bassoon`.
    

המטרה: לגרום ל-`Player` שלנו להיות מסוגל לנגן על ה-`Bassoon`.

יש לשלב מחלקה חיצונית `Bassoon` שמספקת `makeSound(int note)` לתוך מערכת `Instrument` הדורשת `playNote(int note)`.

הפתרון עם Object Adapter:

- יוצרים מתאם: נכתוב מחלקה חדשה בשם BassoonAdapter.
    
- המתאם מממש את הממשק הרצוי: BassoonAdapter יממש את הממשק Instrument (שהוא הממשק שה-Player מצפה לו).
    
- המתאם מחזיק את האובייקט הלא מתאים: BassoonAdapter יחזיק בפנים מופע של Bassoon.
    
- המתאם "מתרגם": כשיקראו למתודה playNote של ה-BassoonAdapter, הוא יקרא למתודה makeSound של ה-Bassoon שהוא מחזיק.
    

הקוד של המתאם ייראה כך (כמו בשקף 24 במעבדה):
```Java
// הממשק שהמערכת מצפה לו
public interface Instrument {
    public void playNote(int note);
}

// המחלקה הקיימת עם הממשק הלא תואם
public class Bassoon {
    public void makeSound(int note) {
        System.out.println("Bassoon making sound for note: " + note);
    }
}

// המתאם
public class BassoonAdapter implements Instrument {

    // מחזיק את האובייקט להתאמה
    private Bassoon bassoon;

    public BassoonAdapter(Bassoon bassoon) {
        this.bassoon = bassoon;
    }

    // מממש את הממשק הרצוי ומתרגם את הקריאה
    @Override
    public void playNote(int note) {
        bassoon.makeSound(note);
    }
}

// איך משתמשים בו?

Bassoon myBassoon = new Bassoon();
// יוצרים את המתאם שעוטף את הבסון
Instrument adaptedBassoon = new BassoonAdapter(myBassoon);
// מעבירים את המתאם (שמציג את עצמו כ-Instrument) לנגן
Player player = new JazzPlayer(adaptedBassoon);
// והמתאם יתרגם ל-makeSound-הנגן יפעיל את playNote, ו
player.playSomething();
```
הערה:

גם אם היינו כותבים BassoonAdapter adaptedBassoon = new BassoonAdapter(myBassoon);  
הכל היה עובד אותו הדבר (בגלל פולימורפיזם).

אז למה בכל זאת מעדיפים לכתוב Instrument adaptedBassoon = ...?  
זוהי מוסכמה חשובה שנקראת "תכנות כנגד הממשק" (Program to the interface).

- היא מדגישה שהקוד שלך מעכשיו והלאה לא צריך להכיר את המחלקה הספציפית BassoonAdapter. כל מה שמעניין אותו זה שהאובייקט הוא Instrument.
    
- זה הופך את הקוד לגמיש יותר. אם מחר תחליט להחליף את BassoonAdapter במתאם אחר (למשל, OboeAdapter), תצטרך לשנות רק את שורת היצירה (new ...), ושאר הקוד שמקבל Instrument לא ישתנה.

### פתרון באמצעות Lambda (Java 8)
OOPDE-W05Lec10 - Generic: שקף 14

מראה איך אפשר להשתמש בביטוי lambda כחלופה קומפקטית למימוש מלא של Adapter באותה דוגמת Bassoon.

- דוגמא 2 היתה מתאימה לגרסאות שלפני JAVA 8, אבל מאז נוספו כלי שמפשטים את הכתיבה.
```Java
// Bassoon.java היא המחלקה הלא תואמת (Adapt ee)
public class Bassoon {
    public void makeSound(int note) { /* ... */ }
}

// יצירת מימוש Instrument באמצעות למדא או Method Reference:
Bassoon bassoon = new Bassoon();

// 1. מימוש באמצעות למדא (מממש את playNote(n))
Instrument adaptedLambda = n -> bassoon.makeSound(n);

// 2. מימוש באמצעות Method Reference (קצר ותמציתי)
Instrument adaptedMethodRef = bassoon::makeSound;

// הלקוח (Player) משתמש במימוש המתורגם:
Player player = new Player(adaptedMethodRef);
```

_**שימוש זה נחשב קומפקטי וקריא יותר, והוא מהווה קיצור דרך אלגנטי למימוש Object Adapter**_.

**הפתרון הזה מתאים רק לממשקים פונקציונליים** (ממשקים עם מתודה אבסטרקטית אחת בלבד), כמו בדוגמת Basson.
דוגמאות מוכרות: Runnable (עם המתודה run()), Comparator (עם המתcompare()), וגם הממשק Instrument מהדוגמה שלנו (עם המתודה playNoודה te()).

- הקריאה ל-playNote לא נמצאת בשורת הלמדא עצמה, אבל הלמדא היא המימוש של playNote.
    
- השורה `Instrument adaptedInstrumentLambda = n -> bassoon.makeSound(n)`;  
    אומרת לג'אווה: "תיצור לי אובייקט חדש שמממש את הממשק Instrument. כשיקראו למתודה playNote שלו עם פרמטר כלשהו (נקרא לו n), פשוט תקרא למתודה makeSound של האובייקט bassoon ותעביר לה את אותו n."
    
- מתי הקריאה ל-playNote מתבצעת? היא מתבצעת מאוחר יותר, כשהאובייקט Player מקבל את adaptedInstrumentLambda ומשתמש בו:
    
```Java
Player player = new Player(adaptedInstrumentLambda);
player.play(); // play()->Player-> adaptedInstrumentLambda.playNote()
```

וכשהקריאה הזו תתבצע, תופעל הלוגיקה שהגדרנו בלמדא - כלומר, תתבצע קריאה ל-`bassoon.makeSound(someNote)`.

---
**קודם:** [[תבניות מבנה (Structural Patterns)]]
**הבא:** [[Decorator]]

**חזרה ל:** [[תבניות מבנה (Structural Patterns)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]