[![Maven Central Latest](https://img.shields.io/maven-central/v/com.cosium.code/git-code-format-maven-plugin.svg)](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.cosium.code%22%20AND%20a%3A%22git-code-format-maven-plugin%22)
[![Build Status](https://github.com/Cosium/git-code-format-maven-plugin/actions/workflows/ci.yml/badge.svg)](https://github.com/Cosium/git-code-format-maven-plugin/actions/workflows/ci.yml)

# Git Code Format Maven Plugin

A maven plugin that automatically deploys code formatters as `pre-commit` git hook.
On commit, the hook will automatically format staged files.

# Breaking changes between 3.x and 4.x

* `Google Java Format` is not enabled by default anymore. `com.cosium.code:google-java-format` must be added as a dependency to the plugin to keep using it.
* `Google Java Format` options declaration structure has changed. You will need to migrate any eventual existing declaration to the new structure described by [the google-java-format-options chapter](#google-java-format-options) .

# Breaking changes between 2.x and 3.x

* [#64](https://github.com/Cosium/git-code-format-maven-plugin/issues/64) `google-java-format 1.8` [dropped support for java 8](https://github.com/google/google-java-format/releases/tag/google-java-format-1.8).
  The minimum supported runtime version for the plugin is JDK 11. i.e. Maven must run on JDK 11+ while the target project can still be built and run using JDK 8.

# Breaking changes between 1.x and 2.x

* [#37](https://github.com/Cosium/git-code-format-maven-plugin/issues/37) To prevent conflicts with other plugins all keys are now 
prefixed with `gcf`. e.g. `-DglobPattern=**/*` becomes `-Dgcf.globPattern=**/*`
* [#38](https://github.com/Cosium/git-code-format-maven-plugin/issues/38) To avoid infringement to Apache Maven Trademark, 
the plugin was renamed to `git-code-format-maven-plugin`. Its new coordinates are 
`com.cosium.code:git-code-format-maven-plugin`.

`1.x` documentation can be found [here](https://github.com/Cosium/git-code-format-maven-plugin/blob/1.39/README.md)

# Automatic code format and validation activation

Add this to your maven project **root** pom.xml :

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.cosium.code</groupId>
      <artifactId>git-code-format-maven-plugin</artifactId>
      <version>${git-code-format-maven-plugin.version}</version>
      <executions>
        <!-- On commit, format the modified files -->
        <execution>
          <id>install-formatter-hook</id>
          <goals>
            <goal>install-hooks</goal>
          </goals>
        </execution>
        <!-- On Maven verify phase, fail if any file
        (including unmodified) is badly formatted -->
        <execution>
          <id>validate-code-format</id>
          <goals>
            <goal>validate-code-format</goal>
          </goals>
        </execution>
      </executions>
      <dependencies>
        <!-- Enable https://github.com/google/google-java-format -->
        <dependency>
          <groupId>com.cosium.code</groupId>
          <artifactId>google-java-format</artifactId>
          <version>${git-code-format-maven-plugin.version}</version>
        </dependency>
      </dependencies>
    </plugin>
  </plugins>
</build>
```

# Manual code formatting

```console
mvn git-code-format:format-code -Dgcf.globPattern=**/*
```

# Manual code format validation

```console
mvn git-code-format:validate-code-format -Dgcf.globPattern=**/*
```

# Google Java Format

## Google Java Format options

The plugin allows you to tweak Google Java Format options :

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.cosium.code</groupId>
      <artifactId>git-code-format-maven-plugin</artifactId>
      <version>${git-code-format-maven-plugin.version}</version>
      <executions>
        <!-- ... -->
      </executions>
      <dependencies>
        <!-- ... -->
      </dependencies>        
      <configuration>
        <formatterOptions>
          <!-- Use AOSP style instead of Google Style (4-space indentation). -->
          <googleJavaFormat.aosp>false</googleJavaFormat.aosp>
          <!-- Format the javadoc -->
          <googleJavaFormat.formatJavadoc>true</googleJavaFormat.formatJavadoc>
          <!-- Fix import order and remove any unused imports, but do no other formatting. -->
          <googleJavaFormat.fixImportsOnly>false</googleJavaFormat.fixImportsOnly>
          <!-- Do not fix the import order. Unused imports will still be removed. -->
          <googleJavaFormat.skipSortingImports>false</googleJavaFormat.skipSortingImports>
          <!-- Do not remove unused imports. Imports will still be sorted. -->
          <googleJavaFormat.skipRemovingUnusedImports>false</googleJavaFormat.skipRemovingUnusedImports>
        </formatterOptions>
      </configuration>
    </plugin>
  </plugins>
</build>
```

## JDK 16+ peculiarities

Since google-java-format uses JDK internal apis, if you need to run the plugin with JDK 16+, you must pass some additional arguments to the JVM.
Those are described at https://github.com/google/google-java-format/releases/tag/v1.10.0.  

Thanks to https://maven.apache.org/configure.html#mvn-jvm-config-file, you should be able to pass them to `.mvn/jvm.config` as follow:

```
--add-exports jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED --add-exports jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED --add-exports jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED --add-exports jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED --add-exports jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
```

# Custom code formatter

Thanks to its code formatter SPI, this plugin can execute any code formatter.

## How to

Note that you can take inspiration from the `google-java-format` module of this project.

1. Implement `com.cosium.code.format_spi.CodeFormatterFactory`. This interface is provided by `com.cosium.code:git-code-format-maven-plugin-spi`.
2. Add your `com.cosium.code.format_spi.CodeFormatterFactory` implementation canonical name in `META-INF/services/com.cosium.code.format_spi.CodeFormatterFactory`.
3. Pack this in a jar that you declare as a dependency in this plugin declaration.

## Example of usage

Suppose: 
- the chosen `configurationId` (declared by `com.cosium.code.format_spi.CodeFormatterFactory#configurationId()`) is `aqme`
- the formatter dependency is `com.aqme.formatter:formatter:1.0`

A plugin declaration making use of this custom code formatter would look like this:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.cosium.code</groupId>
      <artifactId>git-code-format-maven-plugin</artifactId>
      <version>${git-code-format-maven-plugin.version}</version>
      <executions>
        <!-- ... -->
      </executions>
      <dependencies>
        <dependency>
          <groupId>com.aqme.formatter</groupId>
          <artifactId>formatter</artifactId>
          <version>1.0</version>
        </dependency>
      </dependencies>
      <configuration>
        <formatterOptions>
          <aqme.option1>false</aqme.option1>
          <aqme.option2>foo</aqme.option2>
        </formatterOptions>
      </configuration>
    </plugin>
  </plugins>
</build>
```

# Frequently asked questions

## If I have a multi-module project, do I need to install anything in the sub-projects?
You only need to put the plugin in your *root* project pom.xml. By default all submodules will be handled.

## Do I need to run mvn initialize or is that a stage that happens automatically when I run mvn compile or mvn test?
`initialize` is the first phase of the Maven lifecycle. Any goal that you perform (e.g. `compile` or `test`) will automatically trigger `initialize` and thus trigger the git pre-commit hook installation.

## I'm not noticing anything happening.
If after setting up the plugin in your pom, you just executed a maven goal, the only expected output is a pre-commit hook installed in your `.git/hooks` directory. To trigger the automatic formatting, you have to perform a commit of a modified file.
You can also manually [format](#manual-code-formatting) or [validate](#manual-code-format-validation) any file.

## I'd like to skip code formatting in a child project 
I inherit an enterprise parent pom, which I cannot modify, with formatting plugin specified, and I need to turn off formatting for my group's project.
Either use add a ```<skip>true</skip>``` configuration in the inheriting project or set the ```gcf.skip``` property to true.

# How the hook works

On the `initialize` maven phase, `git-code-format:install-hooks` installs a git `pre-commit` hook that looks like this :
```bash
#!/bin/bash
"./.git/hooks/${project.artifactId}.git-code-format.pre-commit.sh"
```
and `.git/hooks/${project.artifactId}.git-code-format.pre-commit.sh` has the following content:
```bash
#!/bin/bash
set -e
"${env.M2_HOME}/bin/mvn" -f "${project.basedir}/pom.xml" git-code-format:on-pre-commit
```

On `pre-commit` git phase, the hook triggers the `git-code-format:on-pre-commit` which formats the code of the modified files.

# Advanced pre-commit pipeline hook
If you wish to modify the output of the pre-commit hook, you can set the `preCommitHookPipeline` configuration.

To completely ignore the hook output, you could use the following configuration:
```xml
      <configuration>
        <preCommitHookPipeline>&gt;/dev/null</preCommitHookPipeline>
      </configuration>
```

To display error lines from the maven output and fail build with any errors, you could use the following configuration:
```xml
      <configuration>
        <preCommitHookPipeline>| grep -F '[ERROR]' || exit 0 &amp;&amp; exit 1</preCommitHookPipeline>
      </configuration>
```
