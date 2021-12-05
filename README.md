![featured](resources/featured.svg)

# fast_i18n

[![pub package](https://img.shields.io/pub/v/fast_i18n.svg)](https://pub.dev/packages/fast_i18n)
<a href="https://github.com/Solido/awesome-flutter">
   <img alt="Awesome Flutter" src="https://img.shields.io/badge/Awesome-Flutter-blue.svg?longCache=true" />
</a>
![ci](https://github.com/Tienisto/flutter-fast-i18n/actions/workflows/ci.yml/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Lightweight i18n solution. Use JSON or YAML files to create typesafe translations.

## About this library

- 🚀 Minimal setup, create JSON files and get started! No configuration needed.
- 📦 Self-contained, you can remove this library after generation.
- 🐞 Bug-resistant, no typos or missing arguments possible due to compiler errors.
- ⚡ Fast, you get translations using native dart method calls, zero parsing!
- 🔨 Configurable, English is not the default language? Configure it in `build.yaml`!

You can see an example of the generated file [here](https://github.com/Tienisto/flutter-fast-i18n/blob/master/example/lib/i18n/strings.g.dart).

This is how you access the translations:

```dart
final t = Translations.of(context); // optional, there is also a static getter without context

String a = t.mainScreen.title;                         // simple use case
String b = t.game.end.highscore(score: 32.6);          // with parameters
String c = t.items(count: 2);                          // with pluralization (using count)
String d = t.greet(name: 'Tom', context: Gender.male); // with custom context
String e = t.intro.step[4];                            // with index
String f = t.error.type['WARNING'];                    // with dynamic key
String g = t['mainScreen.title'];                      // with fully dynamic key

PageData titlePage = t.onboarding.titlePage;
PageData page = t.onboarding.pages[2];
String h = page.title;                                 // with interfaces
```

## Table of Contents

- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Features](#features)
    - [File Types](#-file-types)
    - [Namespaces](#-namespaces)
    - [String Interpolation](#-string-interpolation)
    - [Lists](#-lists)
    - [Linked Translations](#-linked-translations)
    - [Interfaces](#-interfaces)
    - [Locale Enum](#-locale-enum)
    - [Dependency Injection](#-dependency-injection)
    - [Pluralization](#-pluralization)
    - [Custom Contexts](#-custom-contexts)
    - [Maps](#-maps)
    - [Dynamic Keys](#-dynamic-keys)
    - [Fallback](#-fallback)
    - [Recasing](#-recasing)
    - [Compact CSV](#-compact-csv)
    - [Auto Rebuild](#-auto-rebuild)
- [API](#api)
- [FAQ](#faq)

## Getting Started

**Step 1: Add dependencies**

It is recommended to add `fast_i18n` to `dev_dependencies`.

```yaml
dev_dependencies:
  build_runner: any
  fast_i18n: 5.5.0
```

**Step 2: Create JSON files**

Create these files inside your `lib` directory. Preferably in one common package like `lib/i18n`.
Only files having the `.i18n.json` file extension will be detected.
The part after the underscore `_` is the actual locale (e.g. en_US, en-US, fr).
You **must** provide the default translation file (the file without locale extension).

YAML files are also supported (see [File Types](#-file-types)).

Writing translations into assets folder requires extra configuration (see [FAQ](#faq)).

```json5
// File: strings.i18n.json (mandatory, default, fallback)
{
  "hello": "Hello $name",
  "save": "Save",
  "login": {
    "success": "Logged in successfully",
    "fail": "Logged in failed"
  }
}
```

```json5
// File: strings_de.i18n.json
{
  "hello": "Hallo $name",
  "save": "Speichern",
  "login": {
    "success": "Login erfolgreich",
    "fail": "Login fehlgeschlagen"
  }
}
```

**Step 3: Generate the dart code**

```text
flutter pub run fast_i18n
```
alternative (but slower):
```text
flutter pub run build_runner build --delete-conflicting-outputs
```

**Step 4: Initialize**

a) use device locale
```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized(); // add this
  LocaleSettings.useDeviceLocale(); // and this
  runApp(MyApp());
}
```

b) use specific locale
```dart
@override
void initState() {
  super.initState();
  String storedLocale = loadFromStorage(); // your logic here
  LocaleSettings.setLocaleRaw(storedLocale);
}
```

**Step 4a: Flutter locale**

This is optional but recommended.

Standard flutter controls (e.g. back button's tooltip) will also pick the right locale.

```yaml
# File: pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations: # add this
    sdk: flutter
```

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(TranslationProvider(child: MyApp())); // Wrap your app with TranslationProvider
}
```

```dart
MaterialApp(
  locale: TranslationProvider.of(context).flutterLocale, // use provider
  supportedLocales: LocaleSettings.supportedLocales,
  localizationsDelegates: GlobalMaterialLocalizations.delegates,
  child: YourFirstScreen(),
)
```

**Step 4b: iOS configuration**

```
File: ios/Runner/Info.plist

<key>CFBundleLocalizations</key>
<array>
   <string>en</string>
   <string>de</string>
</array>
```

**Step 5: Use your translations**

```dart
import 'package:my_app/i18n/strings.g.dart'; // import

String a = t.login.success; // get translation
```

## Configuration

This is **optional**. This library works without any configuration (in most cases).

For customization, you can create the `build.yaml` file. Place it in the root directory.

```yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          base_locale: fr
          fallback_strategy: base_locale
          input_directory: lib/i18n
          input_file_pattern: .i18n.json
          output_directory: lib/i18n
          output_file_pattern: .g.dart # deprecated, use output_file_name
          output_file_name: translations.g.dart
          namespaces: false
          translate_var: t
          enum_name: AppLocale
          translation_class_visibility: private
          key_case: snake
          key_map_case: camel
          param_case: pascal
          string_interpolation: double_braces
          flat_map: false
          maps:
            - error.codes
            - category
            - iconNames
          pluralization:
            auto: cardinal
            cardinal:
              - someKey.apple
            ordinal:
              - someKey.place
          contexts:
            gender_context:
              enum:
                - male
                - female
              auto: false
              paths:
                - my.path.to.greet
          interfaces:
            PageData: onboarding.pages.*
            PageData2:
              paths:
                - my.path
                - cool.pages.*
              attributes:
                - String title
                - String? content
```

Key|Type|Usage|Default
---|---|---|---
`base_locale`|`String`|locale of default json|`en`
`fallback_strategy`|`none`, `base_locale`|handle missing translations|`none`
`input_directory`|`String`|path to input directory|`null`
`input_file_pattern`|`String`|input file pattern, must end with .json or .yaml|`.i18n.json`
`output_directory`|`String`|path to output directory|`null`
`output_file_pattern`|`String`|deprecated: output file pattern|`.g.dart`
`output_file_name`|`String`|output file name|`null`
`namespaces`|`Boolean`|split into multiple files|`false`
`translate_var`|`String`|translate variable name|`t`
`enum_name`|`String`|enum name|`AppLocale`
`translation_class_visibility`|`private`, `public`|class visibility|`private`
`key_case`|`camel`, `pascal`, `snake`|transform keys (optional)|`null`
`key_map_case`|`camel`, `pascal`, `snake`|transform keys for maps (optional)|`null`
`param_case`|`camel`, `pascal`, `snake`|transform parameters (optional)|`null`
`string_interpolation`|`dart`, `braces`, `double_braces`|string interpolation mode|`dart`
`flat_map`|`Boolean`|generate flat map|`true`
`maps`|`List<String>`|entries which should be accessed via keys|`[]`
`pluralization`/`auto`|`off`, `cardinal`, `ordinal`|detect plurals automatically|`cardinal`
`pluralization`/`cardinal`|`List<String>`|entries which have cardinals|`[]`
`pluralization`/`ordinal`|`List<String>`|entries which have ordinals|`[]`
`<context>`/`enum`|`List<String>`|context forms|no default
`<context>`/`auto`|`Boolean`|auto detect context|`true`
`<context>`/`paths`|`List<String>`|entries using this context|`[]`
`children of interfaces`|`Pairs of Alias:Path`|alias interfaces|`null`

## Features

### ➤ File Types

Type|Supported|Note
---|---|---
JSON|✔|by default
YAML|✔|update `input_file_pattern`
CSV|✔|update `input_file_pattern`

To change to YAML or CSV, please modify input file pattern

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          input_directory: assets/i18n
          input_file_pattern: .i18n.yaml # must end with .json, .yaml or .csv
```

**JSON Example**
```json
{
  "welcome": {
    "title": "Welcome $name"
  }
}
```

**YAML Example**
```yaml
welcome:
  title: Welcome $name
```

**CSV Example**
```csv
welcome.title,Welcome $name
```

You can also combine multiple locales (see [Compact CSV](#-compact-csv)).

### ➤ Namespaces

You can split the translations into multiple files. Each file represents a namespace.

This feature is disabled by default for single-file usage. You must enable it.

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          namespaces: true # enable this feature
          output_directory: lib/i18n # optional
          output_file_name: translations.g.dart # set file name (mandatory)
```

Let's create two namespaces called `widgets` and `dialogs`.

```text
i18n/
- widgets.i18n.json
- widgets_fr.i18n.json
- dialogs.i18n.json
- dialogs_fr.i18n.json
```

You can also use different folders. Only file name matters!

```text
i18n/
  widgets/
    - widgets.i18n.json
    - widgets_fr.i18n.json
  dialogs/
    - dialogs.i18n.json
    - dialogs_fr.i18n.json
```

Now access the translations:

```dart
// t.<namespace>.<path>
String a = t.widgets.welcomeCard.title;
String b = t.dialogs.logout.title;
```

### ➤ String Interpolation

There are three modes configurable via `string_interpolation` in `build.yaml`.

You can always escape them by adding a backslash, e.g. `\{notAnArgument}`.

Mode|Translation|Call
---|---|---
`dart (default)`|`Hello $name. I am ${height}m.`|`t.myKey(name: 'Tom', height: 1.73)`
`braces`|`Hello {name}`|`t.myKey(name: 'Anna')`
`double_braces`|`Hello {{name}}`|`t.myKey(name: 'Tom')`

### ➤ Lists

Lists are fully supported. No configuration needed. You can also put lists or maps inside lists!

```json
{
  "niceList": [
    "hello",
    "nice",
    [
      "first item in nested list",
      "second item in nested list"
    ],
    {
      "wow": "WOW!",
      "ok": "OK!"
    },
    {
      "a map entry": "access via key",
      "another entry": "access via second key"
    }
  ]
}
```

```dart
String a = t.niceList[1]; // "nice"
String b = t.niceList[2][0]; // "first item in nested list"
String c = t.niceList[3].ok; // "OK!"
String d = t.niceList[4]['a map entry']; // "access via key"
```

### ➤ Linked Translations

You can link one translation to another. Add the prefix `@:` followed by the translation key.

```json
{
  "fields": {
    "name": "my name is {firstName}",
    "age": "I am {age} years old"
  },
  "introduce": "Hello, @:fields.name and @:fields.age"
}
```

```dart
String s = t.introduce(firstName: 'Tom', age: 27); // Hello, my name is Tom and I am 27 years old.
```

### ➤ Interfaces

Often, multiple maps have the same structure. You can create a common super class for that.

```json
{
  "onboarding": {
    "whatsNew": {
      "v2": {
        "title": "New in 2.0",
        "rows": [
          "Add sync"
        ]
      },
      "v3": {
        "title": "New in 3.0",
        "rows": [
          "New game modes",
          "And a lot more!"
        ]
      }
    }
  }
}
```

Here we know that all objects inside `whatsNew` have the same attributes. Let's name these objects `ChangeData`.

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          interfaces:
            ChangeData: onboarding.whatsNew.*
```

```dart
void myFunction(ChangeData changes) {
  String title = changes.title;
  List<String> rows = changes.rows;
}

void main() {
  myFunction(t.onboarding.whatsNew.v2);
  myFunction(t.onboarding.whatsNew.v3);
}
```

You can customize the attributes and use different node selectors. 

Please read the [Wiki](https://github.com/Tienisto/flutter-fast-i18n/wiki/Interfaces).

### ➤ Locale Enum

Typesafety is one of the main advantages of this library. No typos. Enjoy exhausted switch-cases!

```dart
// this enum is generated automatically for you
enum AppLocale {
  en,
  fr,
  zhCn,
}
```

```dart
// use cases
LocaleSettings.setLocale(AppLocale.en); // set locale
List<AppLocale> locales = AppLocale.values; // list all supported locales
Locale locale = AppLocale.en.flutterLocale; // convert to native flutter locale
String tag = AppLocale.en.languageTag; // convert to string tag (e.g. en-US)
final t = AppLocale.en.translations; // get translations of one locale
```

### ➤ Dependency Injection

A follow-up feature of locale enums.

You can use your own dependency injection and inject the required translations!

Please set the translations classes public:

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          translation_class_visibility: public
```

```dart
final englishTranslations = AppLocale.en.translations;
final germanTranslations = AppLocale.de.translations;

final String a = germanTranslations.welcome.title; // access the translation

// using get_it
final getIt = GetIt.instance;
getIt.registerSingleton<StringsEn>(AppLocale.de.translations); // set German
final String b = getIt<StringsEn>().welcome.title; // access the translation
getIt.unregister<StringsEn>();
getIt.registerSingleton<StringsEn>(AppLocale.en.translations); // switch to English
```

### ➤ Pluralization

This library uses the concept defined [here](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html).

Some languages have support out of the box. See [here](https://github.com/Tienisto/flutter-fast-i18n/blob/master/lib/src/model/pluralization_resolvers.dart).

Plurals are detected by the following keywords: `zero`, `one`, `two`, `few`, `many`, `other`.

You can access the `num count` but it is optional.

```json5
// File: strings.i18n.json
{
  "someKey": {
    "apple": {
      "one": "I have $count apple.",
      "other": "I have $count apples."
    }
  }
}
```

```dart
String a = t.someKey.apple(count: 1); // I have 1 apple.
String b = t.someKey.apple(count: 2); // I have 2 apples.
```

Plurals are interpreted as cardinals by default. You can configure or disable it.

```json5
// File: strings.i18n.json
{
  "someKey": {
    "apple": {
      "one": "I have $count apple.",
      "other": "I have $count apples."
    },
    "place": {
      "one": "${count}st place.",
      "two": "${count}nd place.",
      "few": "${count}rd place.",
      "other": "${count}th place."
    }
  }
}
```

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          pluralization:
            auto: off
            cardinal:
              - someKey.apple
            ordinal:
              - someKey.place
```

In case your language is not supported, you must provide a custom pluralization resolver:

```dart
// add this before you call the pluralization strings. Otherwise an exception will be thrown.
// you don't need to specify both
LocaleSettings.setPluralResolver(
  language: 'en',
  cardinalResolver: (num n, {String? zero, String? one, String? two, String? few, String? many, String? other}) {
    if (n == 0)
      return zero ?? other!;
    if (n == 1)
      return one ?? other!;
    return other!;
  },
  ordinalResolver: (num n, {String? zero, String? one, String? two, String? few, String? many, String? other}) {
    if (n % 10 == 1 && n % 100 != 11)
      return one ?? other!;
    if (n % 10 == 2 && n % 100 != 12)
      return two ?? other!;
    if (n % 10 == 3 && n % 100 != 13)
      return few ?? other!;
    return other!;
  },
);
```

### ➤ Custom Contexts

You can utilize custom contexts to differentiate between male and female forms.

```json5
// File: strings.i18n.json
{
  "greet": {
    "male": "Hello Mr $name",
    "female": "Hello Ms $name"
  }
}
```

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          contexts:
            gender_context:
              enum:
                - male
                - female
            polite_context:
              enum:
                - polite
                - rude
```

```dart
String a = t.greet(name: 'Maria', context: GenderContext.female);
```

Auto detection is on by default. You can disable auto detection. This may speed up build time.

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          contexts:
            gender_context:
              enum:
                - male
                - female
              auto: false # disable auto detection
              paths: # now you must specify paths manually
                - my.path.to.greet
```

In contrast to pluralization, you **must** provide all forms. Collapse it to save space.

```json5
// File: strings.i18n.json
{
  "greet": {
    "male,female": "Hello $name"
  }
}
```

### ➤ Maps

You can access each translation via string keys by defining maps.

Define the maps in your `build.yaml`. Each configuration item represents the translation tree separated by dots.

Keep in mind that all nice features like autocompletion are gone.

```json5
// File: strings.i18n.json
{
  "a": {
    "hello world": "hello"
  },
  "b": {
    "b0": "hey",
    "b1": {
      "hi there": "hi"
    }
  }
}
```

```yaml
# File: build.yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          maps:
            - a
            - b.b1
```

Now you can access the translations via keys:

```dart
String a = t.a['hello world']; // "hello"
String b = t.b.b0; // "hey"
String c = t.b.b1['hi there']; // "hi"
```

### ➤ Dynamic Keys

A more general solution to [Maps](#-maps).

It is supported out of the box. No configuration needed. Please use this sparingly.

```dart
String a = t['myPath.anotherPath'];
String b = t['myPath.anotherPath.3']; // with index for arrays
String c = t['myPath.anotherPath'](name: 'Tom'); // with arguments
```

### ➤ Fallback

By default, you must provide all translations for all locales. Otherwise, you cannot compile it.

In case of rapid development, you can turn off this feature. Missing translations will fallback to base locale.

```yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          base_locale: en
          fallback_strategy: base_locale  # add this
```

```json5
// English
{
  "hello": "Hello",
  "bye": "Bye"
}
```

```json5
// French
{
  "hello": "Salut",
  // "bye" is missing, fallback to English version
}
```

### ➤ Recasing

By default, no transformations will be applied.

You can change that by specifying `key_case`, `key_map_case` or `param_case`.

Possible cases are: `camel`, `snake` and `pascal`.

```json
{
  "must_be_camel_case": "The parameter is in {snakeCase}",
  "my_map": {
    "this_should_be_in_pascal": "hi"
  }
}
```

```yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          key_case: camel
          key_map_case: pascal
          param_case: snake
          maps:
            - myMap
```

```dart
String a = t.mustBeCamelCase(snake_case: 'nice');
String b = t.myMap['ThisShouldBeInPascal'];
```

### ➤ Compact CSV

Normally, you create a new csv file for each locale:
`strings.i18n.csv`, `strings_fr.i18n.csv`, etc.

You can also merge multiple locales into one single csv file! To do this,
you need at least 3 columns. The first row contains the locale names.

```csv
key,en,de-DE
welcome.title,Welcome $name,Willkommen $name
```

### ➤ Auto Rebuild

You can let the library rebuild automatically for you.
The watch function from `build_runner` is **NOT** maintained.

Just run this command:

```sh
flutter pub run fast_i18n watch
```

## API                                                                                   
                                                                                         
When the dart code has been generated, you will see some useful classes and functions    
                                                                                         
`t` - the translate variable for simple translations                                     
                                                                                         
`Translations.of(context)` - translations which reacts to locale changes                 
                                                                                         
`TranslationProvider` - App wrapper, used for `Translations.of(context)`                 
                                                                                         
`LocaleSettings.useDeviceLocale()` - use the locale of the device                        
                                                                                         
`LocaleSettings.setLocale(AppLocale.en)` - change the locale                             
                                                                                         
`LocaleSettings.setLocaleRaw('de')` - change the locale                                  
                                                                                         
`LocaleSettings.currentLocale` - get the current locale                                  
                                                                                         
`LocaleSettings.baseLocale` - get the base locale                                        
                                                                                         
`LocaleSettings.supportedLocalesRaw` - get the supported locales                         
                                                                                         
`LocaleSettings.supportedLocales` - see step 4a                                          
                                                                                         
`LocaleSettings.setPluralResolver` - set pluralization resolver for unsupported languages

## FAQ

**Can I write the json files in the asset folder?**

Yes. Specify `input_directory` and `output_directory` in `build.yaml`.

```yaml
targets:
  $default:
    builders:
      fast_i18n:
        options:
          input_directory: assets/i18n
          output_directory: lib/i18n
```

**Can I skip translations or use them from base locale?**

Yes. Please set `fallback_strategy: base_locale` in `build.yaml`.

Now you can leave out translations in secondary languages. Missing translations will fallback to base locale.

**Why setLocale doesn't work?**

In most cases, you forgot the `setState` call.

A more elegant solution is to use `TranslationProvider(child: MyApp())` and then get your translation variable with `final t = Translations.of(context)`.
It will automatically trigger a rebuild on `setLocale` for all affected widgets.

**My plural resolver is not specified?**

An exception is thrown by `_missingPluralResolver` because you missed to add `LocaleSettings.setPluralResolver` for the specific language.

See [Pluralization](#-pluralization).

**How does plural / context detection work?**

You can let the library detect plurals or contexts.

For plurals, it checks if any json node has `zero`, `one`, `two`, `few`, `many` or `other` as children.

As soon as an unknown item has been detected, then this json node is **not** a pluralization.

```json5
{
  "fake": {
    "one": "One apple",
    "two": "Two apples",
    "three": "Three apples" // unknown key word 'three', 'fake' is not a pluralization
  }
}
```

For contexts, all enum values must exist.

## License

MIT License

Copyright (c) 2020-2021 Tien Do Nam

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
