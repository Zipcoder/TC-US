## Input/Output streams, readers, writers

-
-
### Input/output streams

- Input streams -- A source to read bytes
- Output streams -- A destination to write bytes

-
#### I/O Streams come from multiple sources

- Come from multiple sources (eg: files, network, memory )
- Not related to the collection Streams API

-
#### I/O Streams vs Readers and writers

- I/O Streams consume and produce (read and write) bytes
- Readers and Writers consume and produce character sequences

-
#### Creating Input Streams

- `Files.newInputStream(path)`
- `new URL(urlstring).openStream()`
- `new ByteArrayInputStream(byteArray)`

Note: lab idea: read html from a url and extract the comments. Expand by finding
content such as script/noscript/meta tags

-
#### Creating Output Streams

- `Files.newOutputStream(path)`
- `URLConnection.getOutputStream()`
- `new ByteArrayOutputStream()`
  - can be converted to `ByteArray` after writing


-
-
#### Know your encoding

Unicode (UTF-8, UTF-16) are common. Many methods have optional encoding and
 default to system encoding.

UTF-16 files start with a Byte-Order-Mark (`BOM`) to indicate if they are
 big-endian or little-endian (two ways of structuring bytes in a file)

-
-
## Paths, Files, Directories



-
### Filesystem Utility Classes

- `Files`
- `Paths`

-
### Paths

A **Path** object represents a hierarchical path on the storage system to a file or directory. In NIO.2, **Path** is a direct replacement for the legacy java.io.**File** class, therefore contains many of the same properties. 

- Path objects represent a route to a file, whether it exists or not. 
- Use them to
 access, create, modify, or delete files and directories.

-

### Paths

Since **Path** is an interface, we use the Factory class **Paths** obtain it.
-
### Creating Paths: relative or absolute paths



```java
Path path1 = Paths.get("someplace/somepic.jpg");
Path path2 = Paths.get("c:\\someplace\\ThisStuff\\myrecords.csv"); 
Path path3 = Paths.get("/users/somdir");

```
-


### Creating Paths: relative or absolute paths

```java

Path path1 = Paths.get("someplace","somepic.jpg");
Path path2 = Paths.get("c:","someplace","ThisStuff","myrecords.csv"); 

Path userDir = Paths.get("/", "users", "username");
Path javaFile = Paths.get("labs", "lab1", "src", "main", "java", "App.java");
//roundabout
Path roundabout = Paths.get("home", ".", "labs", "..", "documents", "..", ".", "Downloads");
 
 
```
-
### Path: Accessing the FileSystem Object

The **Path.getPath()** method comes from **FileSystem**'s method **getPath()**. Since **FileSystem** class has a protected constructor, the factory class **FileSystems** is used to obtain an instance of **FileSystem**:
```java
Path path1 = FileSystems.getDefault().getPath("someplace/somepic.jpg");
Path path2 = FileSystems.getDefault().getPath("c:","someplace","ThisStuff","myrecords.csv");
Path path3 = FileSystems.getDefault().getPath("/users/somdir");
```

-
### Path: Accessing the FileSystem Object

You can construct **Path** objects for remote file systems: 
```
FileSystem fileSystem = FileSystems.getFileSystem( new URI("http://demo.mysfilesystem.com/myfiles"));
Path path = fileSystem.getPath("bunnies.csv");

```
also, the file can be retrieved from the path: 

```
File file = path.toFile();
```

-


#### Remember: 

Path object is **not a file**, it's a representation of a location within the file system. Therefore, most operations available in the Path and Paths classes can be accomplished whether or not referenced file exists. (Hence, the exceptions)


-
### Resolving or normalizing Paths

- `userDir.resolve(javaFile)` resolves to `/users/username/labs/lab1/src/main/java/App.java`
- `roundabout.normalize()` returns `Paths.get("home", "Downloads")`

-
### Directory operations

- `Files.createDirectory(path)` -- like `mkdir path`
- `Files.createDirectories(path)` -- like `mkdir -p path`
- `Files.createFile(path)`
- `Files.exists(path)`

-
### File operations


- `Files.copy(fromPath, toPath, options...)`
- `Files.move(fromPath, toPath, options...)`
- `Files.delete(path)`, `Files.deleteIfExists(path)`

-
### File operation options

||||
|-|-|-|
|READ |ATOMIC_MOVE    |TRUNCATE_EXISTING|
|WRITE|COPY_ATTRIBUTES|REPLACE_EXISTING|
||||

```
Files.move(source, target, REPLACE_EXISTING);
```
-
### File operation options

|Enum Value|Usage|
|:-|:-|
|NOFOLLOW_LINKS|-Test file existing<br>-Read file data<br>-Copy file<br>-Move file|
|FOLLOW_LINKS|Traverse a directory treeCopy file|
|COPY_ATTRIBUTES|Copy file|
|REPLACE_EXISTING|Move file|
|ATOMIC_MOVE|Move file|




-
### Viewing directory contents

List contents of `path` with `Files.list(path)`.  
List contents of `path` and its subdirectories with `Files.walk(path)`

Both methods return a `Stream<Path>` of the entries found (Uses a depth-first
 traversal order)

-
-
## Zip File Systems

ZIP files are just self-contained filesystems. Access them with the
 `FileSystems` and `FileSystem` classes

-
### Read paths from a ZIP file

```Java
FileSystem fs = FileSystems.newFileSystem(Paths.get(zipname), null);
Stream<Path> entries = Files.walk(fs.getPath("/"));
```

Note: second argument is an optional classloader for locating providers

-
#### Writing to ZIP archives

```Java
Path zipPath = Paths.get("archive.zip");
URI uri = new URI("jar", zipPath.toUri().toString(), null);
Map<String, String> env = Collections.singletonMap("create", "true");

try (FileSystem fs = FileSystems.newFileSystem(uri, env);){
  Files.copy(fromPath, fs.getPath("/").resolve(toPath));
}
```

-
-
## URL Connections

-
### Get a URLConnection

Call `openConnection` on a `URL` object eg:

`URLConnection conn = url.openConnection();`

-
### Set request properties

`connection.setRequestProperty("Accept-Charset", "UTF-8, ISO-8859-1, SMOKE-SIGNAL")`

-
### Sending data to a URLConnection

Call `connection.setDoOutput(true)` and then write to `connection.getOutputStream()`

-
### Caveats

- URLConnection class automatically sets `application/x-www-form-urlencoded` content type.
- name/value pairs in urls must be manually encoded using `URLEncoder.encode()`

-
-
## Serialization

- Serialization is a mechanism of converting the state of an object into a byte stream.
- Deserialization is the reverse process where the byte stream is used to recreate the actual Java object in memory.
- This mechanism is used to persist the object.
- Classes must implement `Serializable` to be serialized
- Serialization is safe if all instance variables are primitives, enums, or other Serializable objects
- Collections are Serializable if their elements are.

Note: Shared references to a serialized object must match after deserialization

-
### transient modifier

- marks instance variables that shouldn't be serialized
- Usually used on derived values or non-`Serializable` values

-
#### transient example

```Java
public class Student implements Serializable{
  private List<TestScore> testScores;
  // calculated from testScores
  private transient GradeStats gradeStats;

  public GradeStats getStats(){
    if(gradeStats == null){
      gradeStats = new GradeStats(testScores);
    }
    return GradeStats;
  }
}
```

-
### Customizing serialization behavior

- Override default serialization behavior with:
  - `readObject(ObjectInputStream in)` and
  - `writeObject(ObjectOutputStream out)`
- prevents automatic serialization of instance variables.
- No need to manage superclass instance variables, only `this`

-
### Full Signatures

general interface for serialization

```Java
// from java.io.ObjectOutputStream:
public final void writeObject(Object obj)
                       throws IOException

// from java.io.ObjectInputStream
public final Object readObject()
                  throws IOException,  ClassNotFoundException
```

-
### `readResolve()` and `writeReplace()`

- very rare, but allows you to serialize a proxy object (returned by `writeReplace()`) instead of the actual object
- (for highly controlled initialization -- constructors are not normally called during deserialization).
- Proxy object must construct and return the desired original object in the `readResolve()` method.
- Useful for database-managed objects and was used for singletons prior to `enums`
