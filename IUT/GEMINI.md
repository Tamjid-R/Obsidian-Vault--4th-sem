# Project Workflow: Feynman Learning System

This vault is designed as a "Second Brain." To maintain consistency and clarity, every interaction regarding new information or academic subjects must follow this structure:

1.  **Detailed Note:** Update an existing comprehensive Markdown note containing all critical technical details, specifications, and academic content. **Do NOT create new `.md` files unless specifically instructed by the user.**
2.  **Feynman Explanation:** Include an intuitive explanation using the Feynman Technique (simple analogies, plain English, and "teaching it to a child" logic) **ONLY IF** the user includes the phrase "in simple terms" in their request.
3.  **Examples:** Provide concrete, real-world examples to illustrate the concepts. **Include these by default** as part of the technical note.
4.  **Q&A Formatting:** Any question/Statement found in a note starting with **Q.** must be answered immediately following it starting with **ANS:**. 
    - **Style:** For **ANS:** blocks, provide the answer in **details only**. Do **NOT** include Feynman-style explanations or examples within these blocks.
    - **Restriction:** You must **ONLY** answer existing questions starting with **Q.**. 
    - **Do NOT** create or invent new questions on your own unless specifically instructed by the user.
5.  **Path Persistence:** After every interaction, always ask the user if they want to stay in the current directory path or move to a new one.
    - If the user responds with "y" or does not specify a new path, remain in the same directory and only update files within that scope.
    - If the user mentions a new path, transition all future work to that directory.

**Localized Organization:**
- Subject-specific notes must reside within their respective subfolders (e.g., `/Human/University/3rd Semester/DCN/`).
- **Algorithm Course Specifics:**
    - All algorithms must be documented in `[[All Algorithms folder]]` located in `/Human/University/3rd Semester/Algorithm/`.
    - **Summary Table:** The top of the file must contain a Markdown table with the following columns:
        - Algorithm Name
        - Best Case Time
        - Average Case Time
        - Worst Case Time
        - Space Complexity
        - Use Case (When to use)
        - Keywords (Identification in questions)
    - This table must be updated whenever a new algorithm is added.
    - Each entry must include:
        - Heading with the Algorithm title.
        - C++ implementation.
        - Pseudocode.
        - Recursive analysis: Identify recursive calls, calculate recursion frequency, and provide recursive alternatives for every loop.
        - **Complexity Analysis:** Detail Time and Space complexity for every loop, and provide Best, Average, and Worst-case complexities for the overall algorithm.
        - **Algorithm Questions:** If a question is asked regarding a specific algorithm, update that algorithm's section in `[[All Algorithms folder]]` by inserting the question (starting with `Q.`) and its corresponding answer (starting with `ANS:`).
- Use Obsidian wiki-links `[[...]]` to interconnect related concepts.
