# JavaScript Gettext
JavaScript Gettext from http://jsgettext.sourceforge.net

JavaScript Gettext is an open source JavaScript implementation of the GNU Gettext API. It differs from existing javascript implementations in that it will support all current Gettext features (ex. plural and context support), and will also support loading language catalogs from .mo, .po, or preprocessed json files (converter included).

## Name

Javascript Gettext - Javascript implemenation of GNU Gettext API.

## Synopsis

Optimum caching way

```html
<script language="javascript" src="/path/LC_MESSAGES/myDomain.json"></script>
<script language="javascript" src="/path/Gettext.js'></script>
```

```javascript
// assuming myDomain.json defines variable json_locale_data
var params = {
  "domain" : "myDomain",
  "locale_data" : json_locale_data
};
var gt = new Gettext(params);
// create a shortcut if you'd like
function _ (msgid) { return gt.gettext(msgid); }
alert(_("some string"));
// or use fully named method
alert(gt.gettext("some string"));
// change to use a different "domain"
gt.textdomain("anotherDomain");
alert(gt.gettext("some string"));
```

The other way to load the language lookup is a "link" tag Downside is that not all browsers cache XMLHttpRequests the same way, so caching of the language data isn't guarenteed across page loads. Upside is that it's easy to specify multiple files.

```html
<link rel="gettext" href="/path/LC_MESSAGES/myDomain.json" />
<script language="javascript" src="/path/Gettext.js'></script>
```

```javascript
var gt = new Gettext({ "domain" : "myDomain" });
// rest is the same
```

The reson the shortcuts aren't exported by default is because they'd be glued to the single domain you created. So, if you're adding i18n support to some js library, you should use it as so:

```javascript
if (typeof(MyNamespace) == 'undefined') MyNamespace = {};
MyNamespace.MyClass = function () {
  var gtParms = { "domain" : 'MyNamespace_MyClass' };
  this.gt = new Gettext(gtParams);
  return this;
};
MyNamespace.MyClass.prototype._ = function (msgid) {
  return this.gt.gettext(msgid);
};
MyNamespace.MyClass.prototype.something = function () {
  var myString = this._("this will get translated");
};
```

Adding the shortcuts to a global scope is easier. If that's ok in your app, this is certainly easier.

```javascript
var myGettext = new Gettext({ 'domain' : 'myDomain' });
function _ (msgid) {
  return myGettext.gettext(msgid);
}
alert( _("text") );
```

Data structure of the json data

NOTE: if you're loading via the `<script>` tag, you can only load one file, but it can contain multiple domains.

```javascript
var json_locale_data = {
  "MyDomain" : {
    "" : {
      "header_key" : "header value",
      "header_key" : "header value",
    },
    "msgid" : [ "msgid_plural", "msgstr", "msgstr_plural", "msgstr_pluralN" ],
    "msgctxt\004msgid" : [ null, "msgstr" ],
  },
  "AnotherDomain" : {
  },
}
```

## Description

This is a javascript implementation of GNU Gettext, providing internationalization support for javascript. It differs from existing javascript implementations in that it will support all current Gettext features (ex. plural and context support), and will also support loading language catalogs from .mo, .po, or preprocessed json files (converter included).

The locale initialization differs from that of GNU Gettext / POSIX. Rather than setting the category, domain, and paths, and letting the libs find the right file, you must explicitly load the file at some point. The "domain" will still be honored. Future versions may be expanded to include support for set_locale like features.

## Install

To install this module, simply copy the file lib/Gettext.js to a web accessable location, and reference it from your application.

## Configuration

Configure in one of two ways:

1. **Optimal. Load language definition from statically defined json data.**

  ```html
  <script language="javascript" src="/path/locale/domain.json"></script>
  ```

  ```javascript
  // in domain.json
  json_locale_data = {
    "mydomain" : {
      // po header fields
      "" : {
        "plural-forms" : "...",
        "lang" : "en",
      },
      // all the msgid strings and translations
      "msgid" : [ "msgid_plural", "translation", "plural_translation" ],
    },
  };
  // please see the included bin/po2json script for the details on this format
  ```

  This method also allows you to use unsupported file formats, so long as you can parse them into the above format.

2. **Use AJAX to load language file.**

  Use XMLHttpRequest (actually, SJAX - syncronous) to load an external resource.

  Supported external formats are:

  * **Javascript Object Notation (.json)**

    (see bin/po2json) `type=application/json`

  * **Uniforum Portable Object (.po)**

    (see GNU Gettext's xgettext) `type=application/x-po`

  * **Machine Object (compiled .po) (.mo)**

    NOTE: .mo format isn't actually supported just yet, but support is planned.

    (see GNU Gettext's msgfmt) `type=application/x-mo`

## Methods

The following methods are implemented:

```
new Gettext(args)
textdomain  (domain)
gettext     (msgid)
dgettext    (domainname, msgid)
dcgettext   (domainname, msgid, LC_MESSAGES)
ngettext    (msgid, msgid_plural, count)
dngettext   (domainname, msgid, msgid_plural, count)
dcngettext  (domainname, msgid, msgid_plural, count, LC_MESSAGES)
pgettext    (msgctxt, msgid)
dpgettext   (domainname, msgctxt, msgid)
dcpgettext  (domainname, msgctxt, msgid, LC_MESSAGES)
npgettext   (msgctxt, msgid, msgid_plural, count)
dnpgettext  (domainname, msgctxt, msgid, msgid_plural, count)
dcnpgettext (domainname, msgctxt, msgid, msgid_plural, count, LC_MESSAGES)
strargs     (string, args_array)
```

### new Gettext (args)

Several methods of loading locale data are included. You may specify a plugin or alternative method of loading data by passing the data in as the "locale_data" option. For example:

```javascript
var get_locale_data = function () {
  // plugin does whatever to populate locale_data
  return locale_data;
};
var gt = new Gettext( 'domain' : 'messages',
                      'locale_data' : get_locale_data() );
```

The above can also be used if locale data is specified in a statically included `<script>` tag. Just specify the variable name in the call to new. Ex:

```javascript
var gt = new Gettext( 'domain' : 'messages',
                      'locale_data' : json_locale_data_variable );
```

Finally, you may load the locale data by referencing it in a `<link>` tag. Simply exclude the 'locale_data' option, and all `<link rel="gettext" ...>` items will be tried. The `<link>` should be specified as:

```html
<link rel="gettext" type="application/json" href="/path/to/file.json">
<link rel="gettext" type="text/javascript"  href="/path/to/file.json">
<link rel="gettext" type="application/x-po" href="/path/to/file.po">
<link rel="gettext" type="application/x-mo" href="/path/to/file.mo">
```

args:

#### domain

The Gettext domain, not www.whatev.com. It's usually your applications basename. If the .po file was "myapp.po", this would be "myapp".

#### locale_data

Raw locale data (in json structure). If specified, from_link data will be ignored.

### textdomain( domain )

Set domain for future `gettext()` calls

A message domain is a set of translatable msgid messages. Usually, every software package has its own message domain. The domain name is used to determine the message catalog where a translation is looked up; it must be a non-empty string.

The current message domain is used by the gettext, ngettext, pgettext, npgettext functions, and by the dgettext, dcgettext, dngettext, dcngettext, dpgettext, dcpgettext, dnpgettext and dcnpgettext functions when called with a NULL domainname argument.

If domainname is not NULL, the current message domain is set to domainname.

If domainname is undefined, null, or empty string, the function returns the current message domain.

If successful, the textdomain function returns the current message domain, after possibly changing it. (ie. if you set a new domain, the value returned will NOT be the previous domain).

### gettext( MSGID )

Returns the translation for **MSGID**. Example:

```javascript
alert( gt.gettext("Hello World!\n") );
```

If no translation can be found, the unmodified **MSGID** is returned, i. e. the function can *never* fail, and will *never* mess up your original message.

One common mistake is to interpolate a variable into the string like this:

```javascript
var translated = gt.gettext("Hello " + full_name);
```

The interpolation will happen before it's passed to gettext, and it's unlikely you'll have a translation for every "Hello Tom" and "Hello Dick" and "Hellow Harry" that may arise.

Use `strargs()` (see below) to solve this problem:

```javascript
var translated = Gettext.strargs( gt.gettext("Hello %1"), [full_name] );
```

This is especially useful when multiple replacements are needed, as they may not appear in the same order within the translation. As an English to French example:

```
Expected result: "This is the red ball"
English: "This is the %1 %2"
French:  "C'est le %2 %1"
Code: Gettext.strargs( gt.gettext("This is the %1 %2"), ["red", "ball"] );
```

(The example is stupid because neither color nor thing will get translated here ...).

### dgettext( TEXTDOMAIN, MSGID )

Like `gettext()`, but retrieves the message for the specified **TEXTDOMAIN** instead of the default domain. In case you wonder what a textdomain is, see above section on the `textdomain()` call.

### dcgettext( TEXTDOMAIN, MSGID, CATEGORY )

Like `dgettext()` but retrieves the message from the specified **CATEGORY** instead of the default category `LC_MESSAGES`.

NOTE: the categories are really useless in javascript context. This is here for GNU Gettext API compatibility. In practice, you'll never need to use this. This applies to all the calls including the **CATEGORY**.

### ngettext( MSGID, MSGID_PLURAL, COUNT )

Retrieves the correct translation for **COUNT** items. In legacy software you will often find something like:

```javascript
alert( count + " file(s) deleted.\n" );
```

or

```javascript
printf(count + " file%s deleted.\n", $count == 1 ? '' : 's');
```

*NOTE: javascript lacks a builtin printf, so the above isn't a working example*

The first example looks awkward, the second will only work in English and languages with similar plural rules. Before `ngettext()` was introduced, the best practice for internationalized programs was:

```javascript
if (count == 1) {
  alert( gettext("One file deleted.\n") );
} else {
  printf( gettext("%d files deleted.\n"), count );
}
```

This is a nuisance for the programmer and often still not sufficient for an adequate translation. Many languages have completely different ideas on numerals. Some (French, Italian, ...) treat 0 and 1 alike, others make no distinction at all (Japanese, Korean, Chinese, ...), others have two or more plural forms (Russian, Latvian, Czech, Polish, ...). The solution is:

```javascript
printf( ngettext("One file deleted.\n",
                 "%d files deleted.\n",
                 count), // argument to ngettext!
        count);          // argument to printf!
```

In English, or if no translation can be found, the first argument (**MSGID**) is picked if count is one, the second one otherwise. For other languages, the correct plural form (of 1, 2, 3, 4, ...) is automatically picked, too. You don't have to know anything about the plural rules in the target language, `ngettext()` will take care of that.

This is most of the time sufficient but you will have to prove your creativity in cases like

```
"%d file(s) deleted, and %d file(s) created.\n"
```

That said, javascript lacks `printf()` support. Supplied with Gettext.js is the `strargs()` method, which can be used for these cases:

```javascript
Gettext.strargs( gt.ngettext( "One file deleted.\n",
                              "%d files deleted.\n",
                              count), // argument to ngettext!
                 count); // argument to strargs!
```

NOTE: the variable replacement isn't done for you, so you must do it yourself as in the above.

### dngettext( TEXTDOMAIN, MSGID, MSGID_PLURAL, COUNT )

Like `ngettext()` but retrieves the translation from the specified textdomain instead of the default domain.

### dcngettext( TEXTDOMAIN, MSGID, MSGID_PLURAL, COUNT, CATEGORY )

Like `dngettext()` but retrieves the translation from the specified category, instead of the default category `LC_MESSAGES`.

### pgettext( MSGCTXT, MSGID )

Returns the translation of MSGID, given the context of MSGCTXT.

Both items are used as a unique key into the message catalog.

This allows the translator to have two entries for words that may translate to different foreign words based on their context. For example, the word "View" may be a noun or a verb, which may be used in a menu as File->View or View->Source.

```javascript
alert( pgettext( "Verb: To View", "View" ) );
alert( pgettext( "Noun: A View", "View"  ) );
```

The above will both lookup different entries in the message catalog.

In English, or if no translation can be found, the second argument (**MSGID**) is returned.

### dpgettext( TEXTDOMAIN, MSGCTXT, MSGID )

Like `pgettext()`, but retrieves the message for the specified **TEXTDOMAIN** instead of the default domain.

### dcpgettext( TEXTDOMAIN, MSGCTXT, MSGID, CATEGORY )

Like `dpgettext()` but retrieves the message from the specified **CATEGORY** instead of the default category `LC_MESSAGES`.

### npgettext( MSGCTXT, MSGID, MSGID_PLURAL, COUNT )

Like `ngettext()` with the addition of context as in `pgettext()`.

In English, or if no translation can be found, the second argument (MSGID) is picked if **COUNT** is one, the third one otherwise.

### dnpgettext( TEXTDOMAIN, MSGCTXT, MSGID, MSGID_PLURAL, COUNT )

Like `npgettext()` but retrieves the translation from the specified textdomain instead of the default domain.

### dcnpgettext( TEXTDOMAIN, MSGCTXT, MSGID, MSGID_PLURAL, COUNT, CATEGORY )

Like `dnpgettext()` but retrieves the translation from the specified category, instead of the default category `LC_MESSAGES`.

### strargs (string, argument_array)

```
string : a string that potentially contains formatting characters.
argument_array : an array of positional replacement values
```

This is a utility method to provide some way to support positional parameters within a string, as javascript lacks a `printf()` method.

The format is similar to `printf()`, but greatly simplified (ie. fewer features).

Any percent signs followed by numbers are replaced with the corrosponding item from the **argument_array**.

Example:

```javascript
var string = "%2 roses are red, %1 violets are blue";
var args   = new Array("10", "15");
var result = Gettext.strargs(string, args);
// result is "15 roses are red, 10 violets are blue"
```

The format numbers are 1 based, so the first itme is %1.

A lone percent sign may be escaped by preceeding it with another percent sign.

A percent sign followed by anything other than a number or another percent sign will be passed through as is.

Some more examples should clear up any abmiguity. The following were called with the orig string, and the array as Array("[one]", "[two]") :

```
orig string "blah" becomes "blah"
orig string "" becomes ""
orig string "%%" becomes "%"
orig string "%%%" becomes "%%"
orig string "%%%%" becomes "%%"
orig string "%%%%%" becomes "%%%"
orig string "tom%%dick" becomes "tom%dick"
orig string "thing%1bob" becomes "thing[one]bob"
orig string "thing%1%2bob" becomes "thing[one][two]bob"
orig string "thing%1asdf%2asdf" becomes "thing[one]asdf[two]asdf"
orig string "%1%2%3" becomes "[one][two]"
orig string "tom%1%%2%aDick" becomes "tom[one]%2%aDick"
```

This is especially useful when using plurals, as the string will nearly always contain the number.

It's also useful in translated strings where the translator may have needed to move the position of the parameters.

For example:

```javascript
var count = 14;
Gettext.strargs( gt.ngettext('one banana', '%1 bananas', count), [count] );
```

NOTE: this may be called as an instance method, or as a class method.

```javascript
// instance method:
var gt = new Gettext(params);
gt.strargs(string, args);
// class method:
Gettext.strargs(string, args);
```

## Notes

These are some notes on the internals

### Locale caching

Loaded locale data is currently cached class-wide. This means that if two scripts are both using Gettext.js, and both share the same gettext domain, that domain will only be loaded once. This will allow you to grab a new object many times from different places, utilize the same domain, and share a single translation file. The downside is that a domain won't be RE-loaded if a new object is instantiated on a domain that had already been instantiated.

## BUGS / TODO

### error handling

Currently, there are several places that throw errors. In GNU Gettext, there are no fatal errors, which allows text to still be displayed regardless of how broken the environment becomes. We should evaluate and determine where we want to stand on that issue.

### syncronous only support (no ajax support)

Currently, fetching language data is done purely syncronous, which means the page will halt while those files are fetched/loaded.

This is often what you want, as then following translation requests will actually be translated. However, if all your calls are done dynamically (ie. error handling only or something), loading in the background may be more adventagous.

It's still recommended to use the statically defined `<script ...>` method, which should have the same delay, but it will cache the result.

### domain support

domain support while using shortcut methods like _('string') or i18n('string').

Under normal apps, the domain is usually set globally to the app, and a single language file is used. Under javascript, you may have multiple libraries or applications needing translation support, but the namespace is essentially global.

It's recommended that your app initialize it's own shortcut with it's own domain. (See examples/wrapper/i18n.js for an example.)

Basically, you'll want to accomplish something like this:

```javascript
// in some other .js file that needs i18n
this.i18nObj = new i18n;
this.i18n = this.i18nObj.init('domain');
// do translation
alert( this.i18n("string") );
```

If you use this raw Gettext object, then this is all handled for you, as you have your own object then, and will be calling `myGettextObject.gettext('string')` and such.

### encoding

May want to add encoding/reencoding stuff. See GNU iconv, or the perl module Locale::Recode from libintl-perl.

## Compatibility

This has been tested on the following browsers. It may work on others, but these are all those to which I have access.

FF1.5, FF2, FF3, IE6, IE7, Opera9, Opera10, Safari3.1, Chrome

* FF = Firefox
* IE = Internet Explorer

## Requires

bin/po2json requires perl, and the perl modules Locale::PO and JSON.

## See also

bin/po2json (included), examples/normal/index.html, examples/wrapper/i18n.html, examples/wrapper/i18n.js, Locale::gettext_pp(3pm), POSIX(3pm), gettext(1), gettext(3)

## Author

Copyright (C) 2008, Joshua I. Miller <unrtst@cpan.org>, all rights reserved. See the source code for details.
