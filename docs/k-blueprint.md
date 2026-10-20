# k blueprint

## Concept

k is an agentic system that coordinates multiple agents to work on user directives.

k agentic system:

- Agents:
  - Assistant: the main agent, communicate with user to understand and execute the directives, can summon other agents to work in particular tasks. Assistant lifecycle is limited within a thread.
  - Oracle: the second opinion, more thoughtful than the main agent, often collaborates with the Assistant to work on solution. Oracle lifecycle is limited within a task given by Assistant.
  - Librarian: specialized in research information from the Internet. Librarian lifecycle is limited within a task given by Assistant.
  - Smith: the auditor, verify the work of other agents to keep them in good shape. Smith lifecycle is limited within a task given by Assistant.
  - Each agent has its own instructions, message history, tools, LLM...
- Environment:
  - Custom tools.
  - MCP tools.
- Thread: a chat-like conversation between User and Assistant where they collaborate to achieve some goals, a thread is bound to a workspace, a workspace can have many threads, a thread can be persisted and restored for later use.

## Principles

The following principles were selected to help the system achieve the goals: reliable, maintainable, secure.

### Correctness

The agents must work on source of truth: the specification / documentation, and the tests.

- Specification: provide the essential design of the system, it only describes the most critical points of the system.
- Tests: the deterministic way to prove the system works as expected. It must include the needed tests when the situation requires: pure tests, IO tests, live tests, end-to-end tests, GUI tests, fuzz tests...

### Architecture

The high-level design of the system, must be explicitly defined to set the source of truth for the agents to develop the implementation.

- Modularization: the system is separated into multiple components with their own boundaries, contracts, data-structures.
- Isolation: each component is isolated from the other, has explicit boundaries, hides its implementation details, won't leak the details to the outside world, only exposes to the outside world its contracts.
- Inter-module contracts: the components interact with each other via defined contracts, with their own contract's data-structure or a shared data-structure (will be explicitly defined).
- Observation: the system behaviors must be visible via logging.
  - Log must eliminate sensitive data, e.g. credentials, personal identifiable information...
  - The pure function may not log anything.
  - Log on the integration points, inter-component connections.
  - INFO log has concise information enough to visualize the system behaviors.
  - DEBUG log has detailed information to facilitate debugging and abnormal behaviors investigation.

### Security

_(to be defined)_

### Performance

- Carefully consider the abstraction layers.
- Carefully consider the implementation complexity, algorithm complexity when writing production code.
- Prefer simplest, cleanest design to keep the system as small as possible.
- Performance comparison, improvement must be backed by profiling data, with relevant parameters: clock time, number of instructions, CPU cycles, memory consumption...

## Components

- Configuration:
  - User configuration regarding LLM, credentials and so on...
  - Internal configuration.
- State:
- Agent: the control flow of the coding agent.
- Environment: execute agent actions.
  - Local shell.
  - MCP.
- Model: connect to LLMs.
- Shared: contain shared data models to transfer data between components.
  - Shared-Messages: the messages were transported between agents / environments / models.

### Configuration

Configuration

- profiles:
  - low:
    - agents:
      - main: gpt-5.6-sol-medium
      - oracle: gpt-5.6-sol-high
      - librarian: gpt-5.6-sol-medium
      - smith: gpt-6-astra-medium
  - medium:
    - agents:
      - main: gpt-5.6-sol-high
      - oracle: gpt-5.6-sol-xhigh
      - librarian: gpt-5.6-sol-medium
      - smith: gpt-6-astra-medium
- environments:
  - tool-local-shell:
    - timeout-seconds: 30
  - mcp-unison: ...
- models:
  - gpt-5.6-sol-medium:
    - name: openai/gpt-5.6-sol
    - provider: openrouter
    - reasoning-effort: medium
  - gpt-5.6-sol-high:
    - name: openai/gpt-5.6-sol
    - provider: openrouter
    - reasoning-effort: high
  - gpt-5.6-sol-xhigh:
    - name: openai/gpt-5.6-sol
    - provider: openrouter
    - reasoning-effort: xhigh
  - gpt-6-astra-medium:
    - name: openai/gpt-6-astra
    - provider: openrouter
    - reasoning-effort: medium

### Thread

- Thread is the working unit in the system. It provides the API for user-agent interaction.
- Thread is not nested, but it could contain the messages of the specialist agent in specialist threads, these threads doesn't mean for interaction.

```unison
type Thread
  = {
    id : Uuid,
    messages : [ThreadMessage],
    specialist_threads : [SpecialistThread],
    agent : Agent
    -- Other fields omitted.
  }

type SpecialistThread
  = {
    id : Uuid,
    messages : [SpecialistThreadMessage],
    agent : Agent
    -- Other fields omitted.
  }

type ThreadMessage
  = UserMessage k.shared.model.UserMessage
  | BotMessage k.shared.model.BotMessage
  | BotThought k.shared.model.BotThought
  | ToolCall k.shared.environment.ToolCall
  | ToolResult k.shared.environment.ToolResult

type SpecialistThreadMessage
  = UserMessage k.shared.model.UserMessage
  | BotMessage k.shared.model.BotMessage
  | BotThought k.shared.model.BotThought
  | ToolCall k.shared.environment.ToolCall
  | StandardToolResult k.shared.environment.StandardToolResult

type k.shared.environment.ToolResult
  = SpecialistToolResult k.shared.environment.SpecialistToolResult
  | StandardToolResult k.shared.environment.StandardToolResult

type k.shared.environment.SpecialistToolResult
  = {
    id : Uuid,
    tool_call_id : Uuid,
    output : Text,
    thread_id : Uuid
  }
```

_(work in-progress)_

### Environment

```unison
use k.shared.environment ToolCall ToolDefinition ToolResult

structural ability k.environment.Environment where
  tool_catalogue :
    {Environment} [ToolDefinition]

  execute_tool :
    ToolCall
    ->{Environment} Either k.shared.environment.EnvironmentError ToolResult

structural type k.shared.environment.ErrorCode = ToolNotFound

type k.shared.environment.EnvironmentError =
  { code : k.shared.environment.ErrorCode
  , message : Text
  }
```

- `tool_catalogue`:
  - Return available tools for the agent, based on configuration.
  - Each agent role has different tool catalogue.
  - The catalogue is fixed during the life time of the environment.
- `execute_tool`:
  - Run the given tool and arguments.
  - Tool must be existed in the catalogue.

_(work in-progress)_

### Agent

```unison
type k.agent.Agent =
  { role : Text
  , current_turn : Optional k.agent.AgentTurn
  , status : k.agent.AgentStatus
  }

type k.agent.AgentOperation = Query | Revise

type k.agent.AgentTurn =
  { operation : k.agent.AgentOperation
  , messages : [k.shared.model.RequestMessage]
  , remaining_tool_calls : [k.shared.environment.ToolCall]
  }

type k.agent.AgentStatus
  = Ready
  | QueryingModel
  | ExecutingTools
  | InterruptedModelError k.shared.model.ModelError
  | InterruptedToolError k.shared.environment.EnvironmentError
  | StoppedIncomplete

type k.agent.ProcessingOutcome
  = Completed
  | Interrupted
  | Incomplete
  | Rejected

k.agent.Agent.submit_user_message :
  k.agent.Agent
  -> k.shared.model.UserMessage
  ->{k.environment.Environment, k.model.Model}
    (k.agent.Agent, k.agent.ProcessingOutcome)

k.agent.Agent.revise_user_message :
  k.agent.Agent
  -> k.shared.model.UserMessage
  ->{k.environment.Environment, k.model.Model}
    (k.agent.Agent, k.agent.ProcessingOutcome)

k.agent.Agent.retry :
  k.agent.Agent
  ->{k.environment.Environment, k.model.Model}
    (k.agent.Agent, k.agent.ProcessingOutcome)

k.agent.Agent._process_messages :
  k.agent.Agent
  ->{k.environment.Environment, k.model.Model}
    (k.agent.Agent, k.agent.ProcessingOutcome)
```

_(work in-progress)_

### Model

#### Contract

- The system initializes model based on configuration, instructions, tools, workspace information and credentials. These information remain fixed during the model lifetime, except the information need refreshing or updating such as credentials.
- Model holds the provider specific context internally, e.g. conversation history. Each agent has independent context.
- Agent interacts with model to send input and receive output.
- On query the provider, model receives from caller: messages (user message, tool results). Only one query can be run at a time.
- On receive response from provider, model provides: new messages (assistant messages, tool calls). The model context is only updated when the query is successful. 
- On receive error, model will return a specific error for the caller to react. The model context won't be updated.
- Model won't execute the tool calls, it's the job of other components.
- Model is extensible, supporting new provider won't affect the contract between caller and model.
- Model capabilities:
  - Text generation.
  - Tool call.
- A model lifecycle is started by the initialization and work for only 1 context. The model can be serialized for persistence and restored for later usage. When restoring the agent, the system must check the agent external dependencies such as tools, selected LLM... are still valid. Otherwise, reject new interaction.

#### Implementation

- Model.
  - Initialization:
    - Input:
      - Configuration.
      - Authentication.
      - Instructions.
      - Tools.
    - Output: model instance.
  - Query:
    - Input:
      - User message.
      - Tool results.
    - Output:
      - New reasoning messages.
      - New assistant messages.
      - New tool calls.
  - Serialize.
  - Restore.

- Model handles provider specific state internally.
- Model handles the API calling, connection handling, provider specific data structure...

Model ability

```unison
structural ability k.model.Model where
  query : ModelRequest ->{Model} Either ModelError ModelResponse
  revise : ModelRequest ->{Model} Either ModelError ModelResponse
  serialize : '{Model} Json

type k.shared.model.ModelRequest = { messages : [RequestMessage] }

type k.shared.model.RequestMessage = ToolResult ToolResult | UserMessage UserMessage

type k.shared.model.UserMessage = { id : Uuid, content : Text }

type k.shared.environment.ToolResult = SpecialistToolResult SpecialistToolResult | StandardToolResult StandardToolResult

type k.shared.environment.SpecialistToolResult = { id : Uuid, tool_call_id : Uuid, output : Text, thread_id : Uuid }

type k.shared.environment.StandardToolResult = { id : Uuid, tool_call_id : Uuid, output : Text }

type k.shared.model.ModelResponse = { status : ResponseStatus, messages : [ResponseMessage] }

type k.shared.model.ResponseStatus = Completed | Incomplete | NeedsToolResults

type k.shared.model.ResponseMessage = BotMessage BotMessage | BotThought BotThought | ToolCall ToolCall

type k.shared.model.BotMessage = { id : Uuid, content : Text }

type k.shared.model.BotThought = { id : Uuid, content : Text }

type k.shared.environment.ToolCall = { id : Uuid, name : Text, arguments : Json }

structural type k.shared.model.ErrorCode = Authentication | AccessDenied | BadRequest | ServiceUnavailable | InvalidResponse | InvalidRevision

type k.shared.model.ModelError = { code : ErrorCode, message : Text }
```

- A Model response with status `NeedsToolResults` must contain at least one tool call; otherwise, the Model handler must return `InvalidResponse` without updating its context.
- Model context is managed internally by the ability handler.
- Model.query: submit new messages to push the conversation further. Only 1 query is executed at a time.
- Model.revise: edit the existing message and re-submit to the LLM API for regeneration. This operation will update the context on success, preserve the preceding messages, replace the message content of the editing message, discard the following messages. When the request failed, leave the context unchanged. Discarding messages does not undo external effects.
- Model.serialize: transform internal context to JSON for storage.
- The JSON context can be used to initialize the Model.

```unison
type ModelProvider = CodexProvider | OpenRouterProvider
type ModelContext
  = {
    identifier: Text,
    configuration:
      name: Text,
      reasoning_effort: ?
    instructions: [Text],
    tools: ?
  }
```

#### Initialization

The initialization is the process to create a model. Depending on the configuration, a model will be created to use a specific LLM API with various LLM capability configured. All inputs must be ready before creating the model.

Input:

- Configuration:
  - Identifier.
  - Provider.
  - LLM name.
  - LLM configuration: capability, reasoning effort...
- Credentials:
  - API key: via environment variables.
  - Authentication state: via files.
- System prompt:
  - Base instruction.
  - Environment description.
- Tool catalogue.
  - Custom tools.
  - MCP tools.

Output: configured model, ready for actions.

#### Behaviors

The agent uses the model to perform some actions. Input and output changes on each turn. Supported action: query, revise.

Input:

- Instructions (from model).
- Tool catalogue (from model).
- Messages.
  - User messages.
  - Tool results.
- Opaque state (provider specific information).

Output:

- New messages
  - Assistant message.
  - Reasoning messages.
  - Tool calls.

#### Error handling

Model.query:

- The model context must be replayable so that when the query has error, we can retry the query without issue.
- Must use the stateless API so that the retry won't have any side-effect.
- The model context is only updated when the query is successful.
- The model contract will return an error result and won't handle additional action.
- The error types must be stored in the same place with the model contract.
- Each error must be mapped to an action.
- Possible errors on query:
  - Authentication: the credentials are missing or expired.
  - Access denied: the credentials are correct, but the provider doesn't authorize the request.
  - Bad request: any error about the request being rejected because of the input data, e.g. large context, wrong LLM configuration...
  - Service unavailable: timeout, no availability, internal failure...
  - Invalid response: the response is received but could not parse to return to the caller.

Model.serialize and restoration:

- Possible errors:
  - Bad structure.

#### Cancel a query

_(to be defined)_

