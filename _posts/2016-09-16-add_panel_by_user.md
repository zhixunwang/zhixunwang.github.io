---
layout: post
title:  "Add Panels Based on Username in Shiny"
date:   2016-09-16 11:02:50
categories: [english, science]
comments: true
---

In Shiny, we can add a panel on-the-go based on the value from an input UI easily through `conditionalPanel(condition = "input.variable == 5")`. However, when it comes to achieving the same based on authenticated username for a private app, it's a little tricky.

Shiny stores the authenticated username - the email address - in `session$user`. Since `session` only lives in `shinyServer()`, user information cannot be read directly in `shinyUI()` by `conditionalPanel()`. We have to start in `shinyServer()` and pass something to `shinyUI()` to make this work.

### Solution 1 - `conditionalPanel()`
Though username cannot be referenced directly in `shinyUI()`, there is a work around - a text output can be passed to `shinyUI()` and be used as the condition. One trick is that by default, `renderText()` only really renders when it is called in `shinyUI()`. Therefore, we need to change the default behavior of `outputOptions()` to always render the text no matter whether it has an output in `shinyUI()` so we can use it only in the evaluation in `conditionalPanel()`. Below is the partial code to get it work.

```
shinyServer(function(input, output, session) {
  output$the_condition = renderText({
    if (
      # use is.null(session$user) so it still works when testing locally
      is.null(session$user)
      || session$user == 'itsme@showme.com'
    ) "YES!" else "NO!"
  })
  # set suspendWhenHidden to FALSE so it renders even without output
  outputOptions(output, 'the_condition', suspendWhenHidden = FALSE)
}

shinyUI(
  fluidPage(
    fluidRow(
        column(12,
               conditionalPanel(
                 # the condition
                 condition = "output.the_condition == 'YES!'",
                 # the output
                 h3("Here is the secret table"),
                 DT::dataTableOutput("a_beautiful_table")
               )
        ))
    )
  )
```

### Solution 2 - `insertUI()`
```
shinyServer(function(input, output, session) {
  if (
    is.null(session$user)
    || session$user == 'itsme@showme.com'
  ) {
    insertUI(
      selector = '#uitest1',
      ui = tags$div(
        h3("A Test"),
        a("My test 1", href="https://wangzhixun.com/", target="_blank")
      )
    )
  }
}

shinyUI(
  fluidPage(
    tags$div(id = 'uitest1')
    )
  )
```

### Solution 3 - `renderUI()`
```
shinyServer(function(input, output, session) {
  if (
    is.null(session$user)
    || session$user == 'itsme@showme.com'
  ) {
    output$uitest2 <- renderUI({
      tagList(
        h3("Another Test"),
        a("My test 2", href="https://wangzhixun.com/", target="_blank")
      )
    })
  }
}

shinyUI(
  fluidPage(
    uiOutput("uitest2")
    )
  )
```

The three solutions produce visually exactly the same output on the page for purpose of showing an additional piece of content on the page for certain users.  

* The first one might be logically more intuitive, but the execution is not what `renderText()` is designed for.  
* The second one is designed for creating removable UI parts.  

* The third one is a more powerful alternative for the first one. In addition to creating a new UI section based on some input or username, it allows you to even customize the values of the new input field based on other inputs or username. For example, you want everyone in the company to see the overall KPIs, and you want specific VPs to have an additional input field to select certain departments they are allowed to see. The additional input values can be changed based on the user and the KPIs will change based on which department they selected.  

### Conclusion
Use whichever makes sense to you to achieve the basic functions, but `renderUI()` gives you the most flexibility.
