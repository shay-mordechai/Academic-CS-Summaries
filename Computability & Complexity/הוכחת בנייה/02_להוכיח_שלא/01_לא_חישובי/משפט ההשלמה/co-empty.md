
### שלב 2: הוכחת ש-$\mathbf{co-EMPTY}$ אינה חישובית (הכרעה)

כאן נכנס השימוש ב**משפט ההשלמה** וב**רדוקציה** להוכחת אי-במ"ריות של הקבוצה המשלימה.

כדי להוכיח ש-$\mathbf{\overline{EMPTY}}$ אינה חישובית, עלינו להוכיח ש**המשלים שלה, $\mathbf{EMPTY}$, אינו $\mathbf{במ"ר}$**.

#### הוכחת $\mathbf{EMPTY}$ אינה $\mathbf{במ"ר}$ (אי-קבלה)

#### סיכום באמצעות משפט ההשלמה

מאחר ש-$\mathbf{\overline{EMPTY}}$ היא $\mathbf{במ"ר}$ (שלב 1), אך המשלים שלה ($\mathbf{EMPTY}$) אינו $\mathbf{במ"ר}$ (שלב 2), נובע ממשפט ההשלמה ש-$\mathbf{\overline{EMPTY}}$ **אינה חישובית**.

### התייחסות לדרכים שציינת:

- **רדוקציה מ-$\mathbf{\overline{K}}$**: זו הדרך הנכונה להוכיח ש$\mathbf{EMPTY}$ (המשלים) אינה $\mathbf{במ"ר}$, ומשם להסיק ש$\mathbf{co-EMPTY}$ אינה חישובית.
- **רדוקציה מ-$\mathbf{K}$** ($\mathbf{K} \le_m \mathbf{co-EMPTY}$): ניתן להוכיח זאת גם כן. רדוקציה זו מוכיחה ש**$\mathbf{co-EMPTY}$ אינה חישובית** (כי $\mathbf{K}$ אינה חישובית), והיא משמשת כחלופה מהירה לחלק השני. אולם, היא **אינה מוכיחה** שהקבוצה היא $\mathbf{במ"ר}$ (זו עובדה נפרדת שדורשת את שלב 1).

**לסיכום, לבחינה, עדיף לבחור בדרך המלאה:**

1. **הוכחת במ"ריות** של $\mathbf{co-EMPTY}$ (באמצעות $\exists x \exists t \mathbf{STP}$).
2. **הוכחת אי-במ"ריות** של $\mathbf{EMPTY}$ (באמצעות רדוקציה $\mathbf{\overline{K}} \le_m \mathbf{EMPTY}$).
3. **שימוש במשפט ההשלמה** כדי להסיק ש$\mathbf{co-EMPTY}$ אינה חישובית.


#### שלב ב: הוכחה ש-$\mathbf{EMPTY}$ אינה $\mathbf{במ"ר}$ (אי-קבלה)

$\mathbf{EMPTY}$ היא המשלים של $\mathbf{co-EMPTY}$. כדי להראות שקבוצה $B$ אינה $\mathbf{במ"ר}$, מבצעים רדוקציה מקבוצה $A$ שידוע עליה שהיא אינה $\mathbf{במ"ר}$. הקבוצה הסטנדרטית לכך היא **$\mathbf{\overline{K}}$** (המשלים של $\mathbf{K}$), הידועה כאינה $\mathbf{במ"ר}$.

**הרדוקציה: $\mathbf{\overline{K}} \le_m \mathbf{EMPTY}$**

- **מטרה:** לבנות פונקציה חישובית $f(n)$ כך ש-$n \in \mathbf{\overline{K}}$ אם ורק אם $f(n) \in \mathbf{EMPTY}$.
- **הבנייה:** נגדיר תכנית $P_n$ (שמספרה $f(n)$), אשר מתעלמת מהקלט $X_1$ ומריצה את "הפילטר הסטנדרטי של $\mathbf{K}$": $$ Z \leftarrow \mathbf{\Phi}^{(1)}(n, n) $$
- **נכונות:**
    1. **אם $\mathbf{n \in \mathbf{\overline{K}}}$** ($\mathbf{\Phi}(n, n) \uparrow$): התכנית $P_n$ נכנסת ללולאה אינסופית ו**לא עוצרת על אף קלט** $X_1$. לכן $W_{f(n)} = \emptyset$, כלומר $\mathbf{f(n) \in \mathbf{EMPTY}}$.
    2. **אם $\mathbf{n \notin \mathbf{\overline{K}}}$** ($\mathbf{\Phi}(n, n) \downarrow$): התכנית $P_n$ עוצרת על **כל קלט** $X_1$. לכן $W_{f(n)} = \mathbb{N}$, כלומר $\mathbf{f(n) \notin \mathbf{EMPTY}}$.
- **מסקנה:** מאחר ש-$\mathbf{\overline{K}}$ אינה $\mathbf{במ"ר}$, וביצענו רדוקציה חישובית אליה, נובע ש**$\mathbf{EMPTY}$ אינה $\mathbf{במ"ר}$**.