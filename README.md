# Learning MCP + LangChain

Servers:
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Math")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

@mcp.tool()
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Weather")

@mcp.tool()
async def get_weather(location: str) -> str:
    """Get weather for location."""
    return "It's always sunny in New York"

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

Client Sync:

```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent
import asyncio
from langchain_ollama import ChatOllama


async def get_tools():
    client = MultiServerMCPClient(
        {
            "math": {
                "command": "python",
                # Replace with absolute path to your math_server.py file
                "args": ["math_server.py"],
                "transport": "stdio",
            },
            "weather": {
                # Ensure you start your weather server on port 8000
                "url": "http://localhost:8000/mcp",
                "transport": "streamable_http",
            }
        }
    )
    tools = await client.get_tools()
    return tools


async def get_result(agent, messages):
    result = await agent.ainvoke(
        {"messages": messages}
    )
    return result


def main_3():
    tools = asyncio.run(get_tools())
    print('---')
    for tool in tools:
        print(tool.name)
    print('---')
    chat_llm = ChatOllama(model='llama3.2:latest')
    agent = create_react_agent(
        chat_llm,
        tools
    )
    messages = [{"role": "user", "content": "what's (3 + 5) x 12?"}]
    result = asyncio.run(get_result(agent, messages))
    for message in result["messages"]:
        message.pretty_print()


if __name__ == '__main__':
    main_3()
```

Client Async:
```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent
import asyncio
from langchain_ollama import ChatOllama


async def main():
    client = MultiServerMCPClient(
        {
            "math": {
                "command": "python",
                # Replace with absolute path to your math_server.py file
                "args": ["math_server.py"],
                "transport": "stdio",
            },
            "weather": {
                # Ensure you start your weather server on port 8000
                "url": "http://localhost:8000/mcp",
                "transport": "streamable_http",
            }
        }
    )
    tools = await client.get_tools()
    print('---')
    for tool in tools:
        print(tool.name)
    print('---')
    chat_llm = ChatOllama(model='llama3.2:latest')
    agent = create_react_agent(
        chat_llm,
        tools
    )
    result = await agent.ainvoke(
        {"messages": [{"role": "user", "content": "what's (3 + 5) x 12?"}]}
    )
    for message in result["messages"]:
        message.pretty_print()

    result = await agent.ainvoke(
        {"messages": [{"role": "user", "content": "what is the weather in nyc?"}]}
    )
    for message in result["messages"]:
        message.pretty_print()


if __name__ == '__main__':
    asyncio.run(main())
```




## Credits

- [LangGraph | Use MCP](https://langchain-ai.github.io/langgraph/agents/mcp/#use-mcp-tools)