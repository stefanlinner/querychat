# querychat: Chat with Shiny apps

Imagine typing questions like these directly into your Shiny dashboard, and seeing the results in realtime:

* "Show only penguins that are not species Gentoo and have a bill length greater than 50mm."
* "Show only blue states with an incidence rate greater than 100 per 100,000 people."
* "What is the average mpg of cars with 6 cylinders?"

querychat is a drop-in component for Shiny that allows users to query a data frame using natural language. The results are available as a reactive data frame, so they can be easily used from Shiny outputs, reactive expressions, downloads, etc.

## How it works

### Powered by LLMs

querychat's natural language chat experience is powered by LLMs. You may use any model that [ellmer](https://ellmer.tidyverse.org) supports that has the ability to do tool calls, but we currently recommend (as of March 2025):

* GPT-4o
* Claude 3.5 Sonnet
* Claude 3.7 Sonnet

In our testing, we've found that those models strike a good balance between accuracy and latency. Smaller models like GPT-4o-mini are fine for simple queries but make surprising mistakes with moderately complex ones; and reasoning models like o3-mini slow down responses without providing meaningfully better results.

The small open source models (8B and below) we've tested have fared extremely poorly. Sorry. 🤷

### Powered by SQL

querychat does not have direct access to the raw data; it can _only_ read or filter the data by writing SQL `SELECT` statements. This is crucial for ensuring relability, transparency, and reproducibility:

- **Reliability:** Today's LLMs are excellent at writing SQL, but bad at direct calculation.
- **Transparency:** querychat always displays the SQL to the user, so it can be vetted instead of blindly trusted.
- **Reproducibility:** The SQL query can be easily copied and reused.

Currently, querychat uses DuckDB for its SQL engine. It's extremely fast and has a surprising number of [statistical functions](https://duckdb.org/docs/stable/sql/functions/aggregates.html#statistical-aggregates).

## How to use

First, you'll need an OpenAI API key. See the [instructions from Ellmer](https://ellmer.tidyverse.org/reference/chat_openai.html). (Or use a different LLM provider, see below.)

Here's a very minimal example that shows the three function calls you need to make.

```r
library(shiny)
library(bslib)
library(querychat)

# 1. Configure querychat. This is where you specify the dataset and can also
#    override options like the greeting message, system prompt, model, etc.
querychat_config <- querychat_init(mtcars)

ui <- page_sidebar(
  # 2. Use querychat_sidebar(id) in a bslib::page_sidebar.
  #    Alternatively, use querychat_ui(id) elsewhere if you don't want your
  #    chat interface to live in a sidebar.
  sidebar = querychat_sidebar("chat"),
  DT::DTOutput("dt")
)

server <- function(input, output, session) {

  # 3. Create a querychat object using the config from step 1.
  querychat <- querychat_server("chat", querychat_config)

  output$dt <- DT::renderDT({
    # 4. Use the filtered/sorted data frame anywhere you wish, via the
    #    querychat$df() reactive.
    DT::datatable(querychat$df())
  })
}

shinyApp(ui, server)
```

## Customizing

### Provide a greeting (recommended)

When the querychat UI first appears, you will usually want it to greet the user with some basic instructions. By default, these instructions are auto-generated every time a user arrives; this is slow, wasteful, and unpredictable. Instead, you should create a file called `greeting.md`, and when calling `querychat_init`, pass `greeting = readLines("greeting.md")`.

If you need help coming up with a greeting, your own app can help you! Just launch it and paste this into the chat interface:

> Help me create a greeting for your future users. Include some example questions. Format your suggested greeting as Markdown, in a code block.

And keep giving it feedback until you're happy with the result, which will then be ready to be pasted into `greeting.md`.

Alternatively, you can completely suppress the greeting by passing `greeting = ""`.

### Use a different LLM provider

Provide a `create_chat_func` function that takes a `system_prompt` parameter, and returns an Ellmer chat object. A convenient way to do this is with `purrr::partial`:

```r
querychat_config <- querychat_init(mtcars,
  create_chat_func = purrr::partial(ellmer::chat_claude, model = "claude-3-7-sonnet-latest")
)
```

This would use Claude 3.7 Sonnet instead, which would require you to provide an API key. See the [instructions from Ellmer](https://ellmer.tidyverse.org/reference/chat_claude.html).
