Core Modules & Classes
langgraph.graph: Module for defining graph structures.

StateGraph(StateType):
Syntax: graph = StateGraph(MyStateType)
Explanation: Main class to start building a graph. MyStateType (a TypedDict) defines the data schema passed between nodes.
langgraph.prebuilt: Module for common, pre-defined agent patterns.

ToolExecutor(tools: list):
Syntax: tool_executor = ToolExecutor(my_tools)
Explanation: Creates a runnable node to execute a list of LangChain tools.

create_react_agent(...):
Syntax: agent_app = create_react_agent(llm, tools)
Explanation: Helper to quickly set up a complete ReAct-style agent graph.

langgraph.checkpoint: Module for persisting graph state.

MemorySaver():
Syntax: checkpointer = MemorySaver()
Explanation: In-memory state persistence (good for testing/dev).

SqliteSaver.from_conn_string(conn_string: str):
Syntax: checkpointer = SqliteSaver.from_conn_string("sqlite:///path/to/db.sqlite")
Explanation: Persists state to a SQLite database file.

Graph Definition Functions (methods of StateGraph)
add_node(name: str, node_function: Runnable):
Syntax: graph.add_node("my_node", my_function_or_runnable)
Explanation: Adds a computational step to the graph, named name. node_function is a Python callable or LangChain Runnable.

add_edge(start_node: str, end_node: str):
Syntax: graph.add_edge("node_A", "node_B")
Explanation: Defines a direct, unconditional transition from start_node to end_node.

add_conditional_edges(start_node: str, condition: Callable, conditional_edge_mapping: dict):
Syntax: graph.add_conditional_edges("decider", condition_func, {"option_A": "node_X", "option_B": "node_Y"})
Explanation: Adds transitions where the condition function (applied to start_node's output) returns a key that maps to the next node in conditional_edge_mapping.

set_entry_point(node_name: str):
Syntax: graph.set_entry_point("start_node")
Explanation: Specifies where graph execution begins.

set_finish_point(node_name: str):
Syntax: graph.set_finish_point("final_node")
Explanation: Designates a node as a valid end point for graph execution.

compile(checkpointer: BaseCheckpointSaver = None, ...) -> CompiledGraph:
Syntax: app = graph.compile(checkpointer=MemorySaver())
Explanation: Finalizes the graph definition, validates it, and returns an executable LangChain Runnable. Enables persistence if checkpointer is provided.

Important Constants & Types
START:
Syntax: graph.set_entry_point(START)
Explanation: A constant representing the implicit initial input to the graph, often used with set_entry_point.

END:
Syntax: graph.add_edge("final_step", END) or {"finish": END}
Explanation: A constant signifying the termination of a path in the graph.

TypedDict (from typing):
Syntax: class MyStateType(TypedDict): my_key: str
Explanation: Used to define the schema of the graph's state object, ensuring type safety.

Annotated[list[BaseMessage], add_messages] (from typing & langgraph.graph.message):
Syntax: messages: Annotated[list[BaseMessage], add_messages]
Explanation: Annotation for list[BaseMessage] within StateType that ensures new messages are appended rather than overwriting the list, crucial for conversational memory.


Define State: Create a TypedDict to define the schema of your AgentState, usually with Annotated[list[BaseMessage], add_messages] for conversational memory.

Initialize Graph: workflow = StateGraph(AgentState)
Define Nodes: Create Python functions or LangChain Runnables that take the state as input and return a dictionary of updates to the state.
def call_llm(state): ... return {"messages": [ai_message]}
def execute_tools(state): ... return {"tool_output": result}
def decide_action(state): ... return {"next": "call_tool" if needs_tool else "end"}

Add Nodes to Graph: workflow.add_node("llm_node", call_llm)
Set Entry Point: workflow.set_entry_point("llm_node") (or START to imply direct input)

Add Edges: Define the flow.
workflow.add_edge("llm_node", "decide_action")
workflow.add_conditional_edges("decide_action", lambda state: state["next"], {"call_tool": "execute_tools", "end": END})
workflow.add_edge("execute_tools", "llm_node") (This creates a cycle!)

Compile Graph: app = workflow.compile(checkpointer=MemorySaver())

Invoke/Stream: app.invoke({"messages": [HumanMessage(content="Hello")]})
