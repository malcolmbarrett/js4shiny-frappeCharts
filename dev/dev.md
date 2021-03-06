
# Building an HTML Widget: Frappe Charts

This repository walks through the building of an
[htmlwidget](https://www.htmlwidgets.org/) around the JavaScript library
[Frappe Charts](https://frappe.io/charts).

Because there are many files and moving pieces involved in this process,
I’ve created a git repository that walks through the changes at each
step of the process. With my notes about each step, I’ve included the
SHA linked to the updates made during the step. If you’re viewing this
on GitHub, those SHA hashes should be converted to links that will take
you to a summary of which files changed at each step.

This introduction focuses on the mechanics of htmlwidgets. For a much
more detailed summary, the [HTML
Widgets](https://bookdown.org/yihui/rmarkdown/html-widgets.html) chapter
of the R Markdown book is an excellent introduction.

## Setup R Package

  - [changelog:
    a3f6fd](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/a3f6fdd986d2b98323b5be43e323df4f6a19f1f3)

Create a package for this HTML widget. We’re not going to publish this,
so you can call it whatever you want

``` r
usethis::create_package("frappeCharts")
```

Add a dev script for notes

``` r
dir.create("dev")
file.create("dev/dev.R")
rstudioapi::navigateToFile("dev/dev.R")
```

### Add the R package dependencies for an htmlwidget package

``` r
usethis::use_package("htmlwidgets")
usethis::use_package("htmltools")
usethis::use_package("jsonlite")
usethis::use_package("shiny")
usethis::use_package("yaml")
```

## Setup npm package

  - [changelog: 256f0c](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/256f0ca112b2685608f9a17a4fb4e35d279c9830)

Same process again, but this time for npm.

``` bash
npm init

# or

npm init -y
```

Open `package.json` and take a look

``` json
{
  "name": "frappecharts",
  "version": "0.0.1",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT"
}
```

From Frappe Charts
[docs\#installation](https://frappe.io/charts/docs#installation):

``` bash
npm install frappe-charts
```

We now have a dependency in `package.json` and there’s a
`package-lock.json` file.

``` json
"dependencies": {
  "frappe-charts": "^1.3.0"
}
```

## Ignore node\_modules but add package-lock

There’s also a `node_modules/` folder with `frappe-charts/` inside. Add
`node_modules` to `.Rbuildignore` and `.gitignore`. (BTW, you can and
are supposed to commit `package-lock.json`.)

``` r
usethis::use_build_ignore("node_modules")
usethis::use_build_ignore("package.json")
usethis::use_build_ignore("package-lock.json")
usethis::use_git_ignore("node_modules")
```

## Scaffold the HTML widget

  - [changelog: 38bac2](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/38bac2c65cf54816525076690310008e62ab99a1)

<!-- end list -->

``` r
htmlwidgets::scaffoldWidget("frappeChart")
```

This adds files in `inst/htmlwidgets`

    inst
    └── htmlwidgets
        ├── frappeChart.js    #<< R <-> JS code
        └── frappeChart.yaml  #<< list of dependencies

and creates a file `R/frappeChart.R` with the functions

  - `frappeChart()`
  - `frappeChartOutput()` (for shiny)
  - `renderFrappeChart()` (for shiny)

## Use `npm` to get our dependencies in the right place

  - [changelog: 7abf02](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/7abf0224345a67217c4a476f04eafe581f0ecec0)

`htmlwidgets` load dependencies in a way that’s exactly the same as
using a `<script>` tag in the HTML `<head>`. Look at the [documentation
on Frappe Charts](https://frappe.io/charts/docs#installation) and decide
which file we should use.

Here’s the block from their docs

    <script src="https://cdn.jsdelivr.net/npm/frappe-charts@1.2.4/dist/frappe-charts.min.iife.js"></script>
    <!-- or -->
    <script src="https://unpkg.com/frappe-charts@1.2.4/dist/frappe-charts.min.iife.js"></script>

We need to get our dependecy into a subfolder of `inst/htmlwidgets`.
Convention is `inst/htmlwidgets/lib/<dependency_name>`. Rather than
creating the directoy and copying over, etc., we can have an `npm` build
script do this for us.

To avoid issues with mac/windows, we’ll add a dev dependency on
[`cpy-cli`](https://github.com/sindresorhus/cpy-cli). Dev dependencies
are node modules that are used to build a package, rather than required
for the package to work.

``` bash
npm install cpy-cli --save-dev
```

Then we create the folder `frappe-charts` under `inst/htmlwidgets/lib`
that will hold the Frappe Charts JavaScript dependency. (If the library
included other required files, we would move these too.)

``` r
dir.create("inst/htmlwidets/lib/frappe-charts", recursive = TRUE)
```

And then edit `package.json` to add a copy task. You can define scripts
that are runnable with `npm run <script-name>`. For small build tasks,
this is an easy to implement build solution.

    "scripts": {
      "copy-js": "cpy 'node_modules/frappe-charts/dist/frappe-charts.min.iife*' inst/htmlwidgets/lib/frappe-charts/",
      "build": "npm run copy-js"
    }

Notice that running `npm run build` will also call `npm run copy-js`. If
we had more build tasks related to our JavaScript dependencies, like
linting or testing, we could add them as separate scripts and have them
run in the build process with `npm run <task-1> && npm run <task-2>`,
etc.

## Create a demo html\_document\_plain()

  - [changelog: 036d45](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/036d454f80d6036fc1ba35db92161fd19c053635)

<!-- end list -->

``` r
dir.create("dev/demo")
js4shiny::js4shiny_rmd(path = "dev/demo/demo.Rmd")
```

Use the example in the [Frappe Charts
Docs](https://frappe.io/charts/docs).

``` r
tagList(
  div(id = "chart"),
  htmltools::htmlDependency(
    name = "frappe-charts",
    version = "1.3.0",
    package = "frappeCharts",
    src = "htmlwidgets/lib/frappe-charts",
    script = "frappe-charts.min.iife.js",
    all_files = TRUE
  )
)
```

And copy the JS into a javascript chunk.

⚠️ The dependencies won’t be found until you build/install.

``` r
devtools::document()
devtools::install()
```

If you get a path not found error

    Error: path for html_dependency not found: inst/htmlwidgets/lib/frappe-charts

it’s most likely because

    src = "inst/htmlwidgets/lib/frappe-charts"

should be relative to `inst`.

## Replace the example data with another data set and example

  - [changelog: 8fd703](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/8fd703a08b021b8466171b83506f5fb0bf92f2ac)

The first demo mixes chart types and we don’t want to do that. Use the
example from [Basic
Chart](https://frappe.io/charts/docs/basic/basic_chart#adding-more-datasets).

``` js
const data = {
    labels: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    datasets: [
      { name: "R", values: [18, 40, 30, 35, 8, 52, 17, -4] },
      { name: "Python", values: [30, 50, -10, 15, 18, 32, 27, 14] }
    ]
}
```

Then re-create this data in an R chunk
([changelog: 881c12](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/881c12ffdbdaa017863c918f61fa6208400d6130):

``` r
data <- list(
  labels = c("Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"),
  datasets = list(
    list(name = "R", values = c(18, 40, 30, 35, 8, 52, 17, -4)),
    list(name = "Python", values = c(30, 50, -10, 15, 18, 32, 27, 14))
  )
)
```

To get the data out of R and make it available in the document,
`htmlwidgets` embeds the data in a `<script
type="application/json">...</script>` element in the page. Embed the
data from the R chunk in a `<script>` tag with an ID so that we can find
it later.

``` r
tags$script(
  id = "data",
  type = "application/json",
  htmlwidgets:::toJSON(data)
)
```

Change to `js4shiny::html_document_js()` so that we can see the
`console.log()` from JavaScript just like R code. And then find the
`<script>` tag and get it’s `.textContent`.

``` js
let rData = document.getElementById('data')
rData.textContent
```

Use `JSON.parse()` to turn the data into a JS object and replace the
data used in the chart
([changelog: 7201e4](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/7201e436e72ebddee271cbf7c02a733ac81a5d86).

``` js
let rData = document.getElementById('data')
rData = JSON.parse(rData.textContent)
```

Switch between `data` and `rData` and it should be the same\!

Change the values of the data in the R side to be random so that each
re-run gives a new plot.

~~Delete the `data` in the JS side.~~ Comment out the `data` on the JS
side (but we’ll want to see the structure later).

## Augment data to set options for the chart

  - [changelog: 3e1d9b](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/3e1d9bee03fdf621f5dc5ec46e0e92f603ebe219)

Embed `data` in another list `opts` that will carry additional options,
such as `title`, `type` and `colors`.

Parse the embedded `<script>` and pass the whole object to
`frappe.Chart()`.

Change the colors to

  - `#466683` (dark blue)
  - `#44bc96` (green)
  - `#d33f49` (red)
  - `#993d70` (purple)

## Learn about other options for line charts

  - [changelog: 340d51](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/340d516ee4c7788e4f7e5089c4957ee9ffd1333e)

Read <https://frappe.io/charts/docs/basic/trends_regions> and add and
test additional line options.

Goal: shaded area chart with lines only.

Make the `labels` one week and repeat 4 times. Generate `runif(7 * 4)`
random numbers.

``` r
rep(c("Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"), 4)
```

Find and implement an option to reduce the number of labels on the
x-axis.

## Turn on dots again and make navigable

  - [changelog: 93d4c7](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/93d4c74f4b30a819b5c22fd7cc8ff238fc62f572)

<!-- end list -->

``` r
opts <- list(
  title = "My AwesomeR Chart",
  type = "bar",
  height = 250,
  colors = c("#466683", "#44bc96"),
  data = data,
  axisOptions = list(xIsSeries = TRUE),
  isNavigable = TRUE
)
```

## Add a real data source

  - [changelog: 7a9887](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/7a988739e3b5ff0572f4c16ce5110f52936550c3)

This is where you decide how much work you want to do on the R side and
how much work should be done on the JavaScript side. One thing is clear
though, R users should not be expected to construct the nested list data
structure just to use your HTML widget. We are used to data.frames and
tibbles, so these should be supported out of the box\!

To practice our JavaScript skills, I’ve chosen to do most of the work in
the browser. We’ll just ask our users to give us rectangular data, where
the first column will provide the x axis labels and the remaining
columns are series.

Sidenote: As you can tell, there’s a lot of validation that should
happen, but that part is not as much fun so we’re going to pretend our
users will always give us perfectly formatted data. If you do take on an
htmlwidget package project, *don’t skimp on this step*. Having a
friendly R API will have huge impact on the use of your widget.

We’ll use the `babynames` package for our demo dataset, pick two names
completely at random to compare.

``` r
library(dplyr)
library(babynames)

data <-
  babynames %>% 
  filter(
    name %in% c("Ruth", "August"),
    year >= 1980
  ) %>% 
  group_by(year, name) %>% 
  summarize(n = sum(n)) %>% 
  ungroup() %>% 
  pivot_wider(year, name, values_from = n)
```

At this point the chart won’t work, but you can use the browser dev
console to find the right steps to reformat the data into the expected
format.

We’ll make the **strong** assumption that the tibble in R should always
be formatted with the columns

1.  `labels`
2.  first series…
3.  second series…
4.  etc.

<!-- end list -->

  - `repl_example('reformat-r2js-data')`

<details>

<summary>Answer</summary>

``` js
const chartData = {labels: [], datasets: []}

// Get keys of data, assume that first entry is for labels, the rest are data
let labelColumn = Object.keys(x.data)[0]
let columns = Object.keys(x.data).slice(1)

// First column in x.data is the labels
chartData.labels = x.data[labelColumn]

// Create an appropriate object for each column, reformat data and add to chartData
columns.forEach(function(col) {
  chartData.datasets.push({name: col, values: x.data[col]})
})

x.data = chartData
```

</details>

## This is basically what `htmlwidgets` does, just inside a framework

  - [changelog:
    a0614d](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/a0614d9699aefc0eda82b3e368b48370be0ae9ba)

We now have all of the pieces of an `htmlwidget`, it’s just a bit less
coordinated.

1.  `htmlwidgets` gives us a slightly nicer way of specifying
    dependencies in `inst/htmlwidgets/frappeChart.yaml`. We’ll have to
    update that file.

2.  When we started we added a `div(id = "chart")`. It would be annoying
    to have to make sure that each `id` is always unique. `htmlwidgets`
    will add this `div` for us and give each one a unique id. We won’t
    have to write any code for this, it just happens.

3.  We’ll write an R function that will take input data and options and
    format it into a list, like the `opts` we’ve been using. Then we
    hand the data to `htmlwidgets` and it embeds it in a `<script>` tag
    for us. It will also find that data automatically and make it
    available on the JS side.

4.  Finally, we wrote some code in JavaScript to initialize the chart.
    In the same way, we’ll write some code in
    `inst/htmlwidgets/frappeChart.js` which is where we’ll reformat the
    data and options passed from the R world by htmlwidgets. We also
    need to instantiate the chart object. For advanced usage, this is
    also where we’ll put code that would let us update the widget in
    place without having to re-render the whole chart.

To create the htmlwidget, we’re going to work through each of these
pieces and put them in the right places.

# Make it an htmlwidget

## Declare dependencies

  - [changelog: 969fd9](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/969fd962edf0be8f98ffff1823f8e08960ffb31a)

**FILE:** `inst/htmlwidgets/frappeChart.yaml`

Take the `htmltools::htmlDependency()` and turn it into
`inst/htmlwidgets/frappeChart.yaml`.

``` r
rstudioapi::navigateToFile("inst/htmlwidgets/frappeChart.yaml")
```

Note: keep `htmlwidgets` in `src`\!

## Write the R function

  - [changelog:
    cbc25a](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/cbc25a8f7bbf7e522f369f5fec7c2517ba768656)

**FILE:** `R/frappeChart.r`

Add appropriate arguments to `frappeChart()`.

  - [title](https://frappe.io/charts/docs/reference/configuration#title)
  - [type](https://frappe.io/charts/docs/reference/configuration#type)
  - [colors](https://frappe.io/charts/docs/reference/configuration#colors)?
  - [is\_navigable](https://frappe.io/charts/docs/reference/configuration#isnavigable)

Structure the arguments into `x` and pass `...` for the “extra bits”.

Rebuild the package, then create a new R markdown document:
`js4shiny::js4shiny_doc()`.

Move the code loading `dplyr`, `tidyr`, `babynames` and formatting the
data. Then call `frappeCharts::frappeChart()`.

Render and open dev tools in the browser to see that it “works”. Meaning
that the data and dependencies are included, but the chart won’t. Point
out the random ID. Then go back and change it so we can find the element
better.

## Write JavaScript binding

  - [changelog: 6f141c](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/6f141c4341a2c4f8615df81887e7927d2e765f11)

**FILE:** `inst/htmlwidgets/frappeChart.js`

The final step is to move the Javascript we wrote before into the js
binding.

  - Just put in `console.log(x)`, rebuild, rerender
  - Verify that this `x` looks the same as our `opts` from before
  - Copy all of the JS we wrote to reconfigure the data into the widget
  - Use `el` instead of `#chart`
  - Rebuild, rerender
  - it works\!
  - Try adding other options

### Writing JavaScript in R

  - [changelog: 8d442e](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/8d442e3c842154adbae87dab5e9289cbb1333187)

The [tooltips](https://frappe.io/charts/docs/basic/annotations#tooltips)
can be formatted using the `tooltipOptions` property:

    tooltipOptions: {
        formatTooltipX: d => (d + '').toUpperCase(),
        formatTooltipY: d => d + ' pts',
    }

To write this in R (add to `widget_demo.R`)

``` r
tooltipOptions = list(
  formatTooltipX = htmlwidgets::JS("d => 'Year: ' + d"),
  formatTooltipY = htmlwidgets::JS("d => d + ' babies'")
)
```

## Shiny comes for free\!

  - [changelog: 739d59](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/739d5945010d5e46ab3f9847fd412beb0766805d)

Create a basic Shiny app with

1.  Slider input to pick number of values (1:26 letters)
2.  A new data button that generates new data of same dimension
3.  The data are reactive, `x = letters[1:n]`, `y = runif(n)`
4.  Use `frappeCharts::frappeChartOutput()` linked to
    `frappeCharts::renderFrappeChart()`
      - bar plot
      - fix `tooltipOptions` to turn the `runif()` into a percent.

`dev/shiny/app.R`

Make a mistake in the spelling for `formatTooltipY` and demo how hard it
is for the end user to track down what’s wrong. This points to how
important it is to do the validation on the R side or to do the extra
work to make the R API friendly.

It’s also a good place to demo debug strategies for Shiny and regular
widgets. Open the app in an external window, show the dev console, find
the frappeCharts binding and add a breakpoint. Then reload and show how
you an use the dev console there to figure things out.

## Better data updates

Frappe Charts, like many JS libraries, includes a method for updating
the widget without having to redraw the whole chart/plot/viz/etc.

In Frappe Charts, the [full data
update](https://frappe.io/charts/docs/update_state/modify_data#updating-full-data)
method is

``` js
chart.update(data)
```

where `data` is the `data` part of the initial options object.

Let’s make this work…

### Refactor the JS-side data processing code

[changelog:
b19e33](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/b19e33af8fdca579a8578bcd7a39c6d1e43fb32c)

  - Create a `prepareChartData()` function from the code we wrote for
    `renderValue()`. The goal is that this will let us use the function
    in multiple places.

#### Make `chart` generally available

[changelog:
d11459](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/d114592668ca63f06f593f4f247432eec218894b)

Make the created `chart` object available outside `renderValue()`

#### Expose the context inside the factory function to the world (and yourself)

[changelog:
f0a3bf](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/f0a3bf9fd5e60cda9b2b7ace004f360c36bf6610)

  - bind the factory function context to `el` as `widget`
  - Demo this by opening a rendered widget and showing `widget` as
    attached to the div

#### Expose `chart` with a `chart()` method

[changelog:
f0a3bf](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/f0a3bf9fd5e60cda9b2b7ace004f360c36bf6610)

Add a chart method to the widget object so that we can get to the
current chart.

1.  Demo by finding widget div and running
    
        let c = $0.widget.chart()
        c.addDataPoint(2017, [2500, 1500])

2.  Now, if nothing else, the `chart` object is accessible so others can
    use or extend it.

#### Create an update method that takes new data and updates an existing chart

[changelog: 5da4b6](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/5da4b68b5f60d8e6ee17cc8c4a009121539a2653)

Demo with `app.R`

``` js
let el = document.getElementById('chart')
el.widget.update({x: ['A', 'B', 'C', 'D'], Frequency: [1, 2, 3, 4]})
```

Try with various values. You can increase the number of data points but
you can’t add or change the series.

#### Add a custom message handler that dependes on `HTMLWidgets.shinyMode`

[changelog: 5da4b6](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/5da4b68b5f60d8e6ee17cc8c4a009121539a2653)

``` js
// after factory function
if (HTMLWidgets.shinyMode) {
  Shiny.addCustomMessageHandler('frappeCharts:update', function({id, data}) {
    let el = document.getElementById(id)
    el.widget.update(data)
  })
}
```

Restructure the app code so that the chart initializes with flat data
(0.5). Use `session$sendCustomMessage` to trigger the update.

Note that the JS function above takes `id` and `data` using
destructuring. It’s easy to write `function(id, data)` but this won’t
work because the handler can only take one argument.

Demo the app, now updates are fast\!

#### Write a user-friendly wrapper around `sendCustomMessage` called `updateFrappeChart()`

[changelog: 4706d8](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/4706d89183aaa9a3721599ef13c6f7af4955808b)

#### Now add an event listener to send chart navigation back to Shiny

[changelog: 0b4f7e](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/0b4f7ea16f378ec5a53d81260c8f9056fabbcaba)

Attach the event listener during `renderValue()` and watch for the
`data-select` event. Use the `el.id` to create a new id, like `el.id +
'_selected'`. Send back `index` and `values` from the event.

Add `verbatimTextOutput('selected')` to show `input$chart_selected`.

#### Return better values

[changelog:
e7fe0e](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/e7fe0e1d87977823e6a040434b33e6d5cdf8eac1)

You would probably want to do some work for the user and return more
meaningful values. We’ll probably just copy and paste this during the
workshop, but here’s a potential method.

This function basically reverses the chart processing and and returns a
list that should be a dataframe.

``` js
if (HTMLWidgets.shinyMode && x.isNavigable) {
el.addEventListener('data-select', function(ev) {
  let {index, values} = ev
  let chart = el.widget.chart()
  let label = chart.data.labels[index]
  let names = chart.data.datasets.map(d => d.name)
  let data = values.reduce(function(acc, v, idx) {
    acc[names[idx]] = v
    return acc
  }, {})
  data[labelsName] = label
  Shiny.setInputValue(el.id + '_selected', data)
})
}
```

#### Process the returned data for the user in Shiny

[changelog: 000de6](https://github.com/gadenbuie/js4shiny-frappeCharts/commit/000de60582f277e29983f6c5803de112ca1ade99)

But now in Shiny it needs to go from a list to a data.frame. To do this
we use `shiny::registerInputHandler()` in R and give the input event a
type: `inputId_selected:frappeCharts-selected`.

``` r
.onLoad <- function(libname, pkgname) {
  shiny::registerInputHandler(
    type = "frappeCharts-selected",
    fun = function(value, session, inputName) {
      as.data.frame(value, stringsAsFactors = FALSE)
    }
  )
}
```
