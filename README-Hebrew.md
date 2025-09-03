
# ADVfile\_manager

[![PyPI version](https://badge.fury.io/py/ADVfile_manager.svg)](https://pypi.org/project/ADVfile_manager/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org/)

**מחבר:** Avi Twil
**מאגר (Repo):** [https://github.com/avitwil/ADVfile\_manager](https://github.com/avitwil/ADVfile_manager)

ספרייה אחודה לניהול קבצים בפייתון עם **כתיבה בטוחה (atomic)**, **קאש בזיכרון**, **גיבויים**, **קונטקסט מנג’ר** וניקוי אוטומטי בעת יציאה — תחת API עקבי עבור קבצי **טקסט**, **JSON**, **CSV** ו־**YAML**.

* `TextFile` – קריאה/כתיבה/הוספה, כולל `lines()` ו־`read_line()`.
* `JsonFile` – תומך בשורש מסוג dict או list, כולל `append()`, `get_item()`, `items()`.
* `CsvFile` – מבוסס `DictReader`/`DictWriter`, כולל `read_row()`, `rows()`, שליטה בסדר עמודות.
* `YamlFile` – בדומה ל־`JsonFile`, דורש `PyYAML`.

מחלקת הבסיס `File` מוסיפה **גיבויים**, **שחזור**, **מדיניות שמירת גיבויים (retention)**, **גודל קריא לאדם**, **שליטה בקאש**, ו־**קונטקסט מנג’ר** שמגבה אוטומטית ויכול לשחזר בשגיאה. גיבויי “אףמרליים” (זמניים) נמחיים בשקט דרך **hook של atexit**.

---

## תוכן עניינים

* [למה ADVfile\_manager?](#למה-advfile_manager)
* [התקנה](#התקנה)
* [התחלה מהירה](#התחלה-מהירה)
* [שימוש מפורט](#שימוש-מפורט)

  * [מחלקת בסיס: `File`](#מחלקת-בסיס-file)
  * [`TextFile`](#textfile)
  * [`JsonFile`](#jsonfile)
  * [`CsvFile`](#csvfile)
  * [`YamlFile`](#yamlfile)
* [גיבויים, שמירה וניקוי ביציאה](#גיבויים-שמירה-וניקוי-ביציאה)
* [בטיחות עם קונטקסט מנג’ר](#בטיחות-עם-קונטקסט-מנג’ר)
* [הערות מתקדמות](#הערות-מתקדמות)
* [דוגמאות מלאות](#דוגמאות-מלאות)
* [מדריך פיצ’רים (הסבר שימוש + דוגמאות)](#מדריך-פיצרים-הסבר-שימוש--דוגמאות)
* [רישיון](#רישיון)

---

## למה ADVfile\_manager?

קוד עבודה עם קבצים נוטה להתפזר בין עשרות פונקציות אד־הוק וחזרות.
`ADVfile_manager` נותן ממשק אחיד לכל הפורמטים הנפוצים:

* **כתיבה בטוחה (Atomic)**: החלפה אטומית כדי למנוע קבצים חלקיים/פגומים.
* **גיבויים**: יצירת קבצי `.bak` עם טיימסטמפ, רשימה, שחזור, ושמירת N אחרונים.
* **בטיחות קונטקסט**: `with` מבצע גיבוי בכניסה (אופציונלי) ומשחזר בשגיאה.
* **ניקוי ביציאה**: גיבויים זמניים נמחקים אוטומטית בעת יציאת הפרשן (atexit).
* **עבודה זורמת**: איטרציה על שורות/שורות־CSV/פריטים בלי להעמיס הכל לזיכרון.
* **קאש**: קריאה נשמרת בזיכרון; `clear_cache()` לריענון מהדיסק.

---

## התקנה

### מ־PyPI (מומלץ)

```bash
pip install ADVfile_manager
```

> ל־`YamlFile` צריך [PyYAML](https://pypi.org/project/PyYAML/). אם לא מותקן:

```bash
pip install pyyaml
```

### מהקוד (מקור)

```bash
git clone https://github.com/avitwil/ADVfile_manager
cd ADVfile_manager
pip install -e .
```

---

# 🔎 השוואה: ADVfile\_manager לעומת כלים דומים

| תכונה / כלי              | **ADVfile\_manager**                         | [pathlib](https://docs.python.org/3/library/pathlib.html) | [os/shutil](https://docs.python.org/3/library/shutil.html) | [pandas](https://pandas.pydata.org/) | [ruamel.yaml / PyYAML](https://pyyaml.org/) |
| ------------------------ | -------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------ | ------------------------------------------- |
| **פורמטים נתמכים**       | TXT, JSON, CSV, YAML                         | ניהול נתיבים בלבד                                         | פעולות קבצים (העתק/העבר/מחק)                               | CSV/Excel/JSON/Parquet וכו’          | YAML בלבד                                   |
| **API אחיד**             | ✅                                            | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **קריאה/כתיבה/הוספה**    | ✅                                            | ידני                                                      | ידני                                                       | ✅ (DataFrame)                        | ✅                                           |
| **קאש בזיכרון**          | ✅                                            | ❌                                                         | ❌                                                          | קיים בתוך DF                         | ❌                                           |
| **עזרי שורות/שורות CSV** | ✅ `lines/read_line/rows/read_row`            | ❌                                                         | ❌                                                          | דרך DF                               | ❌                                           |
| **גיבוי ושחזור**         | ✅                                            | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **שמירת גיבויים (N)**    | ✅ `max_backups`                              | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **כתיבה אטומית**         | ✅                                            | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **גודל קריא לאדם**       | ✅ `get_size_human()`                         | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **בטיחות קונטקסט**       | ✅                                            | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **גיבויים זמניים**       | ✅ atexit                                     | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **בקרת ניקוי גלובלי**    | ✅ `set_exit_cleanup/cleanup_backups_for_all` | ❌                                                         | ❌                                                          | ❌                                    | ❌                                           |
| **תלויות**               | אופציונלי: pyyaml                            | אין                                                       | אין                                                        | כבד (numpy וכו’)                     | כן                                          |

**בשורה התחתונה:** אם אתה רוצה ניהול קבצים בטוח, פשוט ועקבי בכל הפורמטים הנפוצים — בלי תלות כבדה — זהו הכלי.

---

## מקרי שימוש לדוגמה

* **קבצי קונפיג**: קריאה/עדכון JSON/YAML בבטיחות מלאה עם אפשרות רולבק.
* **לוגים ודוחות**: הוספת שורות לטקסט/CSV תוך שמירת גיבויים אוטומטיים.
* **עריכה טרנזקציונית**: שימוש ב־`with` למניעת שחיתות נתונים במקרה תקלה.
* **כלי CLI/אוטומציה**: כתיבה אחת שעובדת על כל פורמט, אותו API.

---

## התחלה מהירה

```python
from ADVfile_manager import TextFile, JsonFile, CsvFile, YamlFile

# טקסט
txt = TextFile("notes.txt", "data")
txt.write("first line")
txt.append("second line")
print(txt.read_line(2))     # "second line"
for i, line in txt.lines():
    print(i, line)

# JSON (שורש dict)
j = JsonFile("config.json", "data")
j.write({"users": [{"id": 1}]})
j.append({"active": True})  # עדכון רדוד של dict
print(j.get_item("active")) # True

# CSV
c = CsvFile("table.csv", "data")
c.write([{"name":"Avi","age":30},{"name":"Dana","age":25}], fieldnames=["name","age"])
c.append({"name":"Noa","age":21})
print(c.read_row(2))        # {"name":"Dana","age":"25"}
for idx, row in c.rows():
    print(idx, row)

# YAML
y = YamlFile("config.yaml", "data")
y.write({"app":{"name":"demo"}, "features":["a"]})
y.append({"features":["b"]})  # עדכון רדוד
print(y.get_item("app"))
```

---

## שימוש מפורט

### מחלקת בסיס: `File`

כל המחלקות יורשות מ־`File` ומשתפות את היכולות:

* `read()`, `write(data)`, `append(data)`
* `clear_cache()` — ניקוי הקאש כדי שהקריאה הבאה תהיה מהדיסק
* `get_size()` / `get_size_human()`
* גיבויים: `backup()`, `list_backups()`, `restore(backup_path=None)`, `clear_backups()`
* קונטקסט מנג’ר: `with File(...)(keep_backup=True) as f: ...`
* שליטה גלובלית בניקוי ביציאה: `set_exit_cleanup(enabled: bool)`, `cleanup_backups_for_all()`

#### בנאי

```python
File(
  file_name: str,
  file_path: str | pathlib.Path | None = None,  # ברירת מחדל: הספרייה הנוכחית
  keep_backup: bool = True,                     # לשמור גיבויים
  max_backups: int | None = None                # לשמור רק N האחרונים
)
```

* `keep_backup=False` מסמן אובייקט **אפמרלי**: הגיבויים שלו יימחקו אוטומטית ביציאה (אפשר גם ידנית).
* `max_backups` אוכף שמירת N גיבויים בעת כל `backup()`.

#### גיבויים

* `backup()` יוצר `backups/<file>.<YYYYMMDD_HHMMSS_micro>.bak`
* `list_backups()` מחזיר רשימה ממוינת (הישן→חדש)
* `restore(path=None)` משחזר לפי נתיב מסוים או את האחרון אם `None`
* `clear_backups()` מוחק את כל הגיבויים ומחזיר כמות שנמחקה

#### גודל קריא

* `get_size_human()` מחזיר מחרוזת “12.3 KB”.

---

### `TextFile`

**תוספות:**

* `lines()` → גנרטור `(מספר_שורה, טקסט)`
* `read_line(n)` → קריאת שורה לפי אינדקס 1־מבוסס

```python
txt = TextFile("example.txt", "data")
txt.write("Hello")
txt.append("World")
print(txt.read())           # "Hello\nWorld"
print(txt.read_line(2))     # "World"
for i, line in txt.lines(): # (1, "Hello"), (2, "World")
    print(i, line)
```

---

### `JsonFile`

עובד עם שורש **dict** או **list**.

**תוספות:**

* `get_item(index_or_key)`

  * בשורש רשימה: אינדקס 1־מבוסס (`int`)
  * בשורש מילון: מפתח (`str`)
* `items()`

  * רשימה: `(index, value)` (1־מבוסס)
  * מילון: `(key, value)`
* `append(data)`

  * רשימה: append/extend
  * מילון: `dict.update()` רדוד

```python
# שורש dict
j = JsonFile("conf.json", "data")
j.write({"users":[{"id":1}]})
j.append({"active": True})     # מיזוג רדוד
print(j.get_item("active"))    # True
for k, v in j.items():
    print(k, v)

# שורש list
jl = JsonFile("list.json", "data")
jl.write([{"id":1}])
jl.append({"id":2})
jl.append([{"id":3},{"id":4}])
print(jl.get_item(2))          # {"id":2}
for i, item in jl.items():
    print(i, item)
```

---

### `CsvFile`

**עקרון:** עבודה עם `csv.DictReader/DictWriter` — שורות הן dict.

**תוספות:**

* `write(data, fieldnames=None)` — להגדיר סדר עמודות; אחרת ייגזר מהנתונים
* `read_row(n)` — קריאת שורה לפי אינדקס 1־מבוסס
* `rows()` — גנרטור `(מספר_שורה, dict)`
* `append(dict | iterable[dict])` — הוספת שורה/ות, כולל כתיבת כותרת אם חדש

```python
c = CsvFile("table.csv", "data")
c.write(
    [{"name":"Avi","age":30},{"name":"Dana","age":25}],
    fieldnames=["name","age"]
)
c.append({"name":"Noa","age":21})
print(c.read_row(2))               # {"name":"Dana","age":"25"}
for i, row in c.rows():
    print(i, row)
```

---

### `YamlFile`

בדומה ל־`JsonFile`, על בסיס YAML.
**דורש:** `pip install pyyaml`.

**תוספות:**

* `get_item(index_or_key)` — אינדקס 1־מבוסס לרשימה, מפתח למילון
* `items()` — אותו עיקרון כמו JSON
* `append()` — append/extend לרשימה; `update()` רדוד למילון

```python
from ADVfile_manager import YamlFile

y = YamlFile("config.yaml", "data")
y.write({"app":{"name":"demo"}, "features":["a"]})
y.append({"features":["b"]})
print(y.get_item("app"))
for k, v in y.items():
    print(k, v)
```

---

## גיבויים, שמירה וניקוי ביציאה

* **יצירה**: `path = f.backup()` — עם דיוק עד מיקרושניות.
* **שמירה (Retention)**: `max_backups=N` ישמור רק את N האחרונים בכל `backup()`.
* **רשימה**: `f.list_backups()` — ישן→חדש.
* **שחזור**:

  * האחרון: `f.restore()`
  * ספציפי: `f.restore(path)`
* **מחיקה**: `f.clear_backups()` מחזיר כמה גיבויים נמחקו.

**גיבויים אפמרליים** (`keep_backup=False`):

* מסמן את האובייקט כזמני — הגיבויים שלו יימחקו **אוטומטית** ביציאה (`atexit`).
* שליטה גלובלית:

```python
from ADVfile_manager import set_exit_cleanup, cleanup_backups_for_all

set_exit_cleanup(True)          # ברירת מחדל
set_exit_cleanup(False)         # לכבות ניקוי ביציאה
removed = cleanup_backups_for_all()  # ניקוי ידני (מחזיר כמות קבצים)
```

---

## בטיחות עם קונטקסט מנג’ר

* `with` יוצר **גיבוי בכניסה** (אלא אם `keep_backup=False`).
* אם מתרחשת **שגיאה** בתוך הבלוק ו־`keep_backup=True`, הקובץ **ישוחזר** לגיבוי האחרון.
* **תמיד** מתבצע `clear_cache()` ביציאה כדי שהקריאה הבאה תהיה מהדיסק.

```python
# ברירת מחדל: keep_backup=True
with TextFile("draft.txt", "data") as f:
    f.write("safe transactional edit")

# גיבויים זמניים — יימחקו ביציאה
with TextFile("temp.txt", "data")(keep_backup=False) as f:
    f.write("temporary content")
```

**דוגמה ריאלית לשחזור בשגיאה:**

```python
try:
    with TextFile("draft.txt", "data") as f:
        f.write("New content")
        raise RuntimeError("Simulated crash")
except:
    pass

print("Restored:", TextFile("draft.txt", "data").read())
# -> יחזור לתוכן הקודם
```

---

## הערות מתקדמות

* **כתיבה אטומית**: שימוש בקובץ `*.tmp` ו־`os.replace()` — אין מצב של קובץ “חצי־כתוב”.
* **Pathlib**: כל המחלקות מקבלות `str` או `pathlib.Path` עבור `file_path`.
* **קאש**: `read()` שומר בזיכרון; `clear_cache()` לריענון.
* **סמנטיקת append**:

  * טקסט: מוסיף `"\n"+data` אם הקובץ לא ריק.
  * JSON/YAML:

    * שורש רשימה → append/extend
    * שורש מילון → `dict.update()` רדוד
* **CSV**: קריאה תמיד כמחרוזות (`DictReader`). לאחר `append()`, הקאש שומר עקביות.
* **פייתון**: מומלץ 3.8+.

---

## דוגמאות מלאות

### 1) טקסט + גיבויים + שחזור ספציפי

```python
from ADVfile_manager import TextFile

txt = TextFile("example.txt", "example_data")
txt.write("v1"); b1 = txt.backup()
txt.write("v2"); b2 = txt.backup()
txt.write("v3"); b3 = txt.backup()

print("Backups:", txt.list_backups())
txt.restore(b2)
print("Restored content:", txt.read())  # "v2"
```

### 2) שמירת גיבויים (רק 2 אחרונים)

```python
txt.max_backups = 2
txt.write("v4"); txt.backup()
txt.write("v5"); txt.backup()
print("After retention:", txt.list_backups())  # רק 2 האחרונים
```

### 3) גיבויים זמניים + ניקוי ביציאה/ידני

```python
from ADVfile_manager import TextFile, cleanup_backups_for_all, set_exit_cleanup

with TextFile("temp.txt", "example_data")(keep_backup=False) as f:
    f.write("temporary content")

deleted = cleanup_backups_for_all()
print("Deleted backup files:", deleted)

set_exit_cleanup(False)   # לכבות ניקוי ביציאה
set_exit_cleanup(True)    # להפעיל שוב
```

### 4) CSV עם שליטה בסדר עמודות

```python
from ADVfile_manager import CsvFile

rows = [{"name":"Avi","age":30},{"name":"Dana","age":25}]
c = CsvFile("table.csv", "example_data")
c.write(rows, fieldnames=["name","age"])  # סדר מפורש
c.append({"name":"Noa","age":21})

print(c.read_row(2))      # {"name":"Dana","age":"25"}
for i, row in c.rows():
    print(i, row)
```

### 5) JSON/YAML — dict & list

```python
from ADVfile_manager import JsonFile, YamlFile

# JSON dict
j = JsonFile("data.json", "example_data")
j.write({"users":[{"id":1}]})
j.append({"active": True})
print(j.get_item("active"))  # True

# JSON list
jl = JsonFile("list.json", "example_data")
jl.write([{"id":1}])
jl.append([{"id":2},{"id":3}])
print(jl.get_item(2))        # {"id":2}

# YAML dict
y = YamlFile("config.yaml", "example_data")
y.write({"app":{"name":"demo"},"features":["a"]})
y.append({"features":["b"]})
print(y.get_item("app"))
```

---

## מדריך פיצ’רים (הסבר שימוש + דוגמאות)

להלן **כל הפיצ’רים**, לכל אחד **הסבר שימוש (מה ולמה)** ו־**דוגמאות קוד**.

### ✔️ כתיבה בטוחה (Atomic)

**הסבר שימוש:**
מונע קובץ פגום במקרה קריסה באמצע כתיבה. כותב לקובץ זמני ואז מחליף אטומית את היעד. קריטי לקבצי קונפיג/לוגים.

**דוגמאות:**

```python
from ADVfile_manager import TextFile

cfg = TextFile("settings.txt", "data")
cfg.write("mode=prod")
cfg.write("mode=debug")    # מצב קודם קיים עד לסיום ההחלפה
```

---

### 🧠 קאש בזיכרון & `clear_cache()`

**הסבר שימוש:**
`read()` שומר את התוכן בזיכרון. אם קובץ השתנה מחוץ לתהליך — קרא `clear_cache()` לפני `read()` הבא.

**דוגמאות:**

```python
f = TextFile("cache.txt", "data")
print(f.read())   # קריאה + קאש
# שינוי חיצוני...
f.clear_cache()
print(f.read())   # רענון מהדיסק
```

---

### 📏 גודל קובץ & `get_size_human()`

**הסבר שימוש:**
בדיקות sanity או תצוגה ידידותית למשתמש.

**דוגמאות:**

```python
f = TextFile("size.txt", "data")
f.write("Hello world")
print(f.get_size())         # למשל 11
print(f.get_size_human())   # "11.0 B"
```

---

### 🛟 גיבויים: `backup` / `list_backups` / `restore` / `clear_backups`

**הסבר שימוש:**
לפני שינוי מסוכן — צור גיבוי. אם השתבש — שחזר. ראה רשימת גיבויים ומחק כשסיימת.

**דוגמאות:**

```python
f = TextFile("report.txt", "data")
f.write("v1"); b1 = f.backup()
f.write("v2"); b2 = f.backup()

print(f.list_backups())
f.restore(b1); print(f.read())  # "v1"
f.restore();    print(f.read())  # "v2"

print("Removed:", f.clear_backups())
```

---

### ♻️ שמירת גיבויים (Retention) עם `max_backups`

**הסבר שימוש:**
להגביל שימוש בדיסק — הישאר רק עם N הגיבויים האחרונים. נאכף בכל `backup()`.

**דוגמאות:**

```python
r = TextFile("retain.txt", "data", max_backups=2)
for i in range(5):
    r.write(f"v{i}"); r.backup()
print(r.list_backups())  # רק 2
```

---

### 🧰 קונטקסט מנג’ר (בטיחות טרנזקציונית)

**הסבר שימוש:**
ב־`with` נוצר גיבוי בכניסה. בשגיאה — משחזרים ביציאה. תמיד מנקים קאש.

**דוגמאות:**

```python
with TextFile("draft.txt", "data") as f:
    f.write("transaction")

try:
    with TextFile("draft.txt", "data") as f:
        f.write("new content")
        raise RuntimeError("boom")
except:
    pass

print("Restored:", TextFile("draft.txt", "data").read())
```

---

### 🧹 גיבויים אפמרליים & ניקוי ביציאה

**הסבר שימוש:**
לפעולות זמניות — `keep_backup=False`. הגיבויים יימחקו ביציאה (או ידנית). שליטה גלובלית בניקוי.

**דוגמאות:**

```python
from ADVfile_manager import set_exit_cleanup, cleanup_backups_for_all

with TextFile("tmp.txt", "data")(keep_backup=False) as f:
    f.write("temp")

set_exit_cleanup(False)  # לכבות
set_exit_cleanup(True)   # להפעיל
print("Removed:", cleanup_backups_for_all())
```

---

### 📝 `TextFile`: `lines()` & `read_line()`

**הסבר שימוש:**
עיבוד קבצים גדולים — זרימה לפי שורה. גישה לשורה ספציפית (1־מבוסס).

**דוגמאות:**

```python
poem = TextFile("poem.txt", "data")
poem.write("Line 1"); poem.append("Line 2"); poem.append("Line 3")

print(poem.read_line(2))  # "Line 2"
for i, line in poem.lines():
    print(i, line)
```

---

### 🧩 `JsonFile`: שורש dict/list, `append`, `get_item`, `items`

**הסבר שימוש:**
גמישות מלאה: הוספה לרשימה, מיזוג רדוד למילון. גישה לפי אינדקס/מפתח.

**דוגמאות:**

```python
from ADVfile_manager import JsonFile

conf = JsonFile("conf.json", "data")
conf.write({"users":[{"id":1}]})
conf.append({"active": True})
print(conf.get_item("active"))
for k, v in conf.items():
    print(k, v)

lst = JsonFile("list.json", "data")
lst.write([{"id":1}]); lst.append({"id":2}); lst.append([{"id":3},{"id":4}])
print(lst.get_item(2))
for i, item in lst.items():
    print(i, item)
```

---

### 🧾 `CsvFile`: כתיבה/הוספה, `read_row`, `rows`, סדר עמודות

**הסבר שימוש:**
CSV כ־list of dicts. שליטה בכותרות/סדר, הוספות בטוחות, גישה לשורה ספציפית, זרימה שורה־שורה.

**דוגמאות:**

```python
from ADVfile_manager import CsvFile

rows = [{"name":"Avi","age":30}, {"name":"Dana","age":25}]
csvf = CsvFile("table.csv", "data")
csvf.write(rows, fieldnames=["name","age"])
csvf.append({"name":"Noa","age":21})

print(csvf.read_row(2))    # {"name":"Dana","age":"25"}
for i, row in csvf.rows():
    print(i, row)
```

---

### 🧱 `YamlFile`: שורש dict/list, `append`, `get_item`, `items` (דורש PyYAML)

**הסבר שימוש:**
אותן סמנטיקות כמו JSON, מותאם לקבצי קונפיג.

**דוגמאות:**

```python
from ADVfile_manager import YamlFile

y = YamlFile("config.yaml", "data")
y.write({"app":{"name":"demo"},"features":["a"]})
y.append({"features":["b"]})
print(y.get_item("app"))
for k, v in y.items():
    print(k, v)
```

---

### 🧭 ניהול נתיבים (`str` / `pathlib.Path`)

**הסבר שימוש:**
שילוב קל עם קוד מודרני של נתיבים.

**דוגמאות:**

```python
from pathlib import Path
from ADVfile_manager import TextFile

p = Path("data")
txt = TextFile("notes.txt", p)
txt.write("hello with pathlib")
```

---

## רישיון

**MIT License** — © 2025 Avi Twil.
ראה [`LICENSE`](./LICENSE) לפרטים.

---

שאלות או הצעות? פתחו Issue או PR:
**[https://github.com/avitwil/ADVfile\_manager](https://github.com/avitwil/ADVfile_manager)**
