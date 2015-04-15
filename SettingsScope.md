---
title: Settings Scope
layout: default
---

# Level of applying configurations and manifests #
Configurations have effect on test suites, rather than to specific test cases. That's because a test case can call a function from another test case (using [SelBlocksGlobal](SelBlocksGlobal)). That would be confusing if those test cases were associated with different configurations.

A manifest affects to any test suite under its folder subtree, unless overridden by manifest(s) at a more specific (lower) level or by a set (at any level). Set(s) override 'values' manifest(s), so that teams can share default values via 'values' manifests, yet the team members can override them in their own set(s), serving as preferences, without changing any shared manifest(s).

The actual location of test case(s) doesn't matter. They can even be outside of the test suite folder. So you could run same test case(s) through different test suite(s), each with different combination of manifest(s) and set(s).

Manifests work only if the module associates with folders. Otherwise the test uses the default set (if the module allows sets) or the only existing set (if the module doesn't allow multiple sets), or a set with a given name (if requested by an explicit API call). See [SettingsAPI](SettingsAPI).

## Multi-valued fields ##
Value(s) of a multi-valued field come from a maximum of one source (set or manifest) - the topmost applicable from the list above. Multi-valued fields don't have the values 'merged' from various sets neither from manifests. It would make them more flexible, but also more confusing and fragile. This also applies to FixedMap fields (see [SettingsFields](SettingsFields)).

# Granularity and overriding through sets and manifests #
The rules of applying values from manifests and configuration sets are, in order of priority:
  * the default set (if any) or any associated configuration sets override 'values' manifests
  * associated sets override the default set
  * more local manifests override less local manifests of the same type (either 'values' or 'associations')
  * all those override default value of the field (from definition of its module)

So, a field will have value(s) based on its topmost occurrence within:
  * most locally associated set
  * less locally associated set
  * ...
  * least locally associated set
  * the default set (if any)
  * most local 'values' manifest
  * less local 'values' manifest
  * ...
  * least local 'values' manifest
  * field default value(s) from module definition (only if the module associates with folders)
> > <a href='Hidden comment: TODO Check "(only if the module associates with folders)"'></a>

# Caching #
Manifests are cached
  * temporarily when edited through GUI. You just need to refresh the screen in order for a change to apply.
  * fully when used by Selenium tests (via _SeLiteData.getStorageFromSettings_). You need to restart Firefox in order for a change to apply.
  * optionally when used by other extensions (the developer can choose whether to cache or not)

# Symlinks (on Mac OS/Unix) #
Symlinks are another way to have variety/separation of configuration sets. If you access a test suite via a file path which has symlinked non-leaf folder(s), it can be  associated to partially/fully different configuration sets.

<a href='Hidden comment: Comment: TODO Test how Selenium IDE treats a test suite loaded via a path that depends on symlinks - does it use the provided path, or does it resolved it first? If it resolves the path first, then change this paragraph.'></a>