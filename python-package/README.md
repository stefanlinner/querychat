# querychat for Python

Chat with your Shiny Python apps using natural language.

Imagine typing questions like these directly into your Shiny dashboard, and seeing the results in realtime:

* "Show only data from 2008 with highway MPG greater than 30."
* "What's the average city MPG for SUVs vs compact cars?"
* "Sort the data by highway fuel efficiency descending."

## Installation

```bash
pip install querychat
```

## How to use

First, you'll need access to an LLM that supports tools/function calling. querychat uses [chatlas](https://github.com/posit-dev/chatlas) to interface with various providers.

Here's a minimal example:

```python
import pandas as pd
from shiny import App, ui, reactive
import querychat

# 1. Configure querychat
querychat_config = querychat.init(my_dataframe)

# 2. Define the UI
app_ui = ui.page_sidebar(
    # Use the provided sidebar component
    sidebar=querychat.sidebar("chat"),
    ui.output_table("data_table")
)

# 3. Define server logic
def server(input, output, session):
    # Initialize querychat server
    chat = querychat.server("chat", querychat_config)
    
    # Use the filtered dataframe
    @output
    @render.table
    def data_table():
        return chat["df"]()

# Create Shiny app
app = App(app_ui, server)
```

## Features

querychat uses LLMs to generate SQL queries from natural language, offering:

- **Reliability**: LLMs are excellent at writing SQL but bad at direct calculation
- **Transparency**: SQL is always displayed to the user
- **Reproducibility**: Generated SQL can be copied and reused elsewhere

## Customization

### Configure with your own data

```python
querychat_config = querychat.init(
    df=my_dataframe,
    table_name="my_data",  # Optional: defaults to "data"
    greeting="Welcome! Ask me anything about the data.",
    data_description="""
    This dataset contains information about...
    - column1: Description of column 1
    - column2: Description of column 2
    """,
    extra_instructions="Please use British English spelling conventions."
)
```

### Use a different LLM provider

You can configure which LLM provider to use through chatlas:

```python
import chatlas

my_chat_func = lambda system: chatlas.Chat(
    provider="anthropic",
    model="claude-3-5-sonnet",
    system=system
)

querychat_config = querychat.init(
    df=my_dataframe,
    create_chat_func=my_chat_func
)
```

## How it works

querychat works by:

1. Converting your pandas DataFrame to a DuckDB table
2. Creating a system prompt with your data schema
3. Setting up tools that let the LLM execute SQL queries
4. Processing natural language to SQL queries
5. Returning filtered data to your Shiny app

See the `examples/` directory for more complete examples.