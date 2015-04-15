---
title: Extension Sequencer
layout: default
---

# Summary #
[SeLite Extension Sequencer](https://addons.mozilla.org/en-US/firefox/addon/selite-extension-sequencer/versions/), one of SeLite [AddOns](AddOns), allows Core and IDE extensions of Selenium IDE to declare dependencies. Then it ensures that they are loaded in a correct order.

It only works for extensions packaged as Firefox add-ons (as .xpi files, or through [proxy files](https://developer.mozilla.org/en/Setting_up_extension_development_environment)). It doesn't cover extensions loaded from single Javascript files via Selenium IDE menu Options > Options > General, neither through [BootstrapLoader](BootstrapLoader).

# Use cases #
Say you have two or more extensions of the same type (Core or IDE). Then Firefox and Selenium IDE don't guarantee a specific order of loading them up.

Some extensions may depend on others and require them to be initiated first. For example
  * the dependant tail-intercepts functionality originating in the dependee
  * both extensions tail-intercept same functionality and their order matters
  * initiation of the dependant relies on the dependee

# Loading via Extension Sequencer #
Your add-on needs to have _chrome://my-plugin/content/SeLiteExtensionSequencerManifest.js_ (in UTF-8) containing something like:
```javascript

"use strict";
SeLiteExtensionSequencer.registerPlugin( {
id: 'my-unique@plugin-id',
infoURL: 'https://addons.mozilla.org/en-US/firefox/addon/dummy-test-journey/',
downloadURL: 'https://addons.mozilla.org/en-US/firefox/addon/dummy-test-journey/vesions/' // optional; if not set and infoURL is at addons.mozilla.org, then downloadURL is auto-generated by appending 'versions/'
coreURL: 'chrome://my-plugin/content/extensions/core-extension.js', // optional; it may be an array
ideURL: 'chrome://my-plugin/content/extensons/ide-extension.js', // optional; it may be an array
requisitePlugins: { // optional. These are *direct* dependencies only. E.g.:
'misc@selite.googlecode.com': {
name: 'SeLite Miscellaneous',
infoURL: 'https://addons.mozilla.org/en-US/firefox/addon/dummy-test-train/',
downloadURL: 'https://addons.mozilla.org/en-US/firefox/addon/dummy-test-train/versions/', // optional, see above
compatibleVersion: "0.10", // optional, the minimal value of oldestCompatibleVersion that this requisite add-on must have
minVersion: "0.10" // optional; the minimal version that this requisite add-on must have
},
...
},
nonSequencedRequisitePlugins: { // optional. This is for *direct* dependencies that don't use Extension Sequencer.
'someOtherPlugin@domain': { // structure like above
}
...
},
oldestCompatibleVersion: "0.22", // the oldest version of this add-on that this version (the one being registered) is compatible with. Optional. If present, then it's compared to 'compatibleVersion' in manifests of any add-ons that depend on this add-on.
preActivate: function(api) { // optional
....
}
} );
```

Sequencer will find and process this file. Then it will initiate the plugin after all its sequenced dependencies. If it depends on any add-ons that don't use ExtensionSequencer, then this doesn't guarantee their respective activation order. You can use _window.setTimeout()_ in your Core extension to delay the parts of its activation that depend on any non-sequenced add-ons. Alternatively, encourage the third party to use ExtensionSequencer, too.

## Examples of SeLiteExtensionSequencerManifest.js ##
See _SeLiteExtensionSequencerManifest.js_ files in source of various SeLite [AddOns](AddOns). For full API, see function _SeLiteExtensionSequencer.registerPlugin(prototype)_ in [SeLiteExtensionSequencer.js](https://code.google.com/p/selite/source/browse/extension-sequencer/src/chrome/content/SeLiteExtensionSequencer.js).

You may have multiple add-ons that override same parts of Selenium core and that don't need each other, but if used together then they need to override those parts in a specific order. Then you may want to declare optional dependency between them, so that they are loaded in appropriate order. For examples of dependencies, see _SeLiteExtensionSequencerManifest.js_ and tests in [Shell tests](#shell-tests).

## Validate that SeLite Extension Sequencer is present (optional) ##
If your Firefox add-on has SeLiteExtensionSequencerManifest.js but Extension Sequencer is missing, your add-on won't get activated. Users may then be confused by the lack of functionality. It's worth to make it check whether Extension Sequencer is present. If not, then it should alert the users that they need to install Extension Sequencer.

See [browser.js](https://code.google.com/p/selite/source/browse/db-objects/src/chrome/content/extensions/browser.js) of SeLite DB Objects as an example.

## Shell tests ##
ExtensionSequencer is tested against a list of variations of _SeLiteExtensionSequencerManifest.js_ in several plain [extensions](https://code.google.com/p/selite/source/browse/#git%2Fextension-sequencer%2Fshell-tests%2Fextensions). You can browse their source code at [extension-sequencer/shell-tests](https://code.google.com/p/selite/source/browse/#git%2Fextension-sequencer%2Fshell-tests)<a href='Hidden comment:  (online) or <a href="../selite/extension-sequencer/shell-tests">offline

Unknown end tag for &lt;/a&gt;

'></a>. See [a list of those tests](https://selite.googlecode.com/git/extension-sequencer/shell-tests/tests.html) <a href='Hidden comment:  (online) or <a href="../selite/extension-sequencer/shell-tests/tests.html">offline

Unknown end tag for &lt;/a&gt;

'></a> with test descriptions and expected outputs.

### Installing shell tests ###
The tests use a separate Firefox profile called SeLiteExtensionSequencerTest, with some [extensions](https://code.google.com/p/selite/source/browse/#git%2Fextension-sequencer%2Fshell-tests%2Fextensions). To set up that profile and its add-ons, download source of [Extension Sequencer](https://code.google.com/p/selite/source/browse/#git%2Fextension-sequencer) (or whole SeLite as per [InstallFromSource](InstallFromSource)) and run extension-sequencer/setup\_proxies.bat or setup\_proxies.sh. It will start create and set up that profile in Firefox.

On Windows (and probably on Mac OS, too) you'll need to install [Selenium IDE](http://docs.seleniumhq.org/download/) and apply Windows/Mac OS-specific steps from [InstallFromSource](InstallFromSource).

(You don't need any other SeLite [AddOns](AddOns) for these tests.)

### Running and modifying shell tests ###
Invoke _run\_tests.ps1_, _run\_tests\_mac.sh_ or _run\_tests.sh_.

When changing or re-using those tests, follow the existing special format of their _SeLiteExtensionSequencerManifest.js_ files. For _oldestCompatibleVersion_, _minVersion_ and _compatibleVersion_, enclose the version numbers within "..". Otherwise they do not get compared well when the versions end with 0's. Have any commas separating the fields at the beginning of lines rather than at the end, and have _preActivate_ entry (including the whole function) on one line only. That enables _run\_tests.sh_ and _run\_tests.ps1_ to comment or uncomment those lines.

To debug Extension Sequencer with Firefox Browser Toolbox, visit _chrome://selite-extension-sequencer/content/extensions/invoke.xul_.

# Core extensions loaded twice #
Because of [ThirdPartyIssues](ThirdPartyIssues) > [Selenium issue #6697](http://code.google.com/p/selenium/issues/detail?id=6697), Core extensions get loaded 2x (whether loaded via Selenium IE menu > Options > Options... > Core extension, from an .xpi file or through a proxy file - regardless of ExtensionSequencer - but not when loaded via [BootstrapLoader](BootstrapLoader)). That's OK if the extension just adds new Selenese commands. But it can be a problem if it tail/head intercepts Selenese or Selenium Core. You don't want to intercept Selenese or Selenium Core twice.

In extensions loaded via ExtensionSequencer you can use _SeLiteExtensionSequencer.coreExtensionsLoadedTimes_ to keep track of whether the extension is loaded for the first time or the second one. See [chrome://selite-extension-sequencer/content/SeLiteExtensionSequencer.js](https://code.google.com/p/selite/source/browse/extension-sequencer/src/chrome/content/SeLiteExtensionSequencer.js).

## Global symbols and strict mode ##
That Selenium issue also causes problems when adding new global symbols to Selenese scope and using [JavascriptEssential](JavascriptEssential) > [Strict Javascript](JavascriptEssential#strict-javascript). When Selenium IDE loads a Core extension for the first and second time, it uses different scope and a different 'Selenium' class. If the extension in strict mode defines any global symbols, they are thrown away after the first load: only the second load stays in Selenium scope.

That may tempt you not to use strict mode and set the global symbols as undeclared (without _var_ statement). This would create them in Selenium global scope (bypassing the scope that will be thrown away - see [MDN mozIJSSubScriptLoader.loadSubScript](https://developer.mozilla.org/en-US/docs/XPCOM_Interface_Reference/mozIJSSubScriptLoader#loadSubScript%28%29)). However, there's a way to do it in strict mode. When setting the new global symbols, also set them as fields on an existing object accessible from global scope (e.g. a class constructor). At the beginning of your extension, check whether those fields are set on that (existing) object; if not then set them there and also in the global scope, otherwise retrieve them from that global object and set them in the global scope. See [se-testcase-debug-context.js](https://code.google.com/p/selite/source/browse/testcase-debug-context/src/chrome/content/extensions/se-testcase-debug-context.js).