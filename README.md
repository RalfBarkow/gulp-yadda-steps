# gulp-yadda-steps
[![view on npm](http://img.shields.io/npm/v/gulp-yadda-steps.svg)](https://www.npmjs.org/package/gulp-yadda-steps)
[![npm module downloads per month](http://img.shields.io/npm/dm/gulp-yadda-steps.svg)](https://www.npmjs.org/package/gulp-yadda-steps)
[![Dependency Status](https://david-dm.org/Cellarise/gulp-yadda-steps.svg)](https://david-dm.org/Cellarise/gulp-yadda-steps)

> A gulp task to generate or update Yadda test step libraries from Gherkin features (natural language test scripts).


##Usage 

This gulp task expects a feature file, written in Gherkin syntax, as input, and outputs the matching Yadda test step libraries for this feature file.

### As a gulp task

Require this package and use as part of your gulp task.

```js
var GulpYaddaSteps = require('gulp-yadda-steps');
gulp.src('local.feature')
.pipe(new GulpYaddaSteps())
.pipe(fs.createWriteStream('output.js'));
```


# API
<a name="module_gulp-yadda-steps"></a>
#gulp-yadda-steps
A gulp task to generate or update Yadda test step libraries from Gherkin features (natural language test scripts).

**Params**

- opts `Object` - Task configuration options (see modules Parser and Render for more information)  

**Returns**: `readable-stream/transform`  
**Example**  
 Given the feature file:

```markdown
Feature: Generate test steps from gherkin features
As a developer
I want to be able to generate test step boilerplate code from gherkin features
So that I can focus effort on building quality test steps

Scenario: Generating test steps

Given I have a simple feature file
When I read the feature file
Then a test steps file is generated
```

When you pass the feature file to a `new gulpYaddaSteps()`, and pipe it to a given destination.

```js
var gulpYaddaSteps = require('gulp-yadda-steps');
gulp.src('local.feature')
.pipe(new gulpYaddaSteps())
.pipe(fs.createWriteStream('output.js'));
```

Then you'll get a Yadda style test step library:

```js
"use strict";
var English = require('yadda').localisation.English;

/ Feature: Generate test steps from gherkin features /
module.exports = (function() {
 return English.library()
   /Generating test steps/
   .define("Given I have a simple feature file", function(done) {
       this.assert(false);
       done();
   })
   .define("When I read the feature file", function(done) {
       this.assert(false);
       done();
   })
   .define("Then a test steps file is generated", function(done) {
       this.assert(false);
       done();
   });
})();
```

Note that the output is a vinyl file which will have the filePath overridden if the libraryBasePath and featureBasePath options are set.


**Members**

* [gulp-yadda-steps](#module_gulp-yadda-steps)
  * [gulp-yadda-steps.Parser(opts)](#module_gulp-yadda-steps.Parser)
  * [gulp-yadda-steps.Render(opts)](#module_gulp-yadda-steps.Render)

<a name="module_gulp-yadda-steps.Parser"></a>
##gulp-yadda-steps.Parser(opts)
Parser is a transform stream requiring a valid feature file.  Parser will load test step libraries tagged in the
feature (using @libraries=) and will attempt to load a file with the feature filename and suffix '-steps.js'.
If one or more libraries are found they will be used to find step matches in the feature and filter them from the output.

**Params**

- opts `Object` - Parser configuration options  
  - \[libraryBasePath\] `string` - Specifies a path to the base location for the test step libraries.
E.g. if the base path to the test step library is `Test/unit/steps/` use `path.join(__dirname, './steps/')`
if the script is running from `'Test/unit'`.
Note: featureBasePath must also be set for this option to take effect.  
  - \[featureBasePath\] `string` - Specifies a path to the base location for the features.
Note: libraryBasePath must also be set for this option to take effect.  

**Returns**: `readable-stream/transform`  
**Example**  
Given the feature file:

```markdown
Feature: Generate test steps from gherkin features
As a developer
I want to be able to generate test step boilerplate code from gherkin features
So that I can focus effort on building quality test steps

Scenario: Generating test steps

Given I have a simple feature file
When I read the feature file
Then a test steps file is generated
```

When you pass the feature file to a `new Parser()`, and pipe it to a given destination.

```js
var Parser = require('gulp-yadda-steps').Parser;
gulp.src('local.feature')
.pipe(new Parser())
.pipe(fs.createWriteStream('output.json'));
```

Then you'll get a Yadda parsed JSON output:

```js
{"feature":{"title":"Generate test steps from gherkin features","annotations":{},
"description":["As a developer","I want to be able to generate test step boilerplate code from gherkin features",
"So that I can focus effort on building quality test steps"],
"scenarios":[{"title":"Generating test steps",
"annotations":{},"description":[],
"steps":["Given I have a simple feature file","When I read the feature file","Then a test steps file is generated"]}]}}
```

Note that the output is a vinyl file which will have the filePath overridden if the libraryBasePath and featureBasePath options are set.


<a name="module_gulp-yadda-steps.Render"></a>
##gulp-yadda-steps.Render(opts)
Render is a transform stream requiring a yadda parsed JSON file.  Render will load test step libraries tagged in the
feature (using @libraries=) and will attempt to load a file with the feature filename and suffix '-steps.js'.
If one or more libraries are found they will be used to find step matches in the feature and filter them from the output.

**Params**

- opts `Object` - Parser configuration options  
  - \[template_library='../templates/yadda_library.dust'\] `string` - Specifies a path to a template_library dust file. This file controls the layout of new step libraries.  
  - \[template_insertion='../templates/yadda_insert.dust'\] `string` - Specifies a path to a template_insertion dust file.
This file controls the layout for inserting steps into an existing step library.
This template should use dust partial `{>steps/}` to insert generated steps from template_steps.  
  - \[template_steps='../templates/yadda_steps.dust'\] `string` - Specifies a path to a template_steps dust file. This file controls the layout and generation of test steps.  

**Returns**: `readable-stream/transform`  
**Example**  
 Given a yadda parsed JSON file:

```js
{"feature":{"title":"Generate test steps from gherkin features","annotations":{},
"description":["As a developer","I want to be able to generate test step boilerplate code from gherkin features",
"So that I can focus effort on building quality test steps"],
"scenarios":[{"title":"Generating test steps",
"annotations":{},
"description":[],
"steps":["Given I have a simple feature file","When I read the feature file","Then a test steps file is generated"]}]}}
```

 When you pass the yadda parsed JSON file to a `new Render()`, and pipe it to a given destination.

```js
var Render = require('gulp-yadda-steps').Render;
gulp.src('output.json')
.pipe(new Render())
.pipe(fs.createWriteStream('output.js'));
```

 Then you'll get a Yadda style test step library:

```js
"use strict";
var English = require('yadda').localisation.English;

/ Feature: Generate test steps from gherkin features /
module.exports = (function() {
return English.library()
   /Generating test steps/
   .define("Given I have a simple feature file", function(done) {
       this.assert(false);
       done();
   })
   .define("When I read the feature file", function(done) {
       this.assert(false);
       done();
   })
   .define("Then a test steps file is generated", function(done) {
       this.assert(false);
       done();
   });
})();
```

 Note that the output is a vinyl file which will have the filePath overridden if the libraryBasePath and featureBasePath options are set.



*documented by [jsdoc-to-markdown](https://github.com/75lb/jsdoc-to-markdown)*.


# License

MIT License (MIT)

Copyright (c) 2014 John Barry

## Dependencies
[ansi-regex@0.2.1](&quot;https://github.com/sindresorhus/ansi-regex&quot;) - &quot;MIT&quot;, [ansi-styles@1.1.0](&quot;https://github.com/sindresorhus/ansi-styles&quot;) - &quot;MIT&quot;, [chalk@0.5.1](&quot;https://github.com/sindresorhus/chalk&quot;) - &quot;MIT&quot;, [clone-stats@0.0.1](&quot;https://github.com/hughsk/clone-stats&quot;) - &quot;MIT&quot;, [core-util-is@1.0.1](&quot;https://github.com/isaacs/core-util-is&quot;) - &quot;MIT&quot;, [dateformat@1.0.8-1.2.3](&quot;https://github.com/felixge/node-dateformat&quot;) - &quot;MIT*&quot;, [duplexer2@0.0.2](&quot;https://github.com/deoxxa/duplexer2&quot;) - &quot;BSD&quot;, [dustjs-helpers@1.2.0](&quot;https://github.com/linkedin/dustjs-helpers&quot;) - &quot;MIT&quot;, [dustjs-linkedin@2.3.5](&quot;https://github.com/linkedin/dustjs&quot;) - &quot;MIT&quot;, [escape-string-regexp@1.0.1](&quot;https://github.com/sindresorhus/escape-string-regexp&quot;) - &quot;MIT&quot;, [gulp-util@3.0.0](&quot;https://github.com/wearefractal/gulp-util&quot;) - [&quot;MIT&quot;], [gulp-yadda-steps@0.0.0](&quot;https://github.com/Cellarise/gulp-yadda-steps&quot;) - &quot;MIT License (MIT)&quot;, [has-ansi@0.1.0](&quot;https://github.com/sindresorhus/has-ansi&quot;) - &quot;MIT&quot;, [inherits@2.0.1](&quot;https://github.com/isaacs/inherits&quot;) - &quot;ISC&quot;, [isarray@0.0.1](&quot;https://github.com/juliangruber/isarray&quot;) - &quot;MIT&quot;, [lodash._escapehtmlchar@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._escapestringchar@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._htmlescapes@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._isnative@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._objecttypes@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._reinterpolate@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._reunescapedhtml@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash._shimkeys@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.defaults@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.escape@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.isobject@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.keys@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.template@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.templatesettings@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash.values@2.4.1](&quot;https://github.com/lodash/lodash-cli&quot;) - &quot;MIT&quot;, [lodash@2.4.1](&quot;https://github.com/lodash/lodash&quot;) - &quot;MIT&quot;, [minimist@0.2.0](&quot;https://github.com/substack/minimist&quot;) - &quot;MIT&quot;, [multipipe@0.1.1](&quot;https://github.com/segmentio/multipipe&quot;) - &quot;MIT*&quot;, [readable-stream@1.0.27-1](&quot;https://github.com/isaacs/readable-stream&quot;) - &quot;MIT&quot;, [readable-stream@1.1.13](&quot;https://github.com/isaacs/readable-stream&quot;) - &quot;MIT&quot;, [streamifier@0.1.0](&quot;https://github.com/gagle/node-streamifier&quot;) - &quot;MIT&quot;, [string_decoder@0.10.31](&quot;https://github.com/rvagg/string_decoder&quot;) - &quot;MIT&quot;, [strip-ansi@0.3.0](&quot;https://github.com/sindresorhus/strip-ansi&quot;) - &quot;MIT&quot;, [supports-color@0.2.0](&quot;https://github.com/sindresorhus/supports-color&quot;) - &quot;MIT&quot;, [through2@0.5.1](&quot;https://github.com/rvagg/through2&quot;) - &quot;MIT&quot;, [vinyl@0.2.3](&quot;https://github.com/wearefractal/vinyl&quot;) - [&quot;MIT&quot;], [xtend@3.0.0](&quot;https://github.com/Raynos/xtend&quot;) - [&quot;MIT&quot;], [yadda@0.11.2](&quot;https://github.com/acuminous/yadda&quot;) - &quot;Apache2&quot;, 
*documented by [npm-licenses](http://github.com/AceMetrix/npm-license.git)*.