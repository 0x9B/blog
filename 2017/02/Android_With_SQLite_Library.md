# Android With SQLite Library

If you have tried on writing Android applications, you may know there are
SQLite support in Android which you could easily create, read and write
a database that belongs to your application. However, the default
`cursor.get[DataType]()` only get the column index as an argument,
which the column index is an integer with no meaning itself and hence make
code difficult to read and understand.

In this case, you may create your own wrapper class which extend
CursorWrapper to make the whole thing simpler. Here is an example
for the new created class:

```java
public class OwnCursorWrapper extends CursorWrapper {

  public OwnCursorWrapper(Cursor cursor) {
    super(cursor);
  }

  public String getString(String columnName) {
    return super.getString(getColumnIndexOrThrow(columnName));
  }

  public String getString(String columnName, String defaultValue) {
    String result = getString(columnName);
    return result == null ? defaultValue : result;
  } 
}
```

The above example only provide two new `getString` functions which will
take the column name as argument and return the corresponding result.
But please be reminded that there are some “traps” in Androd’s
SQLite calls. They will be discussed in later posts. (Yes, posts,
as there are many traps)

See you next time~
