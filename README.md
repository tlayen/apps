# Tlayen Apps

This contains a copy of all the source code for all the Tlayen apps.

## Updating

The source code can be updated using the OrientDB console.

First install [SDKMAN!] and [Coursier].

```shell script
> sdk install java 8.0.252.hs-adpt
> sdk use java 8.0.252.hs-adpt
> cs launch com.orientechnologies:orientdb-tools:2.2.31
orientdb> connect remote:HOSTNAME/dsldb admin
orientdb {db=dsldb}> js
```

> ℹ️ Replace `HOSTNAME` with the hostname of the platform instance and ensure that you are able to connect to port 2424.

Then paste the following script:

```js
var Paths = Java.type('java.nio.file.Paths');
var Files = Java.type('java.nio.file.Files');
var Opt = Java.type('java.nio.file.StandardOpenOption');
var dslDir = Paths.get('src/main/erp');

// If this fails then delete the directory manually.
// Files.deleteIfExists(dslDir);

var apps = db.query('select from App');
for (var i = 0; i < apps.length; i++) {
  var app = apps[i];
  var id = app.field('@rid');
  var dslPackage = app.field('dslPackage');
  print('App: id = ' + id + ', dslPackage = ' + dslPackage);

  var dir = dslDir.resolve(dslPackage);
  Files.createDirectories(dir);

  var manifest = dir.resolve('App.properties')
  var fields = app.toMap();
  for each (var field in fields.keySet()) {
    if (!field.equals('files') && !field.equals('@class')) {
      var prop = field + ' = ' + fields.get(field) + '\n';
      Files.write(manifest, prop.getBytes(), Opt.APPEND, Opt.CREATE);
    }
  }
}

var sources = db.query('select from AppSource');
for (var i = 0; i < sources.length; i++) {
  var source = sources[i];
  var id = source.field('@rid');
  var dslPackage = source.field('dslPackage');
  var version = source.field('version');
  var status = source.field('status');
  print('AppSource: id = ' + id + ', dslPackage = ' + dslPackage + ', version = ' + version + ', status = ' + status);

  var dir = dslDir.resolve(dslPackage).resolve(version);
  Files.createDirectories(dir);

  var manifest = dir.resolve('AppSource.properties')
  var fields = source.toMap();
  for each (var field in fields.keySet()) {
    if (!field.equals('files') && !field.equals('@class')) {
      var prop = field + ' = ' + fields.get(field) + '\n';
      Files.write(manifest, prop.getBytes(), Opt.APPEND, Opt.CREATE);
    }
  }

  var files = source.field('files');
  for (var j = 0; j < files.length; j++) {
    var file = files[j];
    var name = file.field('name');
    var content = file.field('content');
    // print('File: name = ' + name);
    var f = dir.resolve(name + '.erp');
    Files.write(f, content.getBytes());
  }
}
end
```

## Latest

Having all versions at once makes it difficult to search and so you can use the [latest] branch to view only the latest
version of each app.

To delete all versions other than the latest run the following Scala script:

```scala worksheet
import java.io.FilenameFilter
import java.nio.file.attribute.BasicFileAttributes

import scala.math.Ordering

implicit val versionOrdering = new Ordering[String] {
  override def compare(x: String, y: String) = {
    compareParts(toVersion(x), toVersion(y))
  }

  private def toVersion(a: String): List[Int] = a.split('.').map(_.toInt).toList

  private def compareParts(a: List[Int], b: List[Int]): Int = {
    a match {
      case Nil => b match {
        case Nil => 0
        case 0 :: bTail => compareParts(a, bTail)
        case _ => -1
      }
      case aHead :: aTail => b match {
        case Nil => aHead
        case `aHead` :: bTail => compareParts(aTail, bTail)
        case bHead :: _ => aHead - bHead
      }
    }
  }
}

import java.nio.file._
import java.io._

val versionFilter = new FilenameFilter {
  override def accept(dir: File, name: String) = name.matches("""\d+(\.\d+)*""")
}

def deleteRecursively(path: Path) = {
  Files.walkFileTree(path, new SimpleFileVisitor[Path] {
    override def visitFile(file: Path, attrs: BasicFileAttributes) = {
      Files.delete(file)
      FileVisitResult.CONTINUE
    }

    override def postVisitDirectory(dir: Path, exc: IOException) = {
      Files.delete(dir)
      FileVisitResult.CONTINUE
    }
  })
}

val erpDir = Paths.get("src/main/erp")
erpDir.toFile.listFiles().foreach { dir =>
  dir
    .list(versionFilter)
    .sorted(versionOrdering.reverse)
    .drop(1)
    .foreach(s => deleteRecursively(dir.toPath.resolve(s)))
}
```

[coursier]: https://get-coursier.io/
[sdkman!]: https://sdkman.io/
