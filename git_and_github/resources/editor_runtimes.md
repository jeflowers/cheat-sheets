# Editor Runtimes ##

These runtime settings tailor the commit message environment to meet best practices and conventional commit standards. These configurations will help ensure your commit messages are clean, readable, and conform to best practices!

#### Summary of Best Practices Across Editors:
- **Wrap at 72 Characters**: Ensures readability in logs and emails.
- **Spell Check**: Avoids typos in important commit messages.
- **Trim Trailing Whitespace**: Keeps codebases clean.
- **No Tabs**: Uses spaces to avoid formatting inconsistencies.
- **Syntax Linting**: Enforces commit message conventions (Conventional Commits, etc.).

---

## Steps to Configure `vi` for Git Commit Messages ##

1. **Create a Git Commit Filetype Configuration**:
   - Create a directory if it doesn’t already exist:
     ```bash
     mkdir -p ~/.vim/ftplugin
     ```
   - Create a file named `gitcommit.vim` inside the `~/.vim/ftplugin` directory:
     ```bash
     vim ~/.vim/ftplugin/gitcommit.vim
     ```
NOTE: 
2. **Add the Configuration**:
   Add the following settings to the file:

   ```vim
   " Set textwidth to 72 for commit message body wrapping
   setlocal textwidth=72

   " Enable spell checking for commit messages
   setlocal spell spelllang=en_us

   " Highlight lines that exceed 72 characters
   highlight OverLength ctermbg=red ctermfg=white guibg=#FF0000 guifg=#FFFFFF
   match OverLength /\%73v.\+/

   " Ensure no auto-wrapping at arbitrary points
   setlocal formatoptions+=n

   " Highlight trailing whitespace
   match ErrorMsg /\s\+$/

   " Use spaces instead of tabs
   setlocal expandtab

   " Set tab size to 4 spaces
   setlocal shiftwidth=4 softtabstop=4
   ```

3. **Ensure Filetype Detection**:
   Vim automatically detects `COMMIT_EDITMSG` files as type `gitcommit`. If not, ensure filetype plugins are enabled by adding this to your `~/.vimrc`:
   ```vim
   filetype plugin on
   ```

---

### Explanation of Settings

- **`textwidth=72`**: Wraps commit message body at 72 characters.
- **`spell spelllang=en_us`**: Enables spell checking for English (US).
- **Highlight Lines Over 72 Characters**:
  - Uses `highlight` and `match` to visually flag overly long lines.
- **`formatoptions+=n`**: Disables automatic line breaking when typing.
- **Highlight Trailing Whitespace**:
  - Prevents unnecessary spaces at the end of lines.
- **`expandtab`** and **`shiftwidth=4`**: Uses spaces (not tabs) for indentation with 4-space width.

---

### Testing Your Configuration
- Open a commit message using `git commit` and ensure:
  - Text wraps at 72 characters.
  - Misspelled words are highlighted.
  - Overlong lines and trailing whitespace are flagged.


### Why Would vi use .vim directory for ftplugin ###

The reason **vi** often uses the `.vim/ftplugin` directory is that on most modern systems, when you invoke `vi`, it is aliased or linked to **Vim** (Vi IMproved), which is a more feature-rich version of the original `vi` editor. Here's why:

---

### Key Reasons for Using `.vim/ftplugin`
1. **Vim's Filetype-Specific Configuration**:
   - Vim supports **filetype plugins**, which allow you to set specific settings for different file types. For example, when editing `COMMIT_EDITMSG` files, Vim recognizes the `gitcommit` filetype.
   - These plugins are stored in `~/.vim/ftplugin/` for user-specific configurations.

2. **Backward Compatibility**:
   - Although Vim is more powerful than the original `vi`, it retains compatibility with `vi`. This ensures that traditional `vi` users can still use `vi` commands, but with added benefits like filetype detection.

3. **Filetype Plugin System**:
   - The `ftplugin` directory is specifically designed for configuring settings based on file types.
   - For example:
     - A `gitcommit.vim` file in `.vim/ftplugin/` applies settings only when editing Git commit messages.
     - A `python.vim` file can hold Python-specific settings.
   - This modular approach avoids cluttering the main `.vimrc` file.

4. **Automatic Filetype Detection**:
   - Vim automatically detects the file type of `COMMIT_EDITMSG` as `gitcommit` when you use `git commit`. This triggers the corresponding plugin file in `~/.vim/ftplugin/`.

---

### Does the Original `vi` Use `.vim/ftplugin`?
No, the original **`vi`** (from Unix) does not have the concept of `ftplugin` or advanced filetype-specific configurations. If you're truly using **`vi`** and not Vim:
- You would need to rely on global settings in `.exrc` or `.vimrc`.
- Filetype-specific features like `ftplugin` or `autocommands` would not be available.

---

### How to Confirm Whether You're Using Vim or Vi
Run the following command:
```bash
vi --version
```
- If you see something like `VIM - Vi IMproved`, you’re using Vim.
- If it outputs something minimalist without mentioning Vim, you’re using the original `vi`.

---

### What If You Want to Support Only Vi?
If you are using the original `vi` and not Vim, you would place the settings in the global configuration file `.exrc`, like this:
```ex
set wrapmargin=8  " Equivalent to 72 characters on an 80-column terminal
set spell         " Enable spell checking (if available)
```
However, many modern features like filetype-specific settings or highlighting are unavailable in traditional `vi`.

---

## Steps to Configure Vim Git Runtime ##
To create a Vim runtime configuration file for Git commit messages that sets `textwidth` to 72, you can follow these steps:

1. Create a Git-specific Vim runtime configuration file at `~/.vim/ftplugin/gitcommit.vim`.
2. Add the configuration to set `textwidth=72`.

Here's the content of the `gitcommit.vim` file:

```vim
" Set textwidth to 72 for Git commit messages
setlocal textwidth=72
```

### Explanation

- **`setlocal`**: Ensures the setting applies only to the current buffer (specific to Git commit messages).
- **`textwidth=72`**: Automatically wraps text at 72 characters.

### Why 72 Columns?

1. **Git Commit Messages**: Keeps messages readable when viewed with tools like `git log` or `git show`.
2. **Email Workflow**: Leaves room for nested reply indicators in an 80-column terminal.

### Additional Tips

- If you don’t have `filetype plugin on` enabled in your Vim configuration, add the following to your `~/.vimrc` to ensure the `gitcommit.vim` file loads for commit messages:
  ```vim
  filetype plugin on
  ```
- To verify it works, open a commit message editor using Git (e.g., `git commit`). The text should wrap at 72 characters.

---

##  Steps to Configure Vim Git Runtime Best Practices ##

Here are additional **best-practice Vim parameters** for editing Git commit messages to improve readability and help follow conventional commit standards:

### Enhanced Configuration for `gitcommit.vim`

```vim
" Set textwidth to 72 for proper wrapping
setlocal textwidth=72

" Enable spell checking for commit messages
setlocal spell spelllang=en_us

" Highlight text that exceeds the textwidth
highlight OverLength ctermbg=red ctermfg=white guibg=#FF0000 guifg=#FFFFFF
match OverLength /\%73v.\+/

" Use no line wrapping beyond textwidth
setlocal wrap

" Prevent auto-wrapping at arbitrary points
setlocal formatoptions+=n

" Highlight trailing whitespace (good for code style compliance)
match ErrorMsg /\s\+$/

" Show the ruler for line and column position
setlocal ruler

" Set file-specific indentation (no tabs for commit messages)
setlocal expandtab shiftwidth=4 softtabstop=4

" Enable line numbers for context if needed (optional)
setlocal number
```

### Breakdown of Settings
- **`textwidth=72`**: Wrap commit message body to 72 columns.
- **`spell spelllang=en_us`**: Check for spelling errors; helpful for typo-free messages.
- **Highlighting Text Over Limit**:
  - Uses `match` and `highlight` to visually flag text that exceeds 72 characters.
- **`wrap`**: Keeps lines visually wrapped, respecting `textwidth`.
- **`formatoptions+=n`**: Prevents arbitrary new-line insertion while typing.
- **Highlighting Trailing Whitespace**:
  - Ensures no extra spaces sneak in, especially for commit bodies.
- **`expandtab` and Indentation**:
  - Avoids introducing tabs in commit messages; uses spaces for formatting if necessary.
- **Optional: Line Numbers (`number`)**:
  - Adds helpful context when reviewing staged changes inline.

### Additional Best Practices
1. **First Line Formatting**:
   - Commit title should be concise (50 characters max) and formatted as:
     ```
     <type>(<scope>): <short description>
     ```
     Example: `fix(parser): resolve edge case with null values`

2. **Spellcheck Suggestions**:
   - Consider abbreviations like `fix`, `feat`, `chore`, `docs`, and ensure no typos appear in the main message body.

3. **Shortcuts**:
   - Add a custom Vim command to count characters in the first line (commit title length):
     ```vim
     command! CheckTitle echo line2byte(2) - 2
     ```
---

To configure **Emacs** for Git commit messages, you can set up specific configurations that apply when editing `COMMIT_EDITMSG` files. Here's how to achieve this:

---

## Steps to Configure Emacs for Git Commit Messages ##

1. **Add a Git Commit Hook to Your `.emacs` or `init.el`**:
   Open your Emacs configuration file (`~/.emacs` or `~/.emacs.d/init.el`) and add the following:

   ```elisp
   ;; Set up Git commit message settings
   (add-hook 'git-commit-mode-hook
             (lambda ()
               ;; Wrap text at 72 characters
               (setq-local fill-column 72)
               (turn-on-auto-fill)

               ;; Enable spell check
               (flyspell-mode 1)

               ;; Highlight overlong lines
               (setq-local whitespace-style '(face lines-tail))
               (setq-local whitespace-line-column 72)
               (whitespace-mode 1)

               ;; Highlight trailing whitespace
               (setq-local show-trailing-whitespace t)))
   ```

2. **Install `magit` (Optional)**:
   - If you’re not already using **Magit** (a powerful Git interface for Emacs), install it via `M-x package-install RET magit RET`.
   - This will automatically set `git-commit-mode` for `COMMIT_EDITMSG` files.

3. **Explanation of Configurations**:
   - **`fill-column 72`**: Sets the text wrapping width to 72 characters.
   - **`turn-on-auto-fill`**: Enables automatic wrapping at the `fill-column`.
   - **`flyspell-mode`**: Enables spell checking for typos in commit messages.
   - **Highlight Overlong Lines**:
     - Configures `whitespace-mode` to flag lines longer than 72 characters.
   - **Highlight Trailing Whitespace**:
     - Ensures any trailing whitespace is visually flagged.

4. **Optional: Use `git-commit-mode` for Non-Magit Users**:
   If you don’t use Magit but still want `git-commit-mode`, you can configure it explicitly:
   ```elisp
   (add-to-list 'auto-mode-alist '("COMMIT_EDITMSG\\'" . git-commit-mode))
   ```

---

### Additional Recommendations

- **Enforce Conventional Commit Standards**:
  Install the `conventional-commit` package for Emacs to assist with properly formatted commit messages:
  ```elisp
  (use-package conventional-commit
    :ensure t
    :hook (git-commit-mode . conventional-commit-setup))
  ```

- **Check Commit Message Title Length**:
  Add a function to warn if the first line (commit title) exceeds 50 characters:
  ```elisp
  (defun check-commit-title-length ()
    (when (and (eq major-mode 'git-commit-mode)
               (> (length (car (split-string (buffer-string) "\n"))) 50))
      (message "Warning: Commit title exceeds 50 characters!")))
  (add-hook 'git-commit-mode-hook 'check-commit-title-length)
  ```

---

### Testing Your Configuration
1. Open a commit message file via `git commit`.
2. Verify:
   - Text wraps automatically at 72 characters.
   - Overlong lines and trailing whitespace are highlighted.
   - Spell checking is enabled.

---

## Steps to Configure for Atom, BBEdit, and Sublime Text for Git Commit Messages ##

### **Atom** ###

Atom uses `.editorconfig` or specific package settings to define runtime configurations for text files like Git commit messages. Here's how you can set up Atom for best practices:

1. **Install Required Packages**:
   - [EditorConfig](https://atom.io/packages/editorconfig) (for `.editorconfig` support).
   - [Spell Check](https://atom.io/packages/spell-check) for inline spell checking.
   - [Git Commit Message Linter](https://atom.io/packages/git-message) for Conventional Commit standard enforcement.

2. **`.editorconfig` for Git Commit Messages**:
   Place an `.editorconfig` file in your repository root or home directory to enforce best practices:

   ```editorconfig
   [COMMIT_EDITMSG]
   max_line_length = 72
   charset = utf-8
   trim_trailing_whitespace = true
   insert_final_newline = true
   indent_style = space
   indent_size = 4
   ```

   - **`max_line_length = 72`**: Enforces 72-character wrapping.
   - **`trim_trailing_whitespace = true`**: Removes unnecessary trailing spaces.
   - **`insert_final_newline = true`**: Adds a final newline for compatibility.
   - **`indent_style = space`**: Avoids tabs in commit messages.

3. **Spell Check Settings**:
   - Go to **Preferences → Packages → Spell Check**.
   - Add `COMMIT_EDITMSG` to the file grammars for spell checking:
     ```
     text.git-commit
     ```

4. **Git Message Package**:
   - Configure Conventional Commit limits (e.g., 50 for titles, 72 for body) in the package settings.

---

### **BBEdit** ###

BBEdit uses language-specific preferences and configuration files. For Git commit messages, you can configure the editor as follows:

1. **Text Wrapping**:
   - Go to **BBEdit → Preferences → Text Wrapping**.
   - Set **Soft Wrap** to 72 characters for the `COMMIT_EDITMSG` file type.

2. **Custom File Settings for Commit Messages**:
   BBEdit allows file-specific preferences. Set these for `COMMIT_EDITMSG`:
   - **Line Endings**: Unix (`LF`).
   - **Soft Wrap Width**: 72.
   - **Enable Auto-Indentation**: Turn off (commit messages typically don’t require indentation).
   - **Tab Width**: 4 spaces (or whatever aligns with your team's style).

3. **Spell Checking**:
   - Go to **Preferences → Text Completion → Spelling**.
   - Enable **Check Spelling as You Type**.
   - Set the dictionary to `English (US)` or your preferred language.

4. **Highlight Overlong Lines**:
   Create a **Clipping** (BBEdit snippet) to highlight lines exceeding 72 characters:
   ```sh
   #!/bin/sh
   awk '{if(length($0)>72) print $0 " <-- TOO LONG"; else print $0}'
   ```
   - Save this as a custom tool in **Scripts → Custom Scripts**.
   - Run it on your commit message buffer.

---

For Sublime Text, you can create runtime settings specifically for Git commit messages by using syntax-specific settings. Here’s how to set it up for best practices:

---

### **Sublime Text** ###

1. **Open a Git Commit Message File**:
   - Open Sublime by running a Git command that triggers the editor, such as `git commit`.
   - This opens the `COMMIT_EDITMSG` file.

2. **Create Syntax-Specific Settings**:
   - Go to **Preferences → Settings – Syntax Specific** while the `COMMIT_EDITMSG` file is open.
   - This creates a settings file scoped to `COMMIT_EDITMSG`.

3. **Add Settings for Git Commit Messages**:
   Add the following settings to the syntax-specific configuration file:

   ```json
   {
       "rulers": [72],                   // Add a visual ruler at 72 characters
       "word_wrap": true,                // Enable word wrapping
       "wrap_width": 72,                 // Set the wrap width to 72 characters
       "spell_check": true,              // Enable spell checking
       "trim_trailing_white_space_on_save": true,  // Remove trailing spaces
       "translate_tabs_to_spaces": true, // Use spaces instead of tabs
       "tab_size": 4                     // Set tab size to 4 spaces
   }
   ```

4. **Enable Git Commit Syntax** (Optional):
   If Sublime doesn’t detect the syntax automatically, you can install a syntax package for Git commit messages:
   - Install the package **GitGutter** via **Package Control** for additional Git integrations.
   - Set the syntax manually to `Git Commit Message` by clicking on the syntax selector in the bottom-right corner of Sublime Text and choosing the correct syntax.

5. **Enable Line Highlighting for Overlength Lines** (Optional):
   - Install the **SublimeLinter** package via **Package Control**.
   - Configure it to highlight lines longer than 72 characters:
     ```json
     "linters": {
         "length": {
             "@disable": false,
             "args": [],
             "excludes": [],
             "max_length": 72
         }
     }
     ```

---

