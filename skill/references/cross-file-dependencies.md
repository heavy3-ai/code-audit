# Cross-File Dependency Gathering

Read this file when you reach step 4 of the Shared Review Pipeline in SKILL.md (code and PR reviews only; plan reviews skip it).

**How to gather:**

1. For each file in `changed_files`, extract its basename (e.g., `utils.py` from `src/utils.py`)
2. Search the project for files importing or referencing the changed files:
   ```bash
   grep -rn --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --include="*.py" --include="*.go" --include="*.rs" --include="*.java" \
     -e "import.*<basename_without_ext>" -e "from.*<basename_without_ext>" -e "require.*<basename_without_ext>" \
     --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=build --exclude-dir=dist --exclude-dir=__pycache__ --exclude-dir=.next --exclude-dir=venv --exclude-dir=.venv \
     --exclude-dir=locales --exclude-dir=locale --exclude-dir=i18n --exclude-dir=translations --exclude-dir=generated --exclude-dir=.generated \
     .
   ```
3. Filter results:
   - **Exclude** files already in `file_contents` (already being reviewed)
   - **Exclude** files already in `test_files`
4. **Cap at 20 files.** If more than 20 unique files found:
   - **Prioritize**: files that call changed functions over files that only import; files closer in the directory tree to the changed files
   - **Deprioritize**: files following the same repetitive pattern (e.g., many locale files all importing the same key)
   - Include the top 20 files with full snippets
   - Add one summary entry keyed `"(+N more files)"` listing just the remaining file paths, one per line (no content snippets)
5. For each included dependent file, capture import line(s) + ~3 lines of surrounding context
6. Add to context JSON as `dependent_files` dict (keyed by file path, values are the relevant snippets)
7. If no dependencies found, omit the `dependent_files` key entirely
