# Dexter 🤖

Dexter is an autonomous financial research agent that thinks, plans, and learns as it works. It performs analysis using task planning, self-reflection, and real-time market data. Think Claude Code, but built specifically for financial research.


<img width="979" height="651" alt="Screenshot 2025-10-14 at 6 12 35 PM" src="https://github.com/user-attachments/assets/5a2859d4-53cf-4638-998a-15cef3c98038" />

## Overview

Dexter takes complex financial questions and turns them into clear, step-by-step research plans. It runs those tasks using live market data, checks its own work, and refines the results until it has a confident, data-backed answer.  

**Key Capabilities:**
- **Intelligent Task Planning**: Automatically decomposes complex queries into structured research steps
- **Autonomous Execution**: Selects and executes the right tools to gather financial data
- **Self-Validation**: Checks its own work and iterates until tasks are complete
- **Real-Time Financial Data**: Access to income statements, balance sheets, and cash flow statements
- **Safety Features**: Built-in loop detection and step limits to prevent runaway execution

[![Twitter Follow](https://img.shields.io/twitter/follow/virattt?style=social)](https://twitter.com/virattt)

<img width="996" height="639" alt="Screenshot 2025-11-22 at 1 45 07 PM" src="https://github.com/user-attachments/assets/8915fd70-82c9-4775-bdf9-78d5baf28a8a" />


### Prerequisites

- [Bun](https://bun.com) runtime (v1.0 or higher)
- OpenAI API key (get [here](https://platform.openai.com/api-keys))
- Financial Datasets API key (get [here](https://financialdatasets.ai))
- Tavily API key (get [here](https://tavily.com)) - optional, for web search

#### Installing Bun

If you don't have Bun installed, you can install it using curl:

**macOS/Linux:**
```bash
curl -fsSL https://bun.com/install | bash
```

**Windows:**
```bash
powershell -c "irm bun.sh/install.ps1|iex"
```

After installation, restart your terminal and verify Bun is installed:
```bash
bun --version
```

### Installing Dexter

1. Clone the repository:
```bash
git clone https://github.com/virattt/dexter.git
cd dexter
```

2. Install dependencies with Bun:
```bash
bun install
```

3. Set up your environment variables:
```bash
# Copy the example environment file (from parent directory)
cp env.example .env

# Edit .env and add your API keys
# OPENAI_API_KEY=your-openai-api-key
# FINANCIAL_DATASETS_API_KEY=your-financial-datasets-api-key
# TAVILY_API_KEY=your-tavily-api-key
```

## Usage

### Starting Dexter

Run Dexter in interactive mode:
```bash
bun start
```

Or with watch mode for development:
```bash
bun dev
```

### CLI Commands

While running Dexter, you can use these commands:

| Command | Description |
|---------|-------------|
| `/model` | Switch between LLM providers (OpenAI, Anthropic, Google) |
| `/clear` | Clear the current conversation and start fresh |
| `/quit` or `/exit` | Exit Dexter |

### Query Queue

Dexter supports queueing multiple queries. Simply type your next question while the agent is processing, and it will be added to the queue and executed automatically when the current query completes.

### Example Queries

Try asking Dexter questions like:
- "What was Apple's revenue growth over the last 4 quarters?"
- "Compare Microsoft and Google's operating margins for 2023"
- "Analyze Tesla's cash flow trends over the past year"
- "What is Amazon's debt-to-equity ratio based on recent financials?"
- "Show me insider trading activity for NVDA in the last 6 months"
- "What are analyst estimates for Tesla's next quarter earnings?"
- "Get the latest 10-K filing highlights for Amazon"
- "Compare P/E ratios of FAANG stocks"
- "What's the current Bitcoin price and 30-day trend?"

### How Dexter Processes Your Query

Dexter uses a multi-phase approach:

1. **Understand Phase**: Extracts your intent, identifies tickers, dates, metrics, and time periods
2. **Plan Phase**: Creates a structured task list with dependencies
3. **Execute Phase**: Runs tasks in parallel (when possible), fetching real-time data
4. **Reflect Phase**: Evaluates data quality and decides whether to iterate or complete
5. **Answer Phase**: Synthesizes all findings into a comprehensive response

The agent may iterate multiple times (up to 5 by default) if it determines more data is needed.

### Available Financial Tools

Dexter has access to 18+ financial data tools:

| Category | Tools |
|----------|-------|
| **Fundamentals** | Income statements, balance sheets, cash flow statements |
| **Prices** | Real-time and historical stock prices, cryptocurrency prices |
| **Metrics** | P/E ratio, debt-to-equity, ROE, ROA, and more |
| **Filings** | 10-K, 10-Q, 8-K SEC filings |
| **Research** | Analyst estimates, financial news, insider trades |
| **Segments** | Revenue breakdown by business segment |
| **Search** | Web search via Tavily (optional) |

### Context Management

Dexter stores tool results in `.dexter/context/` to:
- Avoid re-fetching the same data
- Reduce API calls and latency
- Enable data reuse across iterations

To clear cached data:
```bash
rm -rf .dexter/context/
```

---

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# Required: At least one LLM provider API key
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key      # Optional
GOOGLE_API_KEY=your-google-api-key            # Optional

# Required: Financial data access
FINANCIAL_DATASETS_API_KEY=your-financial-datasets-api-key

# Optional: Web search capability
TAVILY_API_KEY=your-tavily-api-key

# Optional: LangSmith tracing (for debugging)
LANGSMITH_API_KEY=your-langsmith-api-key
LANGSMITH_PROJECT=dexter
LANGSMITH_TRACING=true
```

### LLM Provider Selection

Dexter supports multiple LLM providers:

| Provider | Model | Best For |
|----------|-------|----------|
| OpenAI | GPT-5.2 | General use, fast responses |
| Anthropic | Claude Sonnet 4.5 | Complex reasoning, longer context |
| Google | Gemini 3 | Alternative option |

Switch providers anytime using the `/model` command in the CLI.

### Settings File

Dexter stores settings in `.dexter/settings.json`:

```json
{
  "selectedModel": "openai"
}
```

---

## Troubleshooting

### Installation Issues

#### Bun not found after installation

**Symptom**: `bun: command not found`

**Solutions**:
1. Restart your terminal after installing Bun
2. Add Bun to your PATH manually:
   ```bash
   # macOS/Linux - add to ~/.bashrc or ~/.zshrc
   export BUN_INSTALL="$HOME/.bun"
   export PATH="$BUN_INSTALL/bin:$PATH"
   ```
3. On Windows, run PowerShell as Administrator and reinstall

#### Dependencies fail to install

**Symptom**: `bun install` fails with errors

**Solutions**:
1. Clear Bun's cache:
   ```bash
   bun pm cache rm
   bun install
   ```
2. Delete `node_modules` and reinstall:
   ```bash
   rm -rf node_modules bun.lockb
   bun install
   ```
3. Ensure Bun is v1.0 or higher:
   ```bash
   bun --version
   bun upgrade
   ```

### API Key Issues

#### "API key not found" or "Invalid API key"

**Symptom**: Error message about missing or invalid API key on startup

**Solutions**:
1. Verify your `.env` file exists in the project root
2. Check that keys are correctly formatted (no quotes needed):
   ```bash
   OPENAI_API_KEY=sk-...
   ```
3. Ensure no trailing whitespace in your API keys
4. Verify the key is valid by testing directly:
   ```bash
   curl https://api.openai.com/v1/models \
     -H "Authorization: Bearer $OPENAI_API_KEY"
   ```

#### "Rate limit exceeded"

**Symptom**: API calls fail with rate limit errors

**Solutions**:
1. Wait a few minutes and try again
2. Upgrade your API tier for higher limits
3. Dexter has built-in retry logic with exponential backoff—it will automatically retry

#### Financial Datasets API errors

**Symptom**: Financial data tools return errors or empty results

**Solutions**:
1. Verify your `FINANCIAL_DATASETS_API_KEY` is set correctly
2. Check your API subscription tier—some data may require higher plans
3. Ensure the ticker symbol is valid (e.g., `AAPL` not `Apple`)
4. For historical data, verify the date range is within available data

### Runtime Issues

#### Dexter hangs or becomes unresponsive

**Symptom**: No output or progress for extended periods

**Solutions**:
1. Check your internet connection
2. Verify API services are operational
3. Press `Ctrl+C` to cancel and restart
4. Check for stuck iterations—max iterations limit (5) should prevent infinite loops

#### "Maximum iterations reached"

**Symptom**: Dexter stops after 5 iterations without a complete answer

**Explanation**: This is a safety feature to prevent infinite loops.

**Solutions**:
1. Rephrase your query to be more specific
2. Break complex questions into smaller parts
3. The agent will still provide the best answer it can with available data

#### TypeScript/Compilation errors

**Symptom**: Errors during startup about types or compilation

**Solutions**:
1. Run type checking:
   ```bash
   bun run typecheck
   ```
2. Ensure you're using Bun v1.0+:
   ```bash
   bun upgrade
   ```
3. Reinstall dependencies:
   ```bash
   rm -rf node_modules && bun install
   ```

### Data Quality Issues

#### Incomplete or missing financial data

**Symptom**: Some financial metrics return null or incomplete data

**Possible causes**:
1. **Ticker not covered**: Not all stocks are in the Financial Datasets database
2. **Data not yet available**: Recent quarters may not have data yet
3. **Private companies**: Financial data is only available for public companies

**Solutions**:
1. Try a different time period
2. Use web search for supplementary data (requires Tavily API key)
3. Check if the company is publicly traded

#### Stale or cached data

**Symptom**: Data seems outdated despite expecting fresh results

**Solution**: Clear the context cache:
```bash
rm -rf .dexter/context/
```

### UI/Display Issues

#### Terminal output looks garbled

**Symptom**: Strange characters or broken formatting

**Solutions**:
1. Ensure your terminal supports Unicode and colors
2. Try a different terminal emulator (iTerm2, Windows Terminal, Hyper)
3. Set terminal encoding to UTF-8
4. On Windows, use Windows Terminal instead of CMD

#### Progress indicators not updating

**Symptom**: Phase indicators or task lists don't update

**Solutions**:
1. Ensure your terminal window is large enough
2. Try maximizing the terminal window
3. Some older terminals don't support Ink's refresh—use a modern terminal

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| `ECONNREFUSED` | Cannot connect to API | Check internet connection and API status |
| `401 Unauthorized` | Invalid API key | Verify API key in `.env` |
| `429 Too Many Requests` | Rate limited | Wait and retry, or upgrade API tier |
| `ENOENT: no such file` | Missing file/directory | Run `bun install`, check project structure |
| `Cannot find module` | Missing dependency | Run `bun install` |
| `Zod validation error` | Unexpected LLM response | Retry query, or report as bug if persistent |

### Debugging Tips

1. **Enable LangSmith tracing** for detailed execution logs:
   ```bash
   LANGSMITH_TRACING=true
   LANGSMITH_API_KEY=your-key
   LANGSMITH_PROJECT=dexter
   ```

2. **Check the context directory** to see what data was fetched:
   ```bash
   ls -la .dexter/context/
   cat .dexter/context/*.json | jq .
   ```

3. **Run tests** to verify the setup:
   ```bash
   bun test
   ```

4. **Check for updates**:
   ```bash
   git pull origin main
   bun install
   ```

### Getting Help

If you're still experiencing issues:

1. Check existing [GitHub Issues](https://github.com/virattt/dexter/issues)
2. Open a new issue with:
   - Your Bun version (`bun --version`)
   - Your operating system
   - The full error message
   - Steps to reproduce the issue
3. Follow [@virattt](https://twitter.com/virattt) for updates

## Architecture

Dexter uses a multi-agent architecture with specialized components:

- **Planning Agent**: Analyzes queries and creates structured task lists
- **Action Agent**: Selects appropriate tools and executes research steps
- **Validation Agent**: Verifies task completion and data sufficiency
- **Answer Agent**: Synthesizes findings into comprehensive responses

## Tech Stack

- **Runtime**: [Bun](https://bun.sh)
- **UI Framework**: [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) (terminal UI)
- **LLM Integration**: [LangChain.js](https://js.langchain.com) with multi-provider support (OpenAI, Anthropic, Google)
- **Schema Validation**: [Zod](https://zod.dev)
- **Language**: TypeScript


### Changing Models

Type `/model` in the CLI to switch between:
- GPT 4.1 (OpenAI)
- Claude Sonnet 4.5 (Anthropic)
- Gemini 3 (Google)

## How to Contribute

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

**Important**: Please keep your pull requests small and focused.  This will make it easier to review and merge.


## License

This project is licensed under the MIT License.

