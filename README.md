# Alizer
[release-svg]: https://img.shields.io/nexus/r/com.redhat.devtools.alizer/alizer?server=https%3A%2F%2Frepository.jboss.org%2Fnexus
[nightly-svg]: https://img.shields.io/nexus/s/com.redhat.devtools.alizer/alizer?server=https%3A%2F%2Frepository.jboss.org%2Fnexus
![Build status](https://github.com/redhat-developer/alizer/actions/workflows/CI.yml/badge.svg)
![Release][release-svg]
![Nightly][nightly-svg]

## Overview

Alizer (which stands for Application Analyzer) is a utilily whose goal is to extract informations about an application source code. Such informations are:

- source languages
- frameworks used inside the application
- tools used to build the application

Also based on these informations, Alizer can also select one devfile (cloud workspace file) from a list of available devfiles 
and/or detect components (the concept of component is taken from Odo and its definition can be read on [odo.dev](https://odo.dev/docs/getting-started/basics/#component)).

Alizer comes in 4 different implementations:

- Java library
- CLI
- NPM package
- Go library

so that it can be integrated easily in other projects.

NOTE: Not all implementations support the same features. Please check the table at [alizer-spec](https://github.com/redhat-developer/alizer/blob/main/docs/public/alizer-spec.md#feature-table) for a detailed overview.

## Usage

### Information

First, get access to the Alizer API through a LanguageRecognizer object which can be built like that:
```java
LanguageRecognizer recognizer = new RecognizerFactory().createLanguageRecognizer();
```

Then, access the project information with the `analyze` method:
```java
List<Language> languages = recognizer.analyze("myproject");
```

The result is an ordered list of information for each language detected in the source tree, with the following informations:
- *name*: the name of the detected language
- *framework*: a list of detected frameworks (Quarkus, Flash,...) used by the application
- *tools*: a list of tools (Maven,...) used by the application
- *weight*: a double value that represents the language weight compared to the others.

NOTE: the sum of all weights can be over 100 because a file may be associated to multiple languages and Alizer may not be able to detect it precisely. E.g. a SQL script could be associated to SPLPL, TSQL, PLSQL languages so Alizer will return all 3 with the same weight.

### Devfile selection

It is also possible to select a devfile from a list of existing devfiles (from a devfile registry or other storage) based on the information that we can now extract from the source tree:
```java
DevfileType devfile = recognizer.selectDevFileFromTypes("myproject", devfiles)
```
where devfiles is a list of objects implementing the `DevfileType` interface which defines the following properties:
- *name*: name of the devfile
- *language*: name of the language associated with the devfile
- *projectType*: type of project associated with the devfile
- *tags*: list of tags associated with the devfile

Please note that the devfile object that is returned by the method is one of the object from the list of devfiles given in input for better matching for the user.

### Component detection

Alizer is also able to detect components. The concept of component is taken from Odo and its definition can be read on [odo.dev](https://odo.dev/docs/getting-started/basics/#component).

The detection of a component is based on only one rule. It is discovered if and only if the main language of the component source is one of those that supports component detection (Java, Python, Javascript, Go, ...)

The result is a list of components where each component consists of:
- *path*: root of the component 
- *languages*: list of languages belonging to the component ordered by their relevance.

```java
ComponentRecognizer recognizer = new RecognizerFactory().createComponentRecognizer();
List<Component> components = recognizer.analyze("myproject");
```

## CLI

Alizer is also available through a CLI for applications that can't use the Java API.
The syntax is the following:

```
alizer-cli-$version-runner analyze [-o json] <path>
alizer-cli-$version-runner devfile [-o json] [-r registry-url] <path>
```

### NPM Package

Alizer also offers a NPM package providing helpers for recognize languages/frameworks in a project. It still doesn't support devfile selection.
The syntax is the following:

```
import * as recognizer from '@redhat-developer/alizer';

.....

const languages = await recognizer.detectLanguages('my_project_folder');

.....

```

### Outputs 

Quarkus project output example

```
[
  { name: 'java', builder: 'maven', frameworks: [ 'quarkus' ] },
  { name: 'gcc machine description' }
]
```

SpringBoot project output example

```
[
  { name: 'java', builder: 'maven', frameworks: [ 'springboot' ] },
  { name: 'plsql' },
  { name: 'plpgsql' },
  { name: 'sqlpl' },
  { name: 'tsql' },
  { name: 'javascript' },
  { name: 'batchfile' },
  { name: 'gcc machine description' }
]
```

Django project output example

```
[
  { name: 'python', frameworks: [ 'django' ] },
  { name: 'gcc machine description' },
  { name: 'shell' }
]
```


Contributing
============
This is an open source project open to anyone. This project welcomes contributions and suggestions!

For information on getting started, refer to the [CONTRIBUTING instructions](CONTRIBUTING.md).


Feedback & Questions
====================
If you discover an issue please file a bug and we will fix it as soon as possible.
* File a bug in [GitHub Issues](https://github.com/redhat-developer/alizer/issues).

License
=======
EPL 2.0, See [LICENSE](LICENSE) for more information.
