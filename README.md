# Step-by-Step Guide: Building an MCP Server for Cortex Agents with Snowflake

This guide will walk you through setting up a Model Connectivity Protocol (MCP) server for Cortex Agents, enabling you to connect AI applications like Cursor to your data in Snowflake.

## I. Setting Up Your Snowflake Environment

First, prepare your Snowflake account.

1.  **Sign Up for a Snowflake Trial Account:**
    * Go to the Snowflake website.
    * Sign up for a free 30-day trial account.
    * Provide your name, email, company details, and preferred region.
    * Complete the sign-up process and activate your account via email.

2.  **Get Data via Cortex Knowledge Extension:**
    * Log in to your Snowflake account.
    * Navigate to **Data Products** in the **Marketplace**.
    * Search for "Cortex knowledge extension for the Snowflake docs".
    * Select the extension and click **Get**.
    * Confirm by clicking **Get** again. This makes your Snowflake documentation searchable by Cortex agents.

3.  **Verify Cortex Search Service:**
    * In your Snowflake account, go to **AI & ML** > **Cortex Search**.
    * Select the database related to the Cortex knowledge extension (it will likely have a name similar to the extension).
    * You should see a "doc service" already created with an attached search service.

4.  **Create a Programmatic Access Token (PAT):**
    * Go to **Admin** > **Users & Roles**.
    * Select your user.
    * Click **Generate Token**.
    * Enter a name for the token (e.g., "MCP agents").
    * Set an expiration period (e.g., 1 day).
    * Ensure the token can access **Any** of your roles.
    * Click **Generate**.
    * **Important:** Copy the generated token to your clipboard. You will need it later.
    * **(Optional) Temporarily Bypass Network Policy:**
        * If you anticipate network policy restrictions, you can temporarily bypass them for this PAT.
        * Click **Edit** next to the network policy setting for the token.
        * Choose a temporary bypass duration (e.g., 8 hours).
        * Click **Grant Access**.

## II. Building the MCP Server

Next, set up the MCP server on your local machine.

1.  **Install UV (Universal Virtual Environment Manager):**
    * Open your terminal or command prompt.
    * Run the following command:
        ```bash
        curl -LsSf https://astral.sh/uv/install.sh | sh
        ```
    * Follow any on-screen prompts during the installation.

2.  **Clone the Repository:**
    * In your terminal, run the following command to clone the necessary files:
        ```bash
        git clone https://github.com/AkhilGurrapu/snowflake-mpc-cortex-agent.git
        ```
    * Navigate into the cloned repository directory:
        ```bash
        cd snowflake-mpc-cortex-agent
        ```

3.  **Activate UV Environment and Install Dependencies:**
    * Create and activate the UV virtual environment:
        ```bash
        uv venv
        ```
    * Activate the environment:
        * On macOS/Linux:
            ```bash
            source .venv/bin/activate
            ```
        * On Windows:
            ```bash
            .venv\Scripts\activate
            ```
    * Install the required Python packages (MCP SDK and HTTP client):
        ```bash
        uv add "mpc[cli]" httpx
        ```

4.  **Open the Repository in a Code Editor:**
    * Open the `snowflake-mpc-cortex-agent` folder in your preferred code editor (e.g., VS Code, Sublime Text).

5.  **Modify the Agent Setup (Removing Cortex Analyst components):**
    * In your code editor, open the `cortex_agents.py` file.
    * Locate and **comment out** (by adding a `#` at the beginning of the line) the line referencing `semantic_model_file`.
    * Locate and **remove or comment out** the sections of code related to `analyst_tool_resources`.
    * Locate and **remove or comment out** the sections of code related to `SQL_EXECUTION`.
    * Save the `cortex_agents.py` file.

6.  **Configure Environment Variables:**
    * In the `snowflake-mpc-cortex-agent` directory, find the `.env.template` file.
    * Create a copy of this file and rename the copy to `.env`.
    * Open the `.env` file in your code editor and fill in the following details:
        * **`SNOWFLAKE_ACCOUNT_URL`**:
            * Go back to your Snowflake account in your web browser.
            * Click on your username in the bottom left corner.
            * Select **Connect to Tool**.
            * Copy your **Account Identifier** (it looks like `youraccount.snowflakecomputing.com`).
            * Paste this value into the `.env` file for `SNOWFLAKE_ACCOUNT_URL`.
        * **`SNOWFLAKE_PAT`**: Paste the Programmatic Access Token (PAT) you copied earlier from Snowflake.
        * **`CORTEX_SEARCH_SERVICE`**:
            * In Snowflake, navigate to **AI & ML** > **Cortex Search**.
            * Find your doc service (related to the Snowflake docs extension).
            * Copy the **Key** of your doc service (it might be something like `KEY_DOC_SERVICES`).
            * Paste this value into the `.env` file in the format `<database>.<schema>.<service_name>`.
    * Save the `.env` file.

7.  **Run the MCP Server:**
    * Go back to your terminal, ensuring you are still in the `snowflake-mpc-cortex-agent` directory and the UV virtual environment is active.
    * Run the MCP server:
        ```bash
        uv run cortex_agents.py
        ```
    * You should see output indicating that the server is running. Keep this terminal window open and the server running.

## III. Integrating with Cursor

Finally, connect your running MCP server to the Cursor application.

1.  **Add MCP Host in Cursor:**
    * Open the Cursor application.
    * Go to **Cursor Settings** (often found via a gear icon or in the application menu).
    * Select the **MCP** section.
    * Click on **Add new global MCP server**.

2.  **Edit the MCP JSON Configuration:**
    * A JSON configuration file (usually `mcp.json`) will open in Cursor or your default JSON editor.
    * Add a new entry for your Cortex agent MCP server. It should look similar to this:

        ```json
        {
            "mcpServers": {
                "snowflake_docs_agent": {
                    "command": "uv",
                    "args": [
                        "--directory",
                        "/Users/akhilgurrapu/Documents/Projects/snowflake-mcp-cortex-agent",
                        "run",
                        "cortex_agents.py"
                    ]
                }
            }
        }
        ```
    * **Important Notes:**
        * Replace `"snowflake_docs_agent"` with your desired server name. **Crucially, use underscores (`_`) instead of dashes (`-`) in this name** due to a potential compatibility issue in Cursor.
        * Replace `"/Users/akhilgurrapu/Documents/Projects/snowflake-mcp-cortex-agent"` with the **absolute path** to the `snowflake-mcp-cortex-agent` directory on your computer.
    * Save the `mcp.json` file.

3.  **Verify MCP Server in Cursor:**
    * Go back to the **MCP** section in Cursor settings.
    * You should now see your newly added server (e.g., "snowflake_docs_agent") listed.
    * It should have a green status indicator, signifying that Cursor can connect to your running MCP server.

4.  **Add a Rule to Use MCP (Optional but Recommended):**
    * In Cursor settings, navigate to the **Rules** section.
    * Click **Add new rule**.
    * Create an instruction that tells Cursor when to use your MCP server. For example:
        * **Instruction Trigger:** "When I ask about Snowflake documentation" or "Use snowflake agent for Snowflake questions."
        * **MCP Server to Use:** Select your `snowflake_docs_agent` (or whatever you named it) from the dropdown.
    * Save the rule.

5.  **Test the Integration:**
    * In Cursor, switch to the **AI pane** or chat interface.
    * Ensure you are in **Agents mode** or a mode that allows agent interaction.
    * Ask a question related to the Snowflake documentation (e.g., "How do I create a virtual warehouse in Snowflake?").
    * Cursor should now utilize your MCP server (which connects to Snowflake Cortex Search) to help answer your question.

You have now successfully built an MCP server for Cortex Agents and integrated it with Cursor!
