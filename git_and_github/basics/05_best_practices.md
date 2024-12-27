# Best Practices #

## **0. Effective Tactics** 
For **Git** can turn your repository into a place of smooth management, clarity, and collaboration. The following methods act as lighthouse, showing the way to create more meaningful pledges and improve your workflow:

---

## **1. Commit Message Guidelines**
A good commit message explains what changes were made and why. Here's a simple format:

### **Format of a Commit Message**:
1. **Title (Subject Line)**:
   - Keep it short (50-72 characters).
   - Use the imperative mood (e.g., "Add feature X" instead of "Added feature X").
   - Clearly describe the purpose of the commit.

   Example:
   ```bash
   git commit -m "Fix navigation bar alignment on mobile"
   ```

2. **Description (Optional)**:
   - If the change is complex, provide more context in the description.
   - Mention:
     - Why the change was made.
     - How the change was implemented.
   - Separate the subject line and description with a blank line.

   Example:
   ```bash
   git commit
   ```

   Commit message:
   ```
   Refactor user authentication logic

   Split the user authentication logic into reusable modules to reduce code duplication and simplify testing.
   ```

---

## **2. Commit Early and Often**
- **Why**:
  - Frequent commits ensure you can easily isolate changes and debug issues.
  - Small commits are easier to review and understand.

- **How**:
  - Break down large tasks into smaller, logical changes.
  - For example:
    - One commit for "Add login form UI."
    - Another for "Implement login functionality."

---

## **3. Use Atomic Commits**
- **What It Means**:
  - Each commit should contain one logical change.
  - Avoid bundling unrelated changes into the same commit.

- **Why**:
  - Easier to understand and revert commits if needed.

- **Example**:
  Instead of:
  ```plaintext
  Add user profile page, fix navbar bug, and refactor CSS
  ```

  Break it into:
  ```plaintext
  Add user profile page
  Fix navbar bug on small screens
  Refactor CSS for better readability
  ```

---

## **4. Stage Only What’s Relevant**
- Use `git add` carefully to stage only the changes you want in a specific commit.
- Use interactive staging (`git add -p`) to select specific parts of a file:
  ```bash
  git add -p
  ```
  This ensures unrelated changes don't accidentally end up in the commit.

---

## **5. Write Descriptive Branch Names**
- Use clear branch names to describe the feature or bug fix you’re working on:
  ```plaintext
  feature/user-authentication
  bugfix/navbar-alignment
  chore/update-dependencies
  ```

- This makes it easy to identify the purpose of a branch at a glance.

---

## **6. Test Before Committing**
- Ensure that the code works and passes tests (if applicable) before committing.
- This avoids pushing broken commits to the repository.

---

## **7. Avoid Committing Sensitive Information**
- Double-check that you’re not committing:
  - API keys
  - Passwords
  - Configuration files with sensitive data (e.g., `.env`).
- Use a `.gitignore` file to prevent sensitive files from being tracked:
  ```plaintext
  .env
  node_modules/
  *.log
  ```

---

## **8. Rebase and Squash When Appropriate**
- Use `git rebase` or squash commits to clean up your history before merging:
  - Combine multiple "work-in-progress" commits into a single commit for clarity.
  - Example:
    ```bash
    git rebase -i HEAD~3
    ```

---

## **9. Use Tags for Milestones**
- Tag commits that represent key points in the project (e.g., releases, checkpoints):
  ```bash
  git tag -a v1.0 -m "First stable release"
  git push origin --tags
  ```

---

## **10. Collaborate Effectively**
1. **Use Pull Requests**:
   - Push feature branches to remote repositories and open pull requests for review.
   - Example workflow:
     ```bash
     git checkout -b feature/login-form
     git push origin feature/login-form
     ```

2. **Review Commits**:
   - Review commits in pull requests to ensure code quality and consistency.
   - Provide constructive feedback to collaborators.

---

## **11. Use Tools to Enforce Practices**
- **Lint Commit Messages**:
  Use tools like [Commitlint](https://commitlint.js.org/) to enforce consistent commit messages.
- **Pre-Commit Hooks**:
  Automatically run tests, format code, or lint files before committing:
  ```bash
  npx husky-init && npm install
  ```

---

## **12. Example of an Ideal Commit History**
Here’s what a clean commit history looks like:
```plaintext
3f2a1b2 Add user authentication feature
e1c9b8a Fix navbar alignment issue on mobile
7a5c6d3 Refactor CSS for better readability
39f10a2 Add unit tests for login functionality
```

---

### **Key Takeaways**
- Keep commits **small and focused**.
- Write **clear commit messages**.
- Stage changes carefully to ensure **atomic commits**.
- Clean up your commit history before merging to keep it readable.

These techniques will improve the professionalism of your job and make your repository not only more sustainable in maintenance but also more navigable. Please get in touch if you want to investigate any of these techniques more closely.
