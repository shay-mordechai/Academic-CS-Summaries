## Bridge Pattern (גשר)

Bridge היא תבנית עיצוב **מבנית** ויישום קלאסי של **DIP**.

### אתגר מס' 2: ניתוק הלוגיקה מהחומרה 🌉

הבעיה: המחלקה האפליקטיבית TemperatureSensor (שכבה גבוהה) תלויה ישירות במימוש החומרתי הקונקרטי Nimbus1_0Temperature (שכבה נמוכה).

השלכות הבעיה:

- הפרת DIP: מודול ברמה גבוהה תלוי בפרטי מימוש ברמה נמוכה.
    
- הפרת OCP: כדי להוסיף תמיכה בחומרה חדשה (Nimbus 2.0), נצטרך לשנות את הקוד הקיים של TemperatureSensor.
    
- הפרת ISP (זיהום ממשקים): למעטפת החומרה יש גישה לפרטי יישום של החיישן (כמו רשימת ה-Observers) שהיא לא צריכה.
    

הפתרון: תבנית Bridge (גשר)

תבנית זו נועדה לנתק את ההפשטה (Abstraction) מהמימוש (Implementor) שלה, כך שכל אחד מהם יוכל להתפתח באופן עצמאי.

### המבנה ב-UML
![](Summeries/עיצוב/media/media/image28.png)

המנגנון:

1. ההפשטה (Abstraction): מחלקת TemperatureSensor, המכילה את הלוגיקה העסקית.
    
2. המימוש (Implementor): ממשק חדש ונקי, TemperatureSensorImp, המגדיר רק את הפעולה הבסיסית הנדרשת מהחומרה, למשל int read().
    
3. הגשר: ה-TemperatureSensor מחזיק רפרנס לאובייקט מסוג TemperatureSensorImp. כשהוא צריך לקרוא נתון, הוא פשוט מאציל את הקריאה למימוש שהוא מחזיק: itsTemperatureImp.read().
    

התוצאה: הלוגיקה העסקית מנותקת לחלוטין מהחומרה. תוקנו הפרות ה-DIP, OCP ו-ISP. קל להחליף חומרה על ידי "הזרקת" מימוש אחר (ConcreteImplementor) ל-TemperatureSensor.

---

### שאלות ממבחנים בנושא Bridge

- DIP ו-WMS: תבנית Bridge היא יישום קלאסי של DIP. היא פותרת את התלות בין שכבת האפליקציה (הפשטה) לשכבת החומרה (מימוש) במערכת מזג האוויר. (תשפ"ה א' ב' סגור 2, תשפ"ה ב' ב' סגור 5).
    
- עיצוב השעון ב-WMS: ה-AlarmClock משתמש ב-Bridge כדי לנתק את שעון האפליקציה משעון החומרה (ClockImp). (תשפ"ב א' א' שאלה 3).
    
- שאלות כלליות: בחינת היכולת להחליף מימוש בזמן ריצה, או לפתור חשיפה של מידע פנימי. (תשפ"ג ב' א' שאלה 9, תשפ"ד א' א' שאלה 3ב').
    

---

### שאלות על תבנית Bridge

תבנית זו עוסקת בניתוק ההפשטה מהמימוש, ומאפשרת לשניהם להתפתח בנפרד.

שאלות ממבחנים

- DIP ו-WMS: המיקוד הבלעדי של Bridge הוא פתרון בעיית DIP. (תשפה א ב סגור 2, תשפה ב ב סגור 5)
    
- במקרה הבוחן של WMS, הוא פותר את התלות בין שכבת האפליקציה (ההפשטה) לבין שכבת החומרה (המימוש). שאלות אמריקאיות רבות בוחנות את המימוש הזה (לדוגמה, עיצוב השעון). (תשפב א א)
    
- תשפב א א שאלה 3 (עיצוב השעון במערכת מזג האוויר)
    
- תשפד א א שאלה 3ב (InfoWriter/Printer, פותר חשיפה פנימית).
    
- תשפג ב א שאלה 9 (HeartMonitor, החלפת מימוש getRate בזמן ריצה).
    
- תשפג ב ב שאלה 5 (Bridge פותר DIP).














### מטרה ויישום (ניתוק חומרה)

הבעיה שנפתרה: התלות של שכבת האפליקציה הגבוהה (`TemperatureSensor`) בפרטי המימוש הנמוכים של החומרה (לדוגמה, `Nimbus1_0Temperature` שמממש את `TemperatureSensor`).

- העיצוב המקורי הפר **DIP** (תלות בשכבה נמוכה), **OCP** (החלפת חומרה דורשת שינוי ב`TemperatureSensor`) ו-**ISP** (חשיפת פרטי יישום לא רלוונטיים לחומרה).
- **הפתרון:** הפרדה מוחלטת בין ההפשטה למימוש:
    - **ההפשטה:** `TemperatureSensor` (מכיל את הלוגיקה העסקית וה-Template Method).
    - **ממשק המימוש (Implementor):** ממשק חדש, **`TemperatureImp`** (עם מתודה נקייה כמו `read():int`).
    - **הגשר:** `TemperatureSensor` מחזיק רפרנס פנימי ל-`itsTemperatureImp` ומאציל אליו את הקריאה.
- **התוצאה:** מהנדסי חומרה נדרשים להכיר רק את ממשק `TemperatureImp`.

### קוד דוגמה (שימוש ב-Implementor)

בבנאי של `TemperatureSensor`:

```
public TemperatureSensor(AlarmClock ac, StationToolkit st){
    // ... ac.addListener(T_INTERVAL, this::check);
    // הגשר מחזיק את המימוש שנוצר ע"י ה-Factory
    this.itsTemperatureImp = st.makeTemperature();
}
// בתוך check()
int temp = itsTemperatureImp.read();
// ...
```

---
**קודם:** [[AlarmClock]]
**הבא:** [[WMS - חלק 3 (Abstract Factory)]]

**חזרה ל:** [[מערכת ניטור מזג אוויר (WMS)]]
**תוכן עניינים ראשי:** [[Summeries/עיצוב/תוכן עניינים]]