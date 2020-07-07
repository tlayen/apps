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
var apps = db.query('select from (select dslPackage, last(versions) as version from App) where version is not null');
for (var i = 0; i < apps.length; i++) {
  var app = apps[i];
  var dslPackage = app.field('dslPackage');
  var version = app.field('version');
  print(dslPackage);
  print(version);
  var dirs = dslPackage.split('.');
  var dir = dslDir;
  for (var j = 0; j < dirs.length; j++) {
    var child = dirs[j];
    dir = dir.resolve(child);
  }
  Files.createDirectories(dir);
  var files = version.field('files');
  for (var j = 0; j < files.length; j++) {
    var file = files[j];
    var f = dir.resolve(file.field('name') + '.erp');
    Files.write(f, file.field('content').getBytes());
  }
}
end
```

[coursier]: https://get-coursier.io/
[sdkman!]: https://sdkman.io/
