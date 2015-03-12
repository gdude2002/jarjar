In our imaginary project, the Ant "jar" target looks like:

```
<target name="jar" depends="compile">
    <jar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
    </jar>
</target>
```

To use Jar Jar Links, we define a new task named "jarjar", and
substitute it wherever we used the jar task. Because the `JarJarTask`
class extends the normal Ant Jar task, you can use jarjar without any of
its additional features, if you want:

```
<target name="jar" depends="compile">
    <taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask"
        classpath="lib/jarjar.jar"/>
    <jarjar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
    </jarjar>
</target>
```

Just like with the "jar" task, we can include the contents of another
jar file using the "zipfileset" element. But simply including another
projects classes is not good enough to avoid jar hell, since the class
names remain unchanged and can still conflict with other versions.

To rename the classes, `JarJarTask` adds a new "rule" element. The rule
takes a "pattern" attribute, which uses wildcards to match against
class names, and a "result" attribute, which describes how to
transform the matched names.

In this example we include classes from jaxen.jar and add a rule that
changes any class name starting with "org.jaxen" to start with
"org.example.jaxen" instead (in our imaginary world we control the
example.org domain):

```
<target name="jar" depends="compile">
    <taskdef name="jarjar" classname="com.tonicsystems.jarjar.JarJarTask"
        classpath="lib/jarjar.jar"/>
    <jarjar jarfile="dist/example.jar">
        <fileset dir="build/main"/>
        <zipfileset src="lib/jaxen.jar"/>
        <rule pattern="org.jaxen.**" result="org.example.@1"/>
    </jarjar>
</target>
```

The `**` in the pattern means to match against any valid package
substring. To match against a single package component (by excluding
dots (`.`) from the match), a single `*` may be used instead.

The `@1` in the result is a reference to the `**` portion of the
rule. For every `*` or `**` in the rule, a numbered reference is
available for use in the result. References are numbered from left to
right, starting with `@1`, then `@2`, and so on.

The special `@0` reference refers to the entire class name.