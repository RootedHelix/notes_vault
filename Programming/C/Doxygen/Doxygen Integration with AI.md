
Here’s a step-by-step workflow for integrating Doxygen documentation in your C project:

1. **Install Doxygen and Graphviz**  
   Make sure both are installed on your system.

2. **Create a Doxyfile**  
   Run `doxygen -g Doxyfile` in your project root to generate the config file.

3. **Configure Doxyfile**  
   - Set `EXTRACT_ALL = YES`
   - Set `GENERATE_LATEX = NO`
   - Set `HAVE_DOT = YES`
   - Set `CALL_GRAPH = YES`
   - Add your source, include, and docs folders to `INPUT`
   - Add Markdown files to `FILE_PATTERNS`

4. **Add Documentation Comments**  
   Use your IDE and AI to generate Doxygen-style comments for all public functions and structs.

5. **Write Module/Architecture Docs**  
   Create Markdown files (e.g., `docs/architecture.md`) summarizing subsystems and design.

6. **Run Doxygen**  
   Run `doxygen Doxyfile` to generate documentation.  
   The output will be in the `html/` folder.

7. **Review and Iterate**  
   - Open `html/index.html` to view docs.
   - Update comments and docs as code evolves.
   - Re-run Doxygen after changes.

You can generate Doxygen docs now to see the initial output and verify your setup. Then, keep updating comments and docs as you develop.

### 1) Doxygen → Markdown → MkDocs (most popular)

- Run Doxygen with **XML output** on.
    
- Convert the XML to Markdown with **Doxybook2**.
    
- Point MkDocs (ideally the **Material** theme) at the generated Markdown.  
    This gives you a fully themed, searchable site with your API pages living alongside hand-written guides. [GitHub+1](https://github.com/matusnovak/doxybook2?utm_source=chatgpt.com)
    

Minimal flow:

`doxygen -g # In Doxyfile: GENERATE_XML = YES XML_OUTPUT   = xml`