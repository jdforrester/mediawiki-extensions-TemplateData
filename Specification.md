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

The key words "MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## 2 Terminology

### 2.1 Template

A page on a MediaWiki website that can be transcluded into other pages. Refer to [mediawiki.org](https://www.mediawiki.org/wiki/Help:Templates) for documentation about what templates are capable of and how to create or use them.

### 2.2 Parameter

A key–value pair that may be associated with the transclusion of a template in order to alter the behaviour of a template (and by extend, the content it will return as part of the transclusion).

### 2.3 Author

A program that creates and modifies TemplateData blobs, and distributes them to TemplateData consumer programs.

### 2.4 Consumer

A program interpreting TemplateData blobs supplied to it by a TemplateData author.

### 2.5 User

A person using a TemplateData consumer application.

## 3 Structures

The TemplateData blob MUST have exactly one top-most structure of key–value pairs (hereafter referred to as "objects" having "properties") that follow the specification for the "Root" structure, as specified in section 3.1 below.

All of these objects MUST share the following requirements:

1. They MUST only have properties described in this specification. Additional properties not specified are not permitted. For properties and structures marked as "Required", Authors MUST ensure that they are present.

2. They MUST not have multiple properties with the same key.

Authors MUST enforce all aspects of the specification when creating, modifying and distributing TemplateData blobs.

Requirements for non-Required properties only apply if the property is present.

### 3.1 Root

#### 3.1.1 `description`
* Value: `null` or `InterfaceText`

#### 3.1.2 `params`
* Required
* Value: `Object`

Describes each of the template's parameters. 

Authors MUST ensure that the `params` object maps parameter names to `Param` objects.

#### 3.1.3 `paramOrder`
* Value: `Array`

The logical order of the parameters 

Authors MUST ensure the `paramOrder` array contains all parameter keys exactly once.

Consumers SHOULD display the parameters in this order. 

#### 3.1.4 `sets`
* Required
* Value: `Array`

List of groups of parameters that can be used together.

Authors MUST ensure that the `sets` object maps to `Set` objects. Authors MAY include a parameter in multiple `Set` objects. Authors are NOT REQUIRED to reference each parameter in any of the `Set` objects.

Consumer MAY encourage users to add, edit or remove parameters in the groups specified in this array.

### 3.2 Param
* Value: `Object`

#### 3.2.1 `label`
* Value: `null` or `InterfaceText`
* Default: Key of this `Param` object as referenced from `Root.params`

Label of this parameter.

Authors are RECOMMENDED to provide unique labels for each parameter.

Consumers SHOULD use this when referring to a parameter in the interface for users. 

#### 3.2.2 `description`
* Value: `null` or `InterfaceText`

Explanatory text describing the purpose of this parameter. For example, a template parameter `name` might have as it description _"The name of the author of the book, to positively identify and attribute the claim. The name should be given as it would appear in the author's native language, script and culture."_.

Authors are RECOMMENDED to include this property for each parameter. Authors MAY use this value to include hints as to how to the format the parameter value.

Consumers SHOULD provide users with this description for clarity when selecting and altering parameter values.

#### 3.2.3 `required`
* Value: `boolean`
* Default: `false`

Required status of this parameter. Whether the template being described requires this parameter to have an explicit value in order to function properly. For example, a template used to render a hyperlink to a book viewer might mark a parameter of "ISBN" as required, whereas it might consider parameters such as "publication date" to not be required. 

Authors SHOULD only set this flag if the template will not function correctly without this parameter being set.

Consumers SHOULD encourage users to fill in these parameters, and MAY prevent users from transcluding a template when a required parameter has not been set.

#### 3.2.4 `suggested`
* Value: `boolean`
* Default: `false`

Whether the parameter is recommended to be set to an explicit value for the template transclusion to be useful. This status is less strong indicator of inclusion than `required`. For example, in a book template the author's name might be `suggested`, whereas the title might be `required`.

Authors MUST NOT use `suggested` for parameters without which the template will not function correctly.

Consumers MAY encourage users to fill in a suggested parameter, but SHOULD NOT prevent users transcluding a template when a suggested parameter has not been set.

#### 3.2.5 `deprecated`
* Value: `boolean` or `string`
* Default: `false`

Whether this parameter is discouraged from usage. Set to `false` to indicate it is not deprecated.

Authors are RECOMMENDED to provide a `string` of explanatory text describing why the parameter is deprecated (and what parameter or parameters the user should use instead).

#### 3.2.6 `aliases`
* Value: `Array`

An array of alternative names for the parameter. 

Authors MUST NOT provide a `Param` object for any alias. To describe a discouraged parameter with additional information, authors SHOULD describe the parameter as deprecated instead of alias.

Consumers SHOULD display aliases where entered as secondary to the primary name.

#### 3.2.7 `type`
* Value: `Type`

The kind of value the template expects to be associated with this parameter.

Consumers MAY provide type-specific helper tools for users entering values for parameters with this type.

#### 3.2.8 `inherits`
* Value: `string`

Another parameter from which this parameter will inherit its properites, with any local properties overriding the inherited properties.

Authors MUST ensure that they key maps to another Param object in `Root.params`.

#### 3.2.9 `autovalue`
* Value: `string`

A dynamically-generated default value in wikitext, such as today's date or the editing user's name; this will often involve wikitext substitution, such as `{{subst:CURRENTYEAR}}`.

Consumers MUST provide a parameter's autovalue when a template is inserted or edited, and SHOULD indicate this value to the user and give them the opportunity to set it to another value.

#### 3.2.10 `default`
* Value: `string`

A static default value in wikitext (or description thereof) of a parameter as assumed by the template when the parameter is not present in a transclusion.

Consumers SHOULD indicate this default value to the user when inserting or editing a template.

### 3.3 Set
* Value: `Object`

#### 3.3.1 label
* Required
* Value: `InterfaceText`

Label for this set.

Authors SHOULD ensure these are unique to be distinguishable.

#### 3.3.2 params
* Required
* Value: `Array`

A list of one or more `Root.params` keys.

### 3.4 Type
* Value: `string`
* Default: "unknown"

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
