<!--yml
category: 未分类
date: 2024-05-27 14:56:14
-->

# OS-Copilot: Towards Generalist Computer Agents with Self-Improvement

> 来源：[https://os-copilot.github.io/](https://os-copilot.github.io/)

The design principle of FRIDAY aims to maximize generality by equipping the agent with the ability for self-refinement and self-directed learning. We first use an example to illustrate how FRIDAY operates and emphasize its capacity for self-refinement. Subsequently, we delve into how FRIDAY acquires the proficiency to control unfamiliar applications through self-directed learning.

In the figures above, we use a running example to demonstrate how FRIDAY functions within the OS.

Upon receiving the subtask “Change the system into the Dark mode” (step①), the Configuration Tracker employs dense retrieval to recall relevant information from the long-term memory to construct a prompt (step②). This prompt encompasses related tools, user profiles, OS system versions, and the agent’s working directory.

In this example, no suitable tools are identified (similarities below a specified threshold), prompting activation of the Tool Generator to devise an application-tailored tool for the current subtask (step③). As we can see from Figure (b), the generated tool manifests as a Python class utilizing AppleScript to change systems to dark mode.

Subsequently, with the tool created and the configuration prompt finalized, the Executor processes the prompt, generates an executable action, and executes it (step④). As shown in the bottom of Figure (b), the executor first stores the tool code into a Python file and then executes the code in the command-line terminal.

After execution, the critic evaluates whether the subtask is successfully completed (step⑤). Upon success, the critic assigns a score (using LLMs) ranging from 0 to 10 to the generated tool, with a higher score indicating greater potential for future reuse. In the current implementation, tools scoring above 8 are preserved by updating the tool repository in procedural memory (step ⑦).

However, in the event of a failed execution, the refiner collects feedback from the critic and initiates self-correction (step⑥) of the responsible action, tool, or subtask . The FRIDAY will iterate through steps ④ to ⑥ until the subtask is considered completed or a maximum of three attempts is reached.

Self-directed learning is a crucial ability for humans to acquire information and learn new skills, and it has demonstrated promising results in embodied agents within Minecraft games.

With a pre-defined learning objective, such as mastering spreadsheet manipulation, FRIDAY is prompted to propose a continuous stream of tasks related to the objective, spanning from easy to challenging. FRIDAY then follows this curriculum, resolving these tasks through trial and error, thereby accumulating valuable tools and semantic knowledge throughout the process. Despite its simple design, our evaluation indicates that self-directed learning is crucial for a generalpurpose OS-level agent.