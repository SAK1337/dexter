# Project Context

## Purpose

Dexter is an autonomous financial research agent that uses AI-driven task planning, self-reflection, and real-time market data to answer complex financial research questions. It transforms complex financial queries into structured research tasks, executes them with live market data, validates results through self-reflection, and provides comprehensive, data-backed answers.

**Core Goals:**
- Provide intelligent, autonomous financial research capabilities
- Decompose complex queries into structured, executable tasks
- Fetch and analyze real-time financial data from multiple sources
- Self-validate results and iterate until confident in the answer
- Present findings in clear, data-rich responses

**Target Users:** Financial analysts, investors, researchers, and anyone needing quick access to structured financial data analysis.

## Tech Stack

### Runtime & Language
- **Bun** (v1.0+) - Fast JavaScript runtime with native TypeScript support
- **TypeScript** 5.9.3 - Strict mode enabled

### UI Framework
- **React** 19.2.0 - Component-based UI
- **Ink** 6.5.1 - React for terminal/CLI interfaces
- **ink-spinner**, **ink-text-input** - Terminal UI components

### LLM Integration
- **LangChain.js** - LLM orchestration framework
  - `@langchain/core` - Core abstractions
  - `@langchain/openai` - OpenAI provider (GPT-5.2)
  - `@langchain/anthropic` - Anthropic provider (Claude Sonnet 4.5)
  - `@langchain/google-genai` - Google provider (Gemini 3)
  - `@langchain/community` - Community integrations
  - `@langchain/tavily` - Web search integration

### Schema & Validation
- **Zod** 4.1.13 - Runtime type validation
- **zod-to-json-schema** 3.25.0 - Schema conversion for LLM structured output

### Build & Testing
- **Jest** 29.7.0 - Testing framework
- **ts-jest** - TypeScript support for Jest
- **Babel** 7.28.5 - Transpilation

### Configuration
- **dotenv** - Environment variable management

## Project Conventions

### Code Style

**File Naming:**
- kebab-case for all files (e.g., `task-executor.ts`, `use-agent-execution.ts`)
- `.tsx` extension for React components
- `.ts` extension for pure TypeScript modules

**Naming Conventions:**
- **Classes**: PascalCase (e.g., `Agent`, `PlanPhase`, `ToolExecutor`)
- **Functions**: camelCase (e.g., `callLlm`, `executeTasks`, `getChatModel`)
- **Types/Interfaces**: PascalCase (e.g., `AgentState`, `TaskResult`, `ToolError`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_MAX_ITERATIONS`)
- **React Components**: PascalCase (e.g., `AgentProgressView`, `TaskListView`)
- **React Hooks**: camelCase with `use` prefix (e.g., `useAgentExecution`, `useQueryQueue`)

**Code Organization:**
- Section headers use: `// ============================================================================`
- Related types grouped together with descriptive comments
- Centralized exports via `index.ts` files
- Path alias `@/*` maps to `src/` directory

**TypeScript:**
- Strict mode enabled
- Comprehensive type definitions for all phases and state
- Zod schemas for runtime validation of LLM outputs
- Avoid `any` types - use proper typing or `unknown` with type guards

### Architecture Patterns

**Multi-Phase Agent Architecture:**
```
Query → [Understand] → [Plan] → [Execute] → [Reflect] → (Loop) → [Answer] → Result
```

1. **Understand Phase**: Extract intent, entities (tickers, dates, metrics)
2. **Plan Phase**: Create task list with dependencies
3. **Execute Phase**: Run tasks with just-in-time tool selection
4. **Reflect Phase**: Evaluate data quality, decide to iterate or complete
5. **Answer Phase**: Synthesize final response

**Key Patterns:**
- **Factory Pattern**: `getChatModel()` for LLM provider selection
- **Callback Pattern**: Phase callbacks for UI integration (`onPhaseStart`, `onPlanCreated`, etc.)
- **Context Pattern**: `ToolContextManager` for cross-phase data sharing
- **Dependency Injection**: Options objects passed to constructors
- **Hook Pattern**: React hooks for state management and side effects

**Task Execution Model:**
- Tasks have types: `use_tools` (fetch data) or `reason` (LLM analysis)
- Tasks can declare dependencies on other tasks
- Dependency-aware scheduler enables parallel execution
- Just-in-time tool selection during execution (not planning)

**State Management:**
- React hooks for UI state (no Redux)
- Agent state passed through phase execution
- Tool results persisted to `.dexter/context/` directory

### Testing Strategy

**Framework:** Jest with ts-jest

**Test Commands:**
```bash
bun test          # Run all tests
bun test:watch    # Watch mode
```

**Testing Approach:**
- Unit tests for utility functions and helpers
- Integration tests for tool execution
- Type checking via `bun run typecheck`

**Test File Location:**
- Tests alongside source files or in `__tests__` directories
- Test files use `.test.ts` or `.spec.ts` suffix

### Git Workflow

**Branching:**
- `main` - Primary branch, stable code
- Feature branches for new development

**Commit Style:**
- Concise, descriptive commit messages
- Focus on what changed and why
- Recent examples:
  - "Add LLM provider selection"
  - "Add reflection"
  - "Add answer phase"
  - "Add ExecutedToolRegistry for tracking executed tools"

**Pull Requests:**
- Keep PRs small and focused
- Clear description of changes
- Test before merging

## Domain Context

### Financial Data Domain

**Supported Data Types:**
- **Fundamentals**: Income statements, balance sheets, cash flow statements
- **Market Data**: Real-time and historical stock prices, cryptocurrency prices
- **Metrics**: P/E ratio, debt-to-equity, ROE, ROA, profit margins
- **Filings**: SEC filings (10-K, 10-Q, 8-K)
- **Research**: Analyst estimates, financial news, insider trades
- **Segments**: Revenue breakdown by business segment

**Entity Types:**
- **Tickers**: Stock symbols (e.g., AAPL, MSFT, GOOGL)
- **Dates**: Specific dates or date ranges
- **Periods**: Fiscal quarters (Q1, Q2, Q3, Q4) and fiscal years
- **Metrics**: Financial ratios and measurements
- **Companies**: Company names mapped to tickers

**Common Query Patterns:**
- Revenue/earnings comparisons across quarters or years
- Financial ratio analysis (P/E, debt-to-equity, etc.)
- Trend analysis over time periods
- Company comparisons (competitive analysis)
- Filing and news research

### Agent Behavior

**Iteration Limits:**
- Default max iterations: 5
- Safety feature to prevent infinite loops
- Agent provides best available answer when limit reached

**Tool Selection:**
- Just-in-time selection during execute phase
- LLM chooses appropriate tools based on task requirements
- Tools can be called multiple times with different parameters

**Context Management:**
- Tool results cached in `.dexter/context/`
- MD5 hashing for deduplication (based on tool + args)
- Context loaded selectively for final answer synthesis

## Important Constraints

### Technical Constraints
- **Runtime**: Requires Bun v1.0+ (not compatible with Node.js without modification)
- **API Keys**: At least one LLM provider key required (OpenAI, Anthropic, or Google)
- **Financial Data**: Requires Financial Datasets API key for financial tools
- **Rate Limits**: Subject to API rate limits from all providers
- **Public Companies Only**: Financial data limited to publicly traded companies

### Safety Constraints
- **Max Iterations**: Hard limit of 5 iterations to prevent runaway execution
- **Loop Detection**: Built-in detection for repetitive task patterns
- **Timeout Handling**: API calls have timeout and retry logic

### Data Constraints
- **Data Availability**: Not all tickers covered by Financial Datasets API
- **Recency**: Recent quarters may not have data immediately available
- **Historical Depth**: Historical data availability varies by data type

## External Dependencies

### Required APIs

| Service | Purpose | Required |
|---------|---------|----------|
| **OpenAI API** | LLM provider (GPT-5.2) | One of three LLM providers required |
| **Anthropic API** | LLM provider (Claude Sonnet 4.5) | One of three LLM providers required |
| **Google AI API** | LLM provider (Gemini 3) | One of three LLM providers required |
| **Financial Datasets API** | Financial data (fundamentals, prices, filings) | Yes |

### Optional APIs

| Service | Purpose |
|---------|---------|
| **Tavily API** | Web search for supplementary research |
| **LangSmith** | Tracing and debugging LLM calls |

### API Endpoints

**Financial Datasets API** (`https://api.financialdatasets.ai`):
- `/financials/income-statements`
- `/financials/balance-sheets`
- `/financials/cash-flow-statements`
- `/prices`
- `/financial-metrics`
- `/company/facts`
- `/sec-filings`
- `/analyst-estimates`
- `/news`
- `/insider-trades`
- `/segmented-revenues`
- `/crypto/prices`

### Local Storage

- **`.dexter/context/`**: Cached tool results (JSON files)
- **`.dexter/settings.json`**: User preferences (selected model)
- **`.env`**: Environment variables and API keys
