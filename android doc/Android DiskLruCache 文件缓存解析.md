DiskLruCache 文件缓存解析

DiskLruCache与LruCache都实现了Lru缓存功能，两者都用于图片的三重缓存中。
 LruCache将图片保存在内存，存取速度较快，退出APP后缓存会失效；而DiskLruCache将图片保存在磁盘中，版本号不变下次进入应用后缓存依旧存在 。

# 一、DiskLruCache简介

GitHub：https://github.com/JakeWharton/DiskLruCache

APIDoc：http://jakewharton.github.io/DiskLruCache/

1. 缓存的`key`为`String`类型，且必须匹配正则表达式`[a-z0-9_-]{1,64}`；

2. 一个key可以对应多个value，`value`类型为字节数组，大小在`0 ~ Integer.MAX_VALUE`之间；

3. 缓存的目录必须为专用目录，因为DiskLruCache可能会删除或覆盖该目录下的文件。

4. 添加缓存操作具备原子性，但多个进程不应该同时使用同一个缓存目录。
   
   **添加依赖**

```java
dependencies {
     implementation 'com.jakewharton:disklrucache:2.0.2'
}
```

 **创建DiskLruCache对象**

```java
/* 
* directory – 缓存目录 
* appVersion - 缓存版本 
* valueCount – 每个key对应value的个数 
* maxSize – 缓存大小的上限 
*/ 
DiskLruCache diskLruCache = DiskLruCache.open(directory, 1, 1, 1024 * 1024 * 10);
```

**添加 / 获取 缓存数据 （一对一）**

```java
/**
* 添加一条缓存，一个key对应一个value
 */
public void addDiskCache(String key, String value) throws IOException {
    File cacheDir = context.getCacheDir();
    DiskLruCache diskLruCache = DiskLruCache.open(cacheDir, 1, 1, 1024 * 1024 * 10);
    DiskLruCache.Editor editor = diskLruCache.edit(key);
    // index与valueCount对应，分别为0,1,2...valueCount-1
    editor.newOutputStream(0).write(value.getBytes()); 
    editor.commit();
    diskLruCache.close();
}
```

```java
/**
 * 获取一条缓存，一个key对应一个value
 */
public void getDiskCache(String key) throws IOException {
    File directory = context.getCacheDir();
    DiskLruCache diskLruCache = DiskLruCache.open(directory, 1, 1, 1024 * 1024 * 10);
    String value = diskLruCache.get(key).getString(0);
    diskLruCache.close();
}
```

**添加 / 获取 缓存数据 （一对多）**

```java
 /**
 * 添加一条缓存，1个key对应2个value
 */
public void addDiskCache(String key, String value1, String value2) throws IOException {
    File directory = context.getCacheDir();
    DiskLruCache diskLruCache = DiskLruCache.open(directory, 1, 2, 1024 * 1024 * 10);
    DiskLruCache.Editor editor = diskLruCache.edit(key);
    editor.newOutputStream(0).write(value1.getBytes());
    editor.newOutputStream(1).write(value2.getBytes());
    editor.commit();
    diskLruCache.close();
}
```

```java
/**
 * 添加一条缓存，1个key对应2个value
 */
public void getDiskCache(String key) throws IOException {
    File directory = context.getCacheDir();
    DiskLruCache diskLruCache = DiskLruCache.open(directory, 1, 2, 1024);
    DiskLruCache.Snapshot snapshot = diskLruCache.get(key);
    String value1 = snapshot.getString(0);
    String value2 = snapshot.getString(1);
    diskLruCache.close();
}
```

# 二、源码解析

![](https://gitee.com/ZeTing/UploadImg/raw/main/img/微信截图_20201029144407.png)

存储文件格式如上图

journal 的日志文件，记录了所有的缓存的文件。

前五行是元数据：

1. 固定字符串libcore.io.DiskLruCache，用于标识这是一个 `DiskLruCache` 的日志文件。
2. `DiskLruCache` 版本号，源码中为常量1
3. 开发者声明的应用版本号。
4. 每个 key 对应几个 value，一般都是1。一个 key 支持写入多个值，每个值都是一个文件，文件名是 `key.index`。
5. 一个空行

每次初始化时都要校验这些元数据。

之后是真正的日志数据，可以认为是操作记录，每一行是一个记录，拥有 `state`, `key`, `option` 字段，用空格分隔，其中 `option` 字段根据 `state` 的不同是可选的。

- `DIRTY`: 表示一项数据正在被写入（创建或更新）。后面必须跟着一个 `CLEAN` 或 `REMOVE`。否则这个项目将被视为临时文件，需要清除。
- `CLEAN`: 表示一项数据被成功写入。后面跟着一个数据大小 (Byte)，如果一个 key 对应多个数据，则跟着多个大小。
- `REMOVE`: 表示一项数据已经被删除。
- `READ`: 表示对应项被读取了一次。其实这个不能算是状态，就是一个操作记录。

`DiskLruCache` 存储格式为 LinkedHashMap<String, Entry> lruEntries ；

​    String 代表存储的key

​    Entry 为当前key对应的日志信息

```java
private final class Entry {
    // 当前缓存文件key
  private final String key;

  /** Lengths of this entry's files. */
    // 当前缓存文件大小，可以是多个文件
  private final long[] lengths;

  /** True if this entry has ever been published. */
    // 当前文件是可以被正常读取的
  private boolean readable;

  /** The ongoing edit or null if this entry is not being edited. */
    // 当前正在被编辑的对象
  private Editor currentEditor;

  /** The sequence number of the most recently committed edit to this entry. */
  private long sequenceNumber;

  private Entry(String key) {
    this.key = key;
    this.lengths = new long[valueCount];
  }

  public String getLengths() throws IOException {
    StringBuilder result = new StringBuilder();
    for (long size : lengths) {
      result.append(' ').append(size);
    }
    return result.toString();
  }

  /** Set lengths using decimal numbers like "10123". */
  private void setLengths(String[] strings) throws IOException {
    if (strings.length != valueCount) {
      throw invalidLengths(strings);
    }

    try {
      for (int i = 0; i < strings.length; i++) {
        lengths[i] = Long.parseLong(strings[i]);
      }
    } catch (NumberFormatException e) {
      throw invalidLengths(strings);
    }
  }

  private IOException invalidLengths(String[] strings) throws IOException {
    throw new IOException("unexpected journal line: " + java.util.Arrays.toString(strings));
  }

  public File getCleanFile(int i) {
    return new File(directory, key + "." + i);
  }

  public File getDirtyFile(int i) {
    return new File(directory, key + "." + i + ".tmp");
  }
}
```

## 1.初始化

打开缓存文件 ，DiskLruCache通过`open()`方法新建一个实例，

首先处理日志文件，判断是否存在可用的日志文件，如果存在就读取日志到内存，如果不存在就新建一个日志文件。

```java
/**
 * Opens the cache in {@code directory}, creating a cache if none exists
 * there.
 *
 * @param directory a writable directory
 * @param valueCount the number of values per cache entry. Must be positive.
 * @param maxSize the maximum number of bytes this cache should use to store
 * @throws IOException if reading or writing the cache directory fails
 */
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
    throws IOException {
  // 最大允许缓存不小于0
  if (maxSize <= 0) {
    throw new IllegalArgumentException("maxSize <= 0");
  }
    // 每一个key需要缓存的数据条目不小于0
  if (valueCount <= 0) {
    throw new IllegalArgumentException("valueCount <= 0");
  }

  // If a bkp file exists, use it instead.
    // 如果存在了备份文件
  File backupFile = new File(directory, JOURNAL_FILE_BACKUP);
  if (backupFile.exists()) {
    File journalFile = new File(directory, JOURNAL_FILE);
    // If journal file also exists just delete backup file.
      // 如果日子文件存在就删除备份文件
    if (journalFile.exists()) {
      backupFile.delete();
    } else {
        // 不存在就重命名备份文件
      renameTo(backupFile, journalFile, false);
    }
  }

  // Prefer to pick up where we left off.
  DiskLruCache cache = new DiskLruCache(directory, appVersion, valueCount, maxSize);
    // 如果当前路径下存在缓存文件
  if (cache.journalFile.exists()) {
    try {
        // 读取日子文件，并且校验是否当前可以，校验错误删除日子文件
      cache.readJournal();
        // 计算日志文件已经被缓存文件大小，超出缓存大小，删除缓存
      cache.processJournal();
      cache.journalWriter = new BufferedWriter(
          new OutputStreamWriter(new FileOutputStream(cache.journalFile, true), Util.US_ASCII));
      return cache;
    } catch (IOException journalIsCorrupt) {
      System.out
          .println("DiskLruCache "
              + directory
              + " is corrupt: "
              + journalIsCorrupt.getMessage()
              + ", removing");
      cache.delete();
    }
  }

  // Create a new empty cache.
  directory.mkdirs();
  cache = new DiskLruCache(directory, appVersion, valueCount, maxSize);
    // 读取日子文件，如果不存在就创建
  cache.rebuildJournal();
  return cache;
}
```

我们来看看 readJournal 读取日子文件并对其校验

```java
private void readJournal() throws IOException {
  StrictLineReader reader = new StrictLineReader(new FileInputStream(journalFile), Util.US_ASCII);
  try {
    String magic = reader.readLine();
    String version = reader.readLine();
    String appVersionString = reader.readLine();
    String valueCountString = reader.readLine();
    String blank = reader.readLine();
    // 本地日志文件头部信息校验，错误抛出异常，删除日子文件
    if (!MAGIC.equals(magic)
        || !VERSION_1.equals(version)
        || !Integer.toString(appVersion).equals(appVersionString)
        || !Integer.toString(valueCount).equals(valueCountString)
        || !"".equals(blank)) {
      throw new IOException("unexpected journal header: [" + magic + ", " + version + ", "
          + valueCountString + ", " + blank + "]");
    }

    int lineCount = 0;
    while (true) {
      try {
          //读取日子文件信息保存进入到缓存中
        readJournalLine(reader.readLine());
        lineCount++;
      } catch (EOFException endOfJournal) {
        break;
      }
    }
    redundantOpCount = lineCount - lruEntries.size();
  } finally {
    Util.closeQuietly(reader);
  }
}
```

下一步我来看看看是怎么读取日子文件信息保存进入到缓存中(LinkedHashMap<String, Entry> lruEntries)

```java
private void readJournalLine(String line) throws IOException {
    // 读取每一行数据 line
  int firstSpace = line.indexOf(' ');
  if (firstSpace == -1) {
    throw new IOException("unexpected journal line: " + line);
  }

  int keyBegin = firstSpace + 1;
  int secondSpace = line.indexOf(' ', keyBegin);
  final String key;
  if (secondSpace == -1) {
    key = line.substring(keyBegin);
      // 如果当前数据是以Remove 开头的，就移除当前缓存中的对应key数据
    if (firstSpace == REMOVE.length() && line.startsWith(REMOVE)) {
      lruEntries.remove(key);
      return;
    }
  } else {
    key = line.substring(keyBegin, secondSpace);
  }
    // 获取当前缓存中是否有对应key数据
  Entry entry = lruEntries.get(key);
    // 如果没有创建对应Key存储对象
  if (entry == null) {
    entry = new Entry(key);
    lruEntries.put(key, entry);
  }
// 如果当前数据是以 CLEAN 开头的，就填充对象 entry 保存到缓存中，并且清空当前对象的正在读写的缓存对象
  if (secondSpace != -1 && firstSpace == CLEAN.length() && line.startsWith(CLEAN)) {
    String[] parts = line.substring(secondSpace + 1).split(" ");
    entry.readable = true;
    entry.currentEditor = null;
    entry.setLengths(parts);
  } else if (secondSpace == -1 && firstSpace == DIRTY.length() && line.startsWith(DIRTY)) {
      // 如果当前数据是以 DIRTY 开头的，说明文件正在读写，将读写对象放入到缓存中
    entry.currentEditor = new Editor(entry);
  } else if (secondSpace == -1 && firstSpace == READ.length() && line.startsWith(READ)) {
    // This work was already done by calling lruEntries.get().
  } else {
    throw new IOException("unexpected journal line: " + line);
  }
}
```

继续来看初始化方法 ，日志文件信息都读取到缓存中，下一步来删除超过缓存限制的文件信息

```java
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
    …………
        // 计算日志文件已经被缓存文件大小，超出缓存大小，删除缓存
      cache.processJournal();
    …………
  return cache;
}
```

```java
/**
 * Computes the initial size and collects garbage as a part of opening the
 * cache. Dirty entries are assumed to be inconsistent and will be deleted.
 */
private void processJournal() throws IOException {
    // 如果缓存日子文件存在就直接删除缓存日志文件
  deleteIfExists(journalFileTmp);
  for (Iterator<Entry> i = lruEntries.values().iterator(); i.hasNext(); ) {
    Entry entry = i.next();
    if (entry.currentEditor == null) {
      for (int t = 0; t < valueCount; t++) {
        size += entry.lengths[t];
      }
    } else {
        // 这里说明一下 DIRTY CLEAN 是成对出现了，如果只是出现了一个说明是存储不是完整的文件，就是脏数据，这里直接删除清空
        // 如果当前编辑对象不为空，选是置空当前正在编辑对象
      entry.currentEditor = null;
      for (int t = 0; t < valueCount; t++) {
// 直接删除当前正在编辑的缓存文件
        deleteIfExists(entry.getCleanFile(t));
          // 删除当前正在编辑的临时文件
        deleteIfExists(entry.getDirtyFile(t));
      }
      i.remove();
    }
  }
}
```

写缓存数据

简单写入用例

```java
DiskLruCache diskLruCache = DiskLruCache.open(cacheDir, 1, 1, 1024 * 1024 * 10);
DiskLruCache.Editor editor = diskLruCache.edit(key);
// index与valueCount对应，分别为0,1,2...valueCount-1
editor.newOutputStream(0).write(value.getBytes()); 
editor.commit();
diskLruCache.close();
```

  创建方法方法edit

```java
/**
 * Returns an editor for the entry named {@code key}, or null if another
 * edit is in progress.
 */
public Editor edit(String key) throws IOException {
  return edit(key, ANY_SEQUENCE_NUMBER);
}

private synchronized Editor edit(String key, long expectedSequenceNumber) throws IOException {
    // 验证当前日志编写对象是否存在
  checkNotClosed();
    // 校验当前key
  validateKey(key);
    // 获取当前缓存对象
  Entry entry = lruEntries.get(key);
  if (expectedSequenceNumber != ANY_SEQUENCE_NUMBER && (entry == null
      || entry.sequenceNumber != expectedSequenceNumber)) {
    return null; // Snapshot is stale.
  }
  if (entry == null) {
      //  如果缓存对象为空就创建
    entry = new Entry(key);
    lruEntries.put(key, entry);
  } else if (entry.currentEditor != null) {
      // 如果已经存在编辑对象就直接返回
    return null; // Another edit is in progress.
  }
    // 创建一个编辑对象
  Editor editor = new Editor(entry);
  entry.currentEditor = editor;

  // Flush the journal before creating files to prevent file leaks.
    // 编辑一行 dirty 开头的日志文件
  journalWriter.write(DIRTY + ' ' + key + '\n');
  journalWriter.flush();
  return editor;
}
```

随后通过Editor新建一个输出流，该方法返回一个没有buffer的输出流，参数的index表示该key的第几个缓存文件。如果该输出流在写入时发生错误，这次编辑会在commit()方法被调用的时候终止。该方法返回的输出流是FaultHidingOutputStream，该输出流不抛出IO异常，但是通过标志位标记本次IO操作出错。

```java
/**
 * Returns a new unbuffered output stream to write the value at
 * {@code index}. If the underlying output stream encounters errors
 * when writing to the filesystem, this edit will be aborted when
 * {@link #commit} is called. The returned output stream does not throw
 * IOExceptions.
 */
public OutputStream newOutputStream(int index) throws IOException {
  synchronized (DiskLruCache.this) {
    if (entry.currentEditor != this) {
      throw new IllegalStateException();
    }
    if (!entry.readable) {
      written[index] = true;
    }
    File dirtyFile = entry.getDirtyFile(index);
    FileOutputStream outputStream;
    try {
      outputStream = new FileOutputStream(dirtyFile);
    } catch (FileNotFoundException e) {
      // Attempt to recreate the cache directory.
      directory.mkdirs();
      try {
        outputStream = new FileOutputStream(dirtyFile);
      } catch (FileNotFoundException e2) {
        // We are unable to recover. Silently eat the writes.
        return NULL_OUTPUT_STREAM;
      }
    }
    return new FaultHidingOutputStream(outputStream);
  }
}
```

FaultHidingOutputStream 自己捕获了异常，如果失败了 haserrors = true

```java
private class FaultHidingOutputStream extends FilterOutputStream {
  private FaultHidingOutputStream(OutputStream out) {
    super(out);
  }

  @Override public void write(int oneByte) {
    try {
      out.write(oneByte);
    } catch (IOException e) {
      hasErrors = true;
    }
  }

  @Override public void write(byte[] buffer, int offset, int length) {
    try {
      out.write(buffer, offset, length);
    } catch (IOException e) {
      hasErrors = true;
    }
  }

  @Override public void close() {
    try {
      out.close();
    } catch (IOException e) {
      hasErrors = true;
    }
  }

  @Override public void flush() {
    try {
      out.flush();
    } catch (IOException e) {
      hasErrors = true;
    }
  }
}
```

数据流写入成功后调用`Editor.commit()`告诉editor这次的工作结束了，失败后调用`Editor.abort()`方法

```java
/**
 * Commits this edit so it is visible to readers.  This releases the
 * edit lock so another edit may be started on the same key.
 */
public void commit() throws IOException {
  if (hasErrors) {
    completeEdit(this, false);
    remove(entry.key); // The previous entry is stale.
  } else {
    completeEdit(this, true);
  }
  committed = true;
}
```

流成功后，通知完成编辑

```java
private synchronized void completeEdit(Editor editor, boolean success) throws IOException {
  Entry entry = editor.entry;
  if (entry.currentEditor != editor) {
    throw new IllegalStateException();
  }

  // If this edit is creating the entry for the first time, every index must have a value.
  if (success && !entry.readable) {
    for (int i = 0; i < valueCount; i++) {
      if (!editor.written[i]) {
        editor.abort();
        throw new IllegalStateException("Newly created entry didn't create value for index " + i);
      }
      if (!entry.getDirtyFile(i).exists()) {
        editor.abort();
        return;
      }
    }
  }

  for (int i = 0; i < valueCount; i++) {
      // 获取临时文件
    File dirty = entry.getDirtyFile(i);
    if (success) {
        // 如果流写入成功了 ，如果临时文件存在，就重命名为正式缓存文件
      if (dirty.exists()) {
        File clean = entry.getCleanFile(i);
        dirty.renameTo(clean);
        long oldLength = entry.lengths[i];
        long newLength = clean.length();
        entry.lengths[i] = newLength;
           //重新计算缓存的大小
        size = size - oldLength + newLength;
      }
    } else {
        // 如果流写入失败，删除临时文件
      deleteIfExists(dirty);
    }
  }
//操作次数加一
  redundantOpCount++;
    //将实例的当前编辑器置空
  entry.currentEditor = null;
  if (entry.readable | success) {
      //将实例设置为可读的
    entry.readable = true;
       //向journal文件写入一行CLEAN开头的字符
    journalWriter.write(CLEAN + ' ' + entry.key + entry.getLengths() + '\n');
    if (success) {
      entry.sequenceNumber = nextSequenceNumber++;
    }
  } else {
         //如果不成功，就从集合中删除掉这个缓存
    lruEntries.remove(entry.key);
       //向journal文件写入一行REMOVE开头的字符
    journalWriter.write(REMOVE + ' ' + entry.key + '\n');
  }
  journalWriter.flush();
 //如果缓存总大小已经超过了设定的最大缓存大小或者操作次数超过了2000次，
    // 就开一个线程将集合中的数据删除到小于最大缓存大小为止并重新写journal文件
  if (size > maxSize || journalRebuildRequired()) {
    executorService.submit(cleanupCallable);
  }
}
```

读取方法 get

```java
DiskLruCache diskLruCache = DiskLruCache.open(directory, 1, 1, 1024 * 1024 * 10);
String value = diskLruCache.get(key).getString(0);
diskLruCache.close();
```

```java
/**
 * Returns a snapshot of the entry named {@code key}, or null if it doesn't
 * exist is not currently readable. If a value is returned, it is moved to
 * the head of the LRU queue.
 */
public synchronized Snapshot get(String key) throws IOException {
  checkNotClosed();
  validateKey(key);
  Entry entry = lruEntries.get(key);
  if (entry == null) {
    return null;
  }

  if (!entry.readable) {
    return null;
  }

  // Open all streams eagerly to guarantee that we see a single published
  // snapshot. If we opened streams lazily then the streams could come
  // from different edits.
  InputStream[] ins = new InputStream[valueCount];
  try {
    for (int i = 0; i < valueCount; i++) {
      ins[i] = new FileInputStream(entry.getCleanFile(i));
    }
  } catch (FileNotFoundException e) {
    // A file must have been deleted manually!
    for (int i = 0; i < valueCount; i++) {
      if (ins[i] != null) {
        Util.closeQuietly(ins[i]);
      } else {
        break;
      }
    }
    return null;
  }

  redundantOpCount++;
  journalWriter.append(READ + ' ' + key + '\n');
  if (journalRebuildRequired()) {
    executorService.submit(cleanupCallable);
  }

  return new Snapshot(key, entry.sequenceNumber, ins, entry.lengths);
}
```

通过key获取到了一个snapshot对象，并向这个对象里面传入了一个输入流数组和key。输入流数组之所以是数组，自然是由于我们之前讲到的一个key可以对应多个value,那么每个value对应一个输入流，所有就有了这个对应每一个缓存的输入流数组，会获取到所有缓存的输入流对象 。

```java
/** A snapshot of the values for an entry. */
public final class Snapshot implements Closeable {
 …………

  /** Returns the unbuffered stream with the value for {@code index}. */
  public InputStream getInputStream(int index) {
    return ins[index];
  }

  /** Returns the string value for {@code index}. */
  public String getString(int index) throws IOException {
    return inputStreamToString(getInputStream(index));
  }
…………
    public void close() {
      for (InputStream in : ins) {
        Util.closeQuietly(in);
      }
    }
}
```

 可以通过getInputStream（index）来获取到对应的输入流，有了输入流自然就能读取我们要找的数据了。读取完毕后可以调用close()方法，关闭掉所有的输入流。

删除缓存数据

```java
/**
 * Drops the entry for {@code key} if it exists and can be removed. Entries
 * actively being edited cannot be removed.
 *
 * @return true if an entry was removed.
 */
public synchronized boolean remove(String key) throws IOException {
  checkNotClosed();
  validateKey(key);
  Entry entry = lruEntries.get(key);
  if (entry == null || entry.currentEditor != null) {
    return false;
  }

  for (int i = 0; i < valueCount; i++) {
    File file = entry.getCleanFile(i);
    if (file.exists() && !file.delete()) {
      throw new IOException("failed to delete " + file);
    }
    size -= entry.lengths[i];
    entry.lengths[i] = 0;
  }

  redundantOpCount++;
  journalWriter.append(REMOVE + ' ' + key + '\n');
  lruEntries.remove(key);

  if (journalRebuildRequired()) {
    executorService.submit(cleanupCallable);
  }

  return true;
}
```
