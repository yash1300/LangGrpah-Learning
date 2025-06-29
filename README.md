🔧 Core Modules & Classes
langgraph.graph – For graph definition
StateGraph(StateType):
graph = StateGraph(MyStateType)
Starts graph creation. MyStateType is a TypedDict defining state schema.

langgraph.prebuilt – Pre-defined agents
ToolExecutor(tools: list):
tool_executor = ToolExecutor(my_tools)
Runnable node to execute LangChain tools.

create_react_agent(...):
agent_app = create_react_agent(llm, tools)
Quickly creates a full ReAct-style agent graph.

langgraph.checkpoint – For saving state
MemorySaver():
checkpointer = MemorySaver()
In-memory persistence (good for dev/testing).

SqliteSaver.from_conn_string(conn_string):
checkpointer = SqliteSaver.from_conn_string("sqlite:///path/to/db.sqlite")
Persists to a SQLite DB file.

🔁 Graph Definition Methods (StateGraph)
add_node(name, node_function):
Adds a node. Example: graph.add_node("step", my_func)

add_edge(start, end):
Simple connection. Example: graph.add_edge("A", "B")

add_conditional_edges(start, condition, edge_map):
graph.add_conditional_edges("decide", cond_fn, {"yes": "do_this", "no": "end"})
Adds branching logic based on output.

set_entry_point(node_name):
Example: graph.set_entry_point("start")

set_finish_point(node_name):
Example: graph.set_finish_point("end")

compile(checkpointer):
Finalizes and returns an executable graph.
app = graph.compile(checkpointer=MemorySaver())

🔣 Important Constants & Types
START:
Represents graph input. Use in set_entry_point(START)

END:
Signifies graph termination. Use in edge mapping: {"done": END}

TypedDict:
class MyStateType(TypedDict): my_key: str
Defines type-safe state schema.

Annotated[list[BaseMessage], add_messages]:
Ensures messages list appends instead of overwriting.

🛠️ Workflow Setup Example
Define State:
class AgentState(TypedDict):
messages: Annotated[list[BaseMessage], add_messages]

Initialize Graph:
workflow = StateGraph(AgentState)

Define Nodes:
def call_llm(state): return {"messages": [ai_message]}
def execute_tools(state): return {"tool_output": result}
def decide_action(state): return {"next": "call_tool" if needs_tool else "end"}

Add Nodes & Edges:
workflow.add_node("llm_node", call_llm)
workflow.set_entry_point("llm_node")
workflow.add_edge("llm_node", "decide_action")
workflow.add_conditional_edges("decide_action", lambda s: s["next"],
                                {"call_tool": "execute_tools", "end": END})
workflow.add_edge("execute_tools", "llm_node")  # Creates a cycle

Compile & Run:
app = workflow.compile(checkpointer=MemorySaver())
app.invoke({"messages": [HumanMessage(content="Hello")]})
