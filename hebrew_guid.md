# מדריך שימוש ל-ADVfile_manager (גרסה 1.3.0)

**מחבר:** Avi Twil  
**מאגר:** [https://github.com/avitwil/ADVfile_manager](https://github.com/avitwil/ADVfile_manager)

מבנים אבסטרקטיים אחידים לקבצים ב-Python עם **דפוסי קלט/פלט בטוחים**, **מטמון בזיכרון**, **גיבויים עם חותמות זמן**, **בטיחות מנהל הקשרים (גיבוי אוטומטי ושחזור בשגיאה)**, **ניקוי גיבויים זמניים בסיום המפרש**, ו**חתימת search() אחידה אחת** — על פני קבצי **Text, JSON, CSV, YAML, INI, TOML, XML, ו-Excel**. כולל **ממשקים אסינכרוניים** (`ATextFile`, `AJsonFile`, וכו') לזרימות עבודה לא חוסמות.

---

## למה ADVfile_manager?

* **ממשק API אחד, פורמטים רבים** – קריאה/כתיבה/הוספה/חיפוש באותה דרך עבור TXT/JSON/CSV/YAML/INI/TOML/XML/Excel.
* **זרימות עבודה בטוחות יותר** – מנהל הקשרים מבצע גיבוי בכניסה ומשחזר בשגיאות.
* **גיבויים בשליטתך** – `backup()`, `list_backups()`, `restore()`, `clear_backups()`.
* **עריכות זמניות** – סמן מופע קובץ כ-`keep_backup=False` והגיבויים שלו יימחקו אוטומטית בסיום המפרש.
* **חיפוש אחיד** – חתימת מתודה אחת (`search(pattern=..., key=..., value=..., columns=..., tag=..., attr=..., sheet=...)`) מותאמת לכל פורמט.
* **מטמון בזיכרון** – `read()` מהיר משתמש בתוכן ממטמון; `clear_cache()` לקריאה טרייה מהדיסק.
* **מוכן לאסינכרוני** – `aread/awrite/aappend/asearch` ו-`async with` על מחלקות `A*`.
* **בטיחות מעשית** – משתמש בקובץ זמני + `os.replace()` היכן שמתאים (למשל TOML/XML/Excel) להפחתת סיכון לשחיתות.

> הערה: כתיבות אטומיות מלאות אינן אפשריות בכל פורמט ובכל סביבה. ADVfile_manager משתמש בדפוסים הבטוחים ביותר האפשריים לכל פורמט.

---

## תוכן עניינים

1. [למה ADVfile_manager?](#why-advfile_manager)
2. [התקנה](#installation)
3. [השוואת תכונות](#feature-comparison)
   - [ADVfile_manager מול חלופות נפוצות](#comparison-vs-others)
   - [ADVfile_manager מול ATmulti_file_handler](#comparison-vs-atmulti_file_handler)
4. [התחלה מהירה](#quick-start)
5. [מחלקת בסיס: `File` (חלה על כל הפורמטים)](#base-file)
   - [קונסטרקטור ומאפיינים](#file-constructor)
   - [מנהל הקשרים וגיבויים זמניים](#file-context)
   - [גיבויים / שחזור / ניקוי / רשימה](#file-backups)
   - [מטמון ועזרי גודל קובץ](#file-cache-and-size)
   - [חתימת `search()` אחידה (התנהגות ספציפית לפורמט)](#file-search-contract)
   - [בקרות ניקוי בסיום (ברמת מודול)](#file-exit-cleanup)
6. [TextFile (TXT) — מדריך מלא](#textfile)
   - [קונסטרקטור](#textfile)
   - [קלט/פלט בסיסי ועזרי שורות](#textfile)
   - [חיפוש](#textfile)
   - [גיבויים ובטיחות בהקשרים (TextFile)](#textfile)
7. [JsonFile (JSON) — מדריך מלא](#jsonfile-json--complete-guide)
   - [קונסטרקטור](#jsonfile-json--complete-guide)
   - [קלט/פלט בסיסי, גישות (`get_item`, `items`)](#jsonfile-json--complete-guide)
   - [חיפוש](#jsonfile-json--complete-guide)
8. [CsvFile (CSV) — מדריך מלא](#csvfile-csv--complete-guide)
   - [קונסטרקטור](#csvfile-csv--complete-guide)
   - [write / read / append / read_row / rows](#csvfile-csv--complete-guide)
   - [חיפוש](#csvfile-csv--complete-guide)
9. [YamlFile (YAML) — מדריך מלא](#yamlfile-yaml--complete-guide)
   - [קונסטרקטור](#yamlfile-yaml--complete-guide)
   - [קלט/פלט בסיסי, גישות (`get_item`, `items`)](#yamlfile-yaml--complete-guide)
   - [חיפוש](#yamlfile-yaml--complete-guide)
10. [IniFile (INI) — מדריך מלא](#inifile-ini--complete-guide)
    - [קונסטרקטור](#inifile-ini--complete-guide)
    - [write / read / append](#inifile-ini--complete-guide)
    - [חיפוש](#inifile-ini--complete-guide)
11. [TomlFile (TOML) — מדריך מלא](#tomlfile-toml--complete-guide)
    - [קונסטרקטור](#tomlfile-toml--complete-guide)
    - [write / read / append](#tomlfile-toml--complete-guide)
    - [חיפוש](#tomlfile-toml--complete-guide)
12. [XmlFile (XML) — מדריך מלא](#xmlfile-xml--complete-guide)
    - [קונסטרקטור](#xmlfile-xml--complete-guide)
    - [write / read / append](#xmlfile-xml--complete-guide)
    - [חיפוש](#xmlfile-xml--complete-guide)
13. [ExcelFile (XLSX דרך openpyxl) — מדריך מלא](#excelfile-xlsx-via-openpyxl--complete-guide)
    - [קונסטרקטור](#excelfile-xlsx-via-openpyxl--complete-guide)
    - [write / read / append](#excelfile-xlsx-via-openpyxl--complete-guide)
    - [חיפוש](#excelfile-xlsx-via-openpyxl--complete-guide)
14. [מחלקות A* אסינכרוניות — מדריך מלא](#async-a-classes--complete-guide)
    - [ATextFile](#async-a-classes--complete-guide)
    - [AJsonFile](#async-a-classes--complete-guide)
    - [ACsvFile](#async-a-classes--complete-guide)
    - [AYamlFile](#async-a-classes--complete-guide)
    - [AIniFile](#async-a-classes--complete-guide)
    - [ATomlFile](#async-a-classes--complete-guide)
    - [AXmlFile](#async-a-classes--complete-guide)
    - [AExcelFile](#async-a-classes--complete-guide)
15. [גיבויים, שחזור ובטיחות בהקשרים — ניתוח מעמיק](#backups-restore--context-safety--deep-dive)
16. [כלי עזר גלובליים](#global-utilities)
17. [גליון עזר ל-`search(...)` אחיד](#unified-search-cheatsheet)
18. [התקנה (v1.3.0)](#installation-v130)
19. [טבלאות השוואה](#comparison-tables)
    - [ADVfile_manager מול ספריות פופולריות](#advfile_manager-vs-popular-libraries)
    - [ADVfile_manager מול ATmulti_file_handler שלך](#advfile_manager-vs-your-atmulti_file_handler-hypothetical-prior-tool)
20. [פרויקט לדוגמה — מקצה לקצה](#sample-project--end-to-end)
21. [מפת דרכים (גרסאות הבאות)](#roadmap-next-iterations)

---

## התקנה

```bash
pip install ADVfile_manager
```

**תלות אופציונלית (התקן רק אם אתה זקוק לפורמט):**

* YAML: `pip install pyyaml`
* TOML (קריאה): Python 3.11+ (מובנה `tomllib`) — או `pip install tomli`
* TOML (כתיבה): `pip install tomli-w`
* Excel (XLSX): `pip install openpyxl`

Python **3.8+** מומלץ.

---

## השוואת תכונות

### <a id="comparison-vs-others"></a>ADVfile_manager מול חלופות נפוצות

| תכונה / כלי                               | **ADVfile_manager**                  | `pathlib` (ספרייה סטנדרטית) | `os`/`shutil` (ספרייה סטנדרטית) | `json`/`csv`/`configparser`/`xml.etree` (ספרייה סטנדרטית) | `pandas`                       | `PyYAML`/`ruamel.yaml` | `openpyxl`       |
| ------------------------------------------ | ------------------------------------ | ----------------------------- | --------------------------------- | ----------------------------------------------------------- | ------------------------------ | ---------------------- | ---------------- |
| פורמטים נתמכים                           | TXT/JSON/CSV/YAML/INI/TOML/XML/Excel | נתיבים בלבד                  | פעולות מערכת קבצים              | ניתוח לפי פורמט (מפוצל)                                   | CSV/JSON/Excel/… כ-DataFrames  | YAML בלבד              | Excel בלבד       |
| ממשק API אחיד (קריאה/כתיבה/הוספה/חיפוש) | **כן**                              | לא                            | לא                                | לא (כל מודול שונה)                                        | חלקי (ממוקד DF)               | לא                     | לא               |
| בטיחות בהקשרים (גיבוי + שחזור אוטומטי)   | **כן**                              | לא                            | לא                                | לא                                                          | לא                             | לא                     | לא               |
| כלי גיבויים                               | **כן**                              | לא                            | לא                                | לא                                                          | לא                             | לא                     | לא               |
| גיבויים זמניים (ניקוי atexit)            | **כן**                              | לא                            | לא                                | לא                                                          | לא                             | לא                     | לא               |
| מטמון בזיכרון + `clear_cache()`          | **כן**                              | לא                            | לא                                | לא                                                          | מודל מטמון DF                 | לא                     | לא               |
| חתימת חיפוש אחידה                        | **כן**                              | לא                            | לא                                | לא                                                          | פעולות DataFrame              | לא                     | לא               |
| ממשק אסינכרוני                           | **כן**                              | לא                            | לא                                | לא                                                          | לא (דורש ספריות נוספות)      | לא                     | לא               |
| כתיבות בטוחות נוספות (זמני + החלפה)     | TOML/XML/Excel                       | לא רלוונטי                   | לא רלוונטי                       | בדרך כלל לא                                                | לא                             | לא רלוונטי            | תלוי בשימוש      |
| עקומת למידה                               | **נמוכה**                           | נמוכה                        | נמוכה                            | **גבוהה** (לא עקבית)                                      | בינונית/גבוהה                | נמוכה                 | נמוכה           |

### <a id="comparison-vs-atmulti_file_handler"></a>ADVfile_manager מול **ATmulti_file_handler** (הכלי האחר שלי)

| תחום                                  | **ADVfile_manager 1.3.0**                             | **ATmulti_file_handler**       |
| -------------------------------------- | ------------------------------------------------------ | -------------------------------- |
| חיפוש אחיד על פני פורמטים            | **כן** (`search()` חתימה עובדת בכל מקום)             | חלקי/חסר                       |
| אסינכרוני (`A*` מחלקות)              | **כן** (`aread/awrite/aappend/asearch`, `async with`) | חלקי/חסר                       |
| פורמטים                               | TXT/JSON/CSV/**YAML/INI/TOML/XML/Excel**               | סביר שתת-קבוצה (TXT/JSON/CSV/…)* |
| בטיחות בהקשרים (גיבוי + שחזור אוטומטי) | **כן**                                                | חלקי/חסר                       |
| גיבויים זמניים (ניקוי atexit)        | **כן**                                                | חסר                            |
| כתיבות בטוחות (זמני+החלפה)           | TOML/XML/Excel                                         | משתנה                          |
| תיעוד והתמקדות בהוראה                | **גבוהה מאוד (קובץ README זה)**                      | משתנה                          |

* אם הכלי הקודם שלך כבר תומך בחלק מהתכונות האלה, השדרוג העיקרי כאן הוא **ממשק החיפוש האחיד, הממשק האסינכרוני, וכיסוי הפורמטים הרחב יותר (YAML/INI/TOML/XML/Excel) עם ארגונומיה ובטיחות עקבית.**

---

## התחלה מהירה

```python
from ADVfile_manager import TextFile, JsonFile

base = "example_data"

# טקסט
txt = TextFile("notes.txt", base)
txt.write("שורה ראשונה")
txt.append("שורה שנייה")
print(txt.read())           # "שורה ראשונה\nשורה שנייה"
print(txt.read_line(2))     # "שורה שנייה"
for i, line in txt.lines():
    print(i, line)

# עריכה בטוחה עם גיבוי/שחזור אוטומטי
with TextFile("notes.txt", base) as safe:
    safe.append("בתוך ההקשר")
    # raise Exception("בום")  # הסר הערה כדי לראות שחזור אוטומטי בפעולה

# JSON
j = JsonFile("config.json", base)
j.write({"users": [{"id": 1}]})
j.append({"active": True})
print(j.read())             # {'users': [{'id': 1}], 'active': True}

# חיפוש אחיד (דוגמאות בהמשך המסמך)
hits = list(txt.search(pattern="שנייה"))
print(hits[0]["value"])     # טקסט השורה התואמת
```

---

## <a id="base-file"></a>מחלקת בסיס: `File` (חלה על כל הפורמטים)

כל מחלקה קונקרטית (`TextFile`, `JsonFile`, ...) יורשת מ-`File`, ולכן התכונות כאן **אוניברסליות**.

### <a id="file-constructor"></a>קונסטרקטור ומאפיינים

```python
from ADVfile_manager import TextFile  # כל מחלקת בת יש לה התנהגות בסיס זהה

# file_path נוצר אם חסר; status משקף קיום הקובץ
f = TextFile("example.txt", file_path="example_data")  # keep_backup=True כברירת מחדל

print(f.name)       # "example.txt"
print(f.path)       # "example_data"
print(f.full_path)  # "example_data/example.txt"
print(f.status)     # False בתחילה (עד שתכתוב)
print(f.content)    # None (מטמון ריק)
```

**הערות וחריגות**

* אם התיקייה `file_path` לא קיימת, היא נוצרת אוטומטית.
* `status` משקף את קיום הקובץ בדיסק ב-`full_path`.
* `content` הוא מטמון פנימי. הוא `None` עד ש-`read()` או `write()` מוצלחים ממלאים אותו.

---

### <a id="file-context"></a>מנהל הקשרים וגיבויים זמניים

**למה:** להגנה מפני כתיבות חלקיות או שגיאות בזמן ריצה.
**איך זה עובד:**

* ב-`__enter__`: אם `keep_backup=True`, נלקח גיבוי אוטומטי (אם הקובץ קיים).
* ב-`__exit__`: אם התרחשה שגיאה ו-`keep_backup=True`, הקובץ משוחזר מהגיבוי האחרון.
* המטמון תמיד מנוקה ביציאה.
* אם `keep_backup=False`, הגיבויים של הקובץ מנוקים ביציאה (וגם בסיום המפרש).

```python
from ADVfile_manager import TextFile

# ברירת מחדל: keep_backup=True
with TextFile("plan.txt", "example_data") as t:
    t.write("v1")
    # אם מתרחשת שגיאה כעת, מתבצע restore() עבורך.

# סימון מופע קיים כזמני רק עבור ההקשר הזה:
ephemeral = TextFile("temp.txt", "example_data")
with ephemeral(keep_backup=False) as t:
    t.write("ערך זמני")
# ביציאה: הגיבויים של קובץ זה מוסרים
```

**חריגות**

* אם הקובץ לא קיים ב-`__enter__`, לא נוצר גיבוי (ללא שגיאה).
* אם ניסיון השחזור נכשל כי אין גיבויים (נדיר), הוא פשוט מדלג.

---

### <a id="file-backups"></a>גיבויים / שחזור / ניקוי / רשימה

**למה:** נקודות חזרה בטוחות ושחזור ידני.

```python
from ADVfile_manager import TextFile

t = TextFile("story.txt", "example_data")
t.write("v1"); b1 = t.backup()
t.write("v2"); b2 = t.backup()

print(t.list_backups())   # [ "...story.txt.2025..._..._...bak", "...v2....bak" ]

t.restore()               # שחזור האחרון (v2)
t.restore(b1)             # שחזור גיבוי ספציפי (v1)
removed = t.clear_backups()
print("גיבויים מוסרים:", removed)
```

**חריגות**

* `backup()` מעלה `FileNotFoundError` אם הקובץ לא קיים עדיין.
* `restore()` מעלה `FileNotFoundError` אם אין גיבויים (רק כאשר קוראים לו ידנית ללא גיבויים).

---

### <a id="file-cache-and-size"></a>מטמון ועזרי גודל קובץ

```python
f = TextFile("data.txt", "example_data")
f.write("hello")
print(f.read())            # ממטמון
print(f.get_size())        # בייטים
print(f.get_size_human())  # למשל "6.0 B"
f.clear_cache()
# read() הבא יקרא מחדש מהדיסק:
print(f.read())
```

---

### <a id="file-search-contract"></a>חתימת `search()` אחידה (התנהגות ספציפית לפורמט)

כל מחלקת בת מיישמת `search()` עם **חתימה זהה**:

```python
search(
    pattern: str | None = None,
    *,
    regex: bool = False,
    case: bool = False,
    key: str | None = None,            # פורמטים דמויי מיפוי (JSON/YAML/INI/TOML)
    value: Any = None,                 # התאמה מדויקת לערך
    columns: Sequence[str] | None = None,  # סינון עמודות CSV/Excel
    tag: str | None = None,            # סינון תגיות XML
    attr: dict[str,str] | None = None, # סינון תכונות XML
    sheet: str | None = None,          # סינון גיליונות Excel
    limit: int | None = None,
) -> Iterator[dict]
```

**ערך מוחזר:** **איטרטור של מילוני התאמות**. המפתחות משתנים לפי פורמט אבל עוקבים אחרי בסיס זה:

* `path`: מחרוזת (`/full/path.ext` עם מידע מיקום לשורה/שורה/עמודה כשמתאים)
* `value`: הערך התואם (מחרוזת או טבעי)
* `line` / `row` / `col` / `sheet`: רמזי מיקום כשמתאים
* `context`: מחרוזת קצרה או אובייקט להבנת ההתאמה (**למשל** שורה מלאה לטקסט, מילון שורה ל-CSV/Excel, אלמנט ל-XML).
* חלק מהפורמטים מוסיפים מפתחות כמו `key` (מפתח מילון JSON/YAML/TOML) או `section` (INI).

תראה דוגמאות מעשיות תחת כל סוג קובץ.

---

### <a id="file-exit-cleanup"></a>בקרות ניקוי בסיום (ברמת מודול)

**למה:** כאשר פותחים קבצים במצב זמני (`keep_backup=False`), הגיבויים שלהם נרשמים למחיקה אוטומטית כאשר המפרש מסיים. אתה יכול לשלוט בהתנהגות זו גלובלית.

```python
from ADVfile_manager import set_exit_cleanup, cleanup_backups_for_all, TextFile

# הפעלה/ביטול ניקוי atexit גלובלי (מופעל כברירת מחדל)
set_exit_cleanup(True)
set_exit_cleanup(False)

# ניקוי ידני (מוחק גיבויים לכל הקבצים הזמניים הרשומים)
removed_total = cleanup_backups_for_all()
print(removed_total)
```

---

## <a id="textfile"></a>TextFile (TXT) — מדריך מלא

`TextFile` מיועד לטקסט UTF-8 פשוט. הוא תומך ב-**CRUD מלא** וב-**עזרי שורה**.

### קונסטרקטור

```python
from ADVfile_manager import TextFile

txt = TextFile("log.txt", "example_data")
```

* יוצר `example_data/` אם לא קיים.
* `status` מציין אם `log.txt` קיים כבר.
* `keep_backup=True` כברירת מחדל (מנהל הקשרים יבצע גיבוי אוטומטי).

---

### סקירת מתודות

* **קלט/פלט בסיסי**

  * `write(data: str) -> None`
  * `read() -> str`
  * `append(data: str) -> None`
* **עזרי שורה**

  * `lines() -> Generator[tuple[int, str], None, None]`
  * `read_line(line_number: int) -> str`
* **חיפוש**

  * `search(pattern, regex=False, case=False, limit=None) -> Iterator[dict]`
* **ירושה מ-`File`**

  * `backup()`, `list_backups()`, `restore(backup_path=None)`, `clear_backups()`
  * `clear_cache()`
  * `get_size()`, `get_size_human()`
  * מנהל הקשרים ובקרת זמניות (`with ...`, `obj(keep_backup=False)`)

להלן, כל מתודה כוללת **למה / איך / דוגמה / חריגות**.

---

### `write(data: str) -> None`

**למה:** החלפת כל תוכן הקובץ בבת אחת.

**איך:** פותח במצב טקסט, כותב `data`, מעדכן מטמון בזיכרון ו-`status=True`.

```python
txt = TextFile("a.txt", "example_data")
txt.write("Hello\nWorld")
print(txt.status)   # True
```

**חריגות**

* אם התיקייה לא ניתנת לכתיבה, מערכת ההפעלה תעלה (למשל `PermissionError`).

---

### `read() -> str`

**למה:** קבלת כל תוכן הקובץ (ממטמון לאחר קריאה ראשונה).

**איך:** קורא מהדיסק פעם אחת, שומר תוצאה ב-`txt.content`.

```python
txt = TextFile("a.txt", "example_data")
txt.write("Hello\nWorld")
print(txt.read())       # "Hello\nWorld"
print(txt.read())       # ממטמון (ללא קלט/פלט דיסק)
txt.clear_cache()
print(txt.read())       # קריאה מחודשת מהדיסק
```

**חריגות**

* אם הקובץ לא קיים, `open(..., "rt")` מעלה `FileNotFoundError`.
  צור אותו עם `write()` תחילה או הגן עם `txt.status`.

---

### `append(data: str) -> None`

**למה:** הוספת תוכן לסוף; שומר על סמנטיקת שורות.

**איך:** אם לקובץ כבר יש בייטים (`size > 0`), מוכנסת שורה חדשה אוטומטית.

```python
txt = TextFile("log.txt", "example_data")
txt.write("first")
txt.append("second")
txt.append("third")
print(txt.read())
# first
# second
# third
```

**חריגות**

* זהות ל-`write()` (שגיאות מערכת קבצים).

---

### `lines() -> Generator[(line_number, line_text)]`

**למה:** זרימת קבצים גדולים מבלי לטעון את כל התוכן.

**איך:** מחזיר `(1, "first line")`, `(2, "second line")`, … (ללא `\n` בסוף).

```python
for num, line in TextFile("a.txt", "example_data").lines():
    print(f"{num}: {line}")
```

**חריגות**

* `FileNotFoundError` אם הקובץ לא קיים בזמן פתיחה.

---

### `read_line(line_number: int) -> str`

**למה:** גישה אקראית לשורה ספציפית (מבוסס 1).

**איך:** זורם עד השורה המבוקשת, מחזיר את הטקסט ללא שורה חדשה בסוף.

```python
txt = TextFile("a.txt", "example_data")
txt.write("first\nsecond\nthird")
print(txt.read_line(2))       # "second"
```

**חריגות**

* `IndexError` אם השורה המבוקשת לא קיימת.
* `FileNotFoundError` אם הקובץ חסר.

---

### `search(pattern, *, regex=False, case=False, limit=None)`

**למה:** מציאת שורות המכילות מחרוזת משנה או תואמות regex במהירות.

**איך:** עובר על השורות ומחזיר מילוני התאמות:

```python
txt = TextFile("grep.txt", "example_data")
txt.write("Alpha\nbeta\nALPHA BETA\nGamma")

# מחרוזת משנה לא רגישה לאותיות (ברירת מחדל)
hits = list(txt.search(pattern="alpha"))
print([h["line"] for h in hits])    # [1, 3]

# רגישה לאותיות
hits = list(txt.search(pattern="Alpha", case=True))
print([h["line"] for h in hits])    # [1]

# Regex
hits = list(txt.search(pattern=r"^A.*BETA$", regex=True))
print([h["line"] for h in hits])    # [3]

# הגבלת תוצאות
hits = list(txt.search(pattern="a", limit=2))
print(len(hits))                    # 2
```

**סכמת התאמה (TextFile)**

```python
{
  "path":   "/abs/path/grep.txt:line[3]",
  "value":  "ALPHA BETA",  # השורה התואמת
  "line":   3,
  "row":    None,
  "col":    None,
  "sheet":  None,
  "context":"ALPHA BETA"
}
```

**חריגות**

* אין (מחזיר איטרטור ריק אם `pattern` הוא `None`).

---

### גיבויים ובטיחות בהקשרים עם TextFile (מעשי)

```python
t = TextFile("edit.txt", "example_data")
t.write("v1")
b1 = t.backup()
t.write("v2")
b2 = t.backup()

print(t.list_backups())  # שני גיבויים כעת
t.restore(b1)            # חזרה ל-"v1"
print(t.read())          # "v1"
removed = t.clear_backups()

# הדגמת בטיחות בהקשרים
try:
    with TextFile("edit.txt", "example_data") as safe:
        safe.write("draft")
        raise RuntimeError("oops")   # מפעיל שחזור של הגרסה הקודמת
except RuntimeError:
    pass
print(t.read())  # חזרה ל-"v1"
```

---

### שילוב הכל: כלי יומן טקסט (הדגמה מיני)

```python
from ADVfile_manager import TextFile

base = "example_data"
log  = TextFile("app.log", base)

# התחלת יומן (עריכה טרנזקציונלית בטוחה)
with log:
    log.write("[start] app booting")
    log.append("user=avi")
    # ... אם מתרחשת קריסה, התוכן הקודם (אם קיים) משוחזר

# חיפוש ביומנים
for hit in log.search(pattern="user"):
    print(f"line {hit['line']}: {hit['value']}")

# מחזור חיים של גיבויים
b = log.backup()
print("backup:", b)
print("backups:", log.list_backups())
log.restore()          # האחרון
log.clear_backups()
```

# JsonFile (JSON) — מדריך מלא

`JsonFile` מטפל בקבצי JSON עם שורש **מילון** או **רשימה**.
תומך בסמנטיקת הוספה לשני הסוגים, גישות לפריטים, איטרציה, ו-`search()` אחיד.

---

### קונסטרקטור

```python
from ADVfile_manager import JsonFile

j = JsonFile("data.json", "example_data")
```

* יוצר `example_data/data.json` אם חסר כאשר כותבים.
* טוען אוטומטית לזיכרון בקריאה ראשונה או אם כבר קיים.

---

### סקירת מתודות

* **קלט/פלט בסיסי**

  * `write(data: Any)`
  * `read() -> Any`
  * `append(data: Any)`
* **גישות**

  * `get_item(index_or_key)`
  * `items() -> Iterable`
* **חיפוש**

  * `search(pattern=..., key=..., value=..., regex=False, case=False, limit=None)`

מתודות ירושה: גיבויים, מטמון, עזרי גודל, הקשרים, וכו'.

---

### `write(data: dict | list)`

**למה:** החלפה עם מילון או רשימה של Python כ-JSON.
**איך:** שומר JSON עם הזחה (ברירת מחדל indent=2).

```python
j = JsonFile("d.json", "example_data")
j.write({"users": [{"id": 1}]})
print(j.read())   # {'users': [{'id': 1}]}

jl = JsonFile("l.json", "example_data")
jl.write([{"id": 1}, {"id": 2}])
```

**חריגות**

* אם `data` אינו ניתן לסריאליזציה ל-JSON, `TypeError` מ-`json.dump`.

---

### `read() -> dict | list`

**למה:** קבלת תוכן הקובץ למבני Python.
**איך:** משתמש ב-`json.load`, שומר תוצאה ב-`self.content`.

```python
print(j.read())   # מילון
print(jl.read())  # רשימה
```

**חריגות**

* מעלה `FileNotFoundError` אם הקובץ חסר.
* מעלה `json.JSONDecodeError` אם הקובץ הוא JSON לא תקין.

---

### `append(data: Any)`

**למה:** הוספת תוכן חדש מבלי לכתוב מחדש את כל הקובץ.
**איך:**

* שורש מילון: דורש מילון → מיזוג רדוד (`dict.update`).
* שורש רשימה: מוסיף אלמנט, או מרחיב עם איטרביל.

```python
j = JsonFile("d.json", "example_data")
j.write({"users": [{"id": 1}]})
j.append({"active": True})
print(j.read())  # {'users':[{'id':1}], 'active':True}

jl = JsonFile("l.json", "example_data")
jl.write([{"id": 1}])
jl.append({"id": 2})
jl.append([{"id": 3}, {"id": 4}])
print(jl.read()) # [{'id':1}, {'id':2}, {'id':3}, {'id':4}]
```

**חריגות**

* אם השורש הוא מילון וההוספה אינה מילון → `TypeError`.
* אם השורש אינו רשימה או מילון → `TypeError`.

---

### `get_item(index_or_key)`

**למה:** גישה אקראית נוחה.
**איך:**

* אם השורש הוא רשימה → מצפה למדד שלם מבוסס 1.
* אם השורש הוא מילון → מצפה למפתח מחרוזת.

```python
jl = JsonFile("l.json", "example_data")
jl.write([{"id": 10}, {"id": 20}])
print(jl.get_item(2))  # {'id': 20}

j = JsonFile("d.json", "example_data")
j.write({"x": 42})
print(j.get_item("x"))  # 42
```

**חריגות**

* `TypeError` אם סוג שגוי.
* `IndexError` אם מדד מחוץ לטווח.
* `KeyError` אם מפתח מילון לא קיים.

---

### `items()`

**למה:** איטרציה ישירה על תוכן השורש.
**איך:**

* שורש מילון → מחזיר `(מפתח, ערך)`.
* שורש רשימה → מחזיר `(מדד מבוסס 1, פריט)`.

```python
j = JsonFile("d.json", "example_data")
j.write({"a": 1, "b": 2})
for k, v in j.items():
    print(k, v)   # 'a' 1, 'b' 2

jl = JsonFile("l.json", "example_data")
jl.write([{"id": 1}, {"id": 2}])
for i, item in jl.items():
    print(i, item)  # 1 {'id':1}, 2 {'id':2}
```

---

### `search(pattern=..., key=..., value=..., regex=False, case=False, limit=None)`

**למה:** בדיקת מבני JSON מקוננים עם API אחיד.
**איך:** עובר רקורסיבית על מילונים/רשימות, מתאים מפתחות, ערכים, ודפוסי מחרוזות.

```python
j = JsonFile("u.json", "example_data")
j.write({"users":[{"id":1,"name":"Avi"},{"id":2,"name":"Dana"}], "active":True})

# מצא לפי שם מפתח
hits = list(j.search(pattern="name"))
print([h["value"] for h in hits])  # ["Avi","Dana"]

# מצא לפי מפתח מדויק
hits = list(j.search(key="active"))
print(hits[0]["value"])            # True

# מצא עם regex בערכים
hits = list(j.search(pattern="^A", regex=True))
print([h["value"] for h in hits])  # ["Avi"]
```

**סכמת התאמה (JSON)**

```python
{"path": "/abs/path/u.json", "key": "name", "value": "Avi"}
```

**חריגות**

* אין (מפתח/ערך לא תקינים פשוט לא מחזירים תוצאות).

---

# CsvFile (CSV) — מדריך מלא

`CsvFile` מנהל CSV עם כותרות, באמצעות `csv.DictReader`/`DictWriter`.

---

### קונסטרקטור

```python
from ADVfile_manager import CsvFile

c = CsvFile("table.csv", "example_data")
```

---

### סקירת מתודות

* `write(rows, fieldnames=None)`
* `read() -> list[dict[str,str]]`
* `append(row_or_rows)`
* `read_row(n)`
* `rows() -> generator`
* `search(pattern=..., columns=..., value=...)`

---

### `write(rows, fieldnames=None)`

**למה:** החלפת CSV.
**איך:** כותב כותרות תחילה, ואז שורות.

```python
c = CsvFile("t.csv", "example_data")
c.write([{"name":"Avi","age":30},{"name":"Dana","age":25}])
print(c.read())
```

**חריגות**

* `ValueError` אם שורות ריקות ואין fieldnames מסופקים.

---

### `read() -> list[dict]`

**למה:** טעינת CSV לרשימה של מילונים.
**איך:** מפתחות = כותרות, ערכים = מחרוזות.

```python
rows = c.read()
print(rows[0])  # {"name":"Avi","age":"30"}
```

---

### `append(row_or_rows)`

**למה:** הוספת שורות חדשות מבלי להחליף.
**איך:** מקבל מילון או רשימה של מילונים. מסיק כותרות אם צריך.

```python
c.append({"name":"Noa","age":21})
c.append([{"name":"Lior","age":28},{"name":"Omri","age":33}])
print(c.read())
```

**חריגות**

* מעלה `TypeError` אם הארגומנט אינו מילון או רשימה של מילונים.

---

### `read_row(n)`

**למה:** גישה לשורה ספציפית (מבוסס 1, ללא כותרת).
**איך:** עובר עד השורה.

```python
print(c.read_row(2))   # {"name":"Dana","age":"25"}
```

**חריגות**

* `IndexError` אם השורה לא קיימת.

---

### `rows()`

**למה:** זרימת שורות עצלה.
**איך:** מחזיר `(מספר_שורה, מילון_שורה)`.

```python
for i, row in c.rows():
    print(i, row)
```

---

### `search(pattern=..., columns=None, value=None, regex=False, case=False, limit=None)`

**למה:** מציאת תאים לפי מחרוזת משנה או התאמה מדויקת.
**איך:** עובר על שורות/עמודות.

```python
hits = list(c.search(pattern="Avi"))
print(hits[0]["value"])   # "Avi"

hits = list(c.search(value="21"))
print(hits[0]["row"], hits[0]["col"])   # 3 "age"

hits = list(c.search(pattern="^D", regex=True, columns=["name"]))
print([h["value"] for h in hits])  # ["Dana"]
```

**סכמת התאמה (CSV)**

```python
{
  "path": "/abs/path/table.csv:row[2].name",
  "value": "Dana",
  "row": 2,
  "col": "name",
  "context": "{'name':'Dana','age':'25'}"
}
```

**חריגות**

* אין (פשוט מחזיר ללא תוצאות אם אין התאמות).

---

# YamlFile (YAML) — מדריך מלא

`YamlFile` מנהל תצורות YAML (דורש `PyYAML`).
מתנהג כמו `JsonFile`, עם שורשי מילון/רשימה, סמנטיקת הוספה, וחיפוש אחיד.

---

### קונסטרקטור

```python
from ADVfile_manager import YamlFile

y = YamlFile("config.yaml", "example_data")
```

* מעלה `ImportError` אם `pyyaml` לא מותקן.

---

### סקירת מתודות

* `write(data)`
* `read() -> Any`
* `append(data)`
* `get_item(index_or_key)`
* `items()`
* `search(pattern=..., key=..., value=...)`

---

### `write(data)`

**למה:** שמירת מילון/רשימה של Python כ-YAML.
**איך:** משתמש ב-`yaml.safe_dump`.

```python
y.write({"app":{"name":"demo"}, "features":["a"]})
print(y.read())
```

**חריגות**

* `ImportError` אם PyYAML חסר.
* `yaml.YAMLError` אם השמירה נכשלת.

---

### `read() -> Any`

**למה:** טעינת YAML למבני Python.
**איך:** משתמש ב-`yaml.safe_load`.

```python
print(y.read())   # {'app': {'name': 'demo'}, 'features': ['a']}
```

**חריגות**

* `yaml.YAMLError` אם YAML לא תקין.

---

### `append(data)`

**למה:** עדכון תצורה בקלות.
**איך:**

* שורש מילון → מיזוג רדוד.
* שורש רשימה → הוספה או הרחבה.

```python
y.append({"features":["b"]})
print(y.read())  # {'app': {'name':'demo'}, 'features':['a','b']}
```

**חריגות**

* אם השורש מילון וההוספה לא מילון → `TypeError`.
* אם השורש רשימה וההוספה מסוג שגוי → `TypeError`.

---

### `get_item(index_or_key)`

**למה:** גישה אקראית.
**איך:**

* שורש רשימה → מדד מבוסס 1.
* שורש מילון → מפתח.

```python
print(y.get_item("app"))  # {'name': 'demo'}
```

**חריגות**

* `TypeError` אם סוג הגישה שגוי.
* `KeyError` / `IndexError` אם חסר.

---

### `items()`

**למה:** איטרציה על שורש YAML.
**איך:**

* מילון → `(מפתח, ערך)`
* רשימה → `(מדד, ערך)`

```python
for k, v in y.items():
    print(k, v)
```

---

### `search(...)`

**למה:** בדיקת YAML רקורסיבית (כמו JSON).
**איך:** משתמש ב-`_walk_jsonlike`.

```python
hits = list(y.search(pattern="demo"))
print(hits[0]["value"])  # 'demo'
```

---

# IniFile (INI) — מדריך מלא

`IniFile` עוטף את `configparser` של Python.
קטעים ומפתחות אינם רגישים לאותיות (מנורמלים לאותיות קטנות פנימית).

---

### קונסטרקטור

```python
from ADVfile_manager import IniFile

ini = IniFile("settings.ini", "example_data")
```

---

### סקירת מתודות

* `write(data)`
* `read() -> dict`
* `append(data)`
* `search(pattern=..., key=..., value=...)`

---

### `write(data)`

**למה:** שמירת תצורת INI ממילון.
**איך:** מילון → ConfigParser → קובץ.

```python
ini.write({"server": {"host": "127.0.0.1", "port": "8000"}})
```

**חריגות**

* אם `data` לא מילון-של-מילונים → `TypeError`.

---

### `read() -> dict`

**למה:** טעינת INI למילון Python עם מפתחות באותיות קטנות.
**איך:** קורא דרך ConfigParser, מנרמל.

```python
cfg = ini.read()
print(cfg["server"]["host"])  # "127.0.0.1"
```

---

### `append(data)`

**למה:** מיזוג קטעים/מפתחות נוספים.
**איך:** מיזוג רדוד, כותב חזרה.

```python
ini.append({"server": {"debug": "true"}, "auth": {"enabled": "yes"}})
print(ini.read()["auth"]["enabled"])  # "yes"
```

**חריגות**

* אם `data` לא מילון-של-מילונים → `TypeError`.

---

### `search(...)`

**למה:** מציאת מפתחות/ערכים לפי מחרוזת משנה או regex.
**איך:** עובר על המילון.

```python
hits = list(ini.search(pattern="127"))
print(hits[0]["value"])  # "127.0.0.1"

hits = list(ini.search(key="port"))
print(hits[0]["value"])  # "8000"
```

---

# TomlFile (TOML) — מדריך מלא

`TomlFile` מנהל תצורות TOML (דורש Python 3.11+ `tomllib`, או `tomli/tomli-w`).

---

### קונסטרקטור

```python
from ADVfile_manager import TomlFile

t = TomlFile("cfg.toml", "example_data")
```

---

### סקירת מתודות

* `write(data)`
* `read() -> dict`
* `append(data)`
* `search(pattern=..., key=..., value=...)`

---

### `write(data)`

**למה:** שמירת מילון כ-TOML.
**איך:** דורש `tomli-w`.

```python
t.write({"app":{"name":"demo"}, "flags":{"x":True}})
```

**חריגות**

* `ImportError` אם אין כותב מותקן.

---

### `read() -> dict`

**למה:** טעינת TOML למילון Python.
**איך:** משתמש ב-`tomllib`/`tomli`.

```python
cfg = t.read()
print(cfg["app"]["name"])  # "demo"
```

**חריגות**

* `ImportError` אם אין קורא TOML.

---

### `append(data)`

**למה:** הרחבת תצורה בבטחה.
**איך:** מיזוג עמוק (מילונים ממוזגים רקורסיבית).

```python
t.append({"flags":{"y":False}})
print(t.read()["flags"])  # {"x":True,"y":False}
```

**חריגות**

* `TypeError` אם השורש/נתונים לא מילון.

---

### `search(...)`

**למה:** חקירת TOML לעומק.
**איך:** משתמש ב-`_walk_jsonlike`.

```python
hits = list(t.search(pattern="demo"))
print(hits[0]["value"])  # "demo"
```

---

# XmlFile (XML) — מדריך מלא

`XmlFile` עוטף `xml.etree.ElementTree` לקריאה/כתיבה/העשרת עצי XML וחיפושים מובנים לפי **תגית**, **תכונות**, ו**טקסט**.

> **מתי להשתמש:** תצורות, יצוא נתונים, ומסמכים מכונתיים תואמים שבהם המבנה ההיררכי חשוב.

---

## קונסטרקטור

```python
from ADVfile_manager import XmlFile
import xml.etree.ElementTree as ET

x = XmlFile("data.xml", "example_data")
```

* אם `data.xml` כבר קיים, השורש שלו נטען באופן עצל בקריאה ראשונה (או אוטומטית לאחר `__init__` אם כתבת תוכן בריצה קודמת).

---

## סקירת מתודות

* `write(data: Element | str) -> None`
* `read() -> xml.etree.ElementTree.Element`
* `append(data: Element | Sequence[Element]) -> None`
* `search(pattern=None, *, regex=False, case=False, tag=None, attr=None, value=None, limit=None) -> Iterator[dict]`

---

## `write(data)`

**למה:** שמירת מסמך XML לדיסק.

**איך:**

* מקבל `Element` מלא (שורש) או מחרוזת XML גולמית.
* כותב עם UTF-8 והצהרת XML.
* משתמש בקובץ זמני ו-`os.replace()` לאטומיות.

```python
root = ET.Element("books")
root.append(ET.Element("book", attrib={"id": "1"}))
root.append(ET.Element("book", attrib={"id": "2"}))

x = XmlFile("books.xml", "example_data")
x.write(root)       # או: x.write('<books><book id="1"/></books>')
```

**חריגות**

* `ET.ParseError` אם ניתנת מחרוזת XML לא תקינה.
* `OSError`/`IOError` על שגיאות דיסק.

---

## `read() -> Element`

**למה:** טעינת עץ XML לגישה תכנותית.

**איך:**

* מנתח את הקובץ ומחזיר את **שורש ה-`Element`**.
* שומר את שורש העץ במטמון למופע זה.

```python
root = x.read()
print(root.tag)    # "books"
for el in root.iter("book"):
    print(el.get("id"))
```

**חריגות**

* `ET.ParseError` אם תוכן הקובץ XML לא תקין.
* `FileNotFoundError` אם הקובץ חסר.

---

## `append(data)`

**למה:** הוספת אלמנטים תחת השורש במקום.

**איך:**

* מקבל `Element` יחיד או רצף של `Element`ים.
* מוסיף לשורש הנוכחי וכותב חזרה.

```python
# הוספת אחד
x.append(ET.Element("book", attrib={"id": "3"}))

# הוספת רבים
more = [ET.Element("book", attrib={"id": "4"}),
        ET.Element("book", attrib={"id": "5"})]
x.append(more)
```

**חריגות**

* `TypeError` אם `data` אינו `Element` או רצף של `Element`ים.
* `ET.ParseError` אם הקובץ לא ניתן לקריאה; תקן על ידי כתיבת שורש תקין מחדש.

---

## `search(...)`

**למה:** שאילתת XML לפי **תגית**, **תכונות**, **טקסט**, או שילוב קריטריונים.

**איך (כללי התאמה):**

* `tag="book"` → רק אלמנטים עם תגית זו.
* `attr={"id":"2"}` → האלמנט חייב את כל התכונות הרשומות עם ערכים מדויקים.
* `value="Some text"` → טקסט האלמנט (מנוקה) חייב להתאים לערך זה.
* `pattern="sale"` + `regex`/`case` → התאמת מחרוזת משנה או regex על טקסט האלמנט.
* `limit=N` → הפסק לאחר N התאמות.

**מילון התאמה מוחזר:**
`{"path": file_path, "value": element_text, "context": element}`

```python
# 1) מצא לפי תגית:
hits = list(x.search(tag="book"))
print(len(hits))

# 2) לפי תכונה:
hits = list(x.search(tag="book", attr={"id": "3"}))
print(hits[0]["context"].tag, hits[0]["value"])  # 'book', טקסט האלמנט ('' אם None)

# 3) לפי ערך טקסט מדויק:
chapter = XmlFile("chapter.xml", "example_data")
chapter.write('<chapter><title>Intro</title><title>Advanced</title></chapter>')
hits = list(chapter.search(tag="title", value="Intro"))
print(hits[0]["value"])  # "Intro"

# 4) לפי דפוס (לא רגיש לאותיות):
hits = list(chapter.search(tag="title", pattern="adv", case=False))
print([h["value"] for h in hits])  # ["Advanced"]

# 5) משולב + הגבלה:
hits = list(x.search(tag="book", attr={"category":"Fiction"}, limit=2))
for h in hits:
    el = h["context"]
    print(el.tag, el.attrib)
```

**חריגות**

* אין ספציפיות מ-`search()`; ודא שהקובץ נקרא/מנותח בהצלחה תחילה.

---

# ExcelFile (XLSX דרך openpyxl) — מדריך מלא

`ExcelFile` מספק אינטראקציה פשוטה ובטוחה עם קבצי `.xlsx` דרך `openpyxl`:
**קריאה/כתיבה/הוספה** עם שורות מילוניות (שורה ראשונה היא כותרת), תמיכה בגיליונות מרובים, וחיפוש תאים אחיד.

> **מתי להשתמש:** יצוא/יבוא Excel קל משקל; רישום הדרגתי; מיזוג גיליונות ללא כלי נתונים כבדים.

---

## תלות

* דורש `openpyxl`:

```bash
pip install openpyxl
```

---

## קונסטרקטור

```python
from ADVfile_manager import ExcelFile

xls = ExcelFile("report.xlsx", "example_data", default_sheet="Sheet1")
```

* `default_sheet` משמש כאשר לא מציינים `sheet=` במתודות.
* אם הספר או הגיליון חסרים, מבנים מתאימים נוצרים לפי הצורך.

---

## סקירת מתודות

* `read(*, sheet=None) -> List[Dict[str, Any]]`
* `write(rows, *, sheet=None) -> None`
* `append(row_or_rows, *, sheet=None) -> None`
* `search(pattern=None, *, regex=False, case=False, columns=None, value=None, sheet=None, limit=None) -> Iterator[dict]`

**צורת שורה:** כל שורה היא `dict`, מפתחות הם כותרות עמודות, ערכים הם ערכי תאים.
**חוק כותרות:** שורה ראשונה של הגיליון היא הכותרת; נוצרת אוטומטית בעת כתיבה לגיליון ריק.

---

## `write(rows, *, sheet=None)`

**למה:** יצירה/החלפה של תוכן בגיליון עם **סדר כותרות מוגדר**.

**איך:**

* מקבל רשימה/איטרביל של שורות מילוניות.
* סדר כותרות מסוק מההופעה הראשונה של מפתחות בשורות.
* מנקה/יוצר מחדש את הגיליון היעד, כותב כותרות ושורות.
* משתמש בזמני+החלפה לאטומיות.

```python
rows = [
    {"name": "Avi",  "score": 100},
    {"name": "Dana", "score":  90},
]
xls.write(rows, sheet="S1")

# יצירת גיליון שני:
xls.write([{"name": "Noa", "score": 95}], sheet="S2")
```

**חריגות**

* `ImportError` אם `openpyxl` לא מותקן.
* `TypeError` אם `rows` לא דמויי-מילון.
* `OSError` על שגיאות דיסק.

---

## `read(*, sheet=None) -> List[Dict[str, Any]]`

**למה:** טעינת גיליון לרשימה של שורות מילוניות.

**איך:**

* קורא כותרות משורה 1.
* יוצר מילון לכל שורה (שורות 2..N).
* ממיר תאים חסרים ל-`""` כדי לשמור על סט עמודות יציב.

```python
data = xls.read(sheet="S1")
print(data)
# [{'name': 'Avi', 'score': 100}, {'name': 'Dana', 'score': 90}]
```

**חריגות**

* `FileNotFoundError` אם הספר לא נמצא.
* `KeyError` רק אם פתרון שם הגיליון נכשל פנימית (אנו יוצרים גיליונות בכתיבה/הוספה, אבל קריאה מצפה לגיליון קיים; ודא שהוא קיים).

---

## `append(row_or_rows, *, sheet=None)`

**למה:** הוספת שורות לגיליון קיים (או חדש) תוך שמירה על עקביות כותרות.

**איך:**

* מקבל מילון או איטרביל של מילונים.
* אם הגיליון ריק, כותרות מסוקות מהשורות המוספות ונכתבות תחילה.
* אם כותרות קיימות, מפתחות חסרים בשורה מסוימת נכתבים כ-`""`.

```python
# הוספת שורה אחת ל-S1:
xls.append({"name": "Lior", "score": 88}, sheet="S1")

# הוספת שורות מרובות ל-S2 (עמודה חדשה מופיעה → ריק היכן חסר):
xls.append([{"name": "Omri", "score": 84},
            {"name": "Noa"}],            # 'score' חסר → "" בשורה זו
          sheet="S2")

print(xls.read(sheet="S2"))
# [{'name': 'Noa', 'score': 95}, {'name': 'Omri', 'score': 84}, {'name': 'Noa', 'score': ''}]
```

**חריגות**

* `TypeError` אם השורה אינה מילון או רשימה של מילונים.
* `ImportError` אם `openpyxl` חסר.

---

### `search(pattern=None, *, regex=False, case=False, columns=None, value=None, sheet=None, limit=None)`

**למה:** מציאת תאים לפי **מחרוזת משנה/regex** או **ערך מדויק** על פני עמודות וגיליונות.

**איך:**

* קורא פנימית `read(sheet=...)` → רשימת שורות מילוניות.
* `columns=["colA","colB"]` להגבלת היקף החיפוש.
* `value=...` להתאמת ערך מדויק (ממור לטקסט), `pattern="..."` למחרוזת משנה/regex.
* מחזיר מילוני התאמות:
  `{"path", "value", "row", "col", "sheet", "context"}`

```python
# חיפוש דפוס (לא רגיש לאותיות כברירת מחדל):
hits = list(xls.search(pattern="noa", sheet="S2"))
print([(h["row"], h["col"], h["value"]) for h in hits])

# חיפוש ערך מדויק:
hits = list(xls.search(value=90, columns=["score"], sheet="S1"))
print(hits[0]["value"])  # "90" או 90 תלוי בסוג התא → אנו ממירים לטקסט להשוואה

# Regex (רגיש לאותיות):
hits = list(xls.search(pattern=r"^A..$", regex=True, case=True, columns=["name"], sheet="S1"))
for h in hits:
    print(h["row"], h["col"], h["value"])

# הגבלת תוצאות:
hits = list(xls.search(pattern="o", sheet="S2", limit=2))
print(len(hits))  # 2
```

**חריגות**

* אין ספציפיות מעבר למה ש-`read()` עשוי להעלות אם קובץ/גיליון לא תקינים.

---

## דוגמה מקצה לקצה (Excel + גיבויים + חיפוש)

```python
from ADVfile_manager import ExcelFile

x = ExcelFile("grades.xlsx", "example_data", default_sheet="ClassA")

# כתיבת גיליון טרי:
x.write([{"name":"Avi", "score": 100},
         {"name":"Dana","score":  90}])

# גיבוי לפני שינויים מסוכנים:
bpath = x.backup()

# הוספת שורות:
x.append({"name":"Noa","score":95})
x.append([{"name":"Lior","score":88},{"name":"Omri"}])  # 'score' חסר -> ""

# חיפוש:
print("מצא 90:", list(x.search(value=90, columns=["score"])))
print("שמות עם 'o':", [h["value"] for h in x.search(pattern="o", columns=["name"])])

# שחזור אם צריך:
# x.restore(bpath)
```

# מחלקות A* אסינכרוניות — מדריך מלא

הממשק האסינכרוני משקף את ה-API הסינכרוני באמצעות `asyncio.to_thread(...)`. אתה מקבל את אותה סמנטיקה, אבל לא חוסמת. כל מחלקה אסינכרונית תואמת 1:1 למקבילה הסינכרונית:

* `ATextFile` ↔ `TextFile`
* `AJsonFile` ↔ `JsonFile`
* `ACsvFile` ↔ `CsvFile`
* `AYamlFile` ↔ `YamlFile`
* `AIniFile` ↔ `IniFile`
* `ATomlFile` ↔ `TomlFile`
* `AXmlFile` ↔ `XmlFile`
* `AExcelFile` ↔ `ExcelFile`

כל אחת מספקת:

* `await aread(...)`
* `await awrite(...)`
* `await aappend(...)`
* `await asearch(...)` → מחזיר **רשימה** של התאמות (לא איטרטור)
* `async with ...` מנהל הקשרים (גיבוי אוטומטי בכניסה, שחזור בשגיאה, ניקוי מטמון ביציאה — זהה לסינכרוני)

> ⚠️ **הערות / חריגות**
> • חייבים לרוץ תחת לולאת אירועים (למשל `pytest-asyncio`, `asyncio.run(...)`).
> • חריגות משקפות את הגרסאות הסינכרוניות (למשל `FileNotFoundError`, `ET.ParseError`, `ImportError`).

---

## ATextFile

### קונסטרקטור

```python
from ADVfile_manager import ATextFile

t = ATextFile("notes.txt", "example_data")
```

### מתודות ודוגמאות

#### `await awrite(text: str) -> None`

```python
await t.awrite("hello async")
```

#### `await aread() -> str`

```python
content = await t.aread()
print(content)
```

#### `await aappend(text: str) -> None`

```python
await t.aappend("another line")
```

#### `await asearch(pattern: str, *, regex=False, case=False, limit=None) -> list[dict]`

```python
hits = await t.asearch(pattern="hello")
print(len(hits), hits[0]["line"], hits[0]["value"])
```

#### `async with ATextFile(...)`

```python
try:
    async with ATextFile("draft.txt", "example_data") as f:
        await f.awrite("temp edit")
        raise RuntimeError("boom")
except RuntimeError:
    pass
# קובץ משוחזר אוטומטית מגיבוי; מטמון מנוקה
```

---

## AJsonFile

### קונסטרקטור

```python
from ADVfile_manager import AJsonFile

j = AJsonFile("data.json", "example_data")
```

### מתודות ודוגמאות

#### `await awrite(obj: Any) -> None`

```python
await j.awrite({"users": [{"id": 1}]})
```

#### `await aread() -> Any`

```python
obj = await j.aread()
print(obj["users"][0]["id"])
```

#### `await aappend(data: Any) -> None`

* שורש רשימה → הוספה/הרחבה
* שורש מילון → `update(...)` רדוד
* מעלה `TypeError` אם סוג שגוי לשורש מילון

```python
# שורש מילון
await j.awrite({"users": [{"id": 1}]})
await j.aappend({"active": True})     # בסדר
# סוג שגוי:
try:
    await j.aappend(["not", "a", "dict"])  # מעלה TypeError על שורש מילון
except TypeError:
    ...
```

#### `await asearch(pattern=None, *, key=None, value=None, regex=False, case=False, limit=None) -> list[dict]`

```python
await j.awrite({"users":[{"id":1, "name":"Dana"},{"id":2,"name":"Noa"}], "active": True})

# 1) לפי שם מפתח + דפוס על ערך
hits = await j.asearch(key="name", pattern="no", case=False)
print([h["value"] for h in hits])  # ['Noa']

# 2) לפי ערך מדויק בכל מקום
hits = await j.asearch(value=True)
print(hits)  # לפחות התאמה אחת ל-"active": True
```

#### `async with AJsonFile(...)`

```python
try:
    async with AJsonFile("risky.json", "example_data") as jf:
        await jf.awrite({"state": "bad"})
        raise RuntimeError("oops")
except RuntimeError:
    pass
# משוחזר מהגיבוי האחרון אם קיים
```

---

## ACsvFile

```python
from ADVfile_manager import ACsvFile

c = ACsvFile("table.csv", "example_data")
await c.awrite([{"name":"Avi","age":30},{"name":"Dana","age":25}])  # כותב כותרת
await c.aappend({"name":"Noa"})   # 'age' חסר -> "" בקובץ

rows = await c.aread()
print(rows)

hits = await c.asearch(pattern="avi", case=False, columns=["name"])
print(hits[0]["row"], hits[0]["col"], hits[0]["value"])
```

---

## AYamlFile

> דורש `pyyaml` (`pip install pyyaml`)

```python
from ADVfile_manager import AYamlFile

y = AYamlFile("conf.yaml","example_data")
await y.awrite({"app":{"name":"demo"},"features":["a"]})
await y.aappend({"features":["b"]})  # עדכון רדוד למילון
hits = await y.asearch(key="app")
print(hits[0]["value"])
```

---

## AIniFile

```python
from ADVfile_manager import AIniFile

ini = AIniFile("settings.ini","example_data")
await ini.awrite({"Server":{"Host":"127.0.0.1","Port":"8000"}})
await ini.aappend({"Server":{"Debug":"true"}})

cfg = await ini.aread()
print(cfg["server"]["debug"])  # 'true' (מפתחות באותיות קטנות בזיכרון)

hits = await ini.asearch(key="host")
print(hits[0]["value"])  # "127.0.0.1"
```

---

## ATomlFile

> קריאה: Python 3.11+ (`tomllib`) או `tomli`; כתיבה: `tomli-w`

```python
from ADVfile_manager import ATomlFile

t = ATomlFile("cfg.toml","example_data")
await t.awrite({"app":{"name":"demo","ver":1}, "flags":{"x":True}})
await t.aappend({"flags":{"y":False}})   # מיזוג עמוק מילונים
hits = await t.asearch(key="y", value=False)
print(hits)
```

---

## AXmlFile

```python
from ADVfile_manager import AXmlFile
import xml.etree.ElementTree as ET

x = AXmlFile("data.xml","example_data")
root = ET.Element("books")
root.append(ET.Element("book", attrib={"id":"1"}))
await x.awrite(root)

await x.aappend(ET.Element("book", attrib={"id":"2"}))

hits = await x.asearch(tag="book", attr={"id":"2"})
print(hits[0]["context"].tag, hits[0]["value"])   # 'book', ''
```

---

## AExcelFile

> דורש `openpyxl`

```python
from ADVfile_manager import AExcelFile

xls = AExcelFile("grades.xlsx","example_data", default_sheet="ClassA")
await xls.awrite([{"name":"Avi","score":100},{"name":"Dana","score":90}])
await xls.aappend({"name":"Noa"})  # 'score' חסר -> "" בקובץ

rows = await xls.aread()
print(rows[-1])   # {'name': 'Noa', 'score': ''}

hits = await xls.asearch(pattern="no", columns=["name"])
print([(h["row"], h["value"]) for h in hits])
```

---

# גיבויים, שחזור ובטיחות בהקשרים — ניתוח מעמיק

תכונות אלה נמצאות במחלקת הבסיס `File` ויורשות לכל הסוגים הקונקרטיים.

## למה

* **חזרה אחורה**: אם כתיבה משתבשת, אתה יכול לשחזר את המצב האחרון הידוע כטוב.
* **עריכה טרנזקציונלית**: מנהל הקשרים מגן מפני כתיבות חלקיות.
* **אוטומציה**: גיבויים לא נשמרים ("זמניים") מנוקים אוטומטית בסיום המפרש.

## API ודוגמאות

### `backup() -> str`

צור `.bak` עם חותמת זמן תחת `<path>/backups`.

```python
txt = TextFile("file.txt","example_data")
txt.write("v1")
b = txt.backup()
print("Backup:", b)
```

**מעלה:** `FileNotFoundError` אם קובץ המקור לא קיים.

---

### `list_backups() -> list[str]`

ממוינים (ישן → חדש) נתיבים מוחלטים.

```python
for b in txt.list_backups():
    print(b)
```

---

### `restore(backup_path: str | None = None) -> str`

שחזר מנתיב גיבוי ספציפי, או האחרון אם `None`.

```python
# האחרון
txt.restore()

# ספציפי
backups = txt.list_backups()
txt.restore(backups[-2])
```

**מעלה:** `FileNotFoundError` אם אין גיבויים.

---

### `clear_backups() -> int`

מחק את כל קבצי .bak לקובץ זה; מחזיר ספירה.

```python
removed = txt.clear_backups()
print("Removed backups:", removed)
```

---

### בטיחות מנהל הקשרים (`with ... as f:`)

* **בכניסה**: גיבוי אוטומטי (אם `keep_backup=True`, ברירת מחדל).
* **בשגיאה**: שחזור אוטומטי מהגיבוי האחרון.
* **ביציאה**: מטמון מנוקה.
* **זמני**: אם `keep_backup=False`, גיבויים מנוקים ביציאה.

```python
# בטיחות טרנזקציונלית כברירת מחדל
try:
    with JsonFile("conf.json","example_data") as jf:
        jf.write({"ok": 1})
        raise RuntimeError()
except RuntimeError:
    pass

# גיבויים זמניים (ללא שמירה)
with TextFile("temp.txt","example_data")(keep_backup=False) as f:
    f.write("temporary")
# גיבויים מנוקים כאן
```

---

# כלי עזר גלובליים

### `set_exit_cleanup(enabled: bool) -> None`

הפעל/בטל ניקוי אוטומטי של **גיבויים זמניים** בסיום המפרש. ברירת מחדל: **מופעל**.

```python
from ADVfile_manager import set_exit_cleanup
set_exit_cleanup(False)  # בטל ניקוי אוטומטי
set_exit_cleanup(True)   # הפעל שוב
```

---

### `cleanup_backups_for_all() -> int`

נקה ידנית גיבויים לכל הקבצים **הזמניים** הרשומים; מחזיר סך מוסרים.

```python
from ADVfile_manager import cleanup_backups_for_all
removed = cleanup_backups_for_all()
print("Removed ephemeral backups:", removed)
```

---

# גליון עזר ל-`search(...)` אחיד

כל מחלקות הקבצים מיישמות את אותה חתימה. לא כל הפרמטרים חלים על כל פורמט, אבל הממשק עקבי:

```python
search(
  pattern: str | None = None,
  *,
  regex: bool = False,
  case: bool = False,
  key: str | None = None,       # מפתח מיפוי / שדה (JSON/YAML/INI/TOML)
  value: Any = None,            # התאמה מדויקת לערך
  columns: Sequence[str] | None = None,  # עמודות CSV/Excel
  tag: str | None = None,       # תגית אלמנט XML
  attr: dict[str,str] | None = None,     # תכונות XML
  sheet: str | None = None,     # שם גיליון Excel
  limit: int | None = None,     # מקסימום התאמות
) -> Iterator[dict]
```

* **TextFile**: `pattern`, `regex`, `case`, `limit`
* **JsonFile/YamlFile/TomlFile**: `pattern`, `key`, `value`, `regex`, `case`, `limit`
* **IniFile**: `pattern`, `key`, `value`, `regex`, `case`, `limit` (קטעים/מפתחות מנורמלים לאותיות קטנות בזיכרון)
* **CsvFile**: `pattern`, `columns`, `value`, `regex`, `case`, `limit`
* **XmlFile**: `tag`, `attr`, `pattern`, `value`, `regex`, `case`, `limit`
* **ExcelFile**: `sheet`, `columns`, `pattern`, `value`, `regex`, `case`, `limit`

**מילון התאמה מוחזר** עשוי להכיל:
`{"path","value","line","row","col","sheet","context", ...}`
(רק מפתחות רלוונטיים מוגדרים; אחרים `None`.)

---

# התקנה (v1.3.0)

**התקנה בסיסית (פורמטים ליבה: Text/JSON/CSV/INI/XML):**

```bash
pip install ADVfile_manager
```

**תמיכה ב-YAML:**

```bash
pip install pyyaml
```

**תמיכה ב-TOML:**

* קריאה (Python ≥3.11): מובנה `tomllib`
* קריאה (Python 3.8–3.10):

  ```bash
  pip install tomli
  ```
* כתיבה (כל הגרסאות):

  ```bash
  pip install tomli-w
  ```

**תמיכה ב-Excel:**

```bash
pip install openpyxl
```

**עזרי בדיקה אסינכרוניים (אופציונלי לבדיקות):**

```bash
pip install pytest pytest-asyncio
```

**גרסת Python:** 3.8+ מומלצת.

---

# טבלאות השוואה

## ADVfile_manager מול ספריות פופולריות

| תכונה / כלי              | **ADVfile_manager**       | pathlib (ספרייה סטנדרטית) | os/shutil (ספרייה סטנדרטית) | pandas              | PyYAML / ruamel.yaml | configparser | tomli/tomllib + tomli-w | xml.etree  | openpyxl |
| -------------------------- | -------------------------- | --------------------------- | ----------------------------- | ------------------- | -------------------- | ------------ | ----------------------- | ---------- | -------- |
| ממשק API אחיד על פני פורמטים | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| Text                       | ✅                          | פעולות נתיבים בלבד        | פעולות קבצים בלבד           | ממוקד DF            | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| JSON                       | ✅                          | ❌                          | ❌                            | דרך read_json (DF)  | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| CSV                        | ✅                          | ❌                          | ❌                            | ✅ (כבד)             | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| YAML                       | ✅ (PyYAML)                 | ❌                          | ❌                            | ❌                   | ✅                    | ❌            | ❌                       | ❌          | ❌        |
| INI                        | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ✅            | ❌                       | ❌          | ❌        |
| TOML                       | ✅ (tomli/tomllib, tomli-w) | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ✅                       | ❌          | ❌        |
| XML                        | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ✅          | ❌        |
| Excel                      | ✅ (openpyxl)               | ❌                          | ❌                            | ✅ (כבד)             | ❌                    | ❌            | ❌                       | ❌          | ✅        |
| כתיבות אטומיות (זמני + החלפה) | ✅                          | ❌                          | ❌                            | תלוי                 | לא רלוונטי           | לא רלוונטי   | לא רלוונטי              | תלוי        | תלוי      |
| גיבויים / שחזור           | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| שחזור אוטומטי במנהל הקשרים | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| ממשק חיפוש() אחיד         | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| API אסינכרוני             | ✅                          | ❌                          | ❌                            | ❌                   | ❌                    | ❌            | ❌                       | ❌          | ❌        |
| משקל / תלות                | קל, אופציונלי             | קל מאוד                     | קל מאוד                       | כבד                  | קל                    | קל מאוד       | קל                       | קל מאוד     | בינוני    |

**שורה תחתונה:** ADVfile_manager הוא היחיד ש**קושר הכל יחד** עם API עקבי, בטיחות-ראשונה (קלט/פלט + גיבויים + חיפוש + אסינכרוני). השתמש ב-pandas רק אם אתה *צריך* ניתוח נתונים.

---

## ADVfile_manager מול `ATmulti_file_handler` *שלך* (כלי היפותטי קודם)

| תחום                      | **ADVfile_manager (1.3.0)**                 | ATmulti_file_handler                       |
| ------------------------- | -------------------------------------------- | ------------------------------------------ |
| פורמטים                   | Text, JSON, CSV, YAML, INI, TOML, XML, Excel | בדרך כלל פחות פורמטים (לרוב ללא Excel/XML) |
| חיפוש אחיד                | ✅ חתימה אחת על כל                          | ❌/חלקי (עזרים לפי פורמט)                 |
| בטיחות מנהל הקשרים       | ✅ גיבוי + שחזור אוטומטי                    | חלקי או ידני                              |
| ממשק אסינכרוני           | ✅ מחלקות A* לכל הפורמטים                  | חלקי / אין                                |
| ניקוי גיבויים זמניים    | ✅ (atexit + ידני)                           | ❌                                         |
| טיפול ב-Excel             | ✅ כותרות ראשונות, הוספה חזקה עם ריקים    | ❌/חלקי                                    |
| חיפוש תכונה/תגית XML     | ✅                                           | ❌/חלקי                                    |
| כיסוי בדיקות              | נרחב (סינכרוני + אסינכרוני)               | חלקי                                      |
| תיעוד (קובץ README זה)   | עמוק, דוגמאות לכל מתודה                    | מוגבל                                     |

**ערך שדרוג:** API אסינכרוני מודרני, כיסוי פורמטים רחב יותר, חיפוש אחיד, תכונות בטיחות עקביות, ובדיקות חזקות יותר.

---

# פרויקט לדוגמה — מקצה לקצה

צינור מיני מציאותי שמגע במספר פורמטים, משתמש בגיבויים, בטיחות הקשרים, חיפוש אחיד, ואסינכרוני היכן שימושי.

```python
"""
מטרה:
- טעינת הגדרות (INI + TOML)
- קריאת CSV מקור, העשרה מנתוני JSON ייחוס
- פלט תוצאות לשני JSON ו-Excel
- ספק שאילתות חיפוש מהירות על פלטים
- השתמש בבטיחות טרנזקציונלית + גיבויים
- הדגם שלבים אסינכרוניים
"""

import asyncio
from ADVfile_manager import (
    IniFile, TomlFile, JsonFile, CsvFile, ExcelFile,
    AJsonFile, AExcelFile, set_exit_cleanup
)

BASE = "project_data"

def load_settings():
    ini = IniFile("settings.ini", BASE)
    if not ini.status:
        ini.write({"Server":{"Host":"127.0.0.1","Port":"8000"}})
    cfg = ini.read()

    toml = TomlFile("feature_flags.toml", BASE)
    try:
        flags = toml.read()
    except Exception:
        flags = {"features":{"excel_export": True}}
        toml.write(flags)

    return cfg, flags

def enrich_rows_from_json(rows):
    ref = JsonFile("ref.json", BASE)
    if not ref.status:
        ref.write({"vip_users": ["Avi"]})
    vip_list = ref.read().get("vip_users", [])
    out = []
    for r in rows:
        r = dict(r)
        r["vip"] = r.get("name") in vip_list
        out.append(r)
    return out

def process_sync():
    # 1) קריאת CSV
    src = CsvFile("source.csv", BASE)
    if not src.status:
        src.write([{"name":"Avi","score":100},{"name":"Dana","score":90},{"name":"Noa","score":95}])
    rows = src.read()

    # 2) העשרה עם ייחוס JSON
    enriched = enrich_rows_from_json(rows)

    # 3) כתיבה טרנזקציונלית עם גיבוי
    out_json = JsonFile("result.json", BASE)
    with out_json as jf:
        jf.write({"rows": enriched})

    # 4) חיפוש: מצא VIP
    hits = list(out_json.search(pattern="True", key="vip"))
    print("VIP hits:", hits)

async def async_exports():
    # יצוא Excel אסינכרוני
    aex = AExcelFile("result.xlsx", BASE, default_sheet="Report")
    await aex.awrite([{"name":"Avi","score":100,"vip":True},
                      {"name":"Dana","score":90,"vip":False},
                      {"name":"Noa","score":95,"vip":False}])
    # הוספה
    await aex.aappend({"name":"Omri","score":84,"vip":False})
    # חיפוש לפי דפוס
    hits = await aex.asearch(pattern="o", columns=["name"])
    print("Excel pattern hits:", [(h["row"], h["value"]) for h in hits])

def main():
    set_exit_cleanup(True)

    cfg, flags = load_settings()
    print("INI:", cfg)
    print("Flags:", flags)

    process_sync()
    asyncio.run(async_exports())

if __name__ == "__main__":
    main()
```

**מה זה מראה:**

* טעינת וכתיבת INI + TOML.
* נתוני ייחוס JSON ויבוא CSV.
* כתיבת JSON טרנזקציונלית עם מנהל הקשרים.
* יצוא Excel אסינכרוני כתיבה/הוספה/חיפוש.
* שאילתות חיפוש אחיד (`pattern`, `key`, `columns`, ...).

---

# מפת דרכים (גרסאות הבאות)

רק הפריטים שביקשת:

* 🔐 **הצפנת/פענוח קבצים**
  הצפנה אופציונלית לכל קובץ (כנראה AES-GCM) עם ווים לניהול חומר מפתחות, חלקה בתוך `read`/`write`/`append` וגיבויים שקופים.

* 🧾 **זיהוי שינויים מבוסס האש**
  שמור והצג דיגסטים SHA-256 (ואופציונלי MD5); זיהוי שינויים חיצוניים ואופציונלי ביטול מטמון או העלאת שגיאה בקריאות מיושנות.