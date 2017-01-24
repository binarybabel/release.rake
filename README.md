# Release.rake

[![Latestver](https://lv.binarybabel.org/catalog-api/release.rake/latest.svg?label=latest+version)](https://lv.binarybabel.org/catalog/release.rake/latest) [![Join the chat at https://gitter.im/binarybabel/release.rake](https://badges.gitter.im/binarybabel/release.rake.svg)](https://gitter.im/binarybabel/release.rake?utm_source=badge&utm_medium=badge&utm_content=badge)

Single-file customizable **release + changelog** tool for Ruby projects.  
Designed with Rails in mind, but easily tweaked for any type of project.

**Example Changelog** — Commits since last Git tag are used to help amend your changelog.  
Messages are parsed for common words to generate ASCII or Emoji line indicators.

```
### 1.1 (2017-01-01)

* __`NEW`__ This commit added something
* __`~~~`__ This commit changed something
* __`!!!`__ Fixed a bug here
* __`-!-`__ Removed a feature
```

## Installation 

```
$ cd lib/tasks
$ curl -OL https://git.io/release.rake
```

Review the "Configuration" section of the new rake file.
  * __`version_file`__ points to the hard-coded location of your project's version (sample below)
  * __`version_lambda`__ provides a method for retrieving the current version at runtime

**Need a version file?** — __`config/version.rb`__
```ruby
module MyApp
  VERSION = '1.0.0'
end
```

**Test the installation**

```
$ rake release:current
1.0.0
```

## Usage

__`release.rake`__ is designed as a single-file, and directly installed in your project, to encourage users read, understand, and modify it to meet their needs. Most of the commands are summarized below, but for best results consult the source file.

### Shortcut Commands

The shortcut task section is a great place to start tweaking your install. For example the `:next` task  prereqs determine which version segment should be bumped, or adjust the commit message in the `:prep` task.

```
$ rake release:next
```

* Bumps minor, generates changelog, prints further instructions.
* Does NOT automatically commit changes.
  
```
$ rake release:prep
```

* Bumps minor with "pre" suffix.
* Commits version file change automatically.

### Main Commands

```
$ rake release:changelog
```

* Generate changelog from Git events since last tag.
* Changes are added to the top of the configured `changelog_file`
  * Above any previous release, but below other information you add to the top of the file.

```
$ rake release:bump              # patch, full-release
# 1.0.0 -> 1.0.1

$ rake release:bump[2, pre]      # minor, "pre"-release
# 1.0.0 -> 1.1.pre
```

* Updates configured `version_file`.
* Does NOT automatically commit changes.

```
$ rake release:commit            # commit + tag
```

* Commits changed version/changelog files...
  * Tags release, uses configured `commit_message`
  * Will be signed if `user.signingkey` is set on local repo
* Tag or message can be customized as command parameters.

## Integrating with Versioneer

[Versioneer](https://github.com/binarybabel/gem-versioneer) is a gem for dynamically generating prerelease versions based on commits since last tag, thus avoiding any need to "prep" for future releases.

Adjust your project's version file to manage the hardcoded release in a separate constant.

__`config/version.rb`__
```ruby
require 'versioneer'

module MyApp
  RELEASE = '1.0'
  begin
    VERSION = Versioneer::Config.new(File.expand_path('../../', __FILE__)).to_s
  rescue Versioneer::InvalidRepoError
    VERSION = self::RELEASE
  end
end
```

__`lib/tasks/release.rake`__
```ruby
version_lambda = lambda { Object.const_get(Rails.application.class.name.sub(/::.+$/, ''))::RELEASE }
```
