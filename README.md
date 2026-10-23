![oreilly-logo](images/oreilly.png)

# AI Agents A-Z

## Human approval for RCA remediation

The workflow at `.github/workflows/remediation-approval.yml` provides the Gate 1 approval step
for `ManasaWW/emitter-proj-2`. It shows the RCA, proposed branch, patch, and optional regression
test in the Actions run summary. The remediation job targets the protected GitHub Environment
`remediation-approval`; GitHub pauses it until an authorized reviewer approves. Only then does
the job obtain an Entra token through GitHub OIDC and call the remediation API.

### Configure GitHub Actions

1. Open **Settings > Environments** and create an environment named exactly
   `remediation-approval`.
2. Add the authorized people or teams under **Required reviewers**. Enable **Prevent
   self-review** and restrict deployment branches to trusted branches where available.
   Required-reviewer protection depends on repository visibility and GitHub plan.
3. Add these environment variables:

   | Variable | Value |
   | --- | --- |
   | `AZURE_CLIENT_ID` | Entra application/client ID configured for GitHub OIDC |
   | `AZURE_TENANT_ID` | Entra tenant ID that owns the remediation API |
   | `REMEDIATION_API_SCOPE` | API Application ID URI, e.g. `api://sre-agent-mesh` |
   | `REMEDIATION_AGENT_JSONRPC_URL` | HTTPS endpoint ending `/agents/remediation/v1/jsonrpc` |

   These are identifiers and URLs, not secrets. Do not put access tokens or private keys in
   workflow inputs or run summaries.

### Configure Entra OIDC and API authorization

1. In the Entra app registration identified by `AZURE_CLIENT_ID`, add a federated credential:
   - **Issuer:** `https://token.actions.githubusercontent.com`
   - **Subject:** `repo:ManasaWW/emitter-proj-2:environment:remediation-approval`
   - **Audience:** `api://AzureADTokenExchange`
2. Expose the remediation API app role `Remediation.Invoke` and assign it to the enterprise
   application/service principal represented by `AZURE_CLIENT_ID`.
3. Configure the API to validate token issuer, tenant, audience, and the `Remediation.Invoke`
   role. This workflow does not need an Azure subscription role; it requests a token only for
   the API.

### Run and review Gate 1

1. Push this workflow to the default branch (`main`) if it is not already there.
2. Open **Actions > Approve RCA remediation > Run workflow**.
3. Enter the RCA correlation UUID, a branch beginning `fix/auto-heal-`, root-cause summary,
   unified diff, and optional regression test. The target repository is fixed to this repository.
4. Open the run summary and inspect the RCA and expanded patch/test.
5. Open the waiting `invoke-remediation` deployment and approve or reject it. If rejected or
   cancelled, the Azure remediation endpoint is not called.
6. After approval, inspect the job result. It requires a successful JSON-RPC response, passed
   sandbox status, and a PR URL belonging to `ManasaWW/emitter-proj-2`.

Step 3 does not automatically dispatch this workflow yet. Start it manually and transfer the RCA
fields into the workflow inputs. To automate dispatch later, a trusted service should call
GitHub's `workflow_dispatch` API using a narrowly scoped GitHub App installation token with
Actions write permission. The run will still wait for the same environment approval.

This is Gate 1, not the complete remediation pipeline. The current remediation backend remains a
scaffold: it does not clone the source repository, push a branch, or open a real PR. Rejection
feedback is retained in GitHub Actions history but is not yet persisted to Cosmos DB negative
memory. Do not use this to authorize real repository changes until the backend is completed and
verified.

This repository contains code for both my live course: [O'Reilly Live Online Training for AI Agents A-Z](https://learning.oreilly.com/live-events/ai-agents-a-z/0642572007604) and my video series: [Modern Automated AI Agents: Building Agentic AI to Perform Complex Tasks
](https://learning.oreilly.com/course/modern-automated-ai/9780135414965/)

This course provides a comprehensive guide to understanding, implementing, and managing AI agents both at the prototype stage and in production. Attendees will start with foundational concepts and progressively delve into more advanced topics, including various frameworks like CrewAI, LangChain, and AutoGen as well as building agents from scratch using powerful prompt engineering techniques. The course emphasizes practical application, guiding participants through hands-on exercises to implement and deploy AI agents, evaluate their performance, and iterate on their designs. We will go over key aspects like cost projections, open versus closed source options, and best practices are thoroughly covered to equip attendees with the knowledge to make informed decisions in their AI projects.


## Setup Instructions


### Using Python 3.11 Virtual Environment

At the time of writing, we need a Python virtual environment with Python 3.11.

#### Option 1: Python 3.11 is Already Installed

##### Step 1: Verify Python 3.11 Installation

```bash
python3.11 --version
```

##### Step 2: Create a Virtual Environment

```bash
python3.11 -m venv .venv
```

This creates a `.venv` folder in your current directory.

##### Step 3: Activate the Virtual Environment

- **macOS/Linux:**
  
  ```bash
  source .venv/bin/activate
  ```

- **Windows:**
  
  ```cmd
  .venv\Scripts\activate
  ```

You should see `(.venv)` in your terminal prompt.

##### Step 4: Verify the Python Version

```bash
python --version
```

##### Step 5: Install Packages

```bash
pip install -r requirements.txt
```

##### Step 6: Deactivate the Virtual Environment

```bash
deactivate
```

---

#### Option 2: Install Python 3.11

If you don’t have Python 3.11, follow the steps below for your OS.

##### **macOS (Using Homebrew)**

```bash
brew install python@3.11
```

##### **Ubuntu/Debian**

```bash
sudo apt update
sudo apt install python3.11 python3.11-venv
```

##### **Windows (Using Windows Installer)**

1. Go to [Python Downloads](https://www.python.org/downloads/release/python-3110/).
2. Download the installer for Python 3.11.
3. Run the installer and ensure **"Add Python 3.11 to PATH"** is checked.

### Verify Installation

```bash
python3.11 --version
```

## Notebooks

In the activated environment, run

```bash
python3 -m jupyter notebook
```

- **Using 3rd party agent frameworks**

	- **[Intro to SmolAgents](notebooks/SmolAgents.ipynb)** - An introductory notebook for HuggingFace's SmolAgents

	- **[Intro to Google ADK](notebooks/Google_ADK_Hello_World.ipynb)** - An introductory notebook for Google's Agent Development Kit with Google Search and custom tools
	
	- **[Intro to CrewAI](notebooks/CrewAI_Hello_World.ipynb)** - An introductory notebook for CrewAI

		- See **[the streamlit directory](./streamlit)** for an example of deploying crew on a streamlit app
	
	- **[Intro to Autogen](notebooks/Autogen_HelloWorld.ipynb)** - An introductory notebook for Microsoft's Autogen
	
- **OpenAI**

	- **[Intro to OpenAI Swarm](notebooks/Swarm_Hello_World.ipynb)** - An introductory notebook for OpenAI's Swarm
	
	- **[Intro to OpenAI Agents](notebooks/OpenAI%20Agents.ipynb)** - An introductory notebook for OpenAI's newer Agents SDK

- **LangGraph**
	

	- **[LangGraph Workflows 101](notebooks/LangGraph_Hello_World.ipynb)** - An introductory notebook for LangGraph making a RAG workflow
		
		-  **[Evaluating LangGraph Workflows](notebooks/LangGraph_Workfow_Eval.ipynb)** - Evaluating our RAG example from above

	-  **[Simple ReAct Agents in LangGraph](notebooks/LangGraph_React.ipynb)** - Simple ReAct Agent with tools in Langgraph.

		-  **[ReAct Agents in LangGraph using Ollama](notebooks/LangGraph_React_Local_LLMs.ipynb)** - Use local llama models for your agents

	-  **[ReAct Agents in LangGraph + MCP + Tool Positional Bias](notebooks/LangGraph_React%20-%20MCP%20+%20Tool%20Selection.ipynb)** - Integrating MCP with a ReAct Agent in Langgraph + Testing for Positional Bias

	-  **[Simple MCP Integration](notebooks/simple_mcp.ipynb)** - Connecting a ReAct Agent to an MCP server for tool use

	- **[LangGraph Agents playing Chess](https://colab.research.google.com/drive/1NMb4H8q-N0ZgEdaiDd6qUaBLD21yLejD?usp=sharing)** - An implementation of two ReAct Agents playing Chess with each other

	- **[LangGraph Middleware](notebooks/langgraph_middleware.ipynb)** - Chat compression, guardrails, and more!

	- **[Reasoning LLM Agents](notebooks/reasoning_llm_agents.ipynb)** - Using reasoning-capable LLMs (e.g. o1, o3) as ReAct agents

- **Evaluating Agents**

	- **[Evaluating Agent Output with Rubrics](notebooks/Evaluating_LLMs_with_Rubrics.ipynb)** - Exploring a rubric prompt to evaluate generative output. This notebook also notes positional biases when choosing between agent responses.

		- **[Advanced - Evaluating Alignment](notebooks/evaluating_alignment.ipynb)** - A longer notebook doing a much more in depth analysis on how an LLM can judge agent's responses

 	- **[Evaluating Tool Selection](notebooks/agent_positional_bias_tools.ipynb)** - Calculating the accuracy of tool selection between different LLMs and quantifying the positional bias present in auto-regressive LLMs. See the additions [here for V3 + DeepSeek Distilled Models](notebooks/agent_positional_bias_tools%20-%20DEEPSEEK%20edition.ipynb) and [here for DeepSeek R1](notebooks/agent_positional_bias_tools%20-%20DEEPSEEK%20R1.ipynb) and [here for Llama 4](notebooks/agent_positional_bias_tools%20-%20Llama%204.ipynb)


- **Multi-Agent Systems**

	- **[Parallel Node Execution](notebooks/multi_agent/parallel.ipynb)** - Running LangGraph nodes in parallel with fan-out/fan-in patterns

	- **[Dynamic Agent Network](notebooks/multi_agent/dynamic_network.ipynb)** - A dynamic multi-agent network where agents can route to each other using a generic handoff tool

	- **[Supervisor Pattern](notebooks/multi_agent/supervisor.ipynb)** - A supervisor agent that delegates tasks to specialized sub-agents

- **Building our own agent framework**
	
	- **[First Steps with our own Agent](https://colab.research.google.com/drive/14jAlW2E7ya_aS1M6eUsuHciC1WvLfIif?usp=sharing)** - Working towards building our own agent framework
	
	- See **[Squad Goals](https://github.com/sinanuozdemir/squad-goals)** for a very simple example of my own agent framework
	
		- **[Intro to Squad Goals](notebooks/SquadGoals_Hello_World.ipynb)** - using my own framework to do some basic tasks
		- **[Multimodal Agents](notebooks/squad_visual_agent.ipynb)** - Incorporating Dalle-3 to allow our squad to generate images


- **Modern Agent Paradigms**
	
	-  **[Plan & Execute Agents](notebooks/LangGraph_Plan_Execute.ipynb)** - Plan & Execute Agents use a planner to create multi-step plans with an LLM and an executor to complete each step by invoking tools.

	-  **[Reflection Agents](notebooks/LangGraph_Reflect.ipynb)** - Reflection Agents combine a generator to perform tasks and a reflector to provide feedback and guide improvements.
  
 	-  Using open source [Qwen VL 72B](https://colab.research.google.com/drive/1TYqAtnk1m_gLpCF5KY8WaNNOkfcYuBsy?usp=sharing) to grab bounding boxes of elements
  
  	-  Amazon's Nova Act for Browser Use in Action
  		-   run `python nova_apt.py --caltrain_city "Dogpatch" --bedrooms 2 --baths 2` in the notebooks directory

  	-  **[Computer Use with Reasoning LLMs](notebooks/computer_use_reasoning.ipynb)** - Choose a reasoning LLM and let it try to use my machine by pointing and clicking (🚨**WARNING THIS CODE WILL ALLOW AN AI TO USE YOUR LOCAL MACHINE**🚨)

  	-  **[Computer Use Reasoning Agent](notebooks/computer_use_reasoning_agent.ipynb)** - A LangGraph-based agent for GUI automation with PyAutoGUI and vision models (🚨**WARNING THIS CODE WILL ALLOW AN AI TO USE YOUR LOCAL MACHINE**🚨)

## Instructor

**Sinan Ozdemir** Sinan is a former lecturer of Data Science at Johns Hopkins University and the author of multiple textbooks on data science and machine learning. Additionally, he is the founder of the recently acquired Kylie.ai, an enterprise-grade conversational AI platform with RPA capabilities. He holds a master’s degree in Pure Mathematics from Johns Hopkins University and is based in San Francisco, CA.
