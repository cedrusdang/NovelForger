# ✒️ NovelForger: A LangGraph + Gemini AI Fiction Assistant

## 🎯 Motivation

AI writing tools often generate generic, surface-level content with limited structural coherence or emotional depth. More importantly, they fail to replicate the professional creative workflow of authors and editors in fiction publishing.

**NovelForger** addresses this gap by simulating a full-stack creative process for fiction writing—from structural plot design to AI-generated chapters, automated editorial review, and human-led orchestration. The goal is to create a three-layer workflow, AI-AI-Human, and a two-layer workflow, AI-AI.

---

## 💡 What We Built

**NovelForger** is a LangChain+LangGraph-powered AI system integrating multiple Gemini models into a modular, multilingual novel generation assistant.

The system walks users through:
- Language and genre selection
- Story structure and plot generation
- Character creation (guided or auto-generated)
- Dynamic chapter writing
- AI-based evaluation and rework loop
- Exporting to Markdown/CSV for further use

> It supports multiple languages, with English as the primary language, but also works fine with others. For example, its performance is excellent when trying to create a Vietnamese children's tale novel, even without human interactions.

![image](https://github.com/user-attachments/assets/db9341a8-c905-4887-a210-7455f2502a0b)
***LangChain - Gemini AI Agent's Structure Diagram***

---

## 🔁 Simulating Real Literary Workflows

Instead of simply generating chapters and asking for feedback, NovelForger replicates the **editorial workflow** found in real publishing teams:

1. **Plot Design First**  
 Structured story arcs are created using literary theory (e.g., Hero's Journey, Three-Act, Seven-Point Structure), simulating a professional story architect or head editor.

2. **Character Generation Next**  
 Characters are generated as thematic and structural components of the plot. Their traits and conflicts are aligned with story beats and tension curves.

3. **Contextual Content from Metadata**  
 Chapter context is not randomly invented — it is derived from structured metadata (`story_data`) and long-context grounding through embedding memories.

4. **AI Review and Scoring**  
 A separate Gemini agent reviews generated content and scores it based on internal coherence, tone, and plot fit. This can be used for fast revision and/or editing.

5. **Human-in-the-Loop Management**  
 The human user assumes a **senior editorial role** — overseeing both the creative AI (writer) and evaluative AI (editor), allowing:
   - Better control of narrative structure
   - Efficient decision-making through AI feedback
   - High-level orchestration over generation, revision, and finalization
 However, it is optional and can be fulfilled automatically.

## 🧠 GenAI Capabilities Demonstrated

- ✅ **Structured Output**  
 Output is consistently formatted in structured Markdown for readability and export, alongside Bolloe-style internal state management for chapters, characters, and plot elements. Includes classification logic (e.g., yes/no validation).

> Samples for Markdown formatting and classification boole style out put:

```python
# 0.2 Creative level
creative_level = (
    "Write with vivid imagination and emotional depth. Use expressive language, authentic tone, and unique details. "
    "Avoid clichés or generic phrasing. Be stylistically rich, aiming for a novel's literary quality. "
    "Each output should feel original, emotionally evocative, and highly creative."
)

if isinstance(last_msg, HumanMessage):
    validation_prompt = f"Is '{user_input}' is a valid genre or type of fictional novel? yes - no only."
    resp = invoke_with_retry(classificator, [HumanMessage(content=validation_prompt)]).content.strip().lower()
    if "yes" in resp:
        story_data["novel_type"] = user_input
        return messages + [AIMessage(content=f"Great! You've selected: **{user_input}**. Let's continue.")], True
```

- ✅ **Agents**  
 LangGraph orchestrates a modular, state-aware agent system, where each creative and evaluative step (e.g., language selection, plot generation, evaluation) is modelled as a discrete, controllable node.

- ✅ **Few-Shot Prompting (Semi-Structured)**  
 Story generation and character creation prompts include injected examples, schema patterns, and structural scaffolds to simulate guided creativity.

- ✅ **Long Context Window**  
 Gemini models utilize extended token windows to carry context between story stages, including cross-chapter coherence and character consistency.

Samples

```python
# === Step: Auto generation ===
if user_input == "auto":
    plot_prompt = (f"All output must in {story_data['language']}: create a plot outline that must not have any detail info about characters, as there is other later steps do do so,"
                    f"have plot level as: {plot_level} for a {story_data['novel_type']} novel."
                    f"Format as plain text summary. Creative level: {creative_level}."
                    f"Technique to use is: {storytelling_prompt}. Detail as {plot_level} and with format as {format_control}")
    plot = invoke_with_retry(creative_llm, [HumanMessage(content=plot_prompt)]).content.strip()
    story_data["plot"] = plot
    return messages + [AIMessage(content=f"Here's a plot idea:\n\n{plot}\n\nDo you want to proceed to character creation? (`yes` / `no`)")], "confirm"
```

- ✅ **Context Caching**  
 Metadata, plot outlines, and feedback from previous chapters are cached and reused to maintain thematic alignment and narrative flow.

Sample:

```python
improve_prompt = (
    f"All output must in {lang}. Revise the following chapter using this feedback. Preserve plot and structure.\n\n"
    f"{format_control_final}\n{creative_level}\n\n"
    f"Feedback:\n{story_data['qa_feedback_temp']}\n\nOriginal:\n{temp}"
)
```
- ✅ **Self-Evaluation & Rework Loop**  
 An AI-based reviewer scores chapters and triggers a re-generation process when structural or emotional criteria are not met — mimicking editorial review.

- ✅ **Embeddings & Long-Term Grounding**  
 Character and plot data are embedded into memory layers for vector-based retrieval during content generation and evaluation stages.

- ✅ **Multilingual Generation**  
 The system supports English and Vietnamese, with language-driven prompt routing and culturally adapted storytelling logic.

## 🧪 Implementation Highlights

- **Gemini Model Roles**:
  - `gemini-1.5-flash`: Used for input classification (e.g., language, genre). It's cost-efficient and fast — ideal for deterministic tasks with low latency.
  
  - `gemini-2.0-flash-exp`: Handles creative tasks (plot, characters, chapters) and structural evaluation. Balanced between response diversity and speed.
  
  - `gemini-1.5-pro`: Offer better handling of quotations and deeper evaluation, but were excluded due to quota constraints and higher inference cost during iterative generation loops.
  
- **LangGraph Agent Architecture**:
  - Each LangGraph node encapsulates one structured creative step (e.g., plot design, character generation, evaluation).
  
  - Nodes communicate through state transitions, with evaluation feedback enabling routing, looping, or regeneration.
  
  - The architecture supports rollback and conditional overrides, effectively simulating an editorial workflow.

- **Prompt Sampling Strategy**:
  
  - **Temperature** is dynamically adjusted between `0.6–0.9` based on the generation stage:
  
    - `0.6`: Used for classification and plot structuring to maintain focus and reduce hallucinations.
    
    - `0.9`: Applied to character and chapter generation to promote creativity, emotional depth, and variation.
      
  - **Top-k** is fixed at `1` for strict enforcement and auto (default for Gemini is 40) for another task.
  
  - These values were selected to optimize the trade-off between narrative coherence and imaginative output.
  
```python
from langchain_google_genai import ChatGoogleGenerativeAI
# Create LLM for each task
classificator = ChatGoogleGenerativeAI(
    model="models/gemini-1.5-flash-latest", 
    google_api_key=os.environ["GOOGLE_API_KEY"],
    temperature=0,            # creativity (0.0 = deterministic, 1.0 = diverse)
    top_p=1,                  # nucleus sampling
    max_output_tokens=10      # 15 characters only
)

creative_llm = ChatGoogleGenerativeAI(  # For plot generation
    model="models/gemini-2.0-flash-exp",
    google_api_key=os.environ["GOOGLE_API_KEY"],
    temperature=1,  
    top_p=0.7, 
    max_output_tokens=10000
)

strict_llm = ChatGoogleGenerativeAI(
    model="models/gemini-2.0-flash-exp",
    google_api_key=os.environ["GOOGLE_API_KEY"],
    temperature=0, 
    top_p=1,
    max_output_tokens=2048
)
```

- **Markdown-Only Output**:
  - Each generated chapter is exported as a Markdown block, including:
    - `## Title`
    - A summary
    - Richly formatted narrative content
  - Markdown formatting is prioritized over CSV to preserve paragraph structure, semantic flow, and readability — a central design goal for storytelling integrity.

## Final results

**Here are some parts of the novel, including the good and bad results:**

> **Langchain's chatbot menu:**

![image](https://github.com/user-attachments/assets/92fd67a0-d8a7-48a4-8cf3-9a6acf034d26)

> **High-quality and detailed plot and character build:**

![image](https://github.com/user-attachments/assets/6fe5cfe9-fba2-4d75-9d6a-e4b3be03018c)

> **Detailed review with instructions that can be used to rework the generation as an editor:**

![image](https://github.com/user-attachments/assets/315a97f5-33aa-4eda-b42c-f0740550e68a)

> **High quality, low rate of error Markdown formatting:**

![image](https://github.com/user-attachments/assets/2f69205c-13c7-45d9-8857-f0da1d7ee372)

> **The ability to work in another language, however, does have a medium rate of error, and the AI agent did skip the intruction and went back to English:**

![image](https://github.com/user-attachments/assets/39f38b0b-d753-487f-8606-94b2b9c195ea)

## ⚠️ Limitations & Future Work

The current token window constraint may cause long-form content to be truncated. This version does not implement an advanced long-context memory mechanism to address this.
- AI-generated novels tend to be lengthy, and instructions embedded in prompts can be partially skipped or diluted — reducing control over narrative direction.
- The number of chapters is not yet dynamically linked to the depth or arc of the plot, which may result in premature endings or underdeveloped narratives.

### Planned Improvements:

- Improve instruction adherence and reinforcement strategies to prevent prompt dilution.
- Add content formatting and editing layers to simulate human editorial polish better.
- Enable multiple evaluation passes with granular scoring to identify specific issues (e.g., emotional gaps, structural breaks, pacing errors).
- Expand narrative dimensions to strengthen inter-component connectivity and overall story cohesion.
- Design a more robust GUI to allow human editors to interactively review, annotate, and modify outputs with finer control.
- Translate high-level editorial goals into more AI-digestible instruction sets to align human intent and model output better.


## 🔗 Resources

- [🧠 Kaggle Notebook (Executable)](https://www.kaggle.com/code/cedrusdang/novelforger-gemini-langchain-ai-agent-novelist/edit/run/235129587)
- [📘 GitHub Repository (Blogpost/Source)](https://github.com/cedrusdang/NovelForger)
- [🖹 GitHub Notebook (Source Code)](https://github.com/cedrusdang/NovelForger/blob/main/novelforger-gemini-langchain-ai-agent-novelist.ipynb)
- [🎥 YouTube Demo Walkthrough](https://youtube.com/your-video-link)

---

## 📚 References & Frameworks Used

- K. Field, *Screenplay: The Foundations of Screenwriting*, Delta, 2005. (Three-Act Structure)
- J. Campbell, *The Hero with a Thousand Faces*, New World Library, 2008. (Hero's Journey)
- D. Harmon, "Story Circle Writing Framework." [Online]. https://channel101.fandom.com/wiki/Story_Structure_101
- R. Edgar, *The Seven-Point Story Structure*. https://helpingwritersbecomeauthors.com/secrets-of-story-structure/
- Y. Okawa, "Kishōtenketsu in Japanese Fiction," *Journal of Narrative Theory*, vol. 42, no. 3, 2012.
- L. Truby, *The Anatomy of Story*, Farrar, Straus and Giroux, 2007.
- R. McKee, *Story: Substance, Structure, Style*, HarperCollins, 1997.
- Google, *Gemini Prompt Engineering Guide*, 2025. https://ai.google.dev
- OpenAI, *Creative Prompting for Fiction Writing*, 2024. https://openai.com/research
- LangChain, *LangGraph Documentation*, 2025. https://docs.langchain.com/langgraph

---

Crafted as part of the **GenAI Intensive Capstone 2025 (Google x Kaggle)**.
