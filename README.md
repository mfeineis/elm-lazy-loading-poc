# elm-lazy-loading-poc
Exploring lazy loading modules with Elm


## Demo
Spin up a static http server e.g.

```
npm i -g http-server
http-server -p 3000 .
```

Visit http://localhost:3000 with a modern browser, when you click the button
the `External` module code is loaded on demand and will render a "Hello, External!"
node after the button.


## Strategy (mostly manual for PoC)
The resulting code files are committed into the `/build` directory so you can
check what it looks like. These have been prepared along this recipe:
* Build all Elm modules that should be loaded to `dist/app.full.js`
> npm run build
* Include the module loading snippet into the base bundle and export the function
  via the `_Platform_export(...)`. We're using a whitelist and well known paths
  to the soon to be externalized files because we employ `eval` to evaluate the
  on-demand code within the base bundle context
* Move the module code that should be lazy loaded from the full build
  to their own files e.g. all variables starting with `author$project$External`
* Remove the lazy module exports from the main bundle and add it to the
  externalized files and wrap the code in an IFFE with the "use strict" pragma

Finally we wrap the JS code that actually triggers the module loading into a
`CustomElement` so that we only need to append a DOM node wherever we want
to include the external module.


## Discussion
This approach works but it has drawbacks.

Without the externalization logic being driven by the compiler it's hard to
minify the code and actually leverage dead-code-elimination. If we minify
before separating the external code we have a hard time finding out what
code belongs to which module. Even without minification it's not clear how
to determine what library code is only needed in external modules and could
therefore moved out of the base bundle.

