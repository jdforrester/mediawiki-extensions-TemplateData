# TemplateData Specification

## Living Standard

<dl>
  <dt>This version</dt>
  <dd><a href="https://git.wikimedia.org/blob/mediawiki%2Fextensions%2FTemplateData/master/Specification.md">https://git.wikimedia.org/blob/mediawiki%2Fextensions%2FTemplateData/master/Specification.md</a></dd>
  <dt>Editors</dt>
  <dd>Timo Tijhof, Trevor Parscal, James D. Forrester</dd>
</dl>

***

## 1 Introduction

This document specifies the structure that TemplateData blobs must follow.

The key words "MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119). For readability, these words do not appear in all uppercase letters in this specification.

## 2 Terminology

### 2.1 Template

A page on a MediaWiki website that can be transcluded into other pages. Refer to [mediawiki.org](https://www.mediawiki.org/wiki/Help:Templates) for documentation about what templates are capable of and how to create or use them.

### 2.2 Parameter

A key–value pair that may be associated with the transclusion of a template in order to alter the behaviour of a template (and by extend, the content it will return as part of the transclusion).

### 2.3 Client

A program interpreting TemplateData blobs.

### 2.4 User

The person using an application that is itself, or features, a TemplateData client.

### 2.5 Author

The person writing or modifying a TemplateData blob.

## 3 Structures

The TemplateData blob must have exactly one top-most structure of key–value pairs (hereafter referred to as "objects" having "properties") that follow the specification for the "Root" structure, as specified in section 3.1 below.

All of these objects share the following requirements:

1. They must only have properties described in this specification. Additional properties not specified are not permitted. And, with the exception of those marked as "Optional", none of the specified properties may be omitted.

2. They must not have multiple properties with the same key.

### 3.1 Root

#### 3.1.1 `description`
* Optional
* Value: `null` or `InterfaceText`

#### 3.1.2 `params`
* Requires
* Value: `Object`

Describes each of the template's parameters. The `params` object maps parameter names to `Param` objects.

#### 3.1.3 `paramOrder`
* Optional
* Value: `Array`

The logical order in which parameters should be displayed. The `paramOrder` array must contain all parameter keys exactly once.

#### 3.1.4 `sets`
* Required
* Value: `Array`

List of groups of parameters that may be used together. Any given parameter may be part of multiple sets. A parameter is not required to be referenced in a set. The `sets` array must contain `Set` objects.

### 3.2 Param
* Value: `Object`

#### 3.2.1 `label`
* Optional
* Value: `null` or `InterfaceText`
* Default: Key of this `Param` object as referenced from `Root.params`

Label of this parameter.

Clients should use this when referring to a parameter in the interface for users. Authors are recommended to provide unique labels for each parameter.

#### 3.2.2 `required`
* Optional
* Value: `boolean`
* Default: `false`

Required status of this parameter.

Whether the template being described requires this parameter to have an explicit value in order to function properly. For example, a template used to render a hyperlink to a book viewer, may require a parameter "ISBN", whereas it may consider parameters such as "publication date" to not be required. Clients should encourage users to fill in these parameters, and may prevent users from transcluding a template when a required parameter has not been set.

#### 3.2.3 `suggested`
* Optional
* Value: `boolean`
* Default: `false`

Whether the parameter is recommended to be set to an explicit value for the template transclusion to be useful. This status is less strong indicator of inclusion than `required`. The template must function correctly without a `suggested` parameter being set. For example, in a book template the author's name might be suggested, whereas the title might be required. Clients may encourage users to fill in a suggested parameter, but should not prevent users transcluding a template when a suggested parameter has not been set.

#### 3.2.4 `description`
* Optional
* Value: `null` or `InterfaceText`

Explanatory text describing the purpose of this parameter. May include hints as to how to the format the parameter value. For example, a template parameter "name" might be _"The name of the author of the book, to positively identify and attribute the claim. The name should be given as it would appear in the author's native language, script and culture."_.

#### 3.2.5 `deprecated`
* Optional
* Value: `boolean` or `string`
* Default: `false`

Specified whether this parameter is discouraged from usage. Set to `false` to indicate it is not deprecated. To indicate the parameter being deprecated authors should set this to `true`, or to a `string` of explanatory text describing why the parameter is deprecated (and what parameter the user should use instead).

#### 3.2.6 `aliases`
* Optional
* Value: `Array`

An alias is an alternative name for the parameter that may be used instead of (not in addition to) the primary name. Aliases must not have a dedicated Param object. To describe a discouraged parameter with additional information, authors should describe the parameter as deprecated instead of alias.

#### 3.2.7 `default`
* Optional
* Value: `string`

The default value (or description thereof) of a parameter as assumed by the template when the parameter is not present in a transclusion.

#### 3.2.8 `type`
* Optional
* Value: `Type`

The kind of value the template expects to be associated with this parameter.

#### 3.2.9 `inherits`
* Optional
* Value: `string`

Key to another Param object in `Root.params`. The current Param object will inherit from that one, with local properties overriding the inherited properties.

#### 3.2.10 `autovalue`
* Optional
* Value: `Type`

A dynamically generated default value such as today's date or the editing user's name. Should generally involve wikitext substitution, such as `{{subst:CURRENTMONTHNAME}}`.

### 3.3 Set
* Value: `Object`

#### 3.3.1 label
* Required
* Value: `InterfaceText`

Label for this set.

#### 3.3.2 params
* Required
* Value: `Array`

A list of one or more `Root.params` keys.

### 3.4 Type
* Value: `string`

One of the following:

- `unknown`
  When no type is specified.
- `string`
  Any textual value.
- `number`
  Any numerical value (without decimal points or thousand separators).
- `boolean`
  A boolean value ('1' for true, '0' for false, '' for unknown).
- `date`
  A date in ISO 8601 format, e.g. "2014-05-09" or "2014-05-09T16:01:12Z".
- `wiki-page-name`
  A valid MediaWiki page name for the current wiki. Doesn't have to exist,
  but if not, should be a valid page name to create.
- `wiki-user-name`
  A valid MediaWiki user name for the current wiki. Doesn't have to exist,
  but if not, should be a valid user name to create. Should not include any
  localised or standard namespace prefix ("Foo" not "User:Foo").
- `wiki-file-name`
  A valid MediaWiki file name for the current wiki. Doesn't have to exist,
  but if not, should be a valid file name to upload. Should not include any
  localised or standard namespace prefix ("Foo" not "File:Foo").
- `content`
  Page content (such as text style, links and images etc.).
- `unbalanced-wikitext`
  Raw wikitext that should not be treated as standalone content because it
  is unbalanced (eg. templates concatenating incomplete wikitext as a bigger
  whole such as `{{echo|before=<u>|after=</u>}}`)
- `line`
  Short text field - use for names, labels, and other short-form fields.

### 3.5 InterfaceText
* Value: `string` or `Object`

A free-form string (no wikitext) in the content-language of the wiki, or,
an object containing those strings keyed by language code.

## 4 Examples

### 4.1 The "Unsigned" template

<pre lang="json">
{
	"description": "Label unsigned comments in a conversation.",
	"params": {
		"user": {
			"label": "User's name",
			"type": "wiki-user-name",
			"required": true,
			"description": "User name of person who forgot to sign their comment.",
			"aliases": ["1"]
		},
		"date": {
			"label": "Date",
			"suggested": true,
			"description": {
				"en": "Timestamp of when the comment was posted, in YYYY-MM-DD format."
			},
			"aliases": ["2"],
			"autovalue": '{{subst:#time:Y-m-d}}'
		},
		"year": {
			"label": "Year",
			"type": "number"
		},
		"month": {
			"label": "Month",
			"inherits": "year"
		},
		"day": {
			"label": "Day",
			"inherits": "year"
		},
		"comment": {
			"required": false
		}
	},
	"sets": [
		{
			"label": "Date",
			"params": ["year", "month", "day"]
		}
	]
}
</pre>

Example transclusions
<pre>
{{unsigned|JohnDoe|2012-10-18}}

{{unsigned|user=JohnDoe|year=2012|month=10|day=18|comment=blabla}}
</pre>
