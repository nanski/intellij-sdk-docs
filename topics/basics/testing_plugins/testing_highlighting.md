[//]: # (title: Testing Highlighting)

<!-- Copyright 2000-2022 JetBrains s.r.o. and other contributors. Use of this source code is governed by the Apache 2.0 license that can be found in the LICENSE file. -->

When writing plugin tests, a common task is testing various kinds of highlighting (inspections, annotators, parser error highlighting, etc.).
The IntelliJ Platform provides a dedicated utility and markup format for this task.

To test the highlighting for the file currently loaded into the in-memory editor, you invoke the `checkHighlighting()` method.
The parameters to the method specify which severities should be taken into account when comparing the results with the expected results: errors are always taken into account, whereas warnings, weak warnings, and infos are optional.
To ignore verifying additional highlighting, set parameter `ignoreExtraHighlighting` to `true`.
Alternatively, you can use the `testHighlighting()` method, which loads a <path>testdata</path> file into the in-memory editor and highlights it as a single operation.

## Inspections

If you need to test inspections (rather than generic highlighting provided by a highlighting lexer or annotator), you need to enable inspections that you're testing.
This is done by calling `CodeInsightTestFixture.enableInspections()` in the setup method of your test or directly in a test method, before the call to `checkHighlighting()`.

The expected results of the highlighting are specified directly in the source file.
The platform supports an extensive XML-like markup language for this.
In its simplest form, the markup looks like this:

```xml
<warning descr="expected warning message">code to be highlighted</warning>
```

Or, as a more specific example:

```xml
public int <warning descr="The compareTo() method does not reference 'foo' which is referenced from equals(); inconsistency may result">compareTo</warning>(Simple other) {
  return 0;
}
```

The tag name specifies the severity of the expected highlighting.
The following severities are supported:

* `<error>`
* `<warning>`
* `<weak_warning>`
* `<info>`
* `<inject>` for an injected fragment
* `<symbolName>` for a marker that highlights an identifier according to its type
* any custom severity can be referenced by its name

The tag can also have the following optional attributes:

* `descr` expected (hardcoded) message associated with the highlighter (if not specified, any text will match; if the message contains a quotation mark, it can be escaped by putting two backslash characters before it)
* `bundleMsg` expected message from message bundle in format `[bundleName#] bundleKey [|argument]...`
* `tooltip` expected tooltip message
* `textAttributesKey` expected [`TextAttributesKey`](upsource:///platform/core-api/src/com/intellij/openapi/editor/colors/TextAttributesKey.java) referenced by its `externalName`
* `foregroundColor`, `backgroundColor`, `effectColor` expected colors for the highlighting
* `effectType` expected effect type for the highlighting (see [`EffectType`](upsource:///platform/core-api/src/com/intellij/openapi/editor/markup/EffectType.java))
* `fontType` expected font style for the highlighting (`0` - normal, `1` - bold, `2` - italic, `3` - bold italic)

*Nested* tags are **supported**:
```xml
<warning>warning_highlight<info>warning_and_info_highlight</info>warning_highlight</warning>
```

*Overlapping* tags (annotations) are currently **not supported** in the test framework (but displayed correctly in the editor, albeit this is not an officially supported scenario):
```xml
<warning>warning_highlight<info>warning-and_info_highlight</warning>info_highlight</info>
```
