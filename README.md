SquirrelID [![](https://jitpack.io/v/pavog/SquirrelID.svg)](https://jitpack.io/#pavog/SquirrelID)
==========

SquirrelID is a Java library for working with Mojang profiles.

Functions included in the original upstream repository at [EngineHub/SquirrelID](https://github.com/EngineHub/SquirrelID)
* The resolution of UUIDs from player names in bulk.
* "Last seen" UUID -> name cache implementations.
  * Available as SQLite-backed or in-memory.
* Thread-safe implementations.
* Optional parallel fetching of UUIDs from player names.

Functions added in this fork:
* Reverse resolution: Get name for UUID
* Reverse resolution in bulk

Usage
-----

#### Resolver: Name -> UUID

```java
ProfileService resolver = HttpRepositoryService.forMinecraft();
Profile profile = resolver.findByName("Notch"); // May be null
```

Or in bulk:

```java
ImmutableList<Profile> profiles = resolver.findAllByName(Arrays.asList("Notch", "jeb_"));
```

And in parallel:

```java
int nThreads = 2; // Be kind
ProfileService resolver = HttpRepositoryService.forMinecraft();
ParallelProfileService service = new ParallelProfileService(resolver, nThreads);
service.findAllByName(Arrays.asList("Notch", "jeb_"), new Predicate<Profile>() {
    @Override
    public boolean apply(Profile input) {
        // Do something with the input
        return false;
    }
});
```

#### UUID -> Profile Cache

Choose a cache implementation:

```java
File file = new File("cache.sqlite");
SQLiteCache cache = new SQLiteCache(file);
```

Store entries:

```java
UUID uuid = UUID.fromString("069a79f4-44e9-4726-a5be-fca90e38aaf5");
cache.put(new Profile(uuid, "Notch"));
```

Get the last known profile:

```java
Profile profile = cache.getIfPresent(uuid); // May be null
```

Bulk get last known profile:

```java
ImmutableMap<UUID, Profile> results = cache.getAllPresent(Arrays.asList(uuid));
Profile profile = results.get(uuid); // May be null
```

#### Combined Resolver + Cache

Cache all resolved names:

```java
ProfileCache cache = new HashMapCache(); // Memory cache

CacheForwardingService resolver = new CacheForwardingService(
        HttpRepositoryService.forMinecraft(),
        cache);

Profile profile = resolver.findByName("Notch");
Profile cachedProfile = cache.getIfPresent(profile.getUniqueId());
```

As a dependency
---------------

Note: We recommend shading or shadowing in SquirrelID for distribution, **relocating** the `com.sk89q.squirrelid` package to an internal package without your project.

#### Maven (original repository)

```xml
<repositories>
    <repository>
        <id>sk89q-repo</id>
        <url>http://maven.sk89q.com/repo/</url>
    </repository>
</repositories>
```

```xml
<dependencies>
    <dependency>
        <groupId>com.sk89q</groupId>
        <artifactId>squirrelid</artifactId>
        <version>0.2.0</version>
        <scope>compile</scope>
        <type>jar</type>
    </dependency>
</dependencies>
```

#### Maven (this fork)

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```

```xml
<dependencies>
    <dependency>
        <groupId>com.github.pavog</groupId>
        <artifactId>SquirrelID</artifactId>
        <version>0.6.1</version>
        <scope>compile</scope>
        <type>jar</type>
    </dependency>
</dependencies>
```

#### Gradle (original repository)

```groovy
repositories {
    maven { url "http://maven.sk89q.com/repo/" }
}

dependencies {
    compile 'com.sk89q:squirrelid:0.2.0'
}
```

#### Gradle (this fork)

```groovy
repositories {
    maven { url "https://jitpack.io" }
}

dependencies {
    compile 'com.github.pavog:SquirrelID:0.6.1'
}
```

Compiling
---------

Use Maven 3 to compile SquirrelID.

    mvn clean package

Some of the unit tests are actually integration tests and therefore make
contact with Mojang's servers. That means that the tests may take a
non-trivial amount of time to complete and may even fail if the Mojang
profile servers are unreachable. In the future, these tests may be moved
so that this becomes no longer an issue.

You can disable tests with:

    mvn -DskipTests=true clean package

Contributing
------------

SquirrelID is available under the GNU Lesser General Public License.

We happily accept contributions, especially through pull requests on GitHub.

Links
-----

* [Visit our website](http://www.enginehub.org/)
* [Discord Guild](https://discord.gg/YKbmj7V)
* [IRC channel](http://skq.me/irc/irc.esper.net/sk89q/) (#sk89q on irc.esper.net)
